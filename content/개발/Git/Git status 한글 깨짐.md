## 이슈
Mac에서 Git을 쓰다 보면 한글 파일명이 깨지는 경우가 있습니다.
![[Pasted image 20240226143641.png]]

Git을 사용하다 보면 대부분의 경로는 영문으로 작성하게 되지만, 저의 경우 [[Obsidian]] 및 [[Quartz]] 문서들을 관리하다 보면 경로명이 한글로 작성되는 경우가 많아 자주 겪은 문제입니다. 특히, Git add를 할 때 경로명을 복사해 추가하는 경우가 많은데 이런 경우 불편함이 많았습니다.
## 해결 방법
Git config를 변경함으로써 해결 가능합니다.[^1]
```shell
git config --global core.quotepath false
```

[^1]: https://zlzzlzz2l.tistory.com/50