# Asset Management

처음부터 가이드를 따랐다면 이제 "Hello webpack"을 보여주는 작은 프로젝트를 갖게 될 것입니다. 이제 이미지와 같은 다른 리소스를 통합하여 처리하는 방법을 살펴 보겠습니다.

webpack 이전에 프론트엔드 개발자는 [grunt](https://gruntjs.com/) 및 [gulp](https://gulpjs.com/)와 같은 도구를 사용하여 이러한 리소스를 처리하고 `/src` 폴더에서 `/dist` 또는 `/build` 디렉토리로 복사했습니다. JavaScript 모듈에도 동일한 개념이 사용되었지만 webpack과 같은 도구는 모든 종속성을 동적으로 묶습니다 ([종속성 그래프](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/concepts/dependency-graph.md)라고 하는 것을 생성합니다). 모든 모듈이 종속성을 명시하고 사용하지 않는 모듈을 번들로 묶지 않기 때문에 좋습니다.

가장 멋진 webpack 기능 중 하나는 JavaScript 외에 로더 또는 내장 [Asset Modules](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/guides/asset-modules.md)이 지원하는 다른 유형의 파일도 포함 할 수 있다는 것입니다. 즉, JavaScript에 대해 위에 나열된 동일한 이점 (예 : 명시적 종속성)이 웹 사이트 또는 웹 앱을 빌드 하는데 사용되는 모든 것에 대해 적용될 수 있습니다. 이미 어느 정도 익숙해져 있으므로 CSS부터 시작하겠습니다.

## Setup

시작하기 전에 프로젝트를 약간 변경해 보겠습니다:

**dist/index.html**

```html
<!DOCTYPE html>
<html>
 <head>
   <meta charset="utf-8" />
   <title>Asset Management</title>
 </head>
 <body>
   <script src="bundle.js"></script>
 </body>
</html>
```

**webpack.config.js**

```javascript
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
};
```

## Loading CSS

JavaScript 모듈 내에서 CSS 파일을 `import`하려면 [style-loader](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/loaders/style-loader.md)와 [css-loader](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/loaders/css-loader.md)를 [`module` Configuration](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/configuration/module.md)에 설치하고 추가해야합니다.

```sh
npm install --save-dev style-loader css-loader
```

**webpack.config.js**

```javascript
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
};
```

모듈 로더는 체인으로 연결할 수 있습니다. 체인의 각 로더는 처리된 리소스에 대한 변환을 적용합니다. 체인은 역순으로 실행됩니다. 첫 번째 로더는 결과(변환이 적용된 리소스)를 다음 로더로 전달합니다. 마지막으로 webpack은 체인의 마지막 로더가 JavaScript를 반환할 것으로 예상합니다.

위의 로더 순서를 유지해야 합니다. [`'style-loader'`](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/loaders/style-loader.md)가 먼저 나오고 그다음에 [`'css-loader'`](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/loaders/css-loader.md)가 나옵니다. 이 규칙을 따르지 않으면 webpack에서 오류가 발생할 수 있습니다.

> webpack은 정규 표현식을 사용하여 특정 로더를 찾고 제공해야 하는 파일을 결정합니다. 이 경우 `.css`로 끝나는 모든 파일은 `style-loader` 및 `css-loader`에 제공됩니다.
