#### VSCode에서 에디터 영역을 webview로 사용하는 extension

VSCode는 기본 철학이 VSCode의 모든 영역에 대하여, VSCode가 제공하는 API로 UI에 기여하고, 그외는 모두 막는 방향을 취하고 있다. 따라서 복잡한 형태의 UI를 보여주고 싶은 경우 이를 제공할 방안이 사실 별로 없는 상태이다.

하지만 다행이도, VSCode는 기본적으로 에디터 영역을 Extension에서 제공하는 Contents로 구성 할 수 있도록 API를 제공하고 있다. 즉 에디터 영역은 VSCode의 Extension개발자가 자신만의 Contents를 이용하여 구성 할 수 있다.

좀 더 쉽게 이야기 해서, VSCode의 Extension중 에디터 영역을 자신만의 Contents로 구성하는 Extension은 VSCode의 시작 페이지와 같은 형태의 구성이 가능하다.



##### VSCode에서 제공하는 API

VSCode는 기본적으로 에디터 영역을 컨트롤 할 수 있는 여러 API를 가지고 있다. VSCode의 extension API 문서에서 여러 API중 다음과 같이 에디터의 content를 제공하는 provider를 등록하는 API를 발견하였다.

`registerTextDocumentContentProvider` 라는 이름을 가진 API이다. 이 복잡한 이름의 API는 `vscode.workspace`의 name space영역에 위치하고 있다.

`registerTextDocumentContentProvider`의 함수 시그니쳐와 기본 설명은 아래와 같다.

```js
registerTextDocumentContentProvider(scheme: string, provider: TextDocumentContentProvider): Disposable
```

* 역할
  * text document content provider를 등록\(register\)한다.
  * scheme당 오직 한개의 provider만 등록 가능하다.
* 파라메터
  * scheme
    * 스트링 타입
    * 등록을 위한 고유한 uri 값
  * provider
    * TextDocumentContentProvider 타입

    * content을 제공하는 content provider
* 리턴값

  * Disposable

    * 등록한 provider가 dispose될때 unregisted한다.



즉 extension 구현시 해당 API를 사용해서 에디터 영역에 Contents를 제공 할 수 있다. 해당 API를 사용하려면, 두개의 필수 파라메터를 제공해야 한다. 각각을 살펴 보겠다.



scheme

고유한 uri  값으로 소개되어 있다. uri을 생각하면 보통 프로토콜, 호스트 이름, 패스 이 세가지 요소를 기본적으로 떠올릴 수 있다. 하지만 테스트 결과, 여기서는 고유한 값이기만 하면 되기 때문에, 굳이 프로토콜부터 시작하는 형태를 취하지 않아도 동작한다. 즉 아래와 같이 고유한 값을 설정 하기만 하면 된다.

```java
const Schema = 'gseok-custom-schema';
```



TextDocumentContentProvider 



