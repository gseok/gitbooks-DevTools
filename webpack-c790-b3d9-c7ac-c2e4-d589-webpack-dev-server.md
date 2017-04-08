### webpack 자동 재실행 webpack-dev-server

강력한 bundler tool인 webpack을 자동 재실행 하는 방법을 간략하게 소개한다.

webpack을 여러 언어, 여러 framework에서 사용가능하지만 이 글에서는 react을 개발할때 webpack을 사용하는 예제로 되어 있다.

---

#### webpack의 watch

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

#### webpack-dev-server을 통한 자동 재실행

webpack-dev-server는 `webpack` + `server` 의 기능을 한다.

따라서, 코드 수정시 **자동으로 bundling을 수행**하고 추가로 **서버를 재기동** 하며, 동시에 현재 테스트중인** Browser을 한번 다시 reload** 한다. \(COOL!!!!\)

webpack-dev-server는 webpack와 별개의 module로 되어 있기때문에, 별도의 설치가 필요하며, 또한 webpack-dev-server을 위한 config설정이 별도로 필요하다. 이를 하나씩 소개한다.



##### webpack-dev-server 설치

[npm 모듈](https://www.npmjs.com/package/webpack-dev-server)로 되어 있어 간단히 설치 가능 하다.

```bash
$ npm install webpack-dev-server --save-dev

or

$ npm install webpack-dev-server -g
```



##### webpack-dev-server config 설정

webpack-dev-server의 config 파일은 **`webpack` 의 config 파일을 같이 사용**한다.

따라서 webpack 설정 파일에, webpack-dev-server용 설정 값을 작성하는 형태로 되어있다.





##### webpack-dev-server 사용

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














