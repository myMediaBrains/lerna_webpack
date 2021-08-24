# [lerna webpack eslint prettier](https://blog.shiren.dev/2021-02-15/)

## webpack

webpack 은 모듈 번들러이다. – 즉, webpack 은 하기 소스들의 depedencies 를 ingest 한다.

- webpack 의 목적은 여러 모듈 타입들의 소스들( CommonJS modules, AMD modules, CSS import, Images url, ES moudles)을 unify 하여, 당신의 자바스크립트 코드에 모든 것을 import 하고, shippable output 을 만드는데 있다.

| no  | webpack        | 설명                                                                                                         |
| --- | -------------- | ------------------------------------------------------------------------------------------------------------ |
| 1   | Entry point    | (디폴트) src/index.js                                                                                        |
|     |                | > dependency graph 를 만든다.                                                                                |
|     |                | > entry point 는 여러개가 있을 수 있다.                                                                      |
| 2   | Output         | (디폴트) dist/                                                                                               |
|     |                | > build process 를 거친 최종 출력이다.                                                                       |
|     |                | > 결과는 JavaScript 와 static files 이다.                                                                    |
| 3   | Loaders        | 여러 file extensions 를 처리하는 third-party extensoins 이다.                                                |
|     |                | > 자바스크립트가 아닌 파일들을 transform 하여, 하나의 의존성 모듈을 만든다.                                  |
|     |                | > 예: CSS 용, images 용, txt 파일 용 loader 들이 있다.                                                       |
| 4   | Plugins        | webpack 작동 방식을 변경할 수 있는 third-party exensions 이다.                                               |
|     |                | > 예: HTML, CSS 를 extracting 하는 용도의 plugins                                                            |
|     |                | > 예: 환경변수들을 셋업하는 용도의 plugins                                                                   |
| 5   | Mode           | development 와 production 모드가 존재한다.                                                                   |
|     |                | > production 모드에서는 자동으로 minification 과 기타 optimizations 를 적용한다.                             |
| 6   | Code splitting | 대규모 번들을 방지하는 최적화기법이다. – user interaction 이 일어날 때만 자바스크립트 코드를 로드할 수 있다. |
|     |                | > 코드는 a chunk 로 분리되어진다.                                                                            |

## 기본환경 셋업

| no  | 구분                                                   | 설명               |
| --- | ------------------------------------------------------ | ------------------ |
| 1   | $ git init lerna_webpack && cd lerna_webpack && code . | 프로젝트 폴더      |
| 2   | $ yarn init                                            | package.json       |
| 3   | $ yarn add lerna -D                                    | install            |
| 4   | $ lerna init -i                                        | lerna repo 로 전환 |
| 5   | $ lerna create pgjs                                    | pgjs 패키지 생성   |
| 6   | $ cd packages/pgjs                                     | update             |
|     | > \_\_test\_\_ 폴더 삭제                               | delete             |
|     | > lib > src                                            | modify             |
|     | > src/pgjs.js > src/index.js                           | modify             |

## webpack 셋업

**<u>(1) 초기 셋팅: entry, output, devServer</u>**

| no  | 구분                                                         | 설명    |
| --- | ------------------------------------------------------------ | ------- |
| 1   | $ lerna add -D **webpack** \--scope=pgjs                     | install |
|     | $ lerna add -D **webpack-cli** \--scope=pgjs                 | install |
|     | $ lerna add -D **webpack-dev-server@3.11.2** \--scope=pgjs   | install |
|     | > 프론트 서버를 만들어 주는 도구                             |         |
|     | > webpakc-dev-server 는 3.11.2 버전을 설치한다.              |         |
| 2   | $ touch packages/pgjs/package.json                           | update  |
|     | > **“scripts”: { “dev”**: “webpack **\--mode development**”} | add     |
| 3   | $ touch packages/pgjs/webpack.config.js                      | create  |
|     | > entry, output, devServer 설정                              |         |

packages/pgjs/webpack.config.js

```tsx
const path = require('path');

module.exports = {
  entry: path.resolve(__dirname, './src/index.js'),
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: 'bundle.js',
  },
  devServer: {
    contentBase: path.resolve(__dirname, './dist'),
  },
};
```

**<u>(2) 관련 파일 생성</u>**

| no  | command                                              | desc   |
| --- | ---------------------------------------------------- | ------ |
| 1   | $ mkdir packages/pgjs/src && cd packages/pgjs/src    | create |
|     | > echo ‘console.log(“Hello webpack!”’ > **index.js** | add    |
| 2   | $ touch pacakages/pgjs/dist/index.html               | create |

packages/pgjs/dist/index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Hello Webpack</title>
  </head>
  <body>
    <div>
      <h1>Hello Webpack</h1>
      <script src="./bundle.js"></script>
    </div>
  </body>
</html>
```

**<u>(3) yarn start 명령어 설정 및 실행</u>**

| no  | command                                                                                    | desc   |
| --- | ------------------------------------------------------------------------------------------ | ------ |
| 1   | $ touch packages/pgjs/package.json                                                         | update |
|     | > “scripts”: { “start”: “webpack serve \--config ./webpack.config.js \--mode development”} | add    |
| 2   | $ yarn start                                                                               | run    |

**<u>(4) loaders 셋팅: css-loader, style-loader, sass-loader, sass</u>**

| no  | 구분                                                                 | 설명             |
| --- | -------------------------------------------------------------------- | ---------------- |
| 1   | $ cd packages/pgjs                                                   | 해당 폴더로 이동 |
| 2   | $ yarn add -D css-loader style-loader sass-loader sass               | install          |
|     | > style-loader : DOM에 stylesheet 를 로딩                            |                  |
|     | > css-loader: CSS 파일을 모듈로 로딩                                 |                  |
|     | > sass-loader: import 로 SASS 파일을 로딩                            |                  |
| 3   | $ touch packages/webpack.config.js                                   | update           |
|     | > .css 확장자를 사용하는 경우                                        |                  |
|     | > use 속성에서 loader 의 순서가 매우 중요(우측에서 좌측 순으로 적용) |                  |

webpack.config.js : .css 확장자 사용하는 경우

```tsx
...
	module: {
    rules: [
      {
        test: /\.css$/,
      	use: ["style-loader", "css-loader"]
      }
    ]
  }
...
```

webpack.config.js: .scss 확장자까지 사용하는 경우

```tsx
...
	module: {
    rules: [
      {
        test: /\.(css|scss)$/,
      	use: ["style-loader", "css-loader", "scss-loader"]
      }
    ]
  }
...
```

| no  | command                              | desc |
| --- | ------------------------------------ | ---- |
| 4   | $ touch packages/pgjs/src/style.css  | test |
| 5   | $ touch packages/pgjs/src/index.html | test |
| 6   | $ touch packages/pgjs/src/index.js   | test |
| 7   | $ touch packages/pgjs/src/style.scss | test |
| 8   | $ yarn start                         | run  |

packages/pgjs/src/style.css

```css
h1 {
  color: orange;
}
```

packages/pgjs/src/style.scss

```scss
@import url('https://fonts.googleapis.com/css?family=Karla:weight@400;700&display=swap');

$font: 'Karla', sans-serif;
$primary-color: #3e6f9e;

body {
  font-family: $font;
  color: $primary-color;
}
```

packages/pgjs/src/index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Webpack tutorial</title>
  </head>
  <body>
    <h1>Hello webpack!</h1>
  </body>
</html>
```

packages/pgjs/src/index.js

```tsx
import './style.css';
console.log('Hello webpack!');
```

<u>**(5) loaders 셋팅: babel-loader including react**</u>

babel은 자바스크립트 컴파일러이자 transpiler 이다.

- 최신 자바스크립트를 컴파일하여 거의 모든 브라우저에서 실행할 수 있는 호환성 코드로 변환한다.

| no  | command                                                                                                    | desc    |
| --- | ---------------------------------------------------------------------------------------------------------- | ------- |
| 1   | $ cd packages/pgjs                                                                                         |         |
| 2   | $ yarn add -D @babel/core babel-loader @babel/preset-env @babel/preset-react                               | install |
| 3   | $ yarn add react react-dom                                                                                 | install |
| 4   | $ touch packages/pgjs/.babelrc                                                                             | create  |
|     | > @babel/preset-env: 지정된 브라우저에 따라 최신 자바스크립트 코드를 변환하는 babel plugins 의 컬렉션이다. |         |
|     | > @babel/preset-react: React 코드를 자바스크립트 코드로 전환                                               |         |

packages/pgjs/.babelrc

```json
{
  "presets": ["@babel/preset-env", "@babel/preset-react"]
}
```

| no  | command                                 | desc   |
| --- | --------------------------------------- | ------ |
| 5   | $ touch packages/pgjs/webpack.config.js | update |
|     | > @babel/preset-env 는 .js 확장자       |        |
|     | > @babel/preset-react 는 .jsx 확장자    |        |

packages/pgjs/webpack.config.js

```tsx
...
	module: {
    rules: [
      {
      	...
      },
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: ["babel-loader"]
      }
    ]
  }
```

| no  | command                            | desc      |
| --- | ---------------------------------- | --------- |
| 6   | $ touch packages/pgjs/src/index.js | test      |
|     | > 최신 자바스크립트 코드 테스트    |           |
| 7   | $ yarn start                       | pgjs 폴더 |

packages/pgjs/src/index.js

```tsx
import './style.scss';
console.log('Hello webpack!');

const fancyFunc = () => {
  return [1, 2];
};

const [a, b] = fancyFunc();
```

| no  | command                               | desc |
| --- | ------------------------------------- | ---- |
| 8   | $ touch pacakages/pgjs/src/index.html | test |
|     | > React 코드 테스트                   |      |
| 9   | $ touch packages/pgjs/src/index.js    | test |
|     | > React 코드 테스트                   |      |
| 10  | $ yarn start                          | run  |

packages/pgjs/dist/index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Hello Webpack</title>
  </head>

  <body>
    <div id="app"></div>
    <script src="./bundle.js"></script>
  </body>
</html>
```

packages/pgjs/src/index.js

```tsx
import React from 'react';
import ReactDOM from 'react-dom';

const title = 'React with Webpack and Babel';

ReactDOM.render(
  <div>
    <h1>{title}</h1>
  </div>,
  document.getElementById('app')
);
```

**(6) loaders 설정: HOT module replacement in React **

| no  | command                                 | desc    |
| --- | --------------------------------------- | ------- |
| 1   | $ cd packages/pgjs                      | cd      |
| 2   | $ yarn add -D react-hot-loader          | install |
| 3   | $ touch packages/pgjs/webpack.config.js | update  |

packages/pgjs/webpack.config.js

```tsx
const webpack = require('webpack')
...
  resolve: {
    extensions: ['*', '.js', '.jsx'],
  },
  ...
  plugins: [
    new webpack.HotModuleReplacementPlugin()
  ],
  devServer: {
    ...,
    hot: true,
  },

...
```

| no  | command                            | desc   |
| --- | ---------------------------------- | ------ |
| 4   | $ touch packages/pgjs/src/index.js | update |
|     | > module.hot.accept();             | add    |

packages/pgjs/src/index.js

```tsx
import React from 'react';
import ReactDOM from 'react-dom';

const title = 'React with Webpack and Babel';

ReactDOM.render(
  <div>
    <h1>{title}</h1>
  </div>,
  document.getElementById('app')
);

module.hot.accept();
```

| no   | command                                        | desc   |
| ---- | ---------------------------------------------- | ------ |
| 5    | $ touch packages/pgjs/src/App.js               | create |
| 6    | $ touch packages/pgjs/src/index.js             | update |
|      | > import App                                   |        |
| note | 업데이트될 때마다 localhost:8080 자동 리렌더링 | check  |

packages/pgjs/src/App.js

```tsx
import React from 'react';

const App = ({ title }) => (
  <div>
    <h1>{title}</h1>
  </div>
);

export default App;
```

packages/pgjs/src/index.js

```tsx
import React from 'react';
import ReactDOM from 'react-dom';

import App from './App';

const title = 'React with Webpack and Babel';

ReactDOM.render(<App title={title} />, document.getElementById('app'));

module.hot.accept();
```

**(7) loaders 설정: ESlint**

| no   | command                                                                                                                              | desc    |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------ | ------- |
| 1    | $ cd packages/pgjs                                                                                                                   | cd      |
| 2    | $ yarn add -D eslint eslint-loader babel-eslint eslint-config-airbnb eslint-plugin-import eslint-plugin-jsx-a11y eslint-plugin-react | install |
| 3    | $ touch packages/pgjs/.eslintrc                                                                                                      | create  |
| 4    | $ touch packages/pgjs/.eslintignore                                                                                                  | create  |
| 5    | $ touch packages/pgjs/webpack.config.js                                                                                              | update  |
| note | VSCode settongs.json                                                                                                                 | update  |
|      | > "eslint.options": { “ingorePattern”: “webpack.config.js”, }                                                                        | add     |

packages/pgjs/.eslintrc

```json
{
  "env": {
    "browser": true,
    "es2021": true,
    "node": true
  },
  "extends": ["airbnb", "prettier", "prettier/react"],
  "globals": {
    "Atomics": "readonly",
    "SharedArrayBuffer": "readonly"
  },
  "plugins": ["react", "prettier", "import"],
  "rules": {
    "react/prop-types": 0,
    "no-underscore-dangle": 0,
    "import/imports-first": ["error", "absolute-first"],
    "import/no-extraneous-dependencies": [
      "error",
      {
        "devDependencies": true
      }
    ],
    "import/newline-after-import": "error",
    "import/prefer-default-export": 0,
    "react/prefer-stateless-function": 0,
    "react/jsx-filename-extension": 0,
    "react/jsx-one-expression-per-line": 0,
    "linebreak-style": 0,
    "no-unused-vars": 0

    // ❗️error off : 0 , warning 1,❗️
  }
}
```

packages/pgjs/.eslintignore

```tsx
webpack.config.js;
node_modules;
dist / bundle.js;
```

packages/pgjs/webpack.config.js

```tsx
...
	{
    test: /\.js$/,
    exclude: /node_modules/,
    use: ['babel-loader', 'eslint-loader'],
  },
```

**(8) loaders 설정: Prettier**

| no   | command                                          | desc    |
| ---- | ------------------------------------------------ | ------- |
| 1    | $ yarn global add prettier                       | install |
| 2    | $ yarn add -D prettier                           |         |
| note | VSCode extension: Prettier                       | install |
| note | VSCode settings.json                             | update  |
|      | > “editor.formatOnSave”: false                   | 설정    |
|      | > “[javascript]”: {“editor.foramtOnSave”: true}, | 설정    |
| 2    | $ touch pacakges/pgjs/.prettierrc                | create  |

packages/pgjs/.prettierrc

```json
{
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "semi": true,
  "quoteProps": "as-needed",
  "jsxSingleQuote": false,
  "trailingComma": "es5",
  "arrowParens": "always",
  "endOfLine": "lf",
  "bracketSpacing": true,
  "jsxBracketSameLine": false,
  "requirePragma": false,
  "insertPragma": false,
  "proseWrap": "preserve",
  "vueIndentScriptAndStyle": false
}
```

**<u>(9) eslint-prettier 연동 사용 설정</u>**

| no   | command                                                                      | desc    |
| ---- | ---------------------------------------------------------------------------- | ------- |
| 1    | $ cd packages/pgjs                                                           | cd      |
| 2    | $ yarn add -D eslint-config-prettier eslint-plugin-prettier                  | install |
|      | > eslint-config-prettier: Prettier 와 conflict 하는 ESLint rules 를 turn-off | note    |
|      | > eslint-plugin-prettier: Prettier rules 를 ESLint rules 로 통합             | note    |
| 3    | $ touch packages/pgjs/.eslintrc                                              | update  |
| note | VSCode setting.json                                                          |         |
|      |                                                                              |         |

packages/pgjs/.eslintrc

```json
{
  "parser": "babel-eslint",
  "extends": "prettier",
  "plugins": ["prettier"],
  "rules": {
    "prettier/prettier": "error"
  }
}
```

VSCode settings.json

```json
// eslint와 prettier 연동
"editor.formatOnSave": true,
"javascript.format.enable": false,
"prettier.eslintIntegration": true
```

**(10) loaders 설정: husky & lint-staged**

husky 는 git 에 올리기 전에 코드를 모두 검사 하는 것이고, staged 된 코드만 검사하는 것이 lint-staged 이다.

| no  | command                            | desc    |
| --- | ---------------------------------- | ------- |
| 1   | $ cd packages/pgjs                 | cd      |
| 2   | $ yarn add -D husky lint-staged    | install |
| 3   | $ touch packages/pgjs/package.json | update  |

packages/pgjs/package.json

```json
...
	"lint-staged": {
    "*.js": ["eslint . --fix", "prettier --write"]
  },
	"husky": {
    "hooks": {
      "pre-commit": "eslint . --fix && prettier --write"
    }
  }
...
```
