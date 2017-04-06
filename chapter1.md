### vscode에서 express로 간단 node 서버 설정하기

web app을 개발할때, 특히 client을 개발하면서, 구현한 app \(web client\)가 서버에서 구동되는 모습을 확인해야하는 경우가 있다.

이런경우, 일반적으로는, simple한 형태의 node서버 구현하여 붙이거나, apache tomcat등을 직접 설치 하여야 한다.

이와 같이, 직접 서버를 붙이는 경우, vscode을 통한 개발 환경과 무관하게, 별도의 cmd 창이나, 별도의 os등에 이를 구축하고 운영하여야하는 귀찮음이 존재한다.

vscode에는 vscode을 벋어나지 않고, vscode에서 곧바로 서버를 띄우고 사용할 수 있는 plugin이 존재한다. \(nice!!\)

[vscode의 marketplace](https://marketplace.visualstudio.com/)에 가보면 여러종류의 vscode내장형 server 모듈들이 존재한다.

본인이 주력으로 사용하거나 잘 아는 server모듈을 사용해도 된다.

이 글에서는 express \(node\)서버를 간단하게 소개한다.



#### 소개

아래 그림과 같이 vscode에서 곧바로, 현재 위치를 기준으로 web server를 구동하고, browser을 통해 바로 해당 서버에서 바로 테스트 해볼 수 있다.

![](/assets/vscode-express-review..gif)

#### 설치

vscode의 extension\(plugin\) 설치 방법과 동일하다

##### vscode command을 통한 설치

```
Launch VS Code Quick Open (Ctrl + P)

ext install vscode-express
```

![](/assets/vscode-ext-command-install.png)

##### vscode의 extension tab ui을 통한 설치

![](/assets/vscode-ext-ui-install.png)

vscode의 좌측 extension tab의 ui을 통해 express을 검색하고 설치한다.

두가지 설치 방법 모두, 설치 이후, vscode을 다시 로드 해야 extension이 동작한다.



#### 사용

vscode의 command을 통해 간단하게 server을 실행 할 수 있다.

```
Launch VS Code Command Input Box (Ctrl + Shift + P)

> express: Host Current Workspace
```

![](/assets/vscode-express-ext-run.png)

command로 express을 현재 workspace기준으로 실행하거나, 실행과 동시에 browser을 open하는 등의 동작을 할 수 있다.

기본적으로 express server의 실행이 성공하면 아래와 같이 상단에 메시지가 출력된다.

![](/assets/vscode-express-ext-run-success.png)

하단 출력창에도 express server의 실행에 대한 메시지가 출력된다.

![](/assets/vscode-express-run-output.png)

workspace에 client app이 있다면, 아래와 같이 곧바로 브라우저에서, 구동을 확인해 볼 수 있다. 구동을 확인할때, port주소를 잘 맞추어서 실행해야 한다.

![](/assets/client-run.png)

#### 설정

보통은 기본 설정 상태로 사용가능하지만, 서버를 여러개 띄운다던가. 특정 포트를 사용한다던가. server의 root 패스를 조정하는 등의 추가적인 설정이 필요 할 수 있다.

이러한경우 vscode의 express configuration을 설정해서 사용하면 된다.

```
vscode의 메뉴에서

파일(F) > 기본설정(P) > 설정(S)

을 통해 vscode의 settings.json을 open 한다.
```

settings.json의 검색메뉴에서 express로 검색하면, express extension의 설정을 할 수 있다.

![](/assets/vscode-express-ext-setting.png)

필요에 따라 해당 값을 수정해서 사용하면 된다.







