---
layout: post
title: TypeScript 튜플
date:   2022-07-28 01:17:00 +0300
image:  TypeScript.png
tags:   TypeScript
---

타입스크립트에서 무분별하게 요소가 추가되는 배열이 아닌, 지정된 배열만 허용해주는 튜플에 대해서 알아보자.


---

# 일반적인 TypeScript 배열

{% highlight js %}

const arr: string[] = ["LA", "New-York", "Sanfrancisco"];

{% endhighlight %}


String으로 나열된 배열에서 만일 숫자를 넣게 되면,

{% highlight js %}

const arr: string[] | number[] = ["LA", "New-York", "Sanfrancisco", 20200210];

{% endhighlight %}


해당 배열에는 숫자 혹은 문자열이 들어 간다는 것을 미리 타입을 식별하게 된다.

그러나 배열 안에 어느 인덱스든 문자열, 숫자형이 다 허용되면 타입스크립트의 철저한 타입을 처리하는 방법론에 어긋날 수도 있다.

---

# 지정된 배열 요소를 선언하는 튜플

{% highlight js %}

const arr:  [string, string, string, number] = ["LA", "New-York", "Sanfrancisco", 20200210];

{% endhighlight %}


arr 이라는 배열에 미리 들어갈 타입들을 사전에 다 선언하는 것을 ___튜플___ 이라고 부른다.

---

# 튜플 사용 시 주의해야 할 점

{% highlight js %}

const arr:  [string, string, string, number] = ["LA", "New-York", "Sanfrancisco", 20200210];

arr.push(true)

console.log(arr)
// ["LA", "New-York", "Sanfrancisco", 20200210, true]

{% endhighlight %}

4가지 배열 요소만 지정했으나 push 메서드를 사용하여 다시 콘솔로그를 찍어보면 push 메서드에 담긴 요소가 추가 된다.

그 이유는 엄연히 자바스크립트 상에서 튜플로 지정된 arr은 배열이기 때문에 pop, push, unshift 등등 배열 메서드들이 작동될 수 밖에 없다.
 
사실상 튜플이 지정된 배열에 push, pop 같은 메서드를 사용했을 경우 다시 ___string | number___ 일반적인 배열 타입으로 변경 되는 셈이다.

---

# 튜플의 사용 예

``` 서버 API에서 받아오는 배열인데 Immutable(불변성) 규칙이 필요한 경우 이거나, UI에서 원칙적 변형이 일어날 수 없는 배열들을 다루는 경우 튜플을 사용한다. ```
