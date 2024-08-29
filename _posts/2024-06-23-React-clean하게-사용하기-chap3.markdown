---
layout: post
title:  React clean하게 사용하기 chap3
date:   2024-06-23 20:20:00 +0300
image:  React.png
tags:   react
---

이번 주에는 리액트 렌더링을 효율적으로 최적화하고 관리하는 방법에 대해 클린 코드 내용을 정리하고자 한다.


---


# 무분별하게 내부 컴포넌트를 생성하지 않기

## ❌ 내부 컴포넌트 안에 내부 컴포넌트

{% highlight javascript %}

// 🙅‍♂️ Bad
const AnotherComponent = () => {
  return "AnotherComponent";
};

const InnerComponent = () => {
  return (
    <div>
      <AnotherComponent />
    </div>
  );
};

const OuterComponent = () => {
  return <InnerComponent />;
};

{% endhighlight %}

## ✅ 상위 컴포넌트에서 한 번에 관리하기

{% highlight javascript %}

// 🙆‍♂️ Good
const OuterComponent = () => {
  return (
    <>
      <InnerComponent />
      <AnotherComponent />
    </>
  );
};

{% endhighlight %}

## 코드 해석

내부 컴포넌트를 무분별하게 생성하면 아래와 같은 문제가 발생할 수 있다.

 - 크게 분리할 필요가 없는 컴포넌트를 세부적으로 나누면 해당 컴포넌트의 분석이 복잡해진다.
 - 리팩토링 과정에서 분리가 필요한 시점이 발생하면 작업이 어려워진다.
 - 상위 컴포넌트의 상태 변화로 인해 리렌더링이 실행될 경우, 불필요한 하위 컴포넌트의 재생성이 발생하여 성능이 저하된다.

그러므로 상위 컴포넌트에서 가능한 모든 작업을 표현하도록 구현하는 것이 좋다.
만약 컴포넌트가 너무 복잡해지거나 로직의 정리가 필요할 경우, 또는 반복되는 작업이 다른 페이지에서도 동일하게 구현되고 있다면, 
그때서야 재사용 가능한 모듈화 컴포넌트로 분리하는 작업을 하면 된다.

---

# 숫자 타입으로 조건부 렌더링 관리하지 않기

## ❌ 숫자 0을 통한 비교 연산


{% highlight javascript %}

export const ZeroValueFail = () => {
  // 🙅‍♂️ Bad
  const [data, setData] = useState([]);

  return <>{data.length && <div>hello world</div>}</>;
};

{% endhighlight %}

## ✅ boolean 타입을 통한 비교 연산

{% highlight javascript %}

export const ZeroValue = () => {
  // 🙆‍♂️ Good
  const [data, setData] = useState([]);

  return <>{!!data.length && <div>hello world</div>}</>;
};

{% endhighlight %}

## 코드 해석

숫자 0은 엄연히 값으로 취급되기 때문에 JSX 문법상 텍스트 0으로 렌더링될 수 있다. 
이는 의도치 않게 view 화면에 0이 나타날 수 있음을 의미 한다.
확실한 방법은 boolean 타입을 사용하여 조건부 렌더링을 관리하는 것이 코드와 의도하는 동작을 한다.

---

# List key index 고유의 값으로 관리하기

## ❌ 사용하면 안되는 key index

{% highlight javascript %}

export const ListKeyFail_1 = ({ data }: ListKeyFail_1_Props) => {
  // 🙅‍♂️ Bad
  return (
    <>
      {data.map((item, idx) => (
        <div key={idx}></div>
      ))}
    </>
  );
};

{% endhighlight %}

{% highlight javascript %}

export const ListKeyFail_2 = ({ data }: ListKeyFail_2_Props) => {
  // 🙅‍♂️ Bad
  return (
    <>
      {data.map((item) => (
        <div key={new Date().toString()}>{item.text}</div>
      ))}
    </>
  );
};

{% endhighlight %}

## ✅ 고유의 index로 key 속성 관리하기.

{% highlight javascript %}

export const ListKey = ({ data }: ListKeyFailProps) => {
  // 🙆‍♂️ Good
  return (
    <>
      {data.map((item) => (
        <div key={item.id}>{item.text}</div>
      ))}
    </>
  );
};

{% endhighlight %}

## 코드 해석

먼저 ```ListKeyFail_1```에 관해 문제점은 아래와 같다.

데이터에 추가, 변경, 삭제 같은 동적 변화가 발생하면, 고유의 ID가 아닌 인덱스를 키로 사용할 때 다음과 같은 문제가 발생할 수 있다.

대표적으로 아래 세 가지 문제가 있다.

1. 리스트 항목의 순서가 변경될 때의 문제
 - 예를 들어, 리스트의 첫 번째 항목이 삭제될 경우 나머지 항목의 인덱스가 변경된다.
 - 그러나 리액트는 인덱스를 키로 사용하기 때문에, 각 항목을 이전 상태의 항목과 동일한 것으로 간주하여 예기치 않은 동작을 유발할 수 있다.
   
2. 리액트 상태 관리 문제:
 - 리액트는 키를 기반으로 컴포넌트의 상태를 관리한다.
 - 항목의 순서가 바뀌어 1번 인덱스가 2번 인덱스로 변경되면, 1번 인덱스에 대한 상태가 2번 인덱스로 이동한 항목에 잘못 적용될 수 있다.
   
3.성능 저하:
 - 인덱스 키를 사용하는 경우, 리스트 항목이 자주 변경되면 리액트는 모든 항목을 새롭게 렌더링한다.
 - 이는 불필요한 DOM 업데이트를 유발하고, 성능 저하로 이어질 수 있다.
 
그러나 CRUD 중 Read에만 사용되는 경우, 인덱스를 키로 사용하는 것은 괜찮다.

다음 ```ListKeyFail_2```에 관해 문제점은 아래와 같다.

현재 시간이나 uuid(create_UUID())와 같은 랜덤한 값을 key 연산자로 사용하는 경우가 종종 있다.
그러나 렌더링 시점에서 인라인 코드로 이러한 값을 바로 지정하면, 매번 리렌더링할 때마다 새로운 고유 값을 계속 지정하게 된다.
이로 인해 리액트는 각 요소를 항상 새로운 요소로 간주하게 된다. 따라서 이는 효율적인 렌더링을 방해하고, 성능에 영향을 미칠 수 있다.

만일 임의로 고유의 key 값을 직접 클라이언트에서 생성하여 관리가 필요할 수 있다.
그럴 경우, 새로운 아이템 요소를 추가할 때 함수를 만들고 useEffect를 사용하여 렌더링 전 고유의 id를 추가해주면 무분별하게 id를 생성하는 문제를 막을 수 있다.


대부분 서버 API 응답 데이터를 처리할 때, 리스트 형태의 데이터를 다루게 된다. 
이 경우 서버에서 응답해준 값에 고유의 ID가 있다면, 이를 key 값으로 지정하여 각 항목을 독립적으로 관리하는 것이 좋다. 이렇게 하면 효율적인 렌더링이 가능해지고, 성능 저하를 방지할 수 있다.

---

# Raw HTML 코드의 위험성 방지하기

## ❌ XSS 공격에 취약한 코드

{% highlight javascript %}

const SERVER_DATA = '<p>name: californiaLuv</p>'

export const DangerouslySetInnerHTMLComponentFail = () => {
  const markup = {__html: SERVER_DATA}
  // 🙅‍♂️ Bad
  return <div dangerouslySetInnerHTML={markup} />
};

{% endhighlight %}


## ✅ dangerouslySetInnerHTML 및 보안 라이브러리 활용하기

{% highlight javascript %}

import DOMPurify from 'dompurify';

const SERVER_DATA = '<p>name: californiaLuv</p>'

export const DangerouslySetInnerHTMLComponent = () => {
  const sanitizerInfo = {__html: DOMPurify.sanitize(SERVER_DATA)}
  // 🙆‍♂️ Good
  return <div dangerouslySetInnerHTML={sanitizerInfo} />
};

{% endhighlight %}

## 코드 해석

앞서 설명 전, 생소할 수도 있는 dangerouslySetInnerHTML와 "Raw HTML"에 대해 설명하겠다.

```dangerouslySetInnerHTML```
 - 리액트에서 사용되는 속성으로, HTML을 동적으로 삽입할 때 사용되며, 이 속성을 사용하면 HTML 문자열을 컴포넌트의 DOM 요소에 삽입할 수 있다.
 - 이름에서도 알 수 있듯이 이 속성을 사용하는 것은 "위험"할 수 있으며, XSS 공격에 주의가 필요하다.

```Raw HTML```
 - "Raw HTML" 또는 "순수 HTML"은 HTML 문서에서 그대로 삽입되는 원시적인 형태의 HTML 코드를 의미한다.
 - 이는 HTML 문서의 본문에 직접 작성된 HTML 태그와 속성을 말한다.

위 코드는 XSS(Cross-Site Scripting) 공격에 매우 취약하다.
dangerouslySetInnerHTML을 사용할 때, 사용자 입력이 포함된 HTML을 직접 삽입할 수 있다. 이는 악의적인 사용자가 스크립트를 삽입하여 사이트의 보안을 침해할 수 있는 가능성을 열어두는 꼴이 된다.
예를 들어, 사용자가 입력한 "<script>alert('XSS Attack!');</script>"와 같은 코드가 삽입될 경우, 해당 스크립트가 실행될 수 있다.

그러므로 HTML Sanitizer(HTML 필터링) 라이브러리를 활용하는 것은 dangerouslySetInnerHTML 속성을 사용할 때 중요한 보안적인 조치가 된다. 
HTML Sanitizer는 입력된 HTML에서 안전하지 않은 요소나 스크립트를 제거하거나 이스케이프하여 XSS 공격을 방지하는 역할을 하며 이를 통해 사용자가 입력한 HTML을 안전하게 처리할 수 있다.
