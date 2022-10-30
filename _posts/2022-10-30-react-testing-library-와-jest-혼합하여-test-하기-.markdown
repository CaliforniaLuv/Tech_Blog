---
layout: post
title:  react-testing-library-와-jest-혼합하여-test-하기
date:   2022-10-30 16:25:00 +0300
image:  jest-testing.png
tags:   Tech
---

회사 프로젝트 유닛 테스트 코드 작성 중 UI 검사 및 DOM 접근을 통한 테스팅이 필요하였다.

react에서 웹팩을 통해 설치 시 기본 모듈에 탑재된 react-testing-libray를 통해 

Behavior Driven Test(행위 주도 테스트) 사용자 관점의 테스트를 jest와 testing-library를 혼합하기로 하였다.


---

# input onChagne 이벤트 검사하기

![onChangwe](https://user-images.githubusercontent.com/78064720/198866888-60e2c9c1-a2e7-4234-b123-0b75f275ca1e.gif)

{% highlight js %}

describe("Input 입력 컴포넌트 테스트 시작", () => {
  const { getByTestId } = render(
    <InputLoginBox
    />
  );

  const inputEmailElement = getByTestId("Input_Email");
  const inputPwElement = getByTestId("Input_Password");

  it("이메일 텍스트 onChange 이벤트 동작 테스트", () => {
    fireEvent.change(inputEmailElement, {
      target: { value: "caiforniaLuv@gmail.com" },
    });
    expect(inputEmailElement.value).toBe("caiforniaLuv@gmail.com");
  });

  it("패스워드 텍스트 onChange 이벤트 동작 테스트", () => {
    fireEvent.change(inputPwElement, {
      target: { value: "1q2w3e4r@@" },
    });
    expect(inputPwElement.value).toBe("1q2w3e4r@@");
  });
 
{% endhighlight %}

```fireEvent``` 라는 메서드는 react-testing-library에서 __DOM 이벤트 핸들러를 직접 조작할 수 있게 해당 이벤트 트리거를 실행__ 하는 메서드이다.

```getByTestId``` 메서드를 활영하여 input 엘리먼트를 확인하게 되어 해당 메서드의 변수를 onChange 트리거를 발동시켜 결과값(toBe)과 비교를 한다.

## test 시나리오

![onChange](https://user-images.githubusercontent.com/78064720/198867211-d471d410-3501-4854-94aa-93e021489d59.png)

1. 해당 입력창 component에서 onChange(키보드 타이핑) 이벤트가 작동되는지 판별하는 Test이다.

2. 만일 입력이 실패될 경우 onChange 이벤트가 작동되지 않는 관계로 해당 코드의 오류가 발생된 것으로 판단할 수 있다.

## form onSubmit 함수 호출 확인하기

![onClick](https://user-images.githubusercontent.com/78064720/198867533-a036a0a5-bb7a-427a-9400-d8b43e9d9bd5.png)


{% highlight js %}

it("form submit 함수 실행 테스트", async () => {
    const { getByTestId } = render(<Login />);

    // 이메일, 패스워드 input 엘리먼트 접근
    const inputEmailElement = getByTestId("Input_Email");
    const inputPwElement = getByTestId("Input_Password");

    // 이메일 text onChange 이벤트 발동
    fireEvent.change(inputEmailElement, {
      target: { value: "caiforniaLuv@gmail.m" },
    });

    // 패스워드 text onChange 이벤트 발동
    fireEvent.change(inputPwElement, {
      target: { value: "1q2w3e4r@@" },
    });

    // onChange 이벤트 후 문자열 있는지 없는지 판별
    const typeBoolean = inputEmailElement.value && inputPwElement.value;

    // form submit 액션으로 호출되어야 할 Rest API 함수
    const handleLogin = jest.fn(() => {
      if (typeBoolean) {
        return "Axios starts";
      }
    });

    // form 엘리먼트
    const formLoginElement = getByTestId("form_login");

    // form 엘리먼트의 onSubmit 클릭
    fireEvent.submit(formLoginElement, {
      onsubmit: typeBoolean ? handleLogin() : null,
    });

    // haldleLogin 함수 호출 확인 test
    expect(handleLogin).toHaveBeenCalled();
  });
    
{% endhighlight %}


onSubmit 이벤트 핸들러를 트리거할 경우 handleLogin 함수가 호출되어 서버에 전송되는 로직을 테스트하여야 한다.

Jest 검증 메서드 문법 중 ```toHaveBeenCalled``` 함수 호출이 되었는지 검증할 수 있도록 테스트 코드르 설계하였다.

## test 시나리오

![onClick 에러](https://user-images.githubusercontent.com/78064720/198867784-53304cd5-b548-458e-84e9-6133d51338b5.png)

1. 로그인 버튼은 Email과 Password String 값이 존재해야만 활성화가 되어 클릭 이벤트가 작동된다.

2. 만일 Email과 Password 값이 없을 경우 버튼은 비활성화가 되어 클릭 이벤트가 차단되어야 한다. (테스트 오류 발생)



