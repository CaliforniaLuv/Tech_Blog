---
layout: post
title:  React clean하게 사용하기 chap1
date:   2024-03-30 15:20:00 +0300
image:  react.png
tags:   react
---

작년부터 새로운 프로젝트에 투입되어 올해 2월 다 끝나가는 시점, 프로젝트를 무사히 마무리 지었다.
프로젝트를 끝나고 회고 하면서 크게 느낀 점은 코드를 작성할 때, 깊게 생각하지 않고 무작정 "구현만 되면 된다" 라는 마인드로 작업을 임했다.
매번 코드 리뷰마다 안티패턴을 유발시키는 코드로 인해 지적을 받아, 내 자신이 너무 한심하고 무능하기도 하면서 "내가 이렇게 무책임한 놈이였나?" 라는 생각을 했다.

이번 시간 다시 초심으로 돌아가자는 반성을 하여 'React ```clean``` 하게 사용하는 법' 강의를 보면서 포스트 남기고자 한다.

---

# 상태 변수 초기값의 중요성

## ❌ 초기값 지정하지 않은 상태 변수

{% highlight javascript %}

const InitialState = () => {
  // 🙅‍♂️ Bad
  const [data, setData] = useState();

  useEffect(() => {
    const getData = async () => {
      const response = await fetch("https://www.test.com");
      const result = await response.json();

      setData(result);
    };

    getData();
  }, []);

  return (
    <div>
      {!!data && data.map((item) => <div>{item}</div>)}
    </div>
  );
}

{% endhighlight %}

## ✅ 초기값 지정한 상태 변수

{% highlight javascript %}

const InitialState = () => {
  // 🙆‍♂️ Good
  const [data, setData] = useState([]);

  useEffect(() => {
    const getData = async () => {
      const response = await fetch("https://www.test.com");
      const result = await response.json();

      setData(result);
    };

    getData();
  }, []);

  return (
    <>
      <div>
        {data.map((item) => (
          <div>{item}</div>
        ))}
      </div>
    </>
  );
}

{% endhighlight %}

## 코드 해석

먼저 상태 변수 초기값을 지정 하지 않은 코드는 Side Effect가 발생할 때 위험이 크다.
API 응답 데이터를 받기 전(비동기)에는 undefined 상태인데 return 문에서 바로 map 메서드를 실행하므로 TypeError가 발생할 수 있고
TypeError를 방지하기 위해 방어 코드를 작성해야하는 작업 리소스가 생긴다.

반대로 상태 변수 초기값을 지정하게 되면 빈 배열 TypeError를 방지하게 되고 방어 코드 또한 작성할 필요가 없어진다.


---

# 업데이트 되지 않는 변수는 외부로 뺴기


## ❌ 내부 컴포넌트에 있는 변수

{% highlight javascript %}

export const Constant = () => {
  // 🙅‍♂️ Bad
  const DATA = {
    kr: "한글",
    en: "english",
  };

  return <div>{DATA.en}</div>;
};

{% endhighlight %}

## ✅ 외부로 관리하는 변수

{% highlight javascript %}

// 🙆‍♂️ Good
const DATA = {
  kr: "한글",
  en: "english",
};

export const Constant = () => {
  return <div>{DATA.en}</div>;
};

{% endhighlight %}

## 코드 해석

업데이트가 되지 않는 변수란 값이 변하지 않고(불변성) 고유의 값, 형태를 그대로 유지하는 것을 말하는데 이를 '상수(Constant)' 라고 부른다.
내부 컴포넌트의 변수가 업데이트 되지 않는다면 해당 스코프 내부에 나둘 필요가 없다. 외부로 뺴면 가독성도 좋고 해당 상수가 다른 컴포넌트에도 쓰이게 될 때
관리하기 편하기 때문이다.

---

# 플래그(Flag) 변수로 제어하기

## ❌ 상태 변수로 Mount 시점 제어하기

{% highlight javascript %}

const FlagValue = (token, data, isLoading) => {
  // 🙅‍♂️ Bad
  const [isSuccess, setIsSuccess] = useState(false);
  useEffect(() => {
    if (token && data && isLoading === false) {
      return setIsSuccess(true);
    }
  }, [token, data, isLoading])

  return <div>{isSuccess && "접근 성공하였습니다~"}</div>;
}

{% endhighlight %}

## ✅ Flag 변수로 Mount 시점 제어하기

{% highlight javascript %}

const FlagValue = (token, data, isLoading) => {
  // 🙆‍♂️ Good
  const isSuccess = token && data && isLoading;
  return <div>{isSuccess && "접근 성공하였습니다~"}</div>;
}

{% endhighlight %}

## 코드 해석

플래그(Flag) 변수란?

```
프로그래밍에서 주로 특정 조건 혹은 제어를 위한 조건을 Boolean 타입으로 사용함.
```

즉, React 상에서 상태 변수가 아닌 Flag 변수로 bool 값을 관리하면 깔끔하다.
위 작성한 안티패턴 코드 같이 작성하게 되면 상태값이 많아져 렌더링 영향으로 관리 포인트가 복잡해지기 때문이다.

---

# 불필요한 상태 만들지 않기

## ❌ 불필요한 상태 변수로 데이터 다루기

{% highlight javascript %}

const MOCK_DATA = [
  {
    id: 1,
    completed: true,
  },
  {
    id: 2,
    completed: false,
  },
  {
    id: 3,
    completed: true,
  },
]

const UnnecessaryState = () => {
  // 🙅‍♂️ Bad
  const [saveList, setSaveList] = useState(MOCK_DATA);
  const [list, setList] = useState(MOCK_DATA);

  useEffect(() => {
    const newList = list.filter((item) => item.completed);
    setSaveList(newList);
  }, [list]);

  return (
    <div>
      {saveList.map((item) => (
        <span key={item.id}>{item.id}</span>
      ))}
    </div>
  );
};

{% endhighlight %}

## ✅ 일반 변수로 관리하기

{% highlight javascript %}

const UnnecessaryState = () => {
  // 🙆‍♂️ Good
  const newList = MOCK_DATA.filter((item) => item.completed);

  return (
    <div>
      {newList.map((item) => (
        <span key={item.id}>{item.id}</span>
      ))}
    </div>
  );
};

{% endhighlight %}


## 코드 해석

MOCK_DATA를 상태 변수로 저장하고 그 저장한 값을 가공하여 또 다른 상태 변수에 저장을 하게 되면 렌더링 영향으로 관리 포인트가 매우 복잡해진다.
일반 변수로 가공하여 관리하면 코드가 훨씬 간단하고 가독성이 좋아진다.

---

# 이전(prev) 상태 활용하기

## ❌ 연속적으로 setState 호출 하기

{% highlight javascript %}

const prevState = () => {
  const [number, setNumber] = useState(0)

  const handleNumber = () => {
    // 🙅‍♂️ Bad
    setNumber(number + 1)
    setNumber(number + 1)
    setNumber(number + 1)
  }

  return (
    <button onClick={prevState} />
  )
}

{% endhighlight %}


## ✅ 상태 업데이트 함수로 호출 하기

{% highlight javascript %}

const prevState = () => {
  const [number, setNumber] = useState(0)

  const handleNumber = () => {
    // 🙆‍♂️ Good
    setNumber((prev) => prev + 1)
    setNumber((prev) => prev + 1)
    setNumber((prev) => prev + 1)
  }

  return (
    <button onClick={prevState} />
  )
}

{% endhighlight %}


## 코드 해석

상태 변수의 업데이트(setState)는 __비동기적으로 동작__ 하기 때문에 즉각적으로 반영 되지 않는데, 그 이유는 컴포넌트가 리렌더링 발생 되기 전까지 새로운 값을 갱신하지 않기 때문이다.
그러므로 핸들러 함수에서 연속적으로 setState을 선언하면 __가장 마지막의 setState의 값만 인식__ 하여 업데이트 한다. 연속적인 setState을 동기적으로 처리하기 위해선 
콜백 함수를 활용하여 이전 값을 기억하는 인자를 통해 처리해야 한다.

특히 onChange 같은 이벤트 함수에 setState를 처리하는 로직이 들어갈 경우, 직접적으로 변경하도록 코드를 작성하면 의도치 못한 에러가 발생할 수도 있으니 주의해야 한다.

