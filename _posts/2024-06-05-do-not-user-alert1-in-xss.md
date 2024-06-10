---
title: XSS 취약점 테스트 시 alert(1) 사용 권장하지 않는 이유
categories: [Theory, Web]
tags: [xss, web]
image:
    path: /assets/image_post/20240605134044.png
---

## [0x00] summary
---
xss 취약점 진단 시 `alert(1)` 대신 아래의 문구 사용을 권장한다.
```
alert(document.domain)
alert(window.origin)
```

## [0x01] overview
---
최근 티오리에서 진행한 `[무료 웨비나] 참여자 만족도 95% 이상! 버그바운티 속성 마스터 클래스 웨비나 2차` 웨비나를 들으면서 새로운 정보를 알게되어 이에 대해 한번 정리해보고자 한다.
다양한 사례를 소개해주는 좋은 웨비나였는데, 내용 중 Google 버그바운티에선는 XSS 취약점 진단 시 `alert(1)`의 사용을 권고하지 않는다는 내용이 있었다. 이에 대해 발표자분께도 질문을 해보니,
어디서 동작하는지 정확하게 파악하기 위해 `alert(1)`이 아닌 도메인 위치나 이런 정보들의 출력을 권장한다고 하였다.

그리하여 이에 대해 자세히 알아보고자 자료를 서칭하여 정리해보고자 한다.


## [0x02] alert(1)
---
우선 `alert(1)`은 필자를 포함하여 xss를 테스트할 때 일반적으로 많이 사용하고 있다. 편하고 직관적이고, 키보드 타이핑 수를 줄일 수 있으므로..😒
하지만 위 방식으로 진행할 경우 아래 제한에 걸릴 수 있다고 참고 자료들에서는 설명한다.


### identify domain
- 단순히 `alert(1)`을 입력하여 실행된 경우, 그것이 중요한 도메인에서 실행되었는지 확인이 불가하다. 
- `alert(1)`이 동작했다고 끝이 아니라, 그게 유효한 대상 도메인인지 확인을 해야한다.
- `my.test.com`에서 `alert(document.domain)`을 했을 때 `my.test.com`이 아닌 `sub-my.test-blog.com`와 같이 타 도메인에서 동작하는 코드일 수도 있다는 것이다.


### sandboxed iframes
- 위와 유사하게 샌드박스 형태를 이용하여 사용자 콘텐츠를 격리 시키기 위해 샌드박스 도메인이나 샌드박스 iframe을 사용한다.
- `my.test.com`에서 `alert(document.domain)`을 했을 때 `my.test-sb.com`과 같이 나오는 경우가 존재한다.

이러한 경우 버그바운티 대상 도메인이 아닌 다른 곳에서의 취약점을 발견하게 된 것으로, `리포팅을 하더라도 바운티를 받을 수 없다`는 것이 요지이다.

## [0x03] conclusion
---
그래서 결론적으로 추천하는 방식은 `alert(document.domain)`와 `alert(window.origin)` 같은 형태로, 어느 도메인에서 동작하는지 확인 할 수 있는 코드를 쓰는 것을 추천한다.


## [0x04] reference
---
- Bughunters Google: https://bughunters.google.com/learn/invalid-reports/web-platform/xss/5108550411747328/when-reporting-xss-don-t-use-alert-1
- YOUTUE - Do NOT use alert(1): https://www.youtube.com/watch?v=K6bSPl6NhuQ
- LiveOverflow: https://liveoverflow.com/do-not-use-alert-1-in-xss/