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


---

# form onSubmit 함수 호출 확인하기

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

---

# form onSubmit 함수 호출 후 실패 문구 Text 및 Color 검사하기

<img width="522" alt="스크린샷 2022-11-01 18 54 26" src="https://user-images.githubusercontent.com/78064720/199208064-74a3dec5-7ff8-43f8-939f-2650189e82e6.png">


{% highlight js %}

it("form submit 실패시 발생하는 텍스트 문구 테스트", async () => {
    const user = {
      email: "caiforniaLuv@gmail.com",
      password: "1q2w3e4r@@",
    };

    const statusType = {
      fail_msg400: "비밀번호가 올바르지 않습니다.",
      fail_msg404: "존재하지 않는 email입니다.",
    };

    const { getByTestId } = render(<Login />);

    // 이메일, 패스워드 input 엘리먼트 접근
    const inputEmailElement = getByTestId("Input_Email");
    const inputPwElement = getByTestId("Input_Password");

    // form 엘리먼트
    const formLoginElement = getByTestId("form_login");

    // 이메일 text onChange 이벤트 발동
    fireEvent.change(inputEmailElement, {
      target: { value: "caiforniaLuv@gmail.com" },
    });

    // 패스워드 text onChange 이벤트 발동
    fireEvent.change(inputPwElement, {
      target: { value: "1q2w3e4r@@" },
    });

    // 이메일, 비밀번호 공백 판별
    const typeBoolean = inputEmailElement.value && inputPwElement.value;

    // 로그인 버튼 클릭 후 이메일, 비밀번호 작성한 값에 따라 상태코드 전달
    const loginCheck = (email, password) => {
      if (email === user.email && password === user.password) {
        return true;
      } else if (email !== user.email && password === user.password) {
        return 404;
      } else if (email === user.email && password !== user.password) {
        return 400;
      } else {
        return 0;
      }
    };

    const handleLogin = jest.fn(() => {
      if (loginCheck(inputEmailElement.value, inputPwElement.value) === true) {
        return 200;
      } else {
        return loginCheck(inputEmailElement.value, inputPwElement.value);
      }
    });

    // 로그인 버튼 클릭 함수 (직접 호출하여 리턴값 변수에 저장 필요)
    var login_test = handleLogin();

    // 로그인 버튼 클릭 이벤트 발생
    fireEvent.click(formLoginElement, {
      onsubmit: typeBoolean ? handleLogin() : null,
    });

    // 클릭 이벤트 확인 test
    expect(handleLogin).toHaveBeenCalled();

    render(<LoginFail loginStatus={login_test} loginList={statusType} />);

    // 오류 문구 DOM 접근 함수
    const result = (id) => {
      const failElement = getByTestId(id);
      // 텍스트 color 확인 test
      expect(failElement).toHaveStyle({ color: "#c00a21" });
      return failElement;
    };

    // 로그인 클릭 결과에 따른 상태코드 판별
    if (login_test === 200) {
      expect(null).toBe(null);
    } else if (login_test === 404) {
      // 이메일 틀릴 경우 텍스트 결과 확인 test
      expect(result("Login_Fail_404")).toHaveTextContent(
        "존재하지 않는 email입니다."
      );
    } else if (login_test === 400) {
      // 비밀번호 틀릴 경우 텍스트 결과 확인 test
      expect(result("Login_Fail_400")).toHaveTextContent(
        "비밀번호가 올바르지 않습니다."
      );
    }

{% endhighlight %}
 
## test 시나리오 1

![password](https://user-images.githubusercontent.com/78064720/199208644-b6a7b23a-554c-430c-87e2-991380c869c0.png)

1. 로그인 입력창에 이미 존재하는 이메일을 전송할 경우 404 상태 코드 전송된다.
 
2. 비밀번호가 옳지 않을 경우 400 상태 코드 전송된다.

## test 시나리오 2

![dom](https://user-images.githubusercontent.com/78064720/199208784-4f75851d-1587-4a28-b61f-7c177dd804ad.png)

![color](https://user-images.githubusercontent.com/78064720/199208802-e8791597-1cb1-4b3f-a38b-6bcf5548ada9.png)

1. 로그인 하단 Text의 color가 #c00a1(브라우저 DOM에선 rgb code로 변환) 

2. 만일 color를 다른 code로 작성할 경우 테스트 오류가 발생한다.
