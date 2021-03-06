# Concepts

webpack의 핵심은 최신 JavaScript 애플리케이션을 위한 정적 모듈 번들러입니다. 웹팩이 애플리케이션을 처리 할 때 프로젝트에 필요한 모든 모듈을 매핑하고 하나 이상의 번들을 생성하는 [종속성 그래프(Dependency Graph)](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/concepts/dependency-graph.md)를 내부적으로 빌드합니다.

> JavaScript 모듈 및 webpack 모듈에 대해 자세히 알아보고 싶다면 [여기](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/concepts/modules.md)를 읽어보세요.

4.0.0 버전부터 **webpack은 Configuration 파일에 대한 설정을 필요로 하지 않지만** 더 나은 요구를 충족하려면 [Incredibly Configurable 설정](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/configuration/README.md)이 필요합니다.

핵심 개념만을 이해하고 싶다면 아래의 Core Concepts 을 참조하세요:
- [Entry](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/concepts/README.md#Entry)
- [Output](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/concepts/README.md#Output)
- [Loaders](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/concepts/README.md#Loaders)
- [Plugins](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/concepts/README.md#Plugins)
- [Mode](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/concepts/README.md#Mode)
- [Browser Compatibility](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/concepts/README.md#Browser-Compatibility)

본 문서는 **높은 수준**의 webpack 개념을 제공함과 동시에 상세한 사용법에 대한 링크를 제공하고 있습니다.

Module Bundler에 대한 숨겨진 아이디어와 이것이 어떻게 동작하는지에 대하여 이해하고자 하면 다음 링크를 참고하세요.

- [Manually Bundling an Application](https://www.youtube.com/watch?v=UNMkLHzofQI)
- [Live Coding a Simple Module Bundler](https://www.youtube.com/watch?v=Gc9-7PBqOC8)
- [Detailed Explanation of a Simple Module Bundler](https://github.com/ronami/minipack)

## Entry

진입점(Entry Point)은 웹팩이 내부 종속성 그래프 작성을 시작하는 데 사용해야하는 모듈을 나타냅니다. webpack은 진입점이 (직접/간접 적으로) 의존하는 다른 모듈과 라이브러리를 알아냅니다.

기본적으로 값은 `./src/index.js` 이지만 webpack 구성에서 [진입점(`entry`) 속성을 설정](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/configuration/entry-context.md#entry)하여 다른 (또는 여러 진입 점)을 지정할 수 있습니다. 예를 들면 아래와 같습니다:

**webpack.config.js**

```javascript
module.exports = {
  entry: './path/to/my/entry/file.js'
};
```

> [진입점(Entry Point)](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/concepts/entry-points.md) 섹션에서 자세히 알아보세요.

## Output

**output** 속성은 webpack이 생성한 번들을 내보낼 위치와 이러한 파일의 이름을 지정하는 방법을 지정할 수 있습니다. 기본 출력 파일의 경우 `./dist/main.js`로, 다른 생성된 파일의 경우 `./dist` 폴더로 기본 설정됩니다.

**webpack.config.js**

```javascript
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  }
};
```

위의 예시에서 `output.filename` 및 `output.path` 속성을 사용하여 webpack에 번들의 이름과 번들을 내보낼 위치를 알려줍니다. 상단에 import되는 `path` 모듈은 파일 경로를 조작하는데 사용되는 핵심 [Node.js 모듈](https://nodejs.org/api/modules.html)입니다.

`output` 속성에는 [더 많은 설정](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/configuration/output.md)을 할 수 있으며, 그 개념에 대해 배우고 싶다면 [Output 섹션](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/concepts/output.md)에서 더 많은 것을 배울 수 있습니다.

## Loaders

기본적으로 webpack은 JavaScript 및 JSON 파일 만 이해합니다. **로더(Loaders)**를 사용하면 webpack이 다른 유형의 파일도 처리하고 이를 애플리케이션에서 사용하여 종속성 그래프에 추가 할 수있는 유효한 [모듈](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/concepts/modules.md)로 변환 할 수 있습니다.

> `.css` 파일과 같은 유형의 모듈들을 `import` 할 수 있는 기능은 webpack만의 전용 기능이며, 다른 Bundler나 Task Runner에서 지원되지 않을 수 있음을 유의해야 합니다. webpack 개발진은 이 언어의 확장이 개발자들에게 더 정확한 종속성 그래프를 그려주는 것을 보장한다고 확신합니다.

높은 수준에서 **로더**는 두 가지 속성을 가지고 있습니다.

1. `test` 속성은 어떤 파일이 변환(Transform)되어야 하는지를 나타냅니다.
2. `use` 속성은 어떤 로더(Loader)가 변환에 사용되어 질 것인지를 나타냅니다.

**webpack.config.js**

```javascript
const path = require('path');

module.exports = {
  output: {
    filename: 'my-first-webpack.bundle.js'
  },
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  }
};
```

위의 설정에서는 단일 모듈에 대한 `test`와 `use` 속성이 `rules`에 정의되어 있습니다. 이것은 webpack 컴파일러에게 다음의 것들을 알려줍니다.

> "*.txt파일 중 `require()` 또는 `import` 문을 만나면 Bundles에 추가하기 전에 `raw-loader`를 사용하여 변환하라는 명령을 내려!"

> webpack Configuration에서 규칙을 정의할 때 `rules`가 아니라 `module.rules` 아래에 규칙을 정의한다는 점을 기억하는 것이 중요합니다. 물론, 잘못된 설정을 했을 경우에는 webpack이 잘못되었다고 경고합니다.

> 정규식을 사용하여 파일을 일치시킬 때 `/\.txt$/`는 `'/\.txt$/'`또는 `"/\.txt$/"`와 동일하지 않습니다. 전자는 webpack에 '.txt'로 끝나는 모든 파일과 일치하도록 지시하고 후자는 webpack에 절대 경로 '.txt'를 가진 단일 파일과 일치하도록 지시합니다.

[로더](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/concepts/loaders.md) 섹션에서 자세한 정보와 추가 설정을 확인할 수 있습니다.

## Plugins

로더는 특정 유형의 모듈을 변환하는 데 사용되지만 플러그인을 활용하여 번들 최적화, 리소스 관리 및 환경 변수 주입과 같은 광범위한 작업을 수행할 수 있습니다.

> [플러그인 인터페이스](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/api/plugins.md)와 이를 사용하여 webpack의 기능을 확장하는 방법을 읽어보세요.

플러그인을 사용하려면 `require()`를 사용하여 `plugins` 배열에 추가해야 합니다. 대부분의 플러그인은 옵션을 통해 사용자 정의할 수 있습니다. 다른 목적을 위해 구성에서 플러그인을 여러 번 사용할 수 있으므로 `new` 연산자로 호출하여 플러그인의 인스턴스를 만들어야 합니다.

**webpack.config.js**

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin'); //installed via npm
const webpack = require('webpack'); //to access built-in plugins

module.exports = {
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};
```

위의 예에서 `html-webpack-plugin`은 생성 된 모든 번들을 자동으로 삽입하여 애플리케이션에 대한 HTML 파일을 생성합니다.

> webpack은 다양한 플러그인을 제공합니다! [플러그인 목록](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/plugins/README.md)을 확인하세요.

webpack에서 플러그인을 사용하는 것은 간단합니다. 그러나 추가로 살펴볼 다양한 사용 사례가 많이 있습니다. [여기](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/concepts/plugins.md)에서 자세히 알아보세요.

## Mode

`mode` 매개 변수를 `development`, `production` 또는 `none`으로 설정하여 각 환경에 해당하는 webpack의 내장 최적화를 활성화 할 수 있습니다. 기본값은 `production`입니다.

```javascript
module.exports = {
  mode: 'production'
};
```

[여기](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/configuration/mode.md)에서 `mode` Configuration에 대해 자세히 알아보고 각 값에서 어떤 최적화가 수행되는지 알아보세요.

## Browser Compatibility

webpack은 [ES5를 준수](https://kangax.github.io/compat-table/es5/)하는 모든 브라우저를 지원합니다 (IE8 이하는 지원되지 않습니다). webpack은 [`import()`](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/guides/code-splitting.md#dynamic-imports) 및 [`require.ensure()`](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/guides/code-splitting.md#dynamic-imports)을 사용하려면 `Promise`가 필요합니다. 이전 브라우저를 지원하려면 이러한 표현식을 사용하기 전에 [polyfill을 로드](https://github.com/LeeKyuHyuk/webpack-korean-translation/blob/master/guides/shimming.md)해야 합니다.

## Environment

webpack 5는 Node.js 버전 10.13.0 이상에서 실행됩니다.
