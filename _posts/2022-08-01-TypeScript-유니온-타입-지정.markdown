---
layout: post
title:  TypeScript 유니온 타입 지정
date:   2022-07-24 14:26:00 +0300
image:  TypeScript.png
tags:   TypeScript
---

타입스크립트를 통해 함수의 매개변수에 타입을 지정할 떄 간혹 문자형과 숫자형이 상황에 따라 

선언 되어야 하는 경우가 발생한다. 

이런 경우를 해결하기 위해 존재하는 __유니온 타입__ 에 대해 알아보자.

---

# 함수의 매개변수 문제점

{% highlight js %}

function combine(input1: number, input2: number) {
   const result = input1 + input2
   return result
}

const numCombine = combine(12, 10)

const stringCombine = combine("ab", "cd")

{% endhighlight %}

numCombine은 combine 함수의 매개변수 타입들이 __number__ 특성이라서 큰 문제가 없지만,

stringCombine은 인자들이 __string__ 인 관계로 result 할당 구간에서 컴파일 에러가 발생한다.

---

# 여러 개 타입들을 선언하는 유니온 타입

{% highlight js %}

function combine(input1: number | string, input2: number | string) {
   let result 
   
   if(typeof input1 === "number" && typeof input2 === "number") {
      result = input1 + input2
   } else {
      result = input1.toString() + input2.toString()
   }
   
   return result
}

const numCombine = combine(12, 10)

const stringCombine = combine("ab", "cd")

{% endhighlight %}


위와 같은 방식으로 "|" 기호를 활용한 유니온 타입을 활용하여 런타임 환경에서 숫자형과 문자형을 

상황에 처리될 수 있게 만들 수 있다.


---

# 유니온 타입의 사용 예

1. 런타임 검사 환경에 필요할 경우
2. 여러 타입들을 받아야 할 경우

