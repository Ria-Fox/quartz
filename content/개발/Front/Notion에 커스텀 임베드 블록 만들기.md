Notion에는 임베드 블록이 존재합니다. 이 임베드 블록은 [Iframely](https://iframely.com/)라는 서비스를 이용해서 만들어지기 때문에, Iframely를 통해서 임베딩 될 수 있는 사이트라면 뭐든 임베딩할 수 있습니다([#](https://www.notion.so/ko-kr/help/embed-and-connect-other-apps#notion%EC%9D%98-%EC%9E%84%EB%B2%A0%EB%93%9C-%EA%B8%B0%EB%8A%A5)).
물론 모든 사이트가 Iframely를 통해 임베드 될 수 있지는 않습니다. 어떤 컨텐츠를 iframe 태그로 보여 줄 것인지 지정하는 특정 태그가 존재해야 하고, 해당 컨텐츠 페이지가 따로 존재해야 합니다. 해당 문서에서는 [Iframely의 공식 문서](https://iframely.com/docs/webmasters)를 따라 어떻게 iframely를 통해 Notion 임베드 블록을 생성하는 지 서술합니다.

# 임베드 페이지 생성
우선 iframe 태그로 보일 임베딩 페이지를 따로 생성합니다. 이 때, 임베드 될 미디어 컨텐츠를 지정할 해시 값은 URL input으로 들어가야 하기 때문에, get이나 HTML hash값으로 지정된 미디어를 불러오도록 구현합니다. 이 페이지는 실제 Notion에서 보일 미디어를 지정하므로, 그에 맞게 구현해 주세요.
# 임베드 페이지 연결
임베드 페이지를 생성했다면, 임베드 페이지와 연결되는 임베드 공유 링크 HTML 링크 아래에 `<link>` 태그를 넣어 주셔야 합니다. 태그 상세는 다음과 같습니다.
```html
<link
  rel="iframely"
  href="https://video.vice.com/en_us/embed/5a5f86d2177dd408e5604453"
  media="(aspect-ratio: 1280/720)"
/>
<!--
  [rel]:   Target Iframely,
  [type]:  optional, "text/html" to publish embed as iFrame
  [href]:  with this href as src
  [media]: and with this size
-->
```
Iframely는 위와 같은 링크 태그를 다음과 같은 형식으로 변경하여 반환합니다.
```html
<div style="width: 100%; height: 0px; position: relative; padding-bottom: 56.25%;">
  <iframe
    src="https://video.vice.com/en_us/embed/5a5f86d2177dd408e5604453"
    style="width: 100%; height: 100%; position: absolute;"
    frameborder="0"
  >
  </iframe>
</div>
```