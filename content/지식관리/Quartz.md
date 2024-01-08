Markdown을 이용해 마이크로 블로그를 만들 수 있는 툴입니다. 홈페이지도 Quartz로 꾸며져 있어, 비슷한 블로그를 생성할 수 있습니다.
[Github](https://github.com/jackyzha0/quartz)  |  [홈페이지](https://quartz.jzhao.xyz/)


제 경우에는 원래 기존 [[PARA]] 형식으로 만든 Obsidian vault 중에서 *Resources* 폴더에 SymLink를 연결해 쓰려고 하였으나, 다음 두 가지 문제 때문에 실패했습니다.
- 미디어 폴더도 같이 동기화해야 하는데, 이를 위해서 기존 미디어 폴더를 리소스 안에 넣으면 다른 폴더의 모든 미디어 또한 같이 올라감 -> 네트워크 용량 낭비됨
- Quartz의 문제점인지, 아니면 제 Mac 환경의 문제인지 해당 Symlink가 날아가는 경우가 왕왕 생김

위와 같은 이유로, 현재는 Empty vault를 따로 만들어 관리중입니다.
# 마크다운 형식
Quartz는 기본적으로 Obsidian을 지원하지만, 추가적인 메타데이터를 생성할 수도 있습니다. 이를 **frontmatter**라고 합니다.
```Markdown
---
title: Example Title
draft: false
tags:
  - example-tag
---
 
The rest of your content lives here. You can use **Markdown** here :)
```
