**이 글은 https://inpa.tistory.com/entry/NODE-%F0%9F%93%9A-%EC%B9%B4%EC%B9%B4%EC%98%A4-%EB%A1%9C%EA%B7%B8%EC%9D%B8-Passport-%EA%B5%AC%ED%98%84 블로그를 참고하였습니다.**

## 카카오 로그인 OAuth 신청

카카오 로그인을 위한 카카오 개발자 계정과 로그인 애플리케이션 등록
https://developers.kakao

#### 1. 로그인 후 애플리케이션 추가

자신이 진행하는 포트폴리오 프로젝트 적기
<img src="https://velog.velcdn.com/images/sanizzang00/post/6dc4d775-94a0-4703-a570-b779a87bcf6d/image.png"  height="500"/>

#### 2. 도메인 등록

콜백 받을 도메인을 등록해야 카카오 서버로부터 카카오 계정 정보를 넘겨받을 수 있음.
(도메인은 여러개 적을 수 있다.)
![](https://velog.velcdn.com/images/sanizzang00/post/fa779172-4965-4a3c-8c62-50e65eac07c6/image.png)

#### 3. 카카오 로그인 활성화 후 Redirect URI 적기

카카오 서버에서 어느 주소로 REST API GET 여청을 보낼지 적는 것이다.
![](https://velog.velcdn.com/images/sanizzang00/post/cba51a97-b602-46e6-8558-95c03abbcea9/image.png)

#### 4. 카카오 정보 동의 항목 설정

넘겨 받을 카카오 정보를 어느걸 받을지 설정 한다.
이 항목을 설정하면 사용자는 로그인 할 때 동의서 처리를 한다.
![](https://velog.velcdn.com/images/sanizzang00/post/489d7ac0-ab56-4818-a6dd-0c4623462cb4/image.png)

#### 5. 자신의 프로젝트에 키 등록

REST API키를 환경변수로 설정해서 등록. (kakaoStrategy에서 사용될 예정)

## 카카오 로그인 라우터 전략 구현

카카오 로그인은 로그인 인증과정을 카카오에 맡긴다.
사용자는 번거롭게 새로운 사이트에 회원가입할 필요가 없어 좋고, 서비스 제공자는 로그인 과정을 검증된 SNS에 안심하고 맡길 수 있어 편하고 좋다.
SNS 로그인의 특징은 회원가입 절차가 따로 없다는 것이다.
카카오를 예시로 들경우, 로그인만 하면, 따로 회원가입 절차 없이 사이트에 로그인 할 수 있다.

### passport-kakao 설치

카카오 로그인을 위해선 passport-kakao 설치가 필요하다.

```
npm install passport passport-kakao
```

### 카카오 로그인 인증 전략 처리과정

1. 라우터를 통해 /kakao 요청이 서버로 온다.
2. passport.authenticate('kakao')를 통해서 카카오 로그인 페이지로 이동한다.
3. 카카오 로그인을 하면, 카카오 로그인 developer에서 설정한 redirect url 경로에 따라 식별값을 전달한다.

```
Redirect URL: 해당 클라이언트를 식별하고 식별값(Access token)을 전달할 통로
```

4. /kakao/callback 라우터로 요청이 오게 된다.
5. passport.authenticate('kakao)에서 kakaoStrategy로 인증 전략 시행.

```javascript
{
	clientId: process.env.KAKAO_ID, // 카카오 로그인에서 발급받은 REST API 키
    callbackURL: '/auth/kakao/callback', // 카카오 로그인 Redirect URI 경로
}
/*
clientID에 카카오 앱 아이디 추가
callbackURL: 카카오 로그인 후 카카오가 결과를 전송해줄 URL
accessToken, refreshToken: 로그인 성공 후 카카오가 보내준 토큰
profile: 카카오가 보내준 유저 정보. profile의 정보를 바탕으로 회원가입
*/
async (accessToken, refreshToken, profile, done) => {
```

6. 카카오전략에서 DB에서 가입이력 조사
7. 가입이력 있으면 바로 done()을 보내고, 없다면 바로 회원가입 시키고 성공 done()을 보냄
8. 클라이언트에 세션 쿠키를 보냄으로서 로그인 인증 완료

### 카카오 로그인 인증 전략 코드

브라우저 태그에 요청하기 위한 href=/auth/kakao 링크 태그를 설정한다.
이 카카오톡 로그인 버튼을 누르게 되면, 라우터 GET /auth/kakao로 접근해 카카오 로그인 과정이 시작되게 구성할 것이다.

```HTML
<a id="kakao" href="/auth/kakao" class="btn">카카오톡 로그인</a>
```

그래서 다음과 같이 구성하면, /auth 쪽으로 서버가 요청오면 authRouter(./routes/auth.js) 파일로 가게 될 것이다.

```javascript
const pageRouter = require("./routers/page");
const authRouter = require("./routes/auth");

app.use("/", pageRouter);
app.use("/auth", authRouter);
```

#### routes/auth.js

```javascript
//* 카카오로 로그인하기 라우터 ***********************
//? /kakao로 요청오면, 카카오 로그인 페이지로 가게 되고, 카카오 서버를 통해 카카오 로그인을 하게 되면, 다음 라우터로 요청한다.
router.get("/kakao", passport.authenticate("kakao"));
//? 위에서 카카오 서버 로그인이 되면, 카카오 redirect url 설정에 따라 이쪽 라우터로 오게 된다.
router.get(
  "/kakao/callback",
  //? 그리고 passport 로그인 전략에 의해 kakaoStrategy로 가서 카카오계정 정보와 DB를 비교해서 회원가입시키거나 로그인 처리하게 한다.
  passport.authenticate("kakao", {
    failureRedirect: "/", // kakaoStrategy에서 실패한다면 실행
  }),
  // kakaoStrategy에서 성공한다면 콜백 실행
  (req, res) => {
    res.redirect("/");
  }
);
```

GET /auth/kakao에서 로그인 전략을 수행하는데, 이 때 카카오 로그인창으로 리다이렉트한다.
그리고 로그인 창에서 사용자가 ID/PW를 쳐서 로그인하면, 성공 여부 결과를 GET /auth/kakao/callback으로 받게된다.
그리고 kakao/callback 라우터에서는 카카오 로그인 전략을 다시 수행한다.

로컬 로그인과 다른 점은 passport.authenticate 메서드에 콜백 함수를 제공하지 않는다는 점이다.
카카오 로그인은 로그인 성공 시 내부적으로 req.login을 호출하므로, 우리가 직접 호출할 필요가 없다.
콜백 함수 대신 로그인에 실패했을 때 어디로 이동할지를 failureRedirect속성에 적어준다.
성공했을 때에도 어디로 이동할지를 미들웨어에 적어준다.

#### passport/kakaoStrategy.js

```javascript
const passport = require("passport");
const KakaoStrategy = require("passport-kakao").Strategy;
const User = require("../models/user");
module.exports = () => {
  passport.use(
    new KakaoStrategy(
      {
        clientID: process.env.KAKAO_ID, // 카카오 로그인에서 발급받은 REST API 키
        callbackURL: "/auth/kakao/callback", // 카카오 로그인 Redirect URI 경로
      },
      /*
       * clientID에 카카오 앱 아이디 추가
       * callbackURL: 카카오 로그인 후 카카오가 결과를 전송해줄 URL
       * accessToken, refreshToken: 로그인 성공 후 카카오가 보내준 토큰
       * profile: 카카오가 보내준 유저 정보. profile의 정보를 바탕으로 회원가입
       */
      async (accessToken, refreshToken, profile, done) => {
        console.log("kakao profile", profile);
        try {
          const exUser = await User.findOne({
            // 카카오 플랫폼에서 로그인 했고 & snsId필드에 카카오 아이디가 일치할경우
            where: { snsId: profile.id, provider: "kakao" },
          });
          // 이미 가입된 카카오 프로필이면 성공
          if (exUser) {
            done(null, exUser); // 로그인 인증 완료
          } else {
            // 가입되지 않는 유저면 회원가입 시키고 로그인을 시킨다
            const newUser = await User.create({
              email: profile._json && profile._json.kakao_account_email,
              nick: profile.displayName,
              snsId: profile.id,
              provider: "kakao",
            });
            done(null, newUser); // 회원가입하고 로그인 인증 완료
          }
        } catch (error) {
          console.error(error);
          done(error);
        }
      }
    )
  );
};
```

로컬 로그인과 마찬가지로 첫 번째 인수로 카카오 로그인에 대한 설정을 한다.

1. clientID는 카카오에서 발급해주는 아이디이다. 노출되지 않아야하므로 .env 파일에 넣어줄 것이다.
2. callbackURL은 카카오로부터 인증 결과를 받을 라우터 주소이다.

그 후로 들어오는 두 번째 인수 콜백이 실행된다.
먼저, 기존에 카카오를 통해 회원가입한 사용자가 있는지를 profile.id로 조회한다.
있다면 이미 회원가입이 되어 있는 경우이므로 사용자 정보와 함께 done 함수를 호출하고 전략을 종료한다.

만일 카카오를 통해 회원가입한 사용자가 없다면 해당 사용자의 회원가입을 자동으로 진행한다.
카카오에서는 인증 후 callbackURL에 적힌 주소로 accessToken, refreshToken과 profile을 보내는데, profile에 사용자 정보들이 들어있다.

이후 원하는 정보들을 profile 객체에서 꺼내와 회원가입을 진행하면 된다.
사용자를 생성한 뒤 done 함수를 호출하게 된다.

#### passport/index.js

> kakaoStrategy.js를 passport에 등록한다.

```javascript
const passport = require("passport");
const local = require("./localStrategy"); // 로컬서버로 로그인할때
const kakao = require("./kakaoStrategy"); // 카카오서버로 로그인할때
const User = require("../models/user");
module.exports = () => {
  passport.serializeUser((user, done) => {
    done(null, user.id);
  });
  passport.deserializeUser((id, done) => {
    //? 두번 inner 조인해서 나를 팔로우하는 followerid와 내가 팔로우 하는 followingid를 가져와 테이블을 붙인다
    User.findOne({ where: { id } })
      .then((user) => done(null, user))
      .catch((err) => done(err));
  });
  local();
  kakao(); // 구글 전략 등록
};
```
