### vue/cli?

vue 프로젝트를 굉장히 생성할 수 있게끔 도와주는 도구

프로젝트를 생성을 하게 되면 다양한 설정 파일 폴더, 그리고 프로그램을 구동하기 위한 기본 파일들을 만들어야 되는데 이렇게 일일이 만드는 것도 굉장히 많은 시간을 소비를 해야 한다 그리고 문제는 어떤 개발 리더가 이 프로젝트에 대한 기본 골격을 구성하냐에 따라서 다 달라질 수 있다는 전제가 깔리게 된다.

우리가 매번 동일한 언어로 수많은 프로젝트를 해야되는데 프로젝트를 할 때마다 프로젝트의 폴더 구조나 설정 파일이나 기본 파일들을 매번 새롭게 만들어야 되고 프로젝트 할 때 마다 그게 달라질 수 있다라는 것을 의미한다

이렇게 되면 프로그램을 안정적으로 계속 유지하는게 쉽지 않게된다. 즉 복잡이 높게된다.

이를 위해 vue/cli라는 모듈을 설치하고 vue/cli를 통해서 뷰 프로젝트를 만들면 어떤 프로젝트 팀에서 누가 만들더라도 동일한 구조의 프로젝트 구조가 생성이 되고 그리고 개발자가 직접 코드를 짜서 만들어야 했던 기본 파일들을 vue/cli가 자동으로 생성해준다.

표준화된 뷰 프로젝트 구조를 빠르게 완성할 숭 있다는 장점을 갖게 된다.

또한 프로젝트를 개발을 하고 나중에 배포까지 하는 일련의 과정 또한 유용한 기능을 vue/cli가 제공해 준다.

vue 진영에서는 이렇게 뷰 프로젝트를 만드는 대표적인 지원도구로 vue/cli와 byte라는걸 또 제공한다.

#### Vetur

visual studio vue 개발을 위한 유용한 확장프로그램

```
vue create "프로젝트명"
```

프로젝트 공유 폴더 밑에 뷰를 개발하기 위한 기본 설정 옵션 뿐만 아니고 뷰의 기본 골격 주조에 해당하는 폴더와 수많은 파일들이 자동으로 설치되고 그리고 그걸 구동하기 위한 다양한 모듈까지도 자동으로 한번에 설치가 된다.'

![](https://velog.velcdn.com/images/sanizzang00/post/b84bdd86-b8f0-45c2-875f-a1016f8cd7b5/image.png)

- "private": true
  우리가 만든 프로젝트가 만약에 npm에 등록이 된다면 아무나 접근하지 못하게 private 모드로 사용하겠다. 만약에 false로 한다면 만약 우리가 개발한 프로젝트를 npm 사이트에 등록을 하면 누구나 검색할 수 있게되는 것
  (default 옵션은 true이다)

- "scripts": {}
  다양한 명령어를 스크립트 안에 등록해서 쓸 수 있다. 사용자 입장에서 "npm run serve"라고 하는게 "vue-cli-service serve"라고 하는 것보다 간단하다. (단축키 같은 개념)

- dependencies
  운영 환경으로 배포할 때 가져가야 될 모듈들

- devdependencies
  개발하는 순간에 우리가 사용해야되는 모듈들

우리가 npm run serve라는 명령어를 입력하고 엔터키를 누르는 순간 제일 먼제 실행되는 파일은 main.js 파일이라고 생각하면된다.

vue는 spa 애플리케이션인 결국 index.html 파일 하나만 갖고 뷰 애플리케이션이 구동이 되는 것이다.

vue는 template이라고 보이는 태그안에 html 태그가 작성이 되고 script라는 곳에 우리가 자바스크립트로 코드를 작성하게 되있다.

우리가 npm run serve라는 명령어를 통해서 뷰를 실행시키는 순간에 제일 처음에 실행되는 파일은 main.js이고 main.js에서는 vue를 기반으로해서 createApp을 만들어 낼 건데 제일 처음에 가지고 온 것은 App.vue이고 그리고 그것은 index.html의 id가 App인 곳을 찾아서 거기에다가 mount를 시킨다. 그리고 vue는 Single Page Application이고 index.html 파일 하나만 가지고 구성이된다.

우리가 각종 메뉴를 구성하고 메뉴를 클릭할 때 마다 뭔가 페이지가 전환되는 것이 아니고 이 소스코드(index.html)는 변환되는 게 아무것도 없다. (index.html)<div id="app"></div>에서
실제 바뀌는 컨텐츠에 해당하는 부분만 계속 동적으로 자바스크립트를 이용해서 바뀌게 되는 원리로 동작을 한다.

![](https://velog.velcdn.com/images/sanizzang00/post/c8fa763e-8470-4c84-8286-e31c63d23c3c/image.png)

- Router
  뷰에서 메뉴를 구성을하고 메뉴를 클릭했을 때 화면 이동을 할 수 있게끔 만들어주는 추가적인 모듈

- Vuex
  모든 우리의 뷰 컴포넌트 내에서 공통으로 접근 가능한 어떤 저장소를 만들어서 그 저장소에 데이터를 저장을 하고 상태 관리도 할 수 있게끔 해준다.

- Linter
  자바스크립트 문법을 체크해주고 코딩 컨벤션 또 자바스크립트를 짜는 프로젝트 팀 내에서는 모두가 동일한 코딩 문법(코딩 규칙)을 할 수 있게끔 해주는 모듈

main.js에서 router랑 store를 사용하겠다고 use라는 함수를 이용해 넣어줬다. 그리고 App.vue를 불러와 넣어줬다.

App.vue에는 router-link라는 태그가 있고 Home, About을 클릭할 수 있다. 그때마다
to라고 적혀있는 path로 브라우저 url이 바뀌고 있다. 그런데 이 path에 해당하는 애는

router폴더에 있는 index.js에 똑같은 path가 반드시 존재를 하고 이렇게 메뉴를 클릭할 때마다 to에 적힌 path에 해당하는 것을 찾아서 그때 거기에 연결되어 있는 component란 곳에 연결된 것을 찾아와 화면에 보여주고 있다.

정확히는 App.vue에 있는 <router-view/>라는 부분이 교체가 되는 것이다.

## Vue.js - 라우터 이해하기

![](https://velog.velcdn.com/images/sanizzang00/post/ebe4f736-81c5-408d-bfbe-eb11d538ac90/image.png)

- name
  중복되어선 안됨

- component
  실제 path로 접근을 했을 때 갖고 와서 연결해줄 component. view에는 .vue라는 파일들이 있는데 이런 것들을 다 component라는 용어로 부른다. 이 component는 어떤 component를 여기에 담을 건지에 대한 것이다.

#### 라우터에 compoent를 연결하는 방법

.vue 파일은 js로 컴파일 된다.

chunk-vendors.js에는 우리가 개발한 js 파일 말고 우리가 참조하고 있는 외부 자바스크립트 라이브러리 우리가 설치한 모듈들에 해당하는 스크립트 코드가 chunk-vendors.js에 다 들어가게 된다.

app.js는 router에서 route되어 있는 코드가 자바스크립트 형태로 컴파일이 되서 app.js 파일에 들어가게 된다.

```javascript
component: () =>
  import(
    /* webpackChunkName: "about", webpackPrefetch:true */ "../views/AboutView.vue"
  );
```

prefetch로 등록된 javascript는 당장은 쓸건 아닌데 바로 미래에 사용될 가능성이 높은 리소스들을 이와 같은 형태로 prefetch라고 해서 등록을 해주면 브라우저 캐시에 등록을 하게 된다.

1.

```javascript
import HomeView from "../views/HomeView.vue";

const routes = [
  {
    path: "/",
    name: "home",
    component: HomeView,
  },
];
```

app.js로 무조건 vue로 개발된 화면을 받아오는 파일

2.

```javascript
const routes = [
  {
    path: "/about",
    name: "about",
    // route level code-splitting
    // this generates a separate chunk (about.[hash].js) for this route
    // which is lazy-loaded when the route is visited.
    component: () =>
      import(/* webpackChunkName: "about" */ "../views/AboutView.vue"),
  },
];
```

사이즈가 작아서 메뉴를 클릭한 순간에 서버로 받아도 전혀 문제가 안되는 애들 혹은 사용자가 접속할 빈도가 낮은 것

3.

const routes = [
{
path: '/about',
name: 'about',
// route level code-splitting
// this generates a separate chunk (about.[hash].js) for this route
// which is lazy-loaded when the route is visited.
component: () => import(/_ webpackChunkName: "about", webpackPrefetch:true _/ '../views/AboutView.vue')
}
]

그 페이지에 대한 사이즈가 굉장히 커서 서버로부터 받아오면 느려질 가능성이 높은 것 또한 화면이 열리자 마자 사용자가 바로 들어갈 확률이 굉장히 높은 것

총 3가지 방법이 존재

## 데이터 바인딩

vue는 양방향 데이터 바인딩이 가능!
