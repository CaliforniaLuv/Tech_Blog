---
layout: post
title:  speech to text API를 활용하여 AI 채팅 구현하기
date:   2025-03-01
image:  lifestyle.jpeg
tags:   project
---

최근 회사 프로젝트에 집중하느라 블로깅을 소홀히 했다.

마침 2025년을 맞아 간단한 토이 프로젝트를 만들어볼까 싶어서 퇴근 후 틈틈이 3일 정도 작업해봤다.(다 만들어보니 당일치기로 충분히 만들 사이즈 규모인 거 같기도 하고;)

---

# 토이 프로젝트 소개

![giphy](https://github.com/user-attachments/assets/450c1f06-df03-4e39-b459-0f91ecf6947a)

지난 2024년을 돌아보며, 나만의 2024 오스카상을 수여한다면 공로상은 단연 ‘ChatGPT’에게 돌아갈 것이다. 이처럼 비슷한 기능을 구현하는 토이 프로젝트를 만들어보기로 결정했다.

## 자나깨나 사전 분석

ChatGPT에서 API를 지원한다는 사실을 이미 알고 있었기에, API 연동 작업에는 큰 어려움이 없을 거라 판단했다.

그러나 SST API 연결 과정에서 약 1시간이 소요되었고, 계속해서 ```500``` 에러 상태가 발생했다. 처음에는 설정 문제라고 생각해 여러 차례 공식 문서를 참고하며 테스트해봤지만, 해결되지 않았다. 

결국 잠시 보류하고, 채팅 API 단계로 이동하여 연동하기로 결정했다.

하지만 여기서도 문제가 발생했다. 이번에는 ```429``` 에러 상태가 떴는데, 특이한 점은 400번대 오류는 대부분 클라이언트 측 문제라는 것을 추측하여 OpenAPI 공식 문서를 통해 오류 코드 관련 표를 꼼꼼히 확인해보았다.

<img width="693" alt="스크린샷 2025-02-24 오전 1 56 15" src="https://github.com/user-attachments/assets/2125683e-959f-4bf4-95d6-aa7a23adb12c" />

대충 429 상태 에러코드는 비용 크레딧 문제에 관련된 내용이였고, 머릿속에서 다음과 같은 생각 회로가 그려졌다.

```
결제가 안 된 계정을 사용 → STT API 및 채팅 API 연동 → 채팅 API에서 429 상태 코드 발생 → 혹시 이거 유료 버전만 지원하는 건가?!!!
```

의심이 들어 OpenAPI의 서비스 비용 정보를 확인해보니, STT와 채팅 기능은 모두 유료였다. 나는 지금까지 OpenAPI 측에서 낮은 버전은 무료로 제공해줄 거라고 막연히 생각하고 있었는데(=놀부 심보), 사실 API 서비스는 엄연히 유료였다.
결국, 코드를 작성하기 전에 조금만 더 사전 분석을 철저히 했더라면 이런 어이없는 삽질을 하지 않았을 텐데… 😅

토이 프로젝트에 굳이 돈을 지출해가며 만들 생각은 없었기에 다른 AI API 서비스를 서칭한 결과 구글 AI API에서는 STT 기능과 채팅 기능을 모두 무료로 지원하고 있었고 해당 서비스를 채택하게 되었다.


# 동작 원리

![스크린샷 2025-01-24 오전 1 51 15](https://github.com/user-attachments/assets/fde78b13-087f-4141-b555-cbd6715b7a02)

동작 방식은 다음 사항과 같다.

1. 음성 입력 단계
사용자가 마이크를 통해 질문을 음성으로 입력하면, MediaStream API가 이를 캡처하여 Google AI의 STT(Speech-to-Text) 서비스로 전달한다.

2. 음성을 텍스트로 변환 (STT)
Google AI의 STT 서비스가 음성 데이터를 텍스트로 변환한 후, 변환된 텍스트를 반환한다.

3. 텍스트 분석 및 처리
텍스트 응답이 생성되면, 이를 기반으로 Web Speech API가 분석을 수행한다.

4. AI 응답 처리
텍스트 질문이 Gemini AI로 전달되며, Gemini가 질문의 의도를 파악한 후 적절한 응답을 생성한다.
예를 들어, 사용자가 "한국 날씨는 지금 어때?" 라고 질문하면, Gemini가 해당 요청을 처리하여 응답을 반환한다.

5. 최종 응답 반환
Gemini에서 생성한 응답이 다시 사용자의 화면에 표시되거나, Web Speech API를 통해 음성으로 출력될 수 있다.

___MediaStream API___: 사용자의 음성을 캡처
___Google AI STT___: 음성을 텍스트로 변환
___Gemini AI___: 질문을 분석하고 답변 생성
___Web Speech API___: 생성된 답변을 음성으로 변환 가능

즉, 사용자가 음성으로 질문하면 이를 STT로 변환하고, AI가 분석하여 응답을 제공하는 구조를 생성하였다.

사용된 기술 스택은 Next.js, TailwindCSS로 구성하였다.

---

# 프로젝트 작업 과정

## startRecording 함수 (녹음 시작)

{% highlight js %}

  const startRecording = async () => {
    if (stream === null || pending || mediaRecorder === null) return;

    setRecordingStatus("recording");

    const media = new MediaRecorder(stream, { mimeType: MIME_TYPE });
    mediaRecorder.current = media;
    mediaRecorder.current.start();

    const localAudioChunks: Blob[] = [];
    mediaRecorder.current.ondataavailable = (event) => {
      if (typeof event.data === "undefined") return;
      if (event.data.size === 0) return;
      localAudioChunks.push(event.data);
    };

    setAudioChunks(localAudioChunks);
  };

{% endhighlight %}

## stopRecording 함수 (녹음 종료)

{% highlight js %}

   if (mediaRecorder.current === null || pending) return;

    setRecordingStatus("inactive");
    mediaRecorder.current.stop();
    mediaRecorder.current.onstop = () => {
      const audioBlob = new Blob(audioChunks, { type: MIME_TYPE });
      uploadAudio(audioBlob);
      setAudioChunks([]);
    };

{% endhighlight %}

### 코드 구현 정리

![2025-03-015 54 31-ezgif com-video-to-gif-converter](https://github.com/user-attachments/assets/ac99e1e3-9573-4aa5-b2de-511aa06baabc)

- startRecording()
  - 마이크 스트림을 확인한 후 MediaRecorder 객체 생성
  - start()를 호출하여 녹음 시작
  - 녹음되는 오디오 데이터를 localAudioChunks 배열에 저장

- stopRecording()
  - stop()을 호출하여 녹음 중지
  - onstop 이벤트에서 audioChunks를 Blob으로 변환
  - 변환된 오디오 파일을 서버에 업로드 (uploadAudio(audioBlob))
  - audioChunks를 초기화하여 다음 녹음을 준비

- recordingStatus는 ```녹음 시작```, ```녹음 중```, ```API 응답 대기``` 상태로 이미지를 표현하고 있다.(Gif 참고)

## 녹음 파일 데이터 STT API 요청

{% highlight js %}

  const file = formData.get("audio") as File;

  if (!file || file.size === 0) {
    return {
      response: "오디오 음성 파일이 없습니다.",
      code: ApiResponseCode.AUDIO_FILE_NOT_FOUND,
    };
  }

  const audioBuffer = await file.arrayBuffer();

  // Google STT 클라이언트 초기화
  const client = new SpeechClient({
    credentials: GOOGLE_SPEECH_APPLICATION_CREDENTIALS,
  });

    const request = {
    audio: {
      content: Buffer.from(audioBuffer).toString("base64"),
    },
    config: {
      encoding:
        protos.google.cloud.speech.v1.RecognitionConfig.AudioEncoding.WEBM_OPUS,
      sampleRateHertz: 48000,
      languageCode: "ko-KR",
    },
  };

  // Google STT 호출
  const [response] = await client.recognize(request);

  const speechToText = response.results
    ?.map((result) => result.alternatives?.[0]?.transcript)
    .join("\n");

  if (!speechToText) {
    return {
      response: "정확한 음성을 입력해주세요.",
      code: ApiResponseCode.AUDIO_API_ERROR,
    };
  }

{% endhighlight %}

<img width="341" alt="스크린샷 2025-03-01 오후 7 14 47" src="https://github.com/user-attachments/assets/d547291e-8f00-41a0-ba0e-091fe40fcdbd" />


## STT 변환된 텍스트를 통해 Gemini AI 요청

{% highlight js %}

  // Google Chat 클라이언트 초기화
  const genAI = new GoogleGenerativeAI(
    process.env.GOOGLE_CHAT_APPLICATION_CREDENTIALS_JSON
  );
  const model = genAI.getGenerativeModel({ model: "gemini-1.5-flash" });

  // Google AI CHAT 호출
  try {
    const { response: result } = await model.generateContent(speechToText);

    const uniqueChatRequestId = crypto.randomUUID();

    return {
      id: uniqueChatRequestId,
      sender: result.text(),
      myQuestion: speechToText,
      response: "status 200 OK",
      code: ApiResponseCode.SUCCESS,
    };
  } catch (error: any) {
    console.error("AI Chat Error:", error);
    return {
      response: "AI Chat Error: " + error.message,
      code: ApiResponseCode.CHAT_API_ERROR,
    };
  }

{% endhighlight %}

<img width="324" alt="스크린샷 2025-03-01 오후 7 14 53" src="https://github.com/user-attachments/assets/8f8e0ac2-d5ac-43f7-bb44-fcab1db5ba61" />


### 코드 구현 정리

해당 코드들은 Next.js를 사용하기 때문에 별도의 Node.js 백엔드 서버를 구축할 필요 없이, 녹음된 데이터를 그대로 폼(form)에 담아 전달하는 방식으로 구현했다.

- 오디오 파일 확인 및 변환
  - formData에서 "audio" 키를 가져와 ___파일 객체(File)___로 변환
  - 파일이 없거나 크기가 0이면 오류 응답 반환 (AUDIO_FILE_NOT_FOUND)
  - 오디오 파일을 arrayBuffer()로 변환하여 바이너리 데이터로 준비
 
- Google STT API 호출
  - Google Speech-to-Text(STT) 클라이언트 초기화
  - 오디오 데이터를 Base64 문자열로 변환하여 request.audio.content에 저장
  - 오디오 파일을 arrayBuffer()로 변환하여 바이너리 데이터로 준비
  - client.recognize(request)를 호출하여 음성을 텍스트로 변환
  - 결과에서 가장 확률 높은 변환 텍스트를 추출(speechToText)
  - 변환된 텍스트가 없으면 오류 응답 반환 (AUDIO_API_ERROR)


- Google Generative AI(Chat) 호출
  - Google AI Chat 클라이언트 초기화 (GoogleGenerativeAI)
  - Gemini 1.5 Flash 모델을 사용하여 speechToText를 기반으로 AI 응답 요청
  - 응답을 받아 텍스트 변환 후, ___고유 요청 ID(uniqueChatRequestId)___를 생성하여 반환
  - AI 응답이 성공하면 200 OK 응답 반환
  - AI 응답 중 오류 발생 시, 오류 메시지를 포함한 응답 반환 (CHAT_API_ERROR)

## AI 응답 텍스트 읽기

{% highlight js %}
const [synth, setSynth] = useState<SpeechSynthesis | null>(null);
  const [voice, setVoice] = useState<SpeechSynthesisVoice | null>(null);
  const [pitch, setPitch] = useState(1);
  const [rate, setRate] = useState(1);
  const [volume, setVolume] = useState(1);

  useEffect(() => {
    if (!state.sender || !synth) return;

    const wordsToSay = new SpeechSynthesisUtterance(state.sender);
    wordsToSay.voice = voice;
    wordsToSay.pitch = pitch;
    wordsToSay.rate = rate;
    wordsToSay.volume = volume;

    synth.speak(wordsToSay);

    return () => {
      synth.cancel();
    };
  }, [pitch, rate, state, synth, voice, volume]);

  useEffect(() => {
    setSynth(window.speechSynthesis);
  }, []);

  const handleVoiceChange = (e: React.ChangeEvent<HTMLSelectElement>) => {
    const voices = window.speechSynthesis.getVoices();
    const selectedVoice = voices.find((v) => v.name === e.target.value);
    if (!selectedVoice) return;

    setVoice(selectedVoice);
  };

  const handlePitchChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setPitch(parseFloat(e.target.value));
  };

  const handleRateChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setRate(parseFloat(e.target.value));
  };

  const handleVolumeChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setVolume(parseFloat(e.target.value));
  };

{% endhighlight %}

<img width="520" alt="스크린샷 2025-03-01 오후 7 03 56" src="https://github.com/user-attachments/assets/7df56109-cd09-4c83-b514-30b1a920756d" />

### 코드 구현 정리

- 활용되는 상태 변수 (useState)
  - synth: ___음성 합성 객체(SpeechSynthesis)___를 저장
  - voice: 선택된 음성(SpeechSynthesisVoice) 저장
  - pitch: 음성의 높낮이(1 기본값) 조절
  - rate: 음성 속도(1 기본값) 조절
  - volume: 음성 크기(1 기본값) 조절

 - useEffect를 활용한 음성 출력 (synth.speak)
  - state.sender 값이 있을 때, synth가 존재하면 음성 출력 실행
  - SpeechSynthesisUtterance 객체를 생성하여 ___텍스트(state.sender)___를 음성으로 변환
  - 사용자가 선택한 ___음성(voice), 피치(pitch), 속도(rate), 볼륨(volume)___을 설정
  - synth.speak(wordsToSay)를 호출하여 음성 재생
  - useEffect 클린업 함수에서 synth.cancel()을 실행하여 음성을 중단

 - synth 객체 초기화 (useEffect)
  - window.speechSynthesis를 가져와 음성 합성 객체(synth)를 설정

 - 음성 설정 변경 핸들러
  - 음성 변경 (handleVoiceChange)
     - speechSynthesis.getVoices()를 호출하여 사용 가능한 음성 목록 조회
     - 사용자가 선택한 음성을 찾아서 voice 상태 업데이트.
  - 피치 변경 (handlePitchChange)
     - 사용자가 입력한 값을 parseFloat으로 변환하여 pitch 상태 업데이트
  - 속도 변경 (handleRateChange)
     - 입력값을 parseFloat 변환 후 rate 상태 업데이트
  - 볼륨 변경 (handleVolumeChange)
     - 입력값을 parseFloat 변환 후 volume 상태 업데이트
   
---

# 토이 프로젝트를 진행하면서 느낀 점

![2025-03-017 22 19-ezgif com-video-to-gif-converter](https://github.com/user-attachments/assets/be44cd60-f24a-4d15-bcce-869d0ae0cb78)


1. 무료 API의 한계

무료 버전 API를 사용하다 보니 응답의 질이 기대에 못 미치는 경우가 많았고, 정확한 답변을 원할 때는 유료 API를 고려해야 할 필요성을 느꼈다.

2. 깊이 있는 사고의 중요성

오랜만에 토이 프로젝트를 진행하면서, 내가 원하는 기능을 구현하기 위해 깊이 고민하는 과정이 새로웠다.
단순한 코드 작성이 아니라, 문제를 해결하는 사고 과정 자체가 재미있었다.


3. 블로그 포스팅 다짐

이런 경험을 정리하고 공유하는 것도 중요하다는 생각이 들었고, 올해는 개발 블로그를 자주 포스팅하면서 기록을 남기도록 노력을 해야겠다.

✅ 한마디로, 토이 프로젝트를 통해 기술적 고민의 즐거움을 다시금 느꼈고, 이를 공유하기 위해 블로그 활동을 활발히 할 계획이다! 🚀
 
