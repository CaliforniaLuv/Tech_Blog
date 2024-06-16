---
layout: post
title:  React clean하게 사용하기 chap2
date:   2024-06-17 11:20:00 +0300
image:  React.png
tags:   react
---

한동안 두 개의 프로젝트를 동시에 처리하느라 정신없이 지내다 보니 블로깅을 많이 미뤘다. (개인적인 공부도 많이..) -.-
이번에는 그동안 공부하며 정리한 '```Props```를 깔끔하게 사용하는 방법'에 대해 정리해보자 포스팅을 남긴다.


# 무거운 연산을 활용하는 함수 반환값 효율적으로 관리하기

## ❌ 일반 변수에 저장하기

{% highlight javascript %}

const HeavyValueFail = ({ value }: { value: UserInfo }) => {
  // 🙅‍♂️ Bad
  const copyValue = modifyValue(value);

  return <div>{copyValue}</div>;
};

{% endhighlight %}


## ✅ useMemo를 통해 관리하기

{% highlight javascript %}

const HeavyValue = ({ value }: { value: UserInfo }) => {
  // 🙆‍♂️ Good

  const copyValue = useMemo(() => {
    return modifyValue(value);
  }, [value]);

  return <div>{copyValue}</div>;
};

{% endhighlight %}

## 코드 해석

무거운 연산이 실행되는 특정 함수가 리렌더링마다 동작하면 비효율적이다. 이때, useMemo를 사용하면 메모이제이션을 통해 연산 과정을 최적화할 수 있다.
무거운 연산 값은 가능하면 컴포넌트에 들어오기 전에 처리하여 전달하는 것이 데이터 흐름을 원활하게 유지하는 데 도움이 된다.

---

# Spread 문법 사용하여 Props 처리 간략화

## ✅ 객체 구조 분해 할당(단축 구문 처리) 및 Spread 문법 사용

{% highlight javascript %}

interface IShortProps {
  isView: boolean;
  isLogin: boolean;
  message: string;
  data: string[]
}

export const ShortProps = ({ isView, isLogin, ...props }: IShortProps) => {
  // 🙆‍♂️ Good
  return (
    <Header
      title="headerComponent"
      isLogin={isLogin}
      isView={isView}
      isAdmin
      hasToken
    >
      <MenuComponent {...props} />
    </Header>
  );
};

{% endhighlight %}

## 코드 해석

Props에 Spread 문법을 사용하면 일일히 나열하여 적을 필요도 없고 다른 개발자가 보았을 때 가독성이 좋아진다.
별도로 분리하여 사용해야 하는 Props가 있다면, 객체 구조 분해 할당(단축 구문)을 활용하여 spread 문법에서 제외하도록 코드를 작성하면 된다.
또한, true가 고정값일 경우, value를 생략하고 처리하면 명시적인 코드가 된다.

---

# 자료형 데이터 Props 지양하기

## ❌ 자료형 데이터 그대로 Props 처리

{% highlight javascript %}

const ObjectPropsFail = () => {
  const [data, setData] = useState({ id: 1, list: ["A", "B"] });

  // 🙅‍♂️ Bad
  return <ListContainer data={data} />;
  
{% endhighlight %}

## ✅ 자료형 데이터 분해 후 Props 처리

{% highlight javascript %}

const ObjectProps = () => {
  const [data, setData] = useState({ id: 1, list: ["A", "B"] });

  // 🙆‍♂️ Good
  return <ListContainer id={data.id} list={data.list} />;
};

{% endhighlight %}

## 코드 해석

자료형 데이터의 위험성
```
1. Object.is 메서드는 두 값이 값은 값인지 결정한다.
2. Object.is({hello: "world"}, {hello: "world"}) => false
두 객체는 엄연히 자바스크립트 메모리 상 서로 다른 주소에 저장되므로, 비교 결과는 false입니다.
```

즉, 위 결과에 따라 props에 동일한 값이 전달되더라도 메모리 주소가 다르기 때문에 React는 props가 업데이트된 것으로 인식하여 리렌더링이 발생한다.
ex) 특정 버튼 UI를 클릭할 때마다 list 상태의 첫 번째 인덱스에 "A"를 추가하는 과정에서, 매번 클릭할 때마다 상태 변화가 일어나면서 불필요한 리렌더링이 발생할 수 있다.
이외에도 불변성 유지가 어려워 부모에서 넘긴 값을 자식에서 수정할 수 있다. 이는 의도치 않은 변경일 경우, 추적도 어렵고 에상치 못한 결과가 view 화면에서 나타날 수 있다.

자료형 데이터는 일반 변수로 분리한 뒤, 하위 컴포넌트에 명확하게 전달해 주어야 하며, 하위 컴포넌트를 세부적으로 분리하여 값을 관리하면 불필요한 리렌더링을 방지하게 된다.

---

# ...Props(Spread Props) 주의할 점

## ❌ ...Props 그대로 보내기

{% highlight javascript %}

const CautionPropsFail = (props) => {
  // 🙅‍♂️ Bad
  return <ChildComponent {...props} />;
};
  
{% endhighlight %}

## ✅ 따로 분리하여 Props 처리하기

{% highlight javascript %}

const CautionProps = (props) => {
  const {
    해당_컴포넌트에만_쓰이는_props,
    별도로_관리가_필요한_사용되는_props,
    나머지_props,
  } = props;

  return (
    <div>
      {해당_컴포넌트에만_쓰이는_props}
      <ChildComponent
        별도로_관리가_필요한_사용되는_props={
          별도로_관리가_필요한_사용되는_props
        }
        {...나머지_props}
      />
    </div>
  );
};
  
{% endhighlight %}

## 코드 해석

무분별하게 Spread Props로 넘기면 해당 자식 컴포넌트에 대한 코드 예측을 하기 어렵다. (즉, 유지보수 및 디버깅 과정이 복잡해진다.)

자식 컴포넌트에 사용되는 props만 분리를 하면 코드 가독성이 높아진다.
비록 따로 분리하는 과정은 작업 리소스를 필요로 하지만, 불필요한 props 전달로 인한 디버깅에 소요되는 시간을 사전에 예방할 수 있다.

---

# 각 필요에 따른 컴포넌트를 분리하여 복잡한 컴포넌트 해소하기

## ❌ 하나의 컴포넌트에 많은 props 넘기기

{% highlight javascript %}

const SeparationComponentFail = () => {
  // 🙅‍♂️ Bad
  return (
    <FilterLayout
      onClickTagButton={onClickTagButton}
      onClickFilterButton={onClickFilterButton}
      handleFormData={handleFormData}
      auth={auth}
      userInfo={userInfo}
    />
  );
};
{% endhighlight %}

## ✅ 컴포넌트 분리하여 관리하기

{% highlight javascript %}

const SeparationComponentFail = () => {
  // 🙆‍♂️ Good
  return (
    <FilterLayout
      onClickTagButton={onClickTagButton}
      onClickFilterButton={onClickFilterButton}
    >
      <Certification auth={auth} />
      <FormBox handleFormData={handleFormData} />
      <UserInfoBox userInfo={userInfo} />
    </FilterLayout>
  );
};
  
{% endhighlight %}

## 코드 해석

코드가 지저분하고 유지보수가 어려울 경우, 리펙토링이 필요하여 많은 리소스가 발생할 수 있다.
왜냐하면 많은 컴포넌트 props 중 사용하지 않는 유령 props들이 그대로 남아 있으면, 버그가 발생할 경우 이를 찾기가 어려워진다.
