---
layout: post
title:  React useEffect 종속 배열 함수 선언 시 ESlint 경고
date:   2022-09-23 17:10:00 +0300
image:  React.png
tags:   React
---


useEffect hook 종속 배열에 함수를 선언할 경우 아래와 같은 ESlint 경고가 발생하였다.

```
The 'mapSync' function makes the dependencies of useEffect Hook (at line 14) change on every render. 
Move it inside the useEffect callback.  Alternatively, wrap the definition of 'mapSync' in its own useCallback() Hook 

```

해당 경고가 떴다는 것은 정석대로 코드를 작성하지 않은 관계로 코드를 리펙토링을 해보도록 하겠다.

---

## useEffect 스코프 안에서 생성하기


{% highlight js %}

useEffect(() => {
    if (coordinate) {
      mapSync();
      panoSync();
    }
  }, [coordinate, mapSync, panoSync]);
  
const mapSync = () => {
  const map = new naver.maps.Map("map1", {
      center: new naver.maps.LatLng(latitude, longitude),
      mapTypeControl: true,
      mapTypeId: naver.maps.MapTypeId.SATELLITE,
    });
}

const panoSync = () => {
 var marker = new naver.maps.Marker({
      position: new naver.maps.LatLng(latitude, longitude),
      map: pano,
 });
}
  

{% endhighlight %}


종속 배열의 ___mapSync___ , ___panoSync___ 두 함수를 선언할 경우 ESlint 경고가 발생하였다.

useEffect hook 밖의 스코프에 함수들은 렌더링할 때마다 다시 생성되는데 map API 같은 무거운 코드를 다시 렌더링하는 것은 비효율적이다.

useEffect 안에서 생성하면 종속 배열 요소들이 변화되지 않는 이상 다시 렌더링 되지 않으므로 useEffct 스코프 안에서 함수를 생성해야 한다.


{% highlight js %}

useEffect(() => {

    const mapSync = () => {
        const map = new naver.maps.Map("map1", {
          center: new naver.maps.LatLng(latitude, longitude),
          mapTypeControl: true,
          mapTypeId: naver.maps.MapTypeId.SATELLITE,
        });
    }
    
    const panoSync = () => {
       var marker = new naver.maps.Marker({
        position: new naver.maps.LatLng(latitude, longitude),
        map: pano,
       });
    }
  


    if (coordinate) {
      mapSync();
      panoSync();
    }
  }, [coordinate]);


{% endhighlight %}



### useCallback hook 사용

{% highlight js %}

  const mapSync = useCallback(() => {
        const map = new naver.maps.Map("map1", {
          center: new naver.maps.LatLng(latitude, longitude),
          mapTypeControl: true,
          mapTypeId: naver.maps.MapTypeId.SATELLITE,
        });
    }, [naver.maps.Map, naver.maps.MapTypeId.SATELLITE])
    
    const panoSync = useCallback(() => {
       var marker = new naver.maps.Marker({
        position: new naver.maps.LatLng(latitude, longitude),
        map: pano,
       });
    }, [naver.maps.Marker, pano])

{% endhighlight %}


앞서 설명한 useEffect는 마운트 시에만 결국 동작이 되는데, 동적으로 이벤트가 발생을 하기 위해선 

useCallback을 통해 다시 렌더링되어도 재생성되지 않고 useCallback의 종속 배열 요소 값이 변경될 때만 실행하도록 해주면 좋다.
