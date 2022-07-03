---
layout: post
title:  setTimeout와 requestAnimationFrame
date:   2022-07-03 13:10:00 +0300
image:  05.jpg
tags:   JavaScript
---


이벤트 애니메이션을 사용하다 보면 성능 최적화에 문제가 발생한다.

이를 해결하기 위해서 보통 setTimeout을 사용해서 최적화를 진행하며, 이를 보통 __throttle__ 이라고 부르며 ms를 환산하여 트리거를 작동한다.

---

# setTimeout 활용 (throttle)

{% highlight js %}

const el = document.querySelector("#time");

el.addEventListener("click", () => {
  let start = performance.now()
  function animation() {
    let p = performance.now();
    let d = p - start;
    start = p;
    el.textContent = d
    setTimeout(animation);
  }
  animation();
});


{% endhighlight %}

![setTime](https://user-images.githubusercontent.com/78064720/177047226-39640e8d-d000-4a94-b76e-1f9e69e4e7a6.gif)

대략 5ms가 동작되는데, 보통 모니터들이 1초에 60장의 이미지들을 제어할 수 있다.

그러나 5ms라면 초당 대략 200장의 이미지들의 장면들이 지나가는데, 이는 이미지가 브라우저가 애니메이션 포퍼먼스를 맞추어도 

사용자의 모니터는 초당 200장 성능에 못따라가게 된다.

{% highlight js %}

setTimeout(animation);

{% endhighlight %}

![setTIme60ms](https://user-images.githubusercontent.com/78064720/177047576-eaddf5e4-9a3b-46d4-b4a3-4e5355b6aea4.gif)

직접적으로 setTimeout의 두번쨰 인자에 1000/60이라는 값을 임의로 설정하여 초당 60장의 이미지를 주어야 사용자 성능을 맞추게 된다.

그러나 만일 어떤 사용자들은 초당 100장, 200장의 장면들을 제어할 수 있는 모니터를 사용하고 있다면, 

1000/60 second 설정은 고사양 모니터들에게 강제로 저사양 성능을 선사해주기 떄문에 좋은 방법이 못된다.


--- 


# requestAnimationFrame 활용

{% highlight js %}

const el = document.querySelector("#frame");

el.addEventListener("click", () => {
  let start = performance.now();
  function animation() {
    let p = performance.now();
    let d = p - start;
    start = p;
    el.textContent = d;
    requestAnimationFrame(animation)
  }
  requestAnimationFrame(animation)
});


{% endhighlight %}

![ezgif com-gif-maker](https://user-images.githubusercontent.com/78064720/177049621-5550de88-6658-417e-b24d-0d114024fecf.gif)

자동적으로 사용자의 최적 사양에 맞게 실행되고 있다.


--- 


# setTimeout과 requestAnimationFrame는 비동기인데 왜 다를까?

해당 부분은 [sculove](https://sculove.github.io/post/javascriptflow/)님의 비동기 API에 관한 메커니즘 설명을 참고하기를 바란다.


위 링크를 쉽게 요약하자면

1. setTimeout API의 비동기 task들을 __Task Queue(or macro queue)에 담아둔 후__ js 엔진이 순차적으로 처리

2. Queue에 저장된 비동기 task를 처리하는 시점은 __Call stack이 비어져있을 경우__ 발생 

3. 만일 이 과정이 setTimeout 또는 setInterval에 할당해준 __delay와 싱크가 맞지 않다면 등록해둔 callback은 trigger 되지 않음__


결국 이러한 과정으로 인해 타임 싱크에 의존적이지 않은 requestAnimationFrame가 인터렉티브한 웹 애니메이션을 다루기에 안전한 방식이다.

--- 

# setTimeout처리의 문제점

1. 사용자마다 모니터 hz 성능이 다 다르므로 임의의 값을 주면, 비효율적인 프레임 유실 발생

2. 모바일 과정에서의 배터리 손실 문제 발생
