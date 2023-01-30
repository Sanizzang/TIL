react 프로젝트 생성
npm init react-app my-app

react 프로젝트를 위한
기본적인 기본적인 라이브러리가 설치가 된
상태로 시작이 됨.

- public
  : React로 만든 코드는 결국 자바스크립트로
  만들어지게끔 되어 있다.
  그리고 최종적으로 index.html 파일 하나 갖고 구동을 하게 된다.

- manifest.json
  : favicon 관련된 설정들

- package.json
  : 프로젝트에 대한 정보들을 담고 있음
  - dependencies
    : 실제 react앱을 구동할 때 필요한 모듈들에 대한 정보
  - scripts
    : 스크립트 명령어를 설정
  - eslintConfig
    : 코드가 올바르게 작성되고 있는지 체크해줌
  - browserslist
    : 어떤 브라우저를 지원할 것인가

npm start를 하는 순간 제일 먼저 실행되는 것은 index.js 파일이다.

index.html에 보면 bundle.js가 없다. npm start로 클라이언트 서버를 구동시키면 index.html에 bundle.js 를 추가한다. 우리가 작성한 코드들이 bundle.js에 추가되어 그것을 기준으로 화면들이 나오게 된다. 이때 제일 먼저 시작되는 것이 index.js에 정의되어 있는 코드가 실행된다.

- jsx
  : 자바스크립트를 확장해서 xml 태그를 쓸 수 있게끔 지원해 줌(쉽게 말하자면 자바스크립트 내에서 html을 사용할 수 있게 해준다고 보면됨)
  -> 그것을 위해서 import React를 사용

- ReactDOM
  : index.html에 보면 <div id="root"></div>
  라는 것이 있다.
  또 index.js에 보면 ReactDOM.createRoot(document.getElementById('root'));가 보인다.
  -> 리액트에서(jsx 문법으로) 만들어지는 것들을 렌더링 할 수(넣을 수)있는 ReactDOM의 root로 만들겠다는 것이다.

index.html

```javascript
import App from "./App";

root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

import한 App이 <App /> 태그로 쓰인 것이다.
-> App이란? App.js를 의미

App.js를 열면 return이라는 것 안에 html 태그가 작성되어 있다.(jsx)

<React.StrictMode>를 쓰겠다는 것은 자바스크립트 원격 모듈을 쓰겠다는 의미

- reportWebVitals();
  : WebVital이라는 라이브러리가 있는데, Web 퍼포먼스를 측정하기 위한 자바스크립트 라이브러리이다.

- 정리
  : npm start를 하는 순간 index.js 안에 있는 코드가 실행이 되는데 index.html 안에 있는 div root를 가지고 와 createRoot로 만들어 주고 그 안에 렌더링을 시킬 건데 렌더링 전에 html 코드는 App.js에 정의되어 있다.

#### React 컴포넌트 만드는 방법 (2가지)

- 함수형 컴포넌트

```javascript
import React from "react";

function Home() {
  return <h1>Home 화면 입니다.</h1>;
}

export default Home;
```

- 클래스 컴포넌트

```javascript
import React, { Component } from "react";

class Home extends Component {
  render() {
    return <h1>Home 화면 입니다.</h1>;
  }
}

export default Home;
```

react 안에서 사용하는 html은 html 태그처럼 보이지만 jsx 문법이다. 한마디로 xml이다. 고로 html을 사용하는 것과 100% 동일하지는 않는다.

- npm install react-router-dom@6
  : 메뉴를 클릭하거나 특정 주소를 입력하는 순간 화면이 전환이 되야 한다. 그걸 처리하는 걸 라우팅처리한다고 표현 리액트에서 라우팅 처리를 하기 위해서 가장 많이 쓰는 것중 하나인 react router dom

- <Routes>
  : 브라우저 Path가 바뀔 때 마다 어떤 Component를 맵핑해서 보여줄지를 Route라는 태그 안에 정의를 해야한다.

React(컴포넌트)에서 동적으로 변경되는 값을 "state"라고 부른다.
-> 이 state를 관리하기 위해서 쓸 수 있는 함수가 useState라는 것이 있다.

```javascript
const [num, setNumber] = useState(0);
```

- setNumber
  : num이라는 변수에 대한 setter함수
  -> useState 통해서 state를 관리하겠다고 선언한 변수(num)은 set 함수를 통해서만 상태를 관리할 수 있다.
