---
layout: post
title: React 자동 배칭 (auto batching)
date:   2022-11-08 12:58:00 +0300
image:  React.png
tags:   Tech
---

React 18 버전 이후부터 자동 배칭(auto batching)을 통해 렌더링 성능 개선에 향상 되었다.

18 버전 이전에 사용하지 않은 React 이용자들은 어떤 작동 원리를 통해 렌더링이 되었는지 모르기 때문에 다뤄보고자 한다. 

---

# 배칭은 무엇인가?

배칭이란 React Hook인 state의 다수 요소들이 한꺼번에 하나의 리렌더링이 발생하도록 묶는 것이다.

쉽게 요약하자면, 특정 엘리먼트의 이벤트 핸들러가 발생되어 여러 개의 state 요소의 값들이 업데이트가 될 경우, 

해당 이벤트에 소속된 state들은 하나의 리렌더링으로 동작한다.

{% highlight js %}

function App() {
  const [firstNumber, setFirstNumber] = useState(1);
  const [secondNumber, seSecondNumber] = useState(1);

  const handleClick = () => {
    setFirstNumber(firstNumber + 1); // 리렌더링 발생 ❌
    seSecondNumber(secondNumber + 1); // 리렌더링 발생 ❌
    
    // handleClick 함수 모든 로직 종료 후 리렌더링 발생 ✅
  };
  
  console.log("렌더링 발생"); // 리렌더링 과정을 통해 단 한번만 실행된다.

  return (
    <div className="page">
      <button onClick={handleClick}>버튼 클릭하기</button>
    </div>
  );
}

{% endhighlight %}

위 코드에서 함수형 업데이트를 사용하여도 똑같이 ```handleClick``` 함수의 로직이 종료되지 않는다면 똑같은 리렌더가 즉각적으로 발생하지 않는다.

결국 콘솔 로그의 "렌더링 발생"이라는 문자열은 단 한 번만 실행된다.

만일 두 개 이상의 state 상태 함수가 있는 상태에 리렌더가 각각 발생한다면 웹 브라우저 성능 면에서 효율성이 매우 떨어질 것이다.

# 17버전 이전의 React 리렌더링 과정

{% highlight js %}

function App() {
  const [firstNumber, setFirstNumber] = useState(1);
  const [secondNumber, seSecondNumber] = useState(1);

  const handleClick = () => {
    setFirstNumber(firstNumber + 1); // 리렌더링 발생 ✅
    seSecondNumber(secondNumber + 1); // 리렌더링 발생 ✅
  };
  
  console.log("렌더링 발생"); // 리렌더링 과정을 통해 두 번 실행된다.

  return (
    <div className="page">
      <button onClick={handleClick}>버튼 클릭하기</button>
    </div>
  );
}

{% endhighlight %}

이를 통해 18버전과 전 버전의 차이는 자동 배칭이 이루어지는가에 따라 리렌더 최적화를 명확하게 확인할 수 있다.

비동기 요청, setTimeOut, 이벤트 핸들러는 React에서 제공하는 이벤트들과 동일하게 state 업데이트를 자동 배칭한다.

# 자동 배칭

{% highlight js %}

function App() {
  const [firstNumber, setFirstNumber] = useState(1);
  const [secondNumber, seSecondNumber] = useState(1);

  const handleClick = async() => {
    try {
      await axios.post("www.californiaLuv.com/signup", inputText)
      setFirstNumber(firstNumber + 1); // 리렌더링 발생 ❌
      seSecondNumber(secondNumber + 1); // 리렌더링 발생 ❌
      
      // 비동기 RESTful API 과정이 모두 종료된 후 리렌더링이 발생한다. ✅
    };
    catch(err) {
      setFirstNumber(firstNumber + 1); // 리렌더링 발생 ❌
      seSecondNumber(secondNumber + 1); // 리렌더링 발생 ❌
      
      // 비동기 RESTful API 과정이 모두 종료된 후 리렌더링이 발생한다. ✅
    }
  
  console.log("렌더링 발생"); // 리렌더링 과정을 통해 두 번 실행된다.

  return (
    <div className="page">
      <button onClick={handleClick}>버튼 클릭하기</button>
    </div>
  );
}

{% endhighlight %}

Promise 비동기처리 과정에서 모든 로직이 종료된 후에만 리렌더링이 발생되므로 자동 배칭이 이루어진 것을 확인할 수 있다.


# 배칭 설정을 원하지 않는다면?

{% highlight js %}

import { flushSync } from "react-dom"; 

function App() {
  const [firstNumber, setFirstNumber] = useState(1);
  const [secondNumber, seSecondNumber] = useState(1);

  const handleClick = () => {
    flushSync(() => {
      setFirstNumber(firstNumber + 1); // 리렌더링 발생 ✅
    })
    // React Dom 업데이트 새 완료
    
    flushSync(() => {
    seSecondNumber(secondNumber + 1); // 리렌더링 발생 ✅
    })
    // React Dom 업데이트 새 완료
    
    // 🤔 secondNumber는 과연 2일까 1일까?
    console.log("secondNumber :", secondNumber);
  };
  
  console.log("렌더링 발생"); // 리렌더링 과정을 통해 두 번 실행된다.

  return (
    <div className="page">
      <button onClick={handleClick}>버튼 클릭하기</button>
    </div>
  );
}

{% endhighlight %}

react-dom의 ```flushSync``` 메서드를 활용하면 자동 배칭 트리거를 막을 수 있게 된다.

즉 ```handleClick``` 함수에서 두 번의 렌더링이 발생된다.

그렇다면 두번의 리렌더가 일어나면 ```secondNumber```는 1일지 2일지 궁금할 수도 있을텐데 답은 __1__ 이 나온다.

왜냐하면, 리렌더가 발생하더라도 해당 ```handleClick``` 함수의 로직은 종료가 안된 시점이므로 여전히 1이라는 값이 콘솔 로그에 실행되기 때문이다.


