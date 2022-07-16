---
layout: post
title:  useEffect Rule
date:   2022-07-13 01:10:00 +0300
image:  Javascript.png
tags:   React
---

useEffect Hook 사용하던 중 아래의 오류가 발생하였다.


![스크린샷 2022-07-13 오전 1 44 40](https://user-images.githubusercontent.com/78064720/178789368-92b4275a-6d52-4285-8540-05ccdabd0d19.png)

그대로 해석하면,

```
React Hook "useEffect"는 조건부로 호출됩니다. React Hooks는 모든 구성 요소 렌더링에서 정확히 동일한 순서로 호출되어야 합니다.
```

useEffect는 아래와 같은 코드로 작성하였을 경우 에러가 발생한다. 

---

# 조건문 스코프 에러

{% highlight js %}

import React, { useEffect, useState } from "react";

export default function App() {
  const [count, setCount] = useState(0);

  if (count > 0) {
    useEffect(() => {
      console.log("카운트 발생");
    }, [count]);
  }

  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={() => setCount(count + 1)}>카운트</button>
    </div>
  );
}

{% endhighlight %}

useEffect를 if 조건문의 스코프 상에 종속되어 있으므로 오류가 발생하였다.

## 에러 해결

{% highlight js %}

import React, {useEffect, useState} from 'react';

export default function App() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    if (count > 0) {
      console.log('카운트 발생');
    }
  }, [count]);

  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={() => setCount(count + 1)}>카운트</button>
    </div>
  );
}

{% endhighlight %}

---

# 최상단 조건문 에러

{% highlight js %}

import React, {useEffect, useState} from 'react';

export default function App() {
  const [count, setCount] = useState(0);


  if (count > 0) {
    return <h1>카운터가 0보다 큽니다.</h1>;
  }
  
  useEffect(() => {
    console.log('카운터 발생');
  }, [count]);

  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={() => setCount(count + 1)}>카운터</button>
    </div>
  );
}

{% endhighlight %}

위 문제는 useEffect Hook보다 if 조건문이 상단에 위치를 잡은 관계로 에러가 발생하였다.

(useEffect 뿐만 아니라, 모든 react hook보다 조건문이 최상단에 위치되어 있으면 안된다.)

## 에러 해결

{% highlight js %}

import React, {useEffect, useState} from 'react';

export default function App() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log('카운터 발생);
  }, [count]);

  if (count > 0) {
    return <h1>카운터가 0보다 큽니다./h1>;
  }

  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={() => setCount(count + 1)}>카운터</button>
    </div>
  );
}
{% endhighlight %}
  
---
  
# 결론 
  
1. React Hook은 항상 최상위에 위치되어 있어야 한다.
2. 루프, 조건 또는 중첩 함수 내에서 useEffect 사용하면 안된다.
