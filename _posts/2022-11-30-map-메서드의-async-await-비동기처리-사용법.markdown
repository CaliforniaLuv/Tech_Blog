---
layout: post
title: map 메서드의 async/await 비동기처리 사용법
date:   2022-11-25 14:40:00 +0300
image:  React.png
tags:   React
---


각 배열 요소의 값에 따른 Restful API 요청이 필요한 구간에서 단순하게 map 메서드를 사용하면

__Promise {<pending>}__ 의 값이 콘솔로그에 출력된다.
  
pending은 대기 상태를 의미하며 아직 아무런 실행 조차 이루어지지 않은 채 응답 데이터를 받은 것을 뜻한다.
  
---

  
  
# 문제가 발생한 코드
  
{% highlight js %}  

   const GET_HOUR_DAYILY_INFO = () => {
    const arr = [];

    selectedPlantList.map(async (el) => {
      const hourDayilyData = await hourDayilyListStack(el.id);
      arr.push(hourDayilyData);
    });
    setHourDailyList(arr);
  };

{% endhighlight %}
  

map 메서드 스코프 안에서 await을 설정하면 비동기 처리 과정이 진행될 중 알았지만 앞서 언급한 Promise {<pending>}이 배열에 저장되어 있다.

다시 말해 아무런 비동기 처리 절차를 걷지 않고 대기 상태에서 Restful API 응답 처리가 된 것이다.
  
# Promise.all을 사용하자!
  
{% highlight js %}  
    
  const GET_HOUR_DAYILY_INFO = async () => {
    const arr = [];
    await Promise.all(
      selectedPlantList.map(async (el) => {
        const hourDayilyData = await hourDayilyListStack(el.id);
        arr.push(hourDayilyData);
      })
    );
    setHourDailyList(arr);
  };
 
{% endhighlight %}

__Promise.all__ 문법을 사용하면 배열 요소에 따른 연쇄적 비동기처리 과정을 해결할 수 있다.
  
  
