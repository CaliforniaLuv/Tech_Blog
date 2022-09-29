---
layout: post
title:  Uncaught-TypeError: Cannot read properties of undefined 에러 해결
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

<img width="520" alt="스크린샷 2022-09-30 오전 2 10 10" src="https://user-images.githubusercontent.com/78064720/193095728-74a39d9a-4f01-46c0-96ae-22ab5a4bc48c.png">

