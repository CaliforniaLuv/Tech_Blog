---
layout: post
title: useLayoutEffect는 어떤 용도로 쓰일까?
date:   2022-11-25 14:40:00 +0300
image:  React.png
tags:   React
---

사내 직장 동료 코드를 리뷰하던 중 useLayoutEffect라는 React hook을 처음 보게 되었다.

해당 기능에 대해 어떤 용도로 사용되고 있는지 묻고 싶었지만...

요즘 동료 분 프로젝트가 너무 바쁜 관계로 직접 따로 조사 해보았다.

---

# useLayoutEffect는 어떤 과정으로 구현되는가?

{% highlight js %}

const divRef = useRef()

useLayoutEffect(() => {
   window.innerheight > 500 ? divRef.current.color = "#000" : "#DFDFDF"
}, []);


return (
  <ul>
    <li>
      <div ref={divRef}></div>
    <li>
    <li>
      <div ref={divRef}></div>
    <li>
  </ul>
)

{% endhighlight %}


React 라이프 사이플 주기 중 componentDidMount, componentWillMount에 기능을 수행하는 Hook이다.

해당 역할을 담당하는 대표적인 Hook이 __useEffect__ 가 있는데, 이 둘은 웹 브라우저 구동 매커니즘 과정에 크게 차이가 있다.

- useEffect: __비동기적(asynchronous)__ 으로 실행된다.
- useLayoutEffect: __동기적(synchronous)__ 으로 실행된다.

## useEffect 라이프 사이클 매커니즘

![useEffect](https://user-images.githubusercontent.com/78064720/203919295-62945b2f-b6b1-4b3f-9a4b-1d083117e675.png)


## useLayoutEffect 라이프 사이클 매커니즘

![useLayoutEffect](https://user-images.githubusercontent.com/78064720/203918840-fa1df3f9-fbce-42b6-ac4b-011df1888b34.png)


---

# 그래서 useLayoutEffect 어떤 용도로 사용되는거야?

useEffect는 DOM Tree가 브라우저 상에 구성하기 위해 각 엘리먼트의 속성을 계산하는 과정과 실제 모니터 화면 상 Layout을 구현하는 과정 이후에 동작한다.

만일 해당 엘리먼트의 스타일 속성을 트리거로 사용할 경우 뒤늦게 화면 상에 스타일 속성을 추가하므로  __깜빡임 현상__ 을 볼 수가 있다.

반면 useLayoutEffect은 위 매커니즘 사진을 통해 알 수 있듯 Layout을 구현하는 과정 전에 스타일 속성을 추가하므로 UX 관점에서 매우 효율적이다.

