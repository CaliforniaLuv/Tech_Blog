---
layout: post
title: CleanCode 변수
date:   2022-11-06 12:58:00 +0300
image:  CleanCode.png
tags:   Tech
---

최근 Robert C. Martin's의 Clean Code을 읽다보니 나의 근본도 없는 엉망진창의 코드에 회의감이 빠져 들었다.

계속 다음 장을 넘길 때 마다 밥 아저씨의 비난이 담긴 코드는 보기 고통스러우며 내가 그동안 더닝 크루거 효과 그래프에서 ```우매한 봉우리``` 단계에 있다는 것을 깨달았다.

지금이라도 개선하기 위해 반드시 지켜야 할 Clean Code 룰을 기록 하도록 결심하였다.

---

# 자신만 알아볼 수 있는 작명을 작성하지마라.

## 🤬 Bad Type ❌

{% highlight js %}

function getSignRecord () {
  let d = 15;
  let m = 11;
  ley y = 2012;
  
  return `가입한 날짜는 ${y}년 ${m}월 ${d}일 입니다.`;
}

{% endhighlight %}

## 😄 Good Type ✅

{% highlight js %}

function getSignRecord () {
  let day = 15;
  let month = 11;
  let year = 11;
  
  return `가입한 날짜는 ${year}년 ${month}월 ${day}일 입니다.`;
}

{% endhighlight %}


d, m, y가 대충 day, month, year이라고 이해는 알 수 있다. 

그러나 대충 코드를 작성한다면, 하루 일당이 몇 십만원이나 되는 개발자가 __대충__ 코드 짜서 __대충__ 고객들에게 제품을 제공할 것인가?

명백히 잘못된 행동이다. 코드 하나하나 디테일을 잡아 다른 누군가가 보아도 쉽게 해석할 수 있는 변수명을 작성해주는 것이 개발자의 좋은 그릇이다.


# 명확한 변수명을 작성하라.

## 🤬 Bad Type ❌

{% highlight js %}

let firstTime = 100010;
let zeroTime = 0;
let waitTime;

{% endhighlight %}

## 😄 Good Type ✅

{% highlight js %}

let defaultSettingTime = 100010;
let removeSettingTime = 0;
let delayTime;

{% endhighlight %}

개발자 10명 중에 단 한명도 ```firstTime``` 변수가 무슨 의미인지 아무도 이해를 못한다.

그러면 내 자리에 찾아가기 위해 쓸때 없이 에너지를 소비해서 드디어 저게 초기 세팅 시간이라는 것을 알게 될 것이다.

그리고 ```waitTime``` 이라는 변수 단어를 그대로 해석하면 무언가 이상하다. 

delay와 wait는 의미가 비슷하나 사용되는 상황에 고려하면 메커니즘 영단어에선 delay가 더 옳은 표현이다.

남들이 내 코드의 변수 의미가 무엇인지 궁금증이 없도록 변수명을 신중히 정해줘야 한다.

# 상수값은 대문자와 언더바를 혼합해서 const 키워드를 사용하자.

## 🤬 Bad Type ❌

{% highlight js %}

const payrollTax = 0.1;

{% endhighlight %}

## 😄 Good Type ✅

{% highlight js %}

const PAYROLL_TAX = 0.1;

{% endhighlight %}

세금은 쉽게 변하지 않는다. (물론 정치 법안 개정까지 들어가면 깊으니 그냥 우리 급여 세금이 매달 바뀌지 않는다고 생각하자.)

고정적인 값들은 변수를 재할당과 재선언이 필요 없으니 ```const``` 변수 키워드를 선언하고, 문자열은 __대문자__ 와 __언더바__ 를 혼합해서 사용하면 좋다.

무엇보다 대문자, 언더바 키워드는 __검색하기 쉽다는 점__ 에서 엄청 효율적이다.

# 그 많은 문자 키워드 중에 하필 얘네를 고르는 건데?

## 🤬 Bad Type ❌

{% highlight js %}

const o = 12;
for(let l = 1; l <= o; l++) { 
  l === 12 ? saveNumber(l) : null;
}

{% endhighlight %}

## 😄 Good Type ✅

{% highlight js %}

const limitNumber = 12;
for(let current = 1; current <= limitNumber; current++) { 
  current === 12 ? saveNumber(current) : null;
}

{% endhighlight %}

이건 굳이 언급 안해도 Bad Type 보자마자 화가 난다면 

🤖 삐삑 정상인입니다.

대충 변수를 지정한 ```a, b, num``` 키워드가 선녀로 보일 지경이다.

# 특정 문법을 응용하면 변수가 깨끗하다.

## 🤬 Bad Type ❌

{% highlight js %}

function getMyPhoneNumber(number) {
  let defaultPhoneNumber = number || "000-0000-0000" 
}

{% endhighlight %}

## 😄 Good Type ✅

{% highlight js %}

function getMyPhoneNumber(number = "000-0000-0000") {
}

{% endhighlight %}

javascript 문법인 __함수의 기본값 매개변수(default function parameter)__ 를 활용하면 변수가 훨씬 깔끔하다.

이처럼 특정 언어에서 지원하는 문법을 잘 응용해서 변수를 선언하면 Clean Code가 될 수 있다.



