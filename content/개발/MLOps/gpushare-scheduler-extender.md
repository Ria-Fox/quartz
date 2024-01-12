이 문서는 GPU 공유를 위한 플러그인인 `gpushare-scheduler-extender`에 대해 설명하고, 설치 방법과 사용법에 대해 기술합니다.

# 0. GPU 공유

현재 k8s에서 GPU를 사용하기 위해서는 Nvidia의 공식 플러그인을 사용해야 합니다(Nvidia-device-plugin). 해당 플러그인의 경우 k8s 상 GPU 공유를 위해 Time-slicing 기능을 제공하지만, 다음과 같은 경우에 대해서 time-slice 기능은 오히려 독이 됩니다.

- GPU 메모리 사용량이 `1/time-slice`보다 큰 경우
이 경우에는 하나의 GPU에 배정된 time-slice 개수보다 적은 개수의 프로세스가 들어가도 GPU 메모리는 꽉 찬 상태가 되기 때문에, k8s scheduler는 더 이상 메모리가 없는 GPU에 계속해서 프로세스를 할당하고, 해당 프로세스는 당연히 GPU 메모리가 없어 실패하게 됩니다. k8s에서 pod가 실패하는 경우 컨테이너만 재시작하고 node를 변경하지는 않기 때문에 해당 프로세스나 pod는 영원히 실패하게 됩니다.
- GPU 메모리 사용량이 `1/time-slice`보다 월등히 작지만, 많은 수의 프로세스가 띄워져야 하는 경우
이 경우에는 단순히 time-slice를 늘리는 것으로 해결을 할 수도 있지만, 위에서 서술한 문제가 다시 발생하게 됩니다(해당 프로세스만 쓰는 게 아니라, 다양한 다른 프로세스를 사용하고 있으므로). 결국 GPU 자원을 낭비하거나, 아니면 GPU 자원을 마음대로 사용하는 것이 힘들게 됩니다.

이런 문제가 있어 GPU의 volatile을 기준으로 하는 스케듈링이 아닌, GPU 메모리로 스케듈링하는 커스텀 플러그인이 필요하였으며, 이를 위해 https://github.com/AliyunContainerService/gpushare-scheduler-extender 오픈 소스를 사용하기로 결정하였습니다.

# 1. gpushare-scheduler-extender

해당 플러그인은 nvidia-device-plugin을 대체하여 k8s에서 GPU를 사용할 수 있게 해 주는 플러그인입니다. 다만 기존의 nvidia-device-plugin과 다르게, GPU 공유를 위해서 제공하는 기능은 GPU 메모리 기반의 스케듈링입니다(GPU memory bin-packing). 다만 GPU 메모리를 제한해 주거나 하는 기능은 없고, 사용자가 직접 각 프로세스의 GPU 메모리 사용량을 잘 알고 있어야 하지만 GPU 메모리를 사용자가 직접적으로 관리할 수 있는 이점이 있습니다.

<aside>
💡 Limitation : 하나의 pod 안에서 여러 container가 GPU 리소스를 요청한다면 해당 플러그인이 정상적으로 동작하지 않습니다. 이런 pod는 현재 리뷰 파이프라인 등이 있으며, 이는 deep-07같이 time-slice 플러그인을 남겨 두는 노드에 배포되어야 합니다.

</aside>

사용하게 된다면 리소스를 다음과 같이 요청할 수 있습니다. (`nvidia.com/gpu` → `aliyun.com/gpu-mem`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gpu-test-mem
spec:
  replicas: 3
  selector:
    matchLabels:
      app: gpu-test
  template:
    metadata:
      labels:
        app: gpu-test
    spec:
      containers:
        - image: "docker.io/nvidia/cuda:11.6.2-base-ubuntu20.04"
          command:
            - bash
            - '-c'
            - while true; do sleep 2000; done
          name: recommend-system-container
          resources:
            limits:
              aliyun.com/gpu-mem: "3500"
```

리소스 이름은 `aliyun.com/gpu-mem`이며, GiB를 기준으로 할 지, MiB를 기준으로 할 지에 따라 요청하게 됩니다. 해당 리소스는 정수값만 받기 때문에, MiB를 기준으로 하는 게 더 나을 거라고 생각합니다.

# 2. Installation

설치 과정은 다음의 문서를 따릅니다. : [https://github.com/AliyunContainerService/gpushare-scheduler-extender/blob/master/docs/install.md](https://github.com/AliyunContainerService/gpushare-scheduler-extender/blob/master/docs/install.md)

## 2.0. GPU node 준비

현재 회사의 모든 GPU는 GPU node로서 준비되어 있지만, 만약 새로운 노드를 추가해야 한다면 다음 과정을 진행해 주세요. : [k8s용 GPU node 설정](https://www.notion.so/k8s-GPU-node-f82cc4744de34473831f42a0b3ec4782?pvs=21) 

## 2.1. Control plane에 GPU share scheduler extender deploy

스케듈러 수정을 위한 ClusterRole 및 서비스 계정을 만드는 단계입니다. 추가로 플러그인 deployments도 작성합니다.

```bash
kubectl create -f https://raw.githubusercontent.com/AliyunContainerService/gpushare-scheduler-extender/master/config/gpushare-schd-extender.yaml
```

## 2.2. 스케듈러 설정 수정하기

kube-system 네임스페이스에 있는 Kube-scheduler pod/deploy를 수정하는 과정입니다. 확장 플러그인 설정 파일(scheduler-policy-config.json)을 넣어야 하기 때문에, 컨트롤 플레인의 접속 정보가 필요합니다.

> Notice: If your Kubernetes default scheduler is deployed as **static pod**, don't edit the yaml file inside `/etc/kubernetes/manifest`. You need to edit the yaml file **outside the `/etc/kubernetes/manifest` directory. and copy the yaml file you edited to the '`/etc/kubernetes/manifest/`'** directory, and then kubernetes will update the default static pod with the yaml file automatically.
———
주의사항 : 만약 k8s 기본 스케듈러가 deployments나 deamonset이 아닌 **static pod**으로 배포되어 있는 경우, 각 컨트롤 플레인의 `/etc/kubernetes/manifest` 안의 `kube-scheduler.yaml` 파일을 **바로 수정하지 마시고 해당 디렉토리 외부에 복사해서 수정 후 다시 해당 디렉토리로 복사**해 주셔야 k8s가 자동으로 pod를 업데이트 해 줍니다.
> 

: kube-scheduler는 static pod일 수도, deployments일 수도 있는데 **static pod인 경우 위의 글대로 수정해 주셔야 합니다**.

이하는 k8s 1.22.9버전을 기준으로 설명합니다(1.23 이상이라면 원문을 다시 참조해 주세요).

```bash
# Control plane 안에서 작업 필요
cd /etc/kubernetes
curl -O https://raw.githubusercontent.com/AliyunContainerService/gpushare-scheduler-extender/master/config/scheduler-policy-config.json
```

우선 확장 플러그인 설정 파일을 위와 같이 가져와서, 다음 parameter를 scheduler의 command에 추가해 줍니다.

```yaml
- --policy-config-file=/etc/kubernetes/scheduler-policy-config.json
```

그리고 volumn mount 정보도 추가해 줍니다.

```yaml
- mountPath: /etc/kubernetes/scheduler-policy-config.json
  name: scheduler-policy-config
  readOnly: true
```

```yaml
- hostPath:
      path: /etc/kubernetes/scheduler-policy-config.json
      type: FileOrCreate
  name: scheduler-policy-config
```

정상적으로 Kube-scheduler가 실행되는 지 확인합니다.

## 2.3. gpushare device plugin 배포

이제 실제 대몬셋을 배포합니다. 

```bash
kubectl create -f https://raw.githubusercontent.com/AliyunContainerService/gpushare-device-plugin/master/device-plugin-rbac.yaml
kubectl create -f https://raw.githubusercontent.com/AliyunContainerService/gpushare-device-plugin/master/device-plugin-ds.yaml
```

**(****중요****)**

**이 때 두 번째로 배포한 대몬셋(`gpushare-device-plugin-ds`)의 command 파라미터로 `--memory-unit=GiB`가 있을 텐데, 이 부분을 `--memory-unit=MiB`로 수정하여 기본 단위를 MiB로 수정합니다.**

## 2.4. GPU node에 label 추가하기

이제 마지막으로 GPU를 공유할 GPU 노드에 `gpushare=true` 레이블을 붙여 줍니다. 해당 레이블이 있는 모든 노드에 대해서 GPU sharing이 활성화됩니다.

```bash
kubectl label node <target_node> gpushare=true
```

# 3. Kubectl extension 설치하기

기존의 Nvidia-device-plugin의 경우 GPU가 어떻게 사용되고 있는지 확인하는 데에는 조금 불편함이 있었지만, 새로운 플러그인의 경우 현재 GPU 배포가 어떻게 이루어졌는지 확인할 수 있는 kubectl extension을 배포하고 있습니다. 다만 너무 오래 전에 빌드된 파일이다 보니 대부분의 컴퓨터에서 아키텍쳐 문제로 실행되지 않아, 따로 golang으로 빌드해 주어야 합니다.

- 미리 빌드된 바이너리 문의 안내
    - intel: @헤이즈 [ABE]
    - M1: @존 [ABE]

golang은 각자의 방법대로 설치해 주시고, 빌드 전 다음 명령어로 GOPATH 환경변수를 추가해 주세요.

```bash
export PATH=$PATH:$(go env GOPATH)/bin
export GOPATH=$(go env GOPATH)
```

그 다음에는 다음 명령어로 빌드해 주신 다음,

```bash
mkdir -p $GOPATH/src/github.com/AliyunContainerService
cd $GOPATH/src/github.com/AliyunContainerService
git clone https://github.com/AliyunContainerService/gpushare-device-plugin.git
cd gpushare-device-plugin
go build -o $GOPATH/bin/kubectl-inspect-gpushare cmd/inspect/*.go
```

kubectl이 있는 디렉토리에 넣어주시면 됩니다.

⚠️ 설치 도중에 go.mod 파일 관련 에러가 나면 아래 부분을 확인해주세요.

```bash
# Mac의 경우
sudo cp $GOPATH/bin/kubectl-inspect-gpushare /usr/local/bin/

# 리눅스의 경우
cp $GOPATH/bin/kubectl-inspect-gpushare /usr/bin/

# 필요하다면 실행 가능하도록 권한도 변경해 주세요.
chmod u+x kubectl-inspect-gpushare

# 테스트
kubectl inspect gpushare
```

## ⚠️ 설치 중 go.mod 관련 에러가 나는 경우

이 라이브러리의 go 버전이 옛날 것인 것으로 보여 최신 버전의 go로 빌드를 시도할 경우 `go.mod` (디펜던시 파일) 관련 에러가 날 수 있습니다. ([참고](https://chat.openai.com/share/104fdab4-50cf-432d-b2c0-5da5331c7828))

⚠️ `go build` 명령어 실행 시 다음과 유사한 에러가 나는 경우:

```bash
cmd/inspect/display.go:10:2: no required module provides package github.com/golang/glog: go.mod file not found in current directory or any parent directory; see 'go help modules'
```

1. `go.mod` 파일을 만드는 다음 명령어를 실행해주세요.

```bash
go mod init github.com/AliyunContainerService/gpushare-device-plugin
```

1. `go build -mod=mod` 옵션을 넣어서 다음과 같이 실행해주세요.

```bash
go build -mod=mod -o $GOPATH/bin/kubectl-inspect-gpushare cmd/inspect/*.go
```