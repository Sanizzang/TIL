**참고
https://www.daleseo.com/js-dotenv/
**

### 설치

npm 패키지 매니저를 이용하여 dotenv 라이브러리를 Node.js 프로젝트에 설치

```
$ npm i dotenv
```

### .env 파일 작성

dotenv 라이브러리는 디폴트로 현재 디렉토리에 위치한 .env 파일로 부터 환경 변수를 읽어낸다.
따라서, .env 파일을 생성하고, 그 안에 필요한 환경 변수를 키=값의 포멧으로 나열한다.

```
DB_HOST=localhost
DB_USER=root
DB_PASS=password
```

이렇게 .env 파일에 저장해놓은 환경 변수들을 dotenv 라이브러리를 이용해서 process.env에
설정할 수 있다.

### CommonJS에서 환경 변수 불러오기 (require)

애플리케이션을 구동할 때 제일 먼저 실행되는 자바스크립트 파일(ex. index.js, app.js)의 최상위에 다음과 같이 dotenv 라이브러리를 임포트한 후 config()함수를 호출해주기만 하면 된다.

```javascript
// app.js

require("dotenv").config();

console.log("DB_HOST", process.env.DB_HOST);
```

### ES 모듈에서 환경 변수 불러오기 (import)

```javascript
// env.js

import dotenv from "dotenv";

dotenv.config();
```

```javascript
// index.js

import "./env.js";
import { db_host, db_user, db_pass } from "./db.js";

console.log("DB_HOST:", process.env.DB_HOST);
console.log("DB_USER:", process.env.DB_USER);
console.log("DB_PASS:", process.env.DB_PASS);

console.log({ db_host, db_user, db_pass });
```

### node 커맨드 -r 옵션 사용

본인 프로젝트가 CommonJS 기반인지 ES 모듈 기반인지 신경쓰기 싫다면 node 커맨드를 이용하는
방법도 있다. 애플리케이션을 구동할 때, node 커맨드의 -r 또는 --require 옵션으로
dotenv/config를 넘기는 것이다.

```
$ node -r dotenv/config index.js
```

이 방법을 사용하면 dotenv 라이브러리를 코드에 직접 임포트하지 않아도 .env 파일에 저장된
환경 변수가 process.env에 설정된다.
