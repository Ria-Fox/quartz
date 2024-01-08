# 1. Headers
```
# 제목 1
## 제목 2
### 제목 3
#### 제목 4
```
# 제목 1
## 제목 2
### 제목 3
#### 제목 4
# 2. Bold / Italic / Strikethrough
```
**bold**
__bold__
*italic*
_italic_
~~strikethrough~~
```
**bold**
__bold__
*italic*
_italic_
~~strikethrough~~
# 3. List
```
1.
-
+
*
```
1. List
2. List2
- Bullet list
# 4. Link
```
[링크 텍스트](URL)
```
# 5. Image
```
![alt](이미지 URL)
![alt|가로x세로](이미지 URL)
```
![위키 흑요석](https://upload.wikimedia.org/wikipedia/commons/thumb/8/8c/ObsidianOregon.jpg/360px-ObsidianOregon.jpg)
![위키 흑요석 작은 버전|100x100](https://upload.wikimedia.org/wikipedia/commons/thumb/8/8c/ObsidianOregon.jpg/360px-ObsidianOregon.jpg)
# 6. Blockquotes
```
> 인용구
\- 인용 출처   
```
> 인용구
\- 인용 출처
# 7. 코드
```
인라인 : `printf("asdf")`
블록 : ```
```
# 8. 수평선
```
   ---
   ***
   ___
```
---
***
___
# 9. 각주
```
각주[^1].
여러 줄 각주[^2]
이름을 가진 각주[^note]
인라인 각주 ^[이 때에는 기호가 대괄호 바깥에 있음.]

[^1]: 참조 텍스트
[^2]: 여러 줄의 각주를 작성할 때에는
  두 번의 공백 추가
[^note]: 표기될 때는 여전히 숫자로 표기됨
```
각주[^1].
여러 줄 각주[^2]
이름을 가진 각주[^note]
인라인 각주 ^[이 때에는 기호가 대괄호 바깥에 있음.] -> 편집할 때는 제대로 안 된 것처럼 보이지만 실제로는 정상적으로 됨

[^1]: 참조 텍스트
[^2]: 여러 줄의 각주를 작성할 때에는
	두 번의 공백 추가
[^note]: 표기될 때는 여전히 숫자로 표기됨
# 10. Highlight
```
==하이라이트==
```
==하이라이트==

# 11. Checkbox
```
- [x] strikethrough
- [-] only check
- [ ] empty box
단축키: Ctrl+L
```
- [x] strikethrough
- [-] only check
- [ ] empty box
# 12. 문서 링크
```
[[옵시디언 사용법]]
```
# 13. 문서 임베딩
```
![[옵시디언 사용법]]
```
# 14. 문단 참조
    다른 노트에 있는 태그를 참조하기
```
![[문서 링크#^태그 ID]]
```
# 15. Callout
```
> [!note] Title
>   공백 두 개 이후 내용

서로 다른 색상으로 만들 수 있음
- [!note]
- [!abstract], [!summary], [!tldr]
- [!info]
- [!todo]
- [!tip], [!hint], [!important]
- [!success], [!check], [!done]
- [!question], [!help], [!faq]
- [!warning], [!caution], [!attention]
- [!failure], [!fail], [!missing]
- [!danger], [!error]
- [!bug]
- [!example]
- [!quote], [!cite]

```