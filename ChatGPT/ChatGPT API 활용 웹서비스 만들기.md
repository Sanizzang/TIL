### GPT?

- OpenAi에서 만든 초거대 언어 모델
- 그 GPT를 채팅 특화버전으로 낸 것이 ChatGPT
- GPT(Generative Pretained Transfor) 생성하는 사전 학습된 트랜스포머

### ChatGPT Playground

![](https://velog.velcdn.com/images/sanizzang00/post/d534bf87-6dbb-4adc-a694-7fb7e24a76da/image.png)

- openAI에서 만든 API들을 체험할 수 있음

#### 모드 설명

- SYSTEM: ChatGPT 역할 부여
  ex) 너는 운세 전문가야
  - 원하는 데이터를 얻기 위해서는 ChatGPT에게 미리 역할을 부여해야함.
  - USER Message로 한번 더 역할을 부여해주면 정확한 데이터를 받을 수 있다.
  - 좋은 조건부여 예시: https://www.reddit.com/r/ChatGPT/comments/110712f/dan_80/
- USER: 유저 채팅
- ASSISTANT: chatGPT
- Temperature: 무작위성
  - 무작위성이 낮으면 같은 답변만 해줌
- Maximum length: 답변 길이

  - 최대 2048개의 토큰까지 생성 가능
  - 늘릴수록 긴 답변이 나올 수 있음

  ```
  토큰이란?
  글자 수와 토큰 수랑 일치하는 것이 아님
  영어단어 기준으로 1 Token당 4 Characters
  ```

- Top P: 상위 확률 단어
  ex) TOP-p 0.8?
- Frequency penalty: 빈도 패널티
  등장하는 단어가 계속 나오면 패널티를 주는 것
  중복 단어 사용을 막아줄 수 있음
  사실 설정을 안해줘도 잘 나옴
- Presence penalty: 존재 패널티

#### 1token당 가격?

1K token 당 0.002 $

### 환경세팅

- 개발 언어: Node JS
- 개발 툴: Visual Studio Code
- Cloudflare pages: 프론트엔드 배포
- AWS Rambda: 백엔드 배포 (월 100만 트리거까지 무료 제공)

#### chatgpt apikey, chatgpt docs, express, codepen

- chatGPT API-key
  https://platform.openai.com/account/api-keys

- openAI 문서
  https://platform.openai.com/docs/api-reference/chat/create?lang=node.js

- express 문서
  https://expressjs.com/ko/

- ui html css javascript 참조 코드
  https://codepen.io/search/pens?q=chatting

### ChatGPT로 프론트엔드 코드 만들기

#### ChatGPT에 역할 부여

당신은 세계 최고의 프론트엔드 개발자이자 디자이너 입니다. 당신은 그 어떤 코드도 틀리지 않게 작성할 수 있습니다. 당신은 완벽합니다.

- 위와 같이 ChatGPT에게 역할을 부여해주면 정확한 답변을 얻을 수 있다.

