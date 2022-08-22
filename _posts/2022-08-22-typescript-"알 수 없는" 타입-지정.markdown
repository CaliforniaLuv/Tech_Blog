---
layout: post
title:  TypeScript 알 수 없는 타입 지정
date:   2022-08-22 22:26:00 +0300
image:  TypeScript.png
tags:   TypeScript
---

간혹 특정 변수에 어떤 타입이 사용될 지 알 수 없을 경우, 

사용하기에 용이한 지정이 있는데, 이번 시간 __unknown__ 타입에 대해 알아보자.

---

# any 타입이랑 뭐가 다른거야?

## any 타입

{% highlight js %}

let typeInput: any

typeInput = 5
typeInput = "Hello"

{% endhighlight %}

## unknown 타입

{% highlight js %}

let typeInput: unknown

typeInput = 5
typeInput = "Hello"

{% endhighlight %}

위 any 타입과 unknown 타입을 비교하면 크게 차이가 없을 것이다.

그러나 이렇게 정의하면 엄연히 둘의 차이가 존재한다.

-  any 타입은 어떤 타입들이 들어오든지 모두 받아준다.
-  unknown 타입은 어떤 타입이 들어올지 몰라서 일단 알 수 없는 상태로 지정한다.

# 둘이 다르다는 결정적 증거

![스크린샷 2022-08-22 오후 11 54 13](https://user-images.githubusercontent.com/78064720/185952097-4585601c-fe5f-488a-aa86-a48afd07b3b7.png)

![스크린샷 2022-08-22 오후 11 53 25](https://user-images.githubusercontent.com/78064720/185951920-311d598e-5f08-4144-a6bb-2ae2c8f5e5e3.png)


any의 경우 어떠한 변수에 할당을 받아도 __허용__ 해주지만, 
unknown은 어떠한 타입인지 __모르기__ 때문에 string 타입 변수에 할당 받기에 논리적으로 옳지가 않다.

# unknown 타입의 적절한 사용법


{% highlight js %}

let typeInput: unknown
let typeString: string

typeInput = 5
typeInput = "Hello"

if(typeof typeInput === "string") {
  typeString = typeInput
}

{% endhighlight %}

typeof 키워드를 지정하여 typeInput 변수가 unknown이지만 코드 상에서 많은 로직이 돌아간 뒤 string이 될 것을 확신한 경우,

if 조건문을 통해 string 변수에 할당을 받게 할 수가 있어진다.


## 🤔 그래도 사실..

사실 해당 변수가 어떠한 값들이 들어올 지 미리 예상되는 코드를 작성하고 있다면 간단하게 union 타입을 지정하여 

string | number 형식으로 짜는게 코드 가독성에 더 효율적이다.

unkonwn은 말 그대로 이 변수가 어떠한 타입이 들어올지 __감이 안잡힐떄__ 사용하는 경우이다.

