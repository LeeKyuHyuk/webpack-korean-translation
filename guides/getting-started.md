# Getting Started

webpack은 자바스크립트 모듈을 컴파일하는 데 사용됩니다. 일단 [설치](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/guides/installation.md)되면 [CLI](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/api/cli.md) 또는 [API](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/api/node.md)에서 webpack과 상호 작용할 수 있습니다.

> webpack 5를 실행하기 위해서는 Node.js의 버전은 최소 10.13.0 (LTS) 이상이어야 합니다.

# Basic Setup

먼저 디렉토리를 만들고, `npm`을 초기화하고, [webpack을 로컬에 설치](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/guides/installation.md#local-installation)한 뒤, [`webpack-cli`](https://github.com/webpack/webpack-cli) (명령 줄에서 webpack을 실행하는 데 사용되는 도구)를 설치해 보겠습니다.

```sh
mkdir webpack-demo
cd webpack-demo
npm init -y
npm install webpack webpack-cli --save-dev
```

위의 과정을 마치고, `index.html`과 `src/index.js`를 생성합니다.

아래는 `src/index.js`의 내용입니다.

```javascript
function component() {
  const element = document.createElement('div');

  // 이 줄이 작동하려면 Lodash가 필요합니다.
  element.innerHTML = _.join(['Hello', 'webpack'], ' ');

  return element;
}

document.body.appendChild(component());
```

아래는 `index.html`의 내용입니다.

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Getting Started</title>
    <script src="https://unpkg.com/lodash@4.17.20"></script>
  </head>
  <body>
    <script src="./src/index.js"></script>
  </body>
</html>
```

그리고 패키지를 `private`로 표시하고 `main` 항목을 제거하기 위해 `package.json` 파일을 설정해야 합니다. 이는 실수로 코드가 게시되는 것을 방지하기 위한 것입니다.

> `package.json`의 내부 작동에 대해 자세히 알아보려면 [npm 문서](https://docs.npmjs.com/files/package.json)를 읽는 것이 좋습니다.

아래와 같이 `package.json`의 `"main": "index.js",`를 지우고, `"private": true,`를 추가합니다.

```json
{
  "name": "webpack-demo",
  "version": "1.0.0",
  "description": "",
  "private": true,
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "webpack": "^5.11.1",
    "webpack-cli": "^4.3.1"
  }
}
```

이 예제에서는 `<script>` 태그 사이에 `index.js` 파일 안에 `lodash`가 먼저 로드된 후에 실행되어야 한다는 암묵적 종속성이 내재되어 있습니다. 암묵적 종속성을 갖는 이유는 `index.js` 파일 안에 `lodash`를 명시적으로 선언하지 않고, 글로벌 변수 어딘가에 `_`가 존재한다는 가정만을 하기 때문입니다.

webpack을 사용하지 않고 JavaScript 프로젝트를 관리할 때는 아래와 같은 문제가 있습니다:
- 외부 라이브러리에 의존적인 스크립트는 즉시 반영이 되지 않습니다.
- 의존 모듈이 없거나, 또는 잘못된 순서로 포함(include) 되었을 때 애플리케이션은 원하는 대로 작동하지 못합니다.
- 의존 모듈이 포함(include) 되었지만 사용되지 않을 때도 브라우저는 불필요한 코드까지 다운로드합니다.

# Creating a Bundle

먼저 디렉토리 구조를 약간 변경하여 소스코드 (`./src`)와 배포코드 (`./dist`)를 분리합니다. 소스코드는 우리가 작성하고 편집 할 코드입니다. 배포코드는 최종적으로 브라우저에로드 될 빌드 프로세스의 최소화(Minimized)되고 최적화(Optimized) 된 출력입니다. 다음과 같이 디렉토리 구조를 변경하세요. (`dist` 디렉토리를 추가하고, `dist/index.html` 파일을 생성합니다.)

```
webpack-demo
├── dist
│   └── index.html
├── package.json
└── src
    └── index.js
```

`lodash` 종속성을 `index.js`와 번들로 묶으려면 라이브러리(`lodash`)를 로컬에 설치해야합니다.

```sh
npm install --save lodash
```

> 프로젝트 번들에 번들로 제공되는 패키지를 설치할 때는 `npm install --save`를 사용해야 합니다. 만약 패키지를 개발 목적(linter, testing, etc)으로 사용할 때는 `npm install --save-dev` 명령을 사용하여 설치하세요. 더 자세한 정보는 [npm 문서](https://docs.npmjs.com/cli/install)에서 확인하세요.

자, 이제 스크립트에서 `lodash`를 가져오겠습니다.

아래는 `src/index.js`의 내용입니다.

```javascript
import _ from 'lodash';

function component() {
  const element = document.createElement('div');

  // 이제 이 스크립트로 Lodash를 불러옵니다
  element.innerHTML = _.join(['Hello', 'webpack'], ' ');

  return element;
}

document.body.appendChild(component());
```

이제 스크립트를 번들링 할 것이므로 `index.html` 파일을 업데이트해야 합니다. `lodash`를 가져오는 `<script>`를 제거하고 원시 `./src` 파일 대신 번들을 로드 하도록 `<script>` 태그를 수정하겠습니다.

아래는 `dist/index.html`의 내용입니다.

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Getting Started</title>
  </head>
  <body>
    <script src="main.js"></script>
  </body>
</html>
```

위 설정에서 `index.js`는 명시적으로 `lodash`를 포함하도록 요구하며 `_`로 바인딩하고 있습니다. 이를 `main.js`로 변경함으로써 모듈에 필요한 종속성을 webpack에게 알려주고, webpack은 이 정보를 토대로 Dependency Graph를 구축합니다. 그리고 이 그래프를 이용하여 스크립트가 올바른 순서로 실행되게 최적화된 번들을 생성합니다.

즉, `src/index.js`에 있는 스크립트를 [진입점](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/concepts/entry-points.md)으로 사용하고 `dist/main.js`를 [출력](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/concepts/output.md)으로 생성하는 `npx webpack`을 실행해 보겠습니다. Node 8.2 / npm 5.2.0 이상과 함께 제공되는 `npx` 명령은 처음에 설치 한 webpack 패키지의 webpack 바이너리 (`./node_modules/.bin/webpack`)를 실행합니다.

```
$ npx webpack
asset main.js 69.3 KiB [emitted] [minimized] (name: main) 1 related asset
runtime modules 1000 bytes 5 modules
cacheable modules 530 KiB
  ./src/index.js 275 bytes [built] [code generated]
  ./node_modules/lodash/lodash.js 530 KiB [built] [code generated]

WARNING in configuration
The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/configuration/mode/

webpack 5.11.1 compiled with 1 warning in 1651 ms
```

> 출력은 약간 다를 수 있지만, 성공적으로 빌드가 되었다면 상관없습니다.

아무런 이상이 없다면 `dist/index.html` 파일을 브라우저에서 열었을 때 'Hello webpack'라는 메시지를 보게 될 것입니다.
