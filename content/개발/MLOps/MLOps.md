현재 3년차 MLOps 개발자로서, 다양한 기술 스택과 경험을 할 수 있었습니다. 그 중 기억에 남는 기술들과 경험을 얘기해보려고 합니다. 작성되는 대로 주기적으로 추가됩니다.

## 개발 환경
- [[Ruff 리서치]] : Linting 및 Formatting을 동시에 할 수 있는 툴입니다. 매우 빠르고 가벼워, Pylint를 사용할 때 몇 초간 남는 시간 때문에 집중이 깨지는 경험을 하고 나서 변경하게 되었습니다. Black, Isort, PyLint를 이 툴로 대체할 수 있습니다.

## 인프라
- [[gpushare-scheduler-extender]] : Kubernetes에서는 GPU 지원이 미비한데, 기본적으로 GPU를 사용하기 위해서는 Nvidia device plugin을 설치해야 하고 GPU 리소스 공유를 위해서는 GPU Time-Slice 기능을 사용해야 했습니다. 해당 기능은 GPU time만 격리되고 메모리를 격리할 수 없어, 메모리가 크고 작은 다양한 pod를 deploy하다 보면 어느 하나의 pod는 반드시 OOM으로 죽는 문제가 생겼습니다. 이런 문제를 해결하기 위해 GPU VRAM Memory 수치로 스케듈링할 수 있게 작성된 Kubernetes 관리 프레임워크입니다.