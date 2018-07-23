---
layout: post
title: 리액트 헬로월드의 분해
date: 2018-07-23 09:58 +0800
---

이 글에서는 리액트로 작성된 간단한 코드를 분해하여
리액트가 어떻게 동작하는지를 이해하고자 한다.

## 개발 환경의 준비

이 글은 리눅스 환경을 기준으로 작성되었으나,
운영체제에 의존적인 내용은 없다. 그러나 명령어 정도의 차이는 있을 수 있다.

* Node 8.x LTS 버전을 사용하였다.
* 패키지 관리는 Yarn 을 사용하였다.
* `create-react-app` 을 이용하여 리액트 기반 앱의 뼈대를 생성하였다.
* UI 요소들을 사용하기 위해, `antd` 라이브러리를 사용하였다.

상기의 내용을 자신의 환경에 맞게 준비하여야 한다.

리눅스를 기준으로 설명하면 다음과 같다.

#### 1. nodejs 설치와 yarn 설치

##### nodejs 가 왜 필요한가?

리액트 개발환경을 위해서 백엔드 javascript 런타임 환경인 nodejs 가 설치되어야 한다.
nodejs 는 서버에서 동작하는 Javascript 런타임 환경으로서, 프론트엔드 라이브러리인 리액트와는 상관이 없어 보일 수 있다.

nodejs 는 NPM(Node Package Manager) 라는 훌륭한 자바스크립트 패키지 관리자를 제공하고 있으며,
이를 이용하여 사용자가 개발에 필요한 Javascript 기반의 라이브러리를 손쉽게 설치하고 사용할 수 있다.
리액트도 Javascript 라이브러리이며, NPM 을 이용하여 쉽게 설치할 수 있다.

1. nodejs 는 패키지 관리자인 npm 을 제공하며,
2. 리액트 개발과정에 필요한 각종 라이브러리들을 npm 패키지 관리자를 이용해 설치하고 업데이트 할 수 있다.
3. 따라서 `npm` 을 이용하여 리액트와 그에 의존적인 라이브러리를 설치할 것이다.

개발 과정에만 사용하고, 작성한 리액트 앱이 배포용으로 빌드된 이후에는 nodejs 가 사용되지 않는다.
(리액트 서버측 렌더링을 할것이 아니라면...)

원한다면 리액트 개발환경 구성에 NPM 을 이용하지 않을 수도 있겠으나
react 기동에 필요한 수많은 라이브러리를 직접 내려받아 일일이 설치하여야 할 것이다.

이후 라이브러리의 업데이트라도 발생하면, 영향받는 모든 스크립트들을 일일이 업데이트 하여야 하니,
아주 간단한 구조가 아니라면 사실상 관리가 불가능할 정도로 어려워 진다.

이미 NPM 은 javascript 개발 환경에서 의존성 관리와 배포를 담당하는 중추적인 역할을 담당하고 있으므로
익혀두면 유익하다.

Yarn 은 NPM 과 동일하게 패키지 관리자의 역할을 하나, npm 이 가지고 있는 단점을 극복하기 위해 개발되었다.
사실 `yarn` 명령을 호출하면 `npm` 명령이 wrapping 되어 호출되며, 설정도 npm 의 설정을 우선 따른다.

눈에 보이는 특징으로는 Cache 를 적극적으로 활용하여 NPM 을 이용해 패키지를 설치할때보다 비약적으로 속도가 빨라졌다.
Yarn 도 자바스크립트로 작성된 스크립트 이므로, NPM 을 이용해 설치할 수 있다.

일단 한번 Yarn 을 설치했다면, `npm` 을 사용할 자리에 `yarn` 을 사용하여 대체할 수 있다.

[nodejs.org](https://nodejs.org/ko/download/) 에서 nodejs 바이너리를 내려받을 수 있다.

홈페이지에서 제공하는 문서를 참고하여 설치한다.

#### 2. yarn global add 명령으로 `create-react-app` 을 설치

* yarn 을 이용할 경우: `yarn global add create-react-app`
* npm 을 이용할 경우: `npm install -g create-react-app`

상기의 명령으로, `create-react-app` 패키지를 설치한다.

이 스크립트는 리액트의 실행 환경을 자동으로 설치해준다.

#### 3. `create-react-app` 명령을 이용하여 리액트 앱의 뼈대를 생성.

예제를 위해 `myapp` 이라는 리액트 앱을 생성하였다.

`create-react-app myapp` 

이때, 네트워크 상황에 따라 차이는 있겠으나, 1~2분 정도가 소요된다.

`myapp` 이라는 디렉토리가 생성되며, 내용은 다음과 같다.

```
jehos@E5570:/tmp/myapp$ tree -L 1
myapp/
├── node_modules  # 다양한 Javascript 라이브러리가 NPM 패키지로 배포됨
├── package.json  # `myapp` 이 사용하는 라이브러리의 의존 관계를 기술
├── public        # HTML 파일과 외부 asset 이 포함됨
├── README.md     # 초기값은 공식 React 문서이다.
├── src           # Javascript, import 가능한 CSS 나 Image 같은 Asset
└── yarn.lock     # 현재 설치된 라이브러리의 버전을 고정
```

* 각 디렉토리의 의미는 [공식 문서](https://github.com/facebook/create-react-app/blob/master/packages/react-scripts/template/README.md)를 참고

기존에는 리액트 실행환경을 구성하기 위해 필요한 모든 패키지들을 `npm` 으로 내려받고,
동작에 필수적인 환경설정을 직접 작성했어야 했다.
사용자가 이 작업을 직접 하는것은 고된 일이므로 이것만 해주는 틀(boilerplate)이나 스크립트가 존재하기도 했다.

처음 시작하는 사람이 실행환경 구축하기 위해 다양한 스크립트를 알아야 하는것은 괴로운 일이다.
초기 진입장벽을 낮추기 위해, `create-react-app` 라는 개발환경 구축 스크립트를 공식적으로 제공하게 되었고,
이 도구를 이용하여 자동으로 리액트가 동작하기 위해 필요한 환경을 자동으로 구성할 수 있다.

즉, 이 스크립트로 간단하게 리액트 기반 앱의 뼈대를 만들 수 있으며, 리액트가 뜨는 모습을 볼 수 있다.
소스를 약간 수정하여, 간단한 리액트 앱을 만들거나, 리액트의 동작 원리를 이해할 수도 있다.
초기 진입 장벽 때문에 리액트를 맛보지 못하는 불상사가 많이 사라졌다.
어느정도 개발이 이루어지고 난 이후에는 자동으로 구성된 개발환경을 버리고([eject](https://velopert.com/2037) 참고),
고급 설정을 할 수 있다.

만약, `create-react-app` 을 사용하지 않고 직접 손으로 리액트의 개발환경을 구성해 본다면
동작에 대한 이해가 깊어질 것이다. ( 참조: [[React.JS] 강좌 2편 작업환경 설정하기](https://velopert.com/814) )

### package.json

`create-react-app` 으로 새로운 리액트 프로젝트를 생성하면, 내부적으로 `yarn` 이나 `npm` 을 이용하여 필요한 라이브러리를 설치한다.
그 결과로, `node_modules` 에 필요한 라이브러리가 저장되며, 자동으로 `packages.json` 파일에 의존성이 기록된다.

이 파일은  `yarn` 이나 `npm` 에 의해 의존성을 해소(패키지 설치) 하거나, 스크립트 실행시 참조된다.

`package.json` 을 열어보면 다음과 같은 내용이 있다.

```
{
  "name": "myapp",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "react": "^16.4.1",
    "react-dom": "^16.4.1",
    "react-scripts": "1.1.4"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
```

**dependencies** 는 이 앱이 동작하기 위해 필요한 의존성을 기술한 것이고,
**scripts** 는 `yarn` 이나 `npm` 의 `run` 명령으로 수행할 스크립트이다.

개발서버의 시작이나 빌드, 테스트, 혹은 자동으로 처리하기 원하는 스크립트를 여기에서 선언할 수 있다.
`Makefiles` 의 간소화한 형태라고 이해할 수 있다.

기동 스크립트는 `react-scripts` 로 되어있는데, 이 실행파일은 `node_modules` 에 저장되어 있다.
이것을 직접 지정하는것이 아니라, `yarn run start` 와 같이 실행해주면, 자동으로 `node_modules` 의 실행 바이너리를 탐색하게 된다. 

* [yarn run](https://yarnpkg.com/lang/en/docs/cli/run/)
```
[script] can also be any locally installed executable that is inside node_modules/.bin/.
```

## 개발 서버의 구동

`create-react-app` 명령으로 막 새로운 리액트 프로젝트를 생성했다면, `yarn run start` 명령으로 개발 서버를 구동할 수 있다.
localhost 에서 동작하는 작은 웹서버가 기동되고, 소스코드의 변경을 자동으로 인식하여 제공한다.

만약 소스코드에 정적으로 분석 가능한 오류가 있다면 개발서버를 통해 어느 부분에서 에러가 발생했는지 안내한다.

### Babel 과 Webpack

`react-scripts` 는 webpack 을 호출하여 파일들의 의존성을 해소시키고,
그 중간 처리 과정으로서 babel 을 이용하여 소스 코드를 브라우저에서 실행할 수 있는 형태로 변환한다.

Babel 과 Webpack 은 리액트를 웹 브라우저에서 동작시키기 위한 편의를 제공한다.
즉, 반드시 필요하지는 않으나, 없으면 원활한 개발이 어렵다.

#### Babel

Babel 은 transpiler 라고 불리우며, Compiler 와는 달리 동일한 계층에서의 번역을 담당한다.
현대의 브라우저는 다수 파편화가 되어있지만, W3C 에서 제정한 웹의 표준들을 꾸준히 구현시켜 왔다.
그러나 제정된 표준과, 브라우저 상에서 구현된 현황에서는 다소 차이가 있는것이 사실이다.

브라우저마다 구현의 방식이나 정도가 달라서, 작성된 Javascript 가 정상 동작하는지 테스트 하기 위해서는
브라우저의 호환표를 참조하며 개별 브라우저에 하나씩 켜놓고 테스트 해야 했다.

수년 전까지만 해도 ES5 라고 부르는 Javascript 스펙이 브라우저에 구현되어 있었고,
더 풍부한 문법이 제공되는 ES6(ECMA Script 2015) 스펙을 사용하기에는 어려웠다.
(2018년 현재에는 크롬, 파이어폭스, 사파리, 엣지 와 같은 브라우저 대부분에서 ES6 를 지원한다.
[호환표](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/import#AutoCompatibilityTable) 참조)

이러한 문제를 해결하기 위해, 사용자가 최신의 스펙으로 Javascript 를 개발하면
Babel 이 이를 호환 가능한 스크립트로 번역하여 해결하였다.

또한, JSX(Javascript Extension) 와 같은 표기법을 통해, 사용자에게 익숙한 HTML 태그 형태로 작성하면,
이를 표준 Javascript 로 변환시킬 수도 있다.

한 꼭지에서 자세히 설명하기에는 한계가 있어, 정리하자면 다음과 같다.

즉, 최신 스펙에서 제공하는 문법으로 개발의 편의를 이용하면서,
동시에 다른 브라우저에서도 동작할 수 있는 호환을 보장할 수 있었다.

* [Babel로 ECMAScript 2015 사용하기](https://blog.outsider.ne.kr/1176)

#### Webpack

Webpack 의 역할은 크게 세개로 볼 수 있다.

1. 의존성 해결
2. Webpack 이 이해할 수 있는 형태로 변경해주는 전처리 기능, "로더"
3. 배포에 적합한 형태의 최적화와, Asset 관리

본인이 개발한 웹앱에서 참조하는 Javascript 라이브러리가 다수라면, 이것들을 제공하기 위해
모든 의존성을 파악하여 웹서버에 올려야 할 것이다. 또한, 다수의 파일로 분할되어 있기 때문에
브라우저는 재귀적으로 수백개의 자바스크립트를 연이어서 호출해야 할 것이다.

작은 파일이라도, 잦은 횟수로 호출하게 되면 그 오버헤드가 상당하다.
필요한 내용들을 하나로 수집하여, 호출 횟수를 줄인다면 성능을 증가시킬 수 있다.
필요한 모든 내용을 하나로 끌어 모으기 때문에 용량은 다소 커지지만,
캐시나 CDN, 실시간 압축 과 같은 웹을 지탱하는 인프라의 혜택을 누릴 수 있으므로 큰 문제는 아니다.

말하자면 Webpack 은 C/C++ 에서 말하는 정적 컴파일의 기능을 수행한다고 볼 수 있다.

`react-script` 에 의해 Webpack 이 먼저 호출되고, 각 파일들의 연결을 추적하며 의존성을 해결하는 과정에서
Webpack 의 `loader` 로 등록된 babel 이 웹 브라우저에서 해석할 수 있는 버전의 Javascript 로 변경하게 된다. 

* [Webpack 소개](https://medium.com/@OutOfBedlam/webpack-%EC%86%8C%EA%B0%9C-d595f93d5c28)
* [webpack2 입문 가이드](https://hyunseob.github.io/2017/03/21/webpack2-beginners-guide/)
* [JavaScript 모듈화 도구, webpack](https://d2.naver.com/helloworld/0239818)

## 진입점 (entry)

`yarn run start` 스크립트를 실행하면, `http://localhost:3000` 으로 접근할 수 있는 개발 서버가 기동된다.
상기에서 서술한 것과 같이, `react-scripts` 가 기동되며 소스코드는 babel 과 webpack 을 거쳐 사용자의 브라우저에서 접근할 수 있다.
따라서, 웹 브라우저에서 확인하는 결과물은 babel 에 의해 번역(transpile) 되고, webpack 에 의해 통합된 형태의 결과물이다.

다만, 운영환경에 배포되는 build 의 산출물과, 개발서버에서 제공되는 산출물에는 차이가 있는데,
운영환경에 배포하기 위한 build 의 산출물은 용량을 최소화 하기 위해 변수의 이름을 짧게 바꾸거나,
주석과 공백을 제거, 중복된 코드 제거(Dedup) 하는 Minify 가 실행되고,
코드를 분석하기 힘들게 하기 위한 난독화(Uglify) 과정을 거친다. 
따라서, 개발 환경에서 제공될 때는 이러한 조치가 이루어지지 않아, 사용자가 디버깅 하기가 용이하다.

그렇다면 어디가 시작점일까? 그 시작점을 확인하기 위해서는 인덱스 페이지를 찾았다.
개발 서버가 `http://localhost:3000` 이라고 했을때, 가장 처음에 보이는 화면이 인덱스 페이지 이다.
각 브라우저마다 차이가 있을 수 있으나, 대개 마우스 오른쪽 버튼을 누르면 `소스보기` 메뉴가 있는것을 확인할 수 있다.

`create-react-app` 으로 갓 생성한 프로젝트의 인덱스 페이지의 소스코드는 다음과 같다.

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <meta name="theme-color" content="#000000">
    <!--
      manifest.json provides metadata used when your web app is added to the
      homescreen on Android. See https://developers.google.com/web/fundamentals/engage-and-retain/web-app-manifest/
    -->
    <link rel="manifest" href="/manifest.json">
    <link rel="shortcut icon" href="/favicon.ico">
    <!--
      Notice the use of  in the tags above.
      It will be replaced with the URL of the `public` folder during the build.
      Only files inside the `public` folder can be referenced from the HTML.

      Unlike "/favicon.ico" or "favicon.ico", "/favicon.ico" will
      work correctly both with client-side routing and a non-root public URL.
      Learn how to configure a non-root public URL by running `npm run build`.
    -->
    <title>React App</title>
  </head>
  <body>
    <noscript>
      You need to enable JavaScript to run this app.
    </noscript>
    <div id="root"></div>
    <!--
      This HTML file is a template.
      If you open it directly in the browser, you will see an empty page.

      You can add webfonts, meta tags, or analytics to this file.
      The build step will place the bundled scripts into the <body> tag.

      To begin the development, run `npm start` or `yarn start`.
      To create a production bundle, use `npm run build` or `yarn build`.
    -->
  <script type="text/javascript" src="/static/js/bundle.js"></script></body>
</html>
```

사실 이 내용은, `public/index.html` 과 거의 동일하다.
"거의" 라고 표현한 까닭은, 마지막의 `<script type="text/javascript" src="/static/js/bundle.js"></script>` 부분 때문이다.
이 부분으로 하여금, 개발 웹 서버는 기본 규칙에 의해 index.html 를 브라우저에 제공하고,
index.html 의 가장 마지막 `<script>` 구문에 의해 webpack 에 의해 하나로 "번들"(뭉쳐진) 스크립트 덩어리를 브라우저에서 읽을 수 있도록 한다.

이 부분은 `public/index.html` 에는 없는 내용이며, webpack 에 의해 추가된 내용이다.

* webpack.config.js

```
var webpack = require('webpack');

module.exports = {
    entry: {
        entry: __dirname + '/entry.js'
    },
    output: {
        filename: '[name].bundle.js'
    }
}
```

이때 명시된 `entry.js` 가 webpack 이 접근을 시작하는 진입점이고,
이 파일을 기점으로 의존성 해결이 수행되어 `bundle.js` 가 산출된다.

그렇다면, 이렇게 불러오는 `bundle.js` 에는 어떤 내용이 포함되는가.

## Javascript 와 브라우저

다른 글에서 표현했듯이, 나에게 있어서 웹어플리케이션 또한 프로그램이란 인식을 갖기는 쉽지 않았다.

"프로그램" 의 꼴이라면 무릇 프로세스의 형태를 띄고 시작과 끝이 있는 그 무엇이라는 형태로 이해하고 있었기 때문에,
웹 브라우저 상에서 실행되는 Javascript 는 브라우저를 제어할 수 있는 간단한 도구 정도로만 이해했기 때문이었다.
그러나 `bundle.js` 를 열어보니, Javascript 또한 규모가 거대하고 짜임새 있이 동작하는 하나의 프로그램임을 인식했다.

`bundle.js` 는 앞서 서술한 것처럼, webpack 을 통해 의존성이 연결된 Javascript 라이브러리 들을 하나로 뭉쳐 "번들링"(bundling) 해놓은 결과물이다.
따라서, 리액트 의 실행 코드 또한 이 파일에 함께 포함되어 있다.

리액트는 Javascript 의 라이브러리 라고 표현되어 있지만, 수동적으로 호출당하는 수준의 라이브러리가 아닌,
자신의 상태 정보를 지속적으로 감시하며, 사용자 인터렉션에 의해 변경이 발생하면 화면에 이를 반영한다.

Javascript 로 작성된 React 와 기타 라이브러리들은 webpack 에 의해 번들링되어
브라우저에 의해 실행되고, 
