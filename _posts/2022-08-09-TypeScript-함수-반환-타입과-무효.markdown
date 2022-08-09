---
layout: post
title: TypeScript 함수 반환 타입과 무효
date:   2022-08-09 01:10:00 +0300
image:  TypeScript.png
tags:   TypeScript
---

타입스크립트에서는 함수에도 타입을 지정할 수 있으며 추론 이벤트를 통해 반환문이 없을 경우인 타입에 대해 알아보자.

---


# 함수 반환 타입

{% highlight js %}

function add(n1: number, n2: number) {
  return n1 + n2
}

{% endhighlight %}

<img width="645" alt="스크린샷 2022-08-09 오전 1 39 55" src="https://user-images.githubusercontent.com/78064720/183469078-060c59d5-de91-480b-ad20-3273cd920d7a.png">

구성된 add 함수의 매개변수들은 숫자형이므로 반환 결과는 숫자 타입이 나오게 된다.

해당 add 함수에 커서를 나두면 위 사진과 같은 __number 타입이 나온다는 것을 추론한다.__

{% highlight js %}

function add(n1: number, n2: number) {
  return String(n1) + String(n2)
}

{% endhighlight %}

<img width="673" alt="스크린샷 2022-08-09 오전 1 43 10" src="https://user-images.githubusercontent.com/78064720/183469643-5b1b94a5-78fe-4cd3-8e13-c1aabbc5bff7.png">

매개변수의 숫자 타입이 문자형으로 변환되어 반환 되어도 타입스크립트는 __문자형 타입이 반환될 것이라는 것을 추론하여 나타난다.__

---

# 아무런 반환이 없을 경우 void

{% highlight js %}

function consoleResult(num: number) {
 console.log('result :' + num)
}
consoleResult(add(1, 3))

{% endhighlight %}

<img width="496" alt="스크린샷 2022-08-09 오후 11 33 00" src="https://user-images.githubusercontent.com/78064720/183676350-909f150f-b643-4003-af3f-4e92dea30722.png">


해당 함수에서 반환문이 없고, 함수 내부에서만 로직이 돌아가는 경우 void 라는 타입이 자동으로 추론된다.

---

# 빈 값을 반환할 경우 undefind 


{% highlight js %}

function consoleResult(num: number): undefined {
 console.log('result :' + num)
 return
}
consoleResult(add(1, 3))

{% endhighlight %}

__return 반환문이 비어 있을 경우 undefined 타입을 지정하여 선언__ 할 수도 있고 만일 함수 반환 타입을 작성하지 않을 경우 자연스레 void로 추론한다.



