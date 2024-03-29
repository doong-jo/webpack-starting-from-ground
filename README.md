# [Webpack, 이것만은 알고 가자](https://velog.io/@sdong001/Webpack-%EC%9D%B4%EA%B2%83%EB%A7%8C%EC%9D%80-%EC%95%8C%EA%B3%A0-%EA%B0%80%EC%9E%90)

## 이것만은 알아야 하지 않을까

Webpack은 웹 어플리케이션을 개발할 때 가장 인기 있는 번들러입니다. React 앱을 빌드해주는 도구로 생각하고 CRA의 Webpack 설정을 그대로 사용하기 보다는 이것이 어떻게 동작하는지, 핵심 원리는 무엇인지 이해해야 하지 않을까? 라는 생각에서 글을 적게되었습니다. 하지만 Webpack에서 사용하는 모든 플러그인과 번들러, 최적화 옵션을 공부하기 보다는 Webpack의 동작원리와 핵심 요소들을 이해하고 상황에 맞게 필요한 것들을 학습하는 것이 효율적이라고 생각합니다. 그래서 **이 글에서는 Webpack의 깊이있는 내용보다 얕게 '이것만은 알아야 하지 않을까' 라고 생각되는 내용을 적어봤습니다.** 

## Webpack에 대해서

![webpack](https://user-images.githubusercontent.com/35218826/59730847-eb233b00-927e-11e9-9788-408e699c9e58.png)
[출처](https://medium.com/@paul.allies/webpack-managing-javascript-and-css-dependencies-3b4913f49c58)

Webpack은 모던 자바스크립트 애플리케이션을 위해 사용되는 정적 모듈 번들러입니다. 여기서 번들러는 여러 패키지, 관련된 파일들을 하나로 묶어주는 도구라고 생각할 수 있습니다. 내부적으로는 프로젝트에 필요한 위와 같은 의존성 그래프를 참고하여 하나 이상의 번들로 만들어냅니다.

### Bundle과 Chunk

![](https://res.cloudinary.com/practicaldev/image/fetch/s--4_Wazx_7--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/ukz25x0zjhsu9docg5ob.png)

[출처](https://dev.to/websavi/how-to-code-split-redux-store-to-further-improve-your-apps-performance-4gg9)

#### Bundle

> Webpack에서는 자바스크립트를 포함하여 여러 리소스들을 연결한(Concentarte) 결과물입니다. Bundle은 여러 개의 Chunk로 구성됩니다.

#### Chunk

> 모든 코드를 하나의 거대한 파일(Bundle)로 만들지 않기 위해서 여러개의 Chunk라는 단위로 나눕니다. 코드가 어떻게 나뉘는지에 대한 방법도 임의로 구성할 수 있습니다. 

### 왜 Webpack을 써야할까

우리가 왜 Webpack을 사용해야 하는지 이해하기 위해서는 그 이전 웹에서 자바스크립트를 사용할 때 번들러는 어땠는지 살펴봐야 합니다.

두 가지 방법이 있었는데 하나는 너무 많은 스크립트를 불러오는 것이었습니다. 이 방법은 네트워크 병목을 일으켰고 요청 지연의 문제가 있었습니다. 두번째 방법은 모든 코드를 포함하고 있는 하나의 큰 `.js` 파일을 사용하는 것이었는데, 스코프, 크기, 가독성, 유지보수성에 대한 이유로 문제가 많았습니다.

#### Webpack의 핵심 원리, 해결책 [IIFE, Imeadiately invoked function expressions (즉시 실행 함수 표현)](https://developer.mozilla.org/ko/docs/Glossary/IIFE)

> IIFE는 스크립트 파일을 하나의 IIFE를 사용함으로써 큰 프로젝트에서 스코프 이슈를 해결할 수 있습니다. 따라서 IIFE를 사용함으로써 스코프 충돌에 대해 걱정할 필요 없이 안전하게 코드를 번들링할 수 있습니다.

Webpack은 IIFE를 사용하며 그 밖에 사용하는 도구로써는 Make, Gulp, Grund, Brocooli or Brunch가 있습니다. 이 도구들도 알려진과 같이 번들러(또는 테스크 러너)로써 프로젝트 파일들을 모두 번들링해줍니다. 하지만 다른 도구들과 다르게 Webpack은 [dependency graph](https://Webpack.js.org/concepts/dependency-graph/)를 생성하여 빌드시켜주는 도구로써 크고 복잡한 리소스가 있는 프로젝트에서는 Webpack을 사용하는 것이 좋습니다. (Grunt, Gulp는 오로지 리소스에 대한 툴로 사용되며 dependency graph 개념이 없음) 또한 비슷한 Browsify와 비교한다면 속도면에서 좀 더 우월합니다.

Webpack을 사용하지 않고 번들링을 한다고 가정해본다면, 하나의 파일을 변경한다는 것은 (번들을 새로 만들어야하므로)전체를 다시 빌드한다는 것입니다. 이를 해결하기 위해 번들링을 쉽게하려면 파일들 간에 스크립트를 재사용해야합니다. 하지만 이를 최적화하여 빌드하는 것은 어렵습니다. 또한 코드를 작성한 뒤에 그 코드가 스크립트에서 실제로 사용되는지 아닌지 어떻게 알 수 있을까요?

예를 들어 lodash에서 단일함수 만 사용하더라도 lodash 전체 코드를 추가한 다음 함께 번들링해야합니다. 이러한 코드의 종속성은 어떻게 해결할까요? 코드의 로딩지연은 커지는 것을 막아야하며 이를 해결하기 위해선 개발자의 많은 수작업이 필요합니다.

이러한 상황의 해결하기 위한 방법으로 Webpack을 사용하여 개발자가 직접 하기 어려운 **번들링의 성능과 최적화를 비교적 쉽게 개선할 수 있습니다.** 또한 강력한 기능 중 하나인 [코드 스플리팅](https://webpack.js.org/guides/code-splitting/)을 이용하여 **필요한 모듈만 로드하여 요청 지연을 줄일 수 있습니다.**

### Webpack의 핵심요소

아래는 Webpack의 core라고 할 수 있는 요소들입니다. 이 요소들을 코드와 함께 살펴보겠습니다.

- Entry
- Output
- Loaders
- Plugins
- Mode

#### create-react-app webpack(이하 'CRA') 설정 뜯어보기
CRA는 React로 개발할 때 가장 유용한 보일러플레이트입니다. 이 CRA 설정에는 React를 구성하는데 필요한 Webpack 설정들이 기본으로 작성되어 있습니다. 이 설정들을 보며 React에서 Webpack을 어떻게 설정하는지 살펴보겠습니다.

일반적으로 CRA를 통해 React 환경 구성을하면 Webpack 관련 설정들이 숨겨져있는데 이를 꺼내어 개발자가 직접 수정할 수 있도록 해야합니다.

1. CRA를 통한 React 환경구성

```sh
npx create-react-app myapp
```

2. `eject`를 통해 Webpack 설정을 외부로 꺼내기

```sh
npm run eject

# 또는 yarn
yarn eject
```

이렇게 `eject` 스크립트를 통해 설정을 꺼낼 수 있으며 `config`와 `script` 디렉터리가 생성되는데 `config/webpack.config.js` 에서 Webpack 환경설정을 볼 수 있습니다.

이 요소들을 CRA에서의 `webpack.config.js` 코드에서 모든 설정을 살펴보진 않고 핵심적인 부분만 하나씩 살펴보도록 하겠습니다.

#### 1. Entry / Output

![entry](https://getdrip.s3.amazonaws.com/uploads/image_upload/image/325131/embeddable_a721c3b5-1335-4b71-aeca-cd4d436f554b.png)

entry는 building의 시작점이라고 볼 수 있으며, output은 buidling 된 파일에 결과를 제공합니다.

```js
entry: [
  // dev 모드면 Hot reload(파일의 변경사항 자동 반영) 설정을 추가
  isEnvDevelopment && require.resolve('react-dev-utils/webpackHotDevClient'),

  // src/index.js
  paths.appIndexJs
].filter(Boolean), // true값인 결과만 반환
```

```js
output: {
  // build path, filename 지정
  path: isEnvProduction ? paths.appBuild : undefined,
  filename: isEnvProduction
    ? 'static/js/[name].[contenthash:8].js' // 해시 값을 넣음으로써 로컬에서 캐싱을 유도할 수 있습니다.
    : isEnvDevelopment && 'static/js/bundle.js',
  // more...
},
```

#### 2. Loaders

loader는 번들링할 파일에 대한 규칙을 정하고 실행하는 역할을 담당합니다. 아래의 코드는 모듈이 될 수 있는 파일들을 `babel-loader`를 통해 읽어들이는 것을 볼 수 있습니다.

```js
loaders: [
  {
    test: /\.(js|mjs|jsx|ts|tsx)$/,
    include: paths.appSrc,
    loader: require.resolve("babel-loader")
    // ...
  }
];
```

리소스 파일들 또한 loader를 설정할 수 있습니다.

```js
loaders: [
  {
    test: /\.(js|mjs|jsx|ts|tsx)$/,
    include: paths.appSrc,
    loader: require.resolve("babel-loader")
    // ...
  },
  {
    test: [/\.bmp$/, /\.gif$/, /\.jpe?g$/, /\.png$/],
    loader: require.resolve("url-loader")
  }
];
```

#### 3.Plugins

플러그인은 번들 최적화, 환경 변수 주입과 같은 작업들을 수행할 수 있으며 이외에도 다양한 기능들이 존재합니니다. 아래 코드에서는 `HtmlWebPackPluglin`을 사용하여 `filename`의 값으로 지정된 파일안에 `<script>` 태그를 주입합니다.

```js
plugins: [
  new HtmlWebpackPlugin(
    Object.assign(
      {},
      {
        inject: true,
        template: paths.appHtml // ./src/index.html
      },
      isEnvProduction
        ? {
            minify: {
              removeComments: true
              //...
            }
          }
        : undefined
    )
  )
  // more ...
];
```

#### 4.Mode

webpack을 구동시킬 mode를 지정해줄 수 있습니다. `development`, `production`, `none` 세가지가 있는데 각각의 설정에 따라 webpack의 내장된 최적화를 어떤 방식으로 할지 지정할 수 있습니다. [webpack의 mode 문서](https://webpack.js.org/configuration/mode/)를 참고하여 어떤 최적화가 설정되는지 자세하게 알 수 있습니다. 이를 바탕으로 임의로 `optimization` 속성에 최적화 옵션을 추가할 수 있습니다.

```js
mode: isEnvProduction ? 'production' : isEnvDevelopment && 'development',
```

## 직접 Bundle을 최적화하려면? webpack-bundle-analyzer

번들의 구성사항을 한번에 볼 수 있는 도구입니다. 이를 활용하여 어떤 구성이 bundle의 크기를 얼마나 차지하는지 확인할 수 있고 이를 통해서 Webpack 설정을 수정하거나 라이브러리 교체도 고려해볼 수 있습니다.

[webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer)

![](https://cloud.githubusercontent.com/assets/302213/20628702/93f72404-b338-11e6-92d4-9a365550a701.gif)

## 생각

React로 개발하며 빌드에 대해서는 크게 관심을 가지지 않았었는데 관심을 가지고 시간과 노력을 들일수록 성능을 향상시킬 수 있다는 것을 알았습니다. 또한 SPA를 개발할 때 빌드 도구를 많이 사용하게 되는데 이를 잘 이해하지 못한 채 개발하는 건 마치 카메라의 조리개와 렌즈를 잘 이해하고 있는 사진가가 일반인보다 사진을 잘 찍는 것과 마찬가지 아닐까 생각이 들었습니다. 빌드 도구도 함께 이해한다면 좋은 성능을 낼 수 있을 것 같습니다.


[썸네일 출처](https://www.toptal.com/react/webpack-react-tutorial-pt-1)
