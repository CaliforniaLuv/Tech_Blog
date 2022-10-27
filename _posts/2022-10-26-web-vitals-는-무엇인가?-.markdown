---
layout: post
title:  web vitals는 무엇인가?
date:   2022-10-26 02:07:00 +0300
image:  web.png
tags:   Tech
---


웹 브라우저 성능에 관해 관심을 갖고 있는 상태에서 웹 바이탈(web vitals)라는 사용자 경험을 바탕으로 측정하는

방법론을 알게 되어 향후 프로젝트에 도입하고자 미리 공부하게 되어 기록하였다.


---

# Web Vitals 란?

Google에서 웹 성능에 관해 책정된 __"수준 높은 웹 경험을 제공하는데 필요한 필수 성능 지표 모음과 최적화 가이드라인".__  

웹 브라우저 시장 초기에는 개발자의 역량에 따라 보편적인 기준없이 판단했던 성능 수준을 웹 생명 주기, 사용자의 경험 등에 근거하여 성능 평가 정리 및 측정 기준을 판단하였다.

![image](https://user-images.githubusercontent.com/78064720/198005131-c07faf07-fde0-4773-99cb-0f3ce7426b0e.png)


---

# FP, First Paint

페이지 생성되는 최초 지점에서 각 사용자 디바이스 디스플레이 크기에 따른 첫 픽셀이 뷰-포트에 표시되는데 걸리는 시간.

FP 지점에서 많은 시간이 소요된다면 해당 페이지의 백지 상태(빈 여백)이 길다는 것으로 필요한 리소스를 불러오는데 문제가 있는지 판단할 수 있다.

묙표 시간으로는 1초 이하로 기준을 잡고 있다.

성능 지표의 목표 시간대는 아래와 같다.

```
 render second <= 1s  Good 😄
 render second <= 3s  Need improvement 🙁
 render second > 3s  Poor 😡
```

최초 표시에 영향을 주는 경우는 다음과 같다.

- HTML은 렌더링에 기준이 되어 불어오는데 걸리는 시간은 FP 지연에 영향을 준다.
- CSS 렌더링에 필수 리소스로 불러오는데 걸리는 시간은 FP 지연에 영향을 준다.
- Javascript를 동기적으로 불어오는데 걸리는 시간응ㄴ FP 지연에 영향을 준다. (만일 비동기일 경우 FP에 영향을 주지 않는다.)

---

# LCP, Largest Contentful Paint && FCP, First Contentful Paint

![images_yrnana_post_a0d24faf-b307-4fcc-a4db-b02b595aaa2b_image](https://user-images.githubusercontent.com/78064720/198173007-f07f567d-5259-43a2-b2cb-32ce6dcace97.png)


## LCP

구성된 웹 페이지의 주 컨텐츠(이미지, 텍스트, 영상 등등)가 모두 브라우저 상에 표시돼 사용자가 시각적으로 인식할 수 있기까지 걸리는 시간.

목표 시간으로는 2.5초 이하의 기준으로 잡고 있다. 

성능 지표의 목표 시간대는 아래와 같다.

```
 render second <= 2.5s  Good 😄
 render second <= 4s  Need improvement 🙁
 render second > 4s  Poor 😡
```

주요 컨텐츠인 HTML 요소들은 다음과 같다.

- ```<img>``` 엘리먼트
- ```<image>``` 엘리먼트 내의 ```<svg>``` 엘리먼트
- ```<video>``` 엘리먼트
- url() 함수 속성을 사용한 이미지 타입
- 텍스트 노드 또는 기타 인라인 수준 텍스트 요소를 자식으로 포함하는 블록 수준 엘리먼트


## FCP

웹 페이지의 일부 컨텐츠가 표기 되기 전까지 걸리는 시간.

주요 컨텐츠가 모두 표시되는 시간 전까지 측정하는 FCP와 다르다.

사용자가 페이지가 표시되고 있다는 것을 인지(표시가 다 됐다는 것은 LCP)하는 시점이며, Header 부분이 나오는 시점을 보통 FCP 목표 지점으로 보기도 한다.

목표 시간으로는 1초 이하이다. 

```
 render second <= 1s  Good 😄
 render second <= 3s  Need improvement 🙁
 render second > 3s  Poor 😡
```

컨텐츠가 최초로 표기되는 시간의 기준인 HTML 요소들은 다음과 같다.

- 텍스트 요소
- ```<img>``` 엘리먼트 (배경 이미지 포함)
- ```<svg>``` 엘리먼트
- 흰색이 아닌 ```<canvas>``` 엘리먼트

---

# CLS, Cumulative Layout Shift

페이지 전체 Life cycle(생명 주기) 중 발생하는 모든 레이아웃 이동(shift)에 대한 종합 점수.

사용자가 사용 중 불편함을 느끼거나 오인 입력을 하여 문제가 발생하는지 판단할 수 있는 지표이다.

사용자가 느낄 수 있는 불편한 이용은 아래와 같다.

![ezgif com-gif-maker](https://user-images.githubusercontent.com/78064720/198177865-a6b5c6ea-59b5-4a98-abe3-d277f343bf03.gif)


보통 페이지 콘텐츠의 예상치 않은 위치 변경 현상은 특정 요소의 리소스가 비동기식으로 처리되거나, DOM 요소가 기존 콘텐츠 위의 페이지에 동적으로 추가되기 때문에 발생한다.

대표적 원인은 아래와 같다.

- 이미지, 영상 요소
- 대체 크기보다 크거나 작게 렌더링되는 글꼴
- 동적으로 크기가 조정되어 렌더링되는 광고나 위젯

위 사항들을 방지하기 위해선 __이미지 및 비디오 요소에 항상 크기 속성을 포함하거나 CSS 가로 세로 비율 상자와 같은 방식으로 필요한 공간을 미리 확보__ 해야한다.

레이아웃 이동의 계산 수식은 다음과 같다.

```
Layout Shift Score: Impact Fraction * Distance Fraction
```

-  Impact Fraction: 뷰-포트 전체에 대해 몇 퍼센트 정도 영향을 주는지에 대한 값
-  Distance Fraction: 이동한 거리가 뷰-포트에 비해서 몇 퍼센트 정도 인지에 대한 값



---

# FID, First Input Deleay

사용자가 웹 페이지에서 내부 기능을 처음 시작하고 브라우저가 응답하여 실제로 이벤트 핸들러를 처리하기까지 걸리는 시간.

자바스크립트 문서의 처리 연산이 많을 경우, 사용자가 발생시킨 input Event들은 연산 처리가 완료되기 전까지 대기해야 한다.

사이트의 상호 작용에 대한 반응성 성능이 충분한지 판단할 수 있는 지표가 된다.

목표 시간으로는 100m 초 이내로 기준을 잡고 있다.


```
 render second <= 100ms  Good 😄
 render second <= 300ms  Need improvement 🙁
 render second > 300ms  Poor 😡
```

내부 기능 상호작용과 해당 사항이 아닌 경우는 다음과 같다.

- Click 입력 ✅
- Tab ✅
- Key 입력 ✅
- 스크롤 ❌
- 확대(줌) ❌



