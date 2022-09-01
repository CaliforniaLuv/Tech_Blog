---
layout: post
title:  TypeScript enum(열거형)
date:   2022-07-31 17:10:00 +0300
image:  TypeScript.png
tags:   TypeScript
---

어떠한 특정 상황에 따라 값을 지정하는 경우가 있다. 

예를 들어 메세지 플랫폼에서 어떤 한 유저의 상태를 표시할 때가 대표적이다. (자리 비움, 회의 중 ...)

이럴때 자바스크립트에서는 객체 혹은 변수에 특정 문자열 혹은 숫자형을 지정하여 관리를 하는데

타입스크립트에서는 더 편리하게 관리해주는 enum(열거형) 문법에 대해 알아보자.

---

# 자바스크립트의 불편한점
 
## 변수로 저장
 
  {% highlight js %}
  
  const sleeping = "자리 비움"
  const meeting = "회의 중"
  const connecting = "접속 상태"
  
  {% endhighlight %}
  
  
  이런 형태로 쓰면 직관적이긴 하나 너무 방대하기에 코드가 스파게티 코드로 꼬일게 분명하다.
  
## 객체 형태로 저장
   
   
  {% highlight js %}
   
   const obj = { 
      sleeping: 1
      meeting: 2
      connecting: 3
   }
   
   {% endhighlight %}
   
   객체 형식으로 저장하여 숫자를 별도로 지정하여 문자열 인코드 문제를 막아서 저장하는 방법 또한 좋지만,
   
   특정 키를 외우는 것은 생각보다 헷갈리게 만들 수도 있다.
   
--- 

# enum 활용하기
 
   {% highlight js %}
   
   enum obj { sleeping, meeting, connecting }
   
   const status = {
      name: 'Maximilian',
      age: 30,
      hobbies: ['Sports', 'Cooking'],
      code: obj.sleeping
   };
   
   if (person.code === obj.sleeping) {
    console.log('he's sleeping');
   }
   
   {% endhighlight %}
 
 
 enum에 지정된 요소들은 자연스레 0, 1, 2 이라는 숫자들이 인덱스 형태로 자동 지정된다.
 
 자바스크립트에서는 enum 이라는 키워드가 없으므로 아래와 같이 enum 문법을 해석하게 된다.
 
   {% highlight js %}
   
    var obj;
      (function (obj) {
         obj[obj["sleeping"] = 0] = "sleeping";
         obj[obj["meeting"] = 1] = "meeting";
         obj[obj["connecting"] = 2] = "connecting";
    })(obj || (obj = {}));
    
   {% endhighlight %}
   
   더욱 직관적으로 obj 열거형을 통해 객체 키값을 지정하게 된다.

---


# enum은 객체와 비슷한데 꼭 써야하는가?

enum은 앞서 언급하듯 자바스크립트에서는 __존재하지 않는 문법 키워드__ 이다.

그럼에도 사용해야 하는 이유는

1. 객체는 엄격하게 지정되지 않은 관계로 키의 값은 능동적으로 변경되기 쉽지만, TypeScript enum의 속성은 엄격하여 변경할 수 없다.
2. 객체의 속성 값이 어떤 타입이든 자유롭게 지정되지만, TypeScript에서는 문자열 or 숫자형만 사용할 수 있다.
3. 객체는 가독성에서 스파게티가 될 수 있는 코드이지만, enum을 활용하면 직관적으로 해석이 되므로 코드 해독에 유용하다.
4. REST API 응답을 통해 if 검사 및 타입 배정에 활용하기 유용하다.



   
