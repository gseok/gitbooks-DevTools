## webpack 자동 재실행 webpack-dev-server

강력한 bundler tool인 webpack을 자동 재실행 하는 방법을 간략하게 소개한다.

webpack을 여러 언어, 여러 framework에서 사용가능하지만 이 글에서는 react을 개발할때 webpack을 사용하는 예제로 되어 있다.

---

### webpack의 watch

webpack에는 file의 변경을 감지해서 webpack을 다시 구동하는 옵션이  기본적으로 존재한다.

CLI환경에서, `$ webpack --help` 명령을 이용해보면, `--watch` 옵션을 확인 할 수 있다.

```bash
λ webpack --help
...

Basic options:
  --context    The root directory for resolving entry point and stats
                                       [string] [default: The current directory]
  --entry      The entry point                                          [string]
  --watch, -w  Watch the filesystem for changes                        [boolean]

...
```

해당 옵션을 사용하면, 기본적으로 file의 변경에 따라 webpack을 다시 구동하여서, webpack을 통한

bundling\(일종의 빌드, bundle.js을 생성\)을 지속적으로 할 수 있다.

webpack은 bundling을 하여 최종적으로는 `dist` 위치에 bundle.js을 생성한다.

이러한 bundling작업이 시간이 오래 걸리고, bundle.js는 그야말로 여러 모듈을을 하나로 묶은 하나의 파일을 생성할 뿐, 그 이상의 동작은 하지 않는다.

따라서 개발자는 테스트를 위해 별도의 서버 \(e.g, express\)을  이용하고, 서버에서 특정 파일이변경되면, 서버를 다시 시작 하는 모듈\(e.g. nodemon\)을 사용해야 한다.

webpack에서는 이러한 불편한 점을 줄이고, 개발자의 개발 생산성을 향상 시킬 수 있도록, **webpack-dev-server** 을 제공한다.

**기억할점**

* webpack은 bundler다
  * 여러 모듈을 하나의 파일로 만들어 준다. \(bundle.js\)
* webpack 명령어를 사용하면, `webpack.config.js` 에 설정된 `output` path 위치에 `bundle.js`가 만들어진다.
* webpack은 시간이 좀 걸리는 동작이다.
* webpack은 서버을 직접 지원하는건 아니다.

---

### webpack-dev-server을 통한 자동 재실행

webpack-dev-server는 `webpack` + `server` 의 기능을 한다.

따라서, 코드 수정시 **자동으로 bundling을 수행**하고 추가로 **서버를 재기동** 하며, 동시에 현재 테스트중인** Browser을 한번 다시 reload** 한다. \(COOL!!!!\)

webpack-dev-server는 webpack와 별개의 module로 되어 있기때문에, 별도의 설치가 필요하며, 또한 webpack-dev-server을 위한 config설정이 별도로 필요하다. 이를 하나씩 소개한다.

#### webpack-dev-server 설치

[npm 모듈](https://www.npmjs.com/package/webpack-dev-server)로 되어 있어 간단히 설치 가능 하다.

```bash
$ npm install webpack-dev-server --save-dev

or

$ npm install webpack-dev-server -g
```

#### webpack-dev-server config 기본 설정

webpack-dev-server의 config 파일은 `webpack`** 의 config 파일을 같이 사용**한다.

따라서 webpack 설정 파일에, webpack-dev-server용 설정 값을 작성하는 형태로 되어있다.

아래는 전형적인 예시이다.

```js
const path = require('path');

module.exports = {
    entry: [
        './index.js'
    ],

    output: {
        path: path.join(__dirname, 'dist/assets'),
        filename: 'bundle.js'
    },

    devServer: {
        port: 8080,
        contentBase: path.join(__dirname, 'dist')
    },

    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: [/node_modules/],
                use: [{
                    loader: 'babel-loader',
                    options: { presets: ['es2015', 'react'] }
                }],
            }
        ]
    }
};
```

살펴보면, `devServer` 옵션이 추가된 것을 볼 수 있다. webpack-dev-server의 server는 express로 되어있다고 한다.

server의 포트와, server의 base path을 설정 할 수 있다. 해당 설청을하고 webpack-dev-server을 구동하면, webpack을 통한 bundling이 되고, server가 구동 되는 것을 확인 할 수 있다.

**여기서 주의할점**

> **sever 구동을 확인한뒤, 위 설정상 output 위치인 **`dist/assets`**에 가보면, **`bundle.js`** 가 없다. **

?????????  왜 일까?

webpack은 실제 bundling을 해서 `bundle.js` 을 `dist/assets` 에 file write을 한다.

하지만..

> **webpack-dev-server는 **`In Memory`** 에서 **`bundle.js`** 을 생성하고, 별도의 file wirte을 하지 않는다.**
>
> **사용자가 webpack-dev-server가 띄우 server 에서 **`bundle.js`** 을 요청하면..**
>
> **\(즉 browser에서 &lt;script src='bundle.js\`&gt;을 하면..\), webpack-dev-server 가 In Memory에 생성한 bundle.js을 **
>
> **실제 파일 대신 주게 된다. 따라서 bundling 속도가 빠르고, Incremental build가 가능하며..**
>
> **개발중 변경한 내용을 빠르게 반영해보면서 테스트가 가능하다**



**기억하기**

* webpack은 bundle.js 을 실제 output 위치에 생성
* webpack-dev-server는 bundle.js을 in-memory에 생성, 실제 파일 미생성.

여기까지 기본 설정에 대한 설명을 하였다. 서버가 통합된 형태에 `In Memory` 로 bundling이 된다. 하지만, 우리가 원하는 개발중 `on the fly` 기능은 아직 설정이 되어 있지 않다. 해당 내용을 아래 추가로 설명한다.







#### webpack-dev-server config hot module replace설정

사실 우리\(개발자\)가 원하는 것은 아래와 같다고 볼 수 있다.

코드를 수정하면...

* 서버가 자동 재시작 or 수정된 코드가 서버에 자동 반영
* 수정된 코드를 테스트 중인 브라우저도 자동 재시작
* Bundle된 코드를 테스트 중인 브라우저에서 Debug 하고 싶음

서버 설정 입장에서...

* 서버의 basePath와 테스트하는 code가 반영될 위치가 다를 수 있음
* Debug는 계속 잘 되어야 함.

이와 같은 조건을 만족 시킬 수 있는 설정을 살펴보자

```js
const path = require('path');
const webpack = require('webpack');

module.exports = {
    devtool: 'inline-source-map',

    entry: [
        './assets/js/reactComponents/about.js'
    ],

    output: {
        path: path.join(__dirname + '/_site/assets/js/reactComponents/'),
        publicPath: '/assets/js/reactComponents/',
        filename: 'about.js'
    },

    plugins: [
        // webpack-dev-server enhancement plugins
        new webpack.HotModuleReplacementPlugin()
    ],

    devServer: {
        hot: true,
        port: 4000,
        contentBase: path.join(__dirname, '_site')
    },

    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: [/node_modules/],
                use: [{
                    loader: 'babel-loader',
                    options: { presets: ['es2015', 'react'] }
                }],
            }
        ]
    }
};
```

하나씩 살펴보겠다.

##### **devtool**

```js
devtool: 'inline-source-map'
```

> 모듈러\(babel\)가 원본 코드를 bundle.js로 바꾸거나, 중간에 minify 되어도, debug을 할 수 있도록
>
> source-map 을 생성하는 옵션

참고: [https://webpack.js.org/configuration/devtool/](https://webpack.js.org/configuration/devtool/)

source map을 생성해야 debuging이 가능하다. 개발중에는 inline-source-map을 추천한다.



##### **entry**

```js
entry: [
    './assets/js/reactComponents/about.js'
],
```

> webpack에게 알려주는 bundling되야 하는 app의 시작점, 시작 js파일

react의 경우 일반적으로  index.js 에 아래와 같은 코드로 구성되어 있다.

```js
import React from 'react';
import ReactDOM from 'react-dom';

class App extends React.Component {
    render() {
        return (
            <div>
                hello world
            </div>
        );
    }
}

const root = document.getElementById('root');
ReactDOM.render(<App />, root);
```

##### **output**

```js
output: {
    path: path.join(__dirname + '/_site/assets/js/reactComponents/'),
    publicPath: '/assets/js/reactComponents/',
    filename: 'about.js'
},
```

> path: webpack에게 알려주는 bundle.js 파일이 쓰여야 하는 위치
>
> filename: bundle.js 파일 이름, bundle.js가 보통이나, 사용자가 변해서 만들 수 있다.
>
> publicPath: devServer의 contentBase와 bundle.js의 위치가 다를때 webpack-dev-server 한테,  bundle.js위 위치 추적을 알리는 역할, webpack-dev-server의 경우 bundle.js가 in-memory로 동작하기 때문에, 추적해야 하는 위치를 명확히 알려야 debuging이 가능하다. \(매우중요.!!!\)

devServer 설정과 output 설정이 잘 맞아야, 정상적인 debugging이 가능하다.



##### **plugins**

```

```

webpack 확장 모듈, hot moule replace을 사용하기 위해서는 반드시 `webpack.HotModuleReplacementPlugin` 을 추가해야 한다. 별도의 npm설치는 필요없고, 최상위에서 webpack을 가져와서, new로생성하면 된다.





#### webpack-dev-server 사용

CLI 명령어로 바로 사용 가능하다. config 파일을 별도로 명시하지 않으면 기본으로 `webpack.config.js` 을 사용한다.

CLI 명령은 다음과 같이 수행하면 된다.

```bash
$ node_modules/.bin/webpack-dev-server

or

$ webpack-dev-server
```

config을 지정해서 사용하는 것도 가능하다.

```
$ webpack-dev-server --config webpack.config.dev.js
```

정상적으로 실행되면 아래 그림과 같이, `bundling이 수행`되고, `server가 구동`된다.

![](/assets/webpack-dev-server-run.png)



