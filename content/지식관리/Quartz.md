Markdown을 이용해 마이크로 블로그를 만들 수 있는 툴입니다. 홈페이지도 Quartz로 꾸며져 있어, 비슷한 블로그를 생성할 수 있습니다.
[Github](https://github.com/jackyzha0/quartz)  |  [홈페이지](https://quartz.jzhao.xyz/)


제 경우에는 원래 기존 [[PARA]] 형식으로 만든 [[Obsidian]] vault 중에서 *Resources* 폴더에 SymLink를 연결해 쓰려고 하였으나, 다음 두 가지 문제 때문에 실패했습니다.
- 미디어 폴더도 같이 동기화해야 하는데, 이를 위해서 기존 미디어 폴더를 리소스 안에 넣으면 다른 폴더의 모든 미디어 또한 같이 올라감 -> 네트워크 용량 낭비됨
- Quartz의 문제점인지, 아니면 제 Mac 환경의 문제인지 해당 Symlink가 날아가는 경우가 왕왕 생김

위와 같은 이유로, 현재는 Empty vault를 따로 만들어 관리중입니다.

# 설치
기본적으로는 홈페이지의 설치 방식을 따라가면 됩니다. 다만 기본적으로 Github 계정이 있어야 하고, 호스팅 또는 Github에 Quartz를 올리기 위해서는 호스팅 가능한 서버와 도메인이 있거나 Github 지식이 있어야 합니다. 기본적으로 Git은 설치되어 있다고 가정합니다.
## 설치 전
Node와 NPM을 설치해야 합니다. [Node.js 설치 페이지](https://nodejs.org/en/download)에서 각 운영체제에 맞는 인스톨러를 다운로드하여 설치합니다.
## Quartz 레포지토리 가져오기
Node와 NPM이 설치되었다면, 다음 코드를 입력하여 Quartz 코드를 다운로드합니다.
```shell
git clone https://github.com/jackyzha0/quartz.git
cd quartz
npm i
npx quartz create
```
이 때, 저처럼 Github으로 Quartz를 다루고자 하신다면 해당 깃헙 레포지토리를 Fork하여 자신의 레포지토리로 만들고, 그 레포지토리를 클론해 주셔야 됩니다.

`npx quartz create` 명령어를 입력하면 Quartz의 content들을 어떻게 저장하고 링크를 관리할 지 물어볼 텐데, 개인적인 추천으로는 `Empty Quartz`, `Treat links as shortest path`를 선택하시길 추천드립니다. Obsidian과 함께 쓰기 가장 좋은 설정이라고 생각합니다.
## Quartz를 Github.io에 올리기
Quartz의 설치가 완료되었다면, Quartz 폴더 안의 `.github/workflow` 폴더 안에 `deploy.yml` 파일을 생성하고 다음 코드를 입력합니다.
```yaml
name: Deploy Quartz site to GitHub Pages
 
on:
  push:
    branches:
      - v4
 
permissions:
  contents: read
  pages: write
  id-token: write
 
concurrency:
  group: "pages"
  cancel-in-progress: false
 
jobs:
  build:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0 # Fetch all history for git info
      - uses: actions/setup-node@v3
        with:
          node-version: 18.14
      - name: Install Dependencies
        run: npm ci
      - name: Build Quartz
        run: npx quartz build
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v2
        with:
          path: public
 
  deploy:
    needs: build
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
```

그 다음에는 자신의 레포지토리의 **설정**에 들어가서 **Pages** -> **Source**를 **Github Actions**로 바꿔 주신 다음, `npx quartz sync`를 입력하여 동기화해 주면 `<자신의 깃헙 아이디>.github.io/<레포지토리 이름>` 주소로 Quartz가 올라온 것을 확인하실 수 있습니다. 이후의 설정이나 글 작성법은 홈페이지를 참고해 주세요.

# 마크다운 형식
Quartz는 기본적으로 [[Obsidian]]을 지원하지만, 추가적인 메타데이터를 생성할 수도 있습니다. 이를 **frontmatter**라고 합니다.
```Markdown
---
title: Example Title
draft: false
tags:
  - example-tag
---
 
The rest of your content lives here. You can use **Markdown** here :)
```
