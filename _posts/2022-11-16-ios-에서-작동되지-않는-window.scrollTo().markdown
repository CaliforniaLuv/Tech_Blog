---
layout: post
title: TypeScript 튜플
date:   2022-11-16 01:07:00 +0300
image:  Javascript.png
tags:   Javascript
---

데스크탑, 모바일, OS 환경 QA 테스트를 진행 도중 모바일 환경에서 엉뚱하게 스크롤 이벤트가 동작되는 버그를 발견하였다.

여러 방법을 시도 도중 믿었던 이벤트 트리거에 발등을 찍혀 기록하고자 한다.

---

# 원하는 위치에 스크롤 이벤트가 동작되지 않는 scrollTo

{% highlight js %}

window.scrollTo(0, Element_Id_Y)

{% endhighlight %}

데스크탑에서는 문제 없이 특정 엘리먼트의 Y 좌표 값에 위치되어 동작을 하지만,

iOS 환경에서는 원치 않은 엘리먼트 Y 좌표에 동작을 하였다.(아이러니하게도 갤럭시에서는 동작이 잘됨....)


# 비슷한 이벤트 트리거 scroll

{% highlight js %}

window.scroll({top: Element_Id_Y})

{% endhighlight %}

```scroll``` 이벤트는 iOS 환경에서도 특정 엘리먼트 Y 좌표에 알맞게 스크롤 이벤트가 동작되었다.

# 원인

[스택오버플로우](https://stackoverflow.com/questions/15691569/javascript-issue-with-scrollto-in-chrome/15694294#15694294)에 따르면,

Chrome의 웹 엔진은 너무 빨라서 scrollTo() 작업이 Chrome의 기존 엘리먼트 Y 좌표 이벤트 전에 실행된다고 한다.

그러나 내가 궁금한 점은 안드로이드 환경인 갤럭시에서는 잘 작동되고 오로직 iOS 환경에서만 버그가 발생하여 웹 브라우저 엔진과는 다른 버그가 있는지 찾을 필요가 있어보인다...

