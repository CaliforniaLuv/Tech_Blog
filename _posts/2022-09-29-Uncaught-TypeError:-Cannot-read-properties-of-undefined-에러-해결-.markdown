---
layout: post
title:  Uncaught-TypeError Cannot read properties of undefined 에러 해결
date:   2022-09-30 02:07:00 +0300
image:  Javascript.png
tags:   JavaScript
---

배열 요소에 객체가 들어간 데이터를 배열 메서드 혹은 반복문으로 처리하다가 

이런 에러를 겪은 적이 자주 있을 것이다.

```
Uncaught TypeError: Cannot read properties of undefined (reading 'Keyword')

```

REST API 응답을 통해 사용되는 DB들이 주로 배열 안에 객체들로 구성되어 있는데, 

이런 에러를 접근 했을 경우 어떻게 처리해야 하는지 방법에 대해 알아보자.

---


# TypeError 발생


{% highlight js %}

const data = [
    {
        name: "톰 크루즈",
        filmography: ["Top Gun", "Jerry Macguire", "Vanilla Sky", "mission impossible"],
        information: {
            age: 62
        }
    }, 
    {
        name: "에밀리 블런트",
        filmography: ["The Devil Wears Prada", "Edge of Tomorrow", "A Quiet Place"],
        information: {
            age: 39
        }
    },
    {
        name: "릴리 콜린스",
        filmography: ["Love, Rosie", "Okja", "Emily in Paris"],
    },
]

{% endhighlight %}


{% highlight js %}

data.forEach((el) => {
    if(el.information.age) {
        console.log(`${el.name}의 나이는 ${el.information.age} 입니다.`)
    } else {
        console.log(`${el.name}의 나이를 알 수가 없어`)
    }
})

{% endhighlight %}

data라는 배열 요소의 객체에 information 프로퍼티를 접근하면 아래와 같은 에러가 발생한다.

<img width="520" alt="스크린샷 2022-09-30 오전 2 10 10" src="https://user-images.githubusercontent.com/78064720/193095728-74a39d9a-4f01-46c0-96ae-22ab5a4bc48c.png">

해당 오류가 발생하 이유는 3번째 요소에는 information이라는 프로퍼티가 존재하지 않는데

forEach 메서드로 접근 하였기에 TypeError가 발생한 것이다.

# 조건문으로 존재하지 않는 객체 프로퍼티 차단하기

{% highlight js %}

data.forEach((el) => {
    if(el.information && el.information.age) {
        console.log(`${el.name}의 나이는 ${el.information.age} 입니다.`)
    } else {
        console.log(`${el.name}의 나이를 알 수가 없어`)
    }
})

{% endhighlight %}

조건문을 이용하여 if 문에 해당 프로퍼티가 없을 경우 접근을 차단하여 else 문에 빠져 나가게 한다.

# Optional chaining 검증하기

{% highlight js %}

data.forEach((el) => {
    if(el.information?.age) {
        console.log(`${el.name}의 나이는 ${el.information.age} 입니다.`)
    } else {
        console.log(`${el.name}의 나이를 알 수가 없어`)
    }
})

{% endhighlight %}

Optional chaining은 객체 프로퍼티에 ```?``` 문법을 사용하여 연결된 객체 요소에 깊숙히 접근하여 속성 값을 읽도록 한다. 

만일 속성 값이 존재하지 않을 경우, ```undefined``` 값으로 타입을 처리해주며, 더이상 TypeError가 발생하지 않는다.

Optional chaining를 사용할 경우 기존 객체 프로퍼티에 조건문을 상세하게 설정하는 것보다 ```코드가 간단하고 가독성이 편리``` 하다.


