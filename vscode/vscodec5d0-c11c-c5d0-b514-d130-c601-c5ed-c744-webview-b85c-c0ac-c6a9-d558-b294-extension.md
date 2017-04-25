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

> scheme

고유한 uri  값으로 소개되어 있다. uri을 생각하면 보통 프로토콜, 호스트 이름, 패스 이 세가지 요소를 기본적으로 떠올릴 수 있다. 하지만 테스트 결과, 여기서는 고유한 값이기만 하면 되기 때문에, 굳이 프로토콜부터 시작하는 형태를 취하지 않아도 동작한다. 즉 아래와 같이 고유한 값을 설정 하기만 하면 된다.

```java
const Schema = 'gseok-custom-schema';
```

> TextDocumentContentProvider

VSCode에서 정의한 Class이다. vscode의 name space 영역에 위치하고 있다. VSCode에서 에디터 영역에 content을 제공하는 클래스로, `TextDocumentContentProvider`를 상속 구현하여 extension만의 ui를 구현 할 수 있다. API 문서에 해당 클래스에 대한 설명은 다음과 같다.

text document content provider는 읽기전용 문서\(document\)을 에디터에 제공 할 수 있다. 여기서 읽기전용의 의미는, 일반적인 에디터가, 사용자가 코드를 작성하고, 이를 저장하는 형태 즉 쓰기와 읽기를 한다면, 읽기전용의 경우, md파일을 기반으로 생성된 정적인 html 페이지와 같이, 사용자가 작성을 하지는 못하고, 사용자가 글을 읽고, 혹시 글에 링크가 있으면 그걸 눌러볼 수 있는 수준의 문서를 의미한다.

그리고 text document content provider는 uri-scheme로 등록되어 있어야 한다. uri-scheme를 이용해서 에디터를 열면 해당 uri-scheme로 등록된 text document content provider을 통해 content을 제공한다.

`TextDocumentContentProvider`는 아래와 같은 하나의 event와 하나의 method를 구현해야 한다.

Event

* onDidChage
  * 옵션널하기 때문에 구현해도 되고 안해도 된다.
  * resource 변경시 이벤트가 발생한다.

Method

* `provideTextDocumentContent(uri: Uri, token: CancellationToken): ProviderResult`
* 실제 content을 제공하는 method이다. 해당 함수를 구현해서, extension만의 UI 구성 할 수 있다.
* 해당 메소드의 파라메터는 아래와 같은 의미를 가진다.
  * uri: 해당 메소드를 구현하는 TextDocumentContentProvider가 등록된 스키와와 동일한 스키마
  * token: 캔슬토큰
  * ProviderResult: 리턴값이고, string 또는 thenable이 가능하다. 어떤 type을 사용하던 최종적으로는 content string이 되어야 한다.

##### VSCode에서 실제 구현

앞에서 VSCode에서 제공하는 API를 간략하게 살펴보았다. 이제 해당 API와 Class을 사용해서 실제 적용하는 예제를 코드와 함께 설명하도록 하겠다.

* TextDocumentContentProvider 클래스를 상속받는 클래스 구현

```js
import * as vscode from 'vscode';

class CustomTextDocumentContentProvider implements vscode.TextDocumentContentProvider {
    private _onDidChange = new vscode.EventEmitter<vscode.Uri>();

    public async provideTextDocumentContent(uri: vscode.Uri, token: vscode.CancellationToken): Promise<string> {
        return `
            <head>
                <!-- inner content's lib or src loaded here -->
            </head>

            <body>
                <!-- extension own content here -->
                <div>
                    Hello This is My custom Read Only Editor Contents
                </div>
            </body>
        `;
    }

    get onDidChange(): vscode.Event<vscode.Uri> {
        return this._onDidChange.event;
    }
}
```

* uri-scheme을 사용한 등록

```js
let provider = new CustomTextDocumentContentProvider();
const Schema = 'gseok-custom-schema';

let registration = vscode.workspace.registerTextDocumentContentProvider(Schema, provider);
```

* 등록한 provider을 editor로 open

아래 예제에서는,  extension에서 등록한 command가 수행되면, 그때 editor을 open하는 형태로 되어 있다.

```js
export function activate(context: vscode.ExtensionContext) {
    let provider = new CustomTextDocumentContentProvider();
    const Schema = 'gseok-custom-schema';

    let registration = vscode.workspace.registerTextDocumentContentProvider(Schema, provider);

    let disposable = vscode.commands.registerCommand('gseok', () => {
        let previewUri = vscode.Uri.parse(Schema + '://authority/');

        return vscode.commands.executeCommand('vscode.previewHtml', previewUri,
                vscode.ViewColumn.One, 'Gseok Test Title').then((success) => {
            // editor open success after called here
            console.log('success open editor', success);
        }, (reason) => {
            vscode.window.showErrorMessage(reason);
        });
    });

    context.subscriptions.push(disposable, registration);
}
```

gseok라는 command가 VSCode에서 사용자에 의해 호출 되면, `vscode.previewHtml` 커맨드를 실행 하고. 이때 등록한  uri-scheme로 되어 있는 uri를 통해서,  예제에서 작성한 `CustomTextDocumentContentProvider`의 `provideTextDocumentContent`가 불리게 된다. 이후 `provideTextDocumentContent`가 제공한 content가 VSCode에서 정의한 `WebView`의 content로 등록되면서, 화면에 editor가 열리게 된다.

여기까지 진행하면, 기본적인 web page 형태의 content을 `VSCode`의 editor로 제공 하는게 가능하다.

하지만 추가적으로 제공한 content가 자체적인 lib, css을 사용하고 동작하게 하는 부분이 필요 할 수 있다.

아래에 이어서 위와 같은 추가 적인 궁금증에 대하여 설명해 보도록 한다.

##### 이벤트 처리 및 외부 소스 로드

`VSCode`에서는 `TextDocumentContent`을 본인들이 정의한 `WebView`로 `warpping`하여 화면에 뿌리고 있다. 거기에 더하여, `window`, `document`와 같은 객체를 기본 VScode에서는 접근 할 수 없도록 하고 있다.

하지만, custom으로 제공한 `TextDocumentContent`는 일종의 `iframe`와 같이 `VSCode` 본래의 html와는 별개로 작성 할 수 있다.

위 기본 예제에도 있듯, head 부터 body까지 별로도 생성 가능한 형태이다. 따라서, 제공하는 어떤 content에 본인만의 css나 lib, logic을 추가하는 것이 가능하다.

아래 소스로 설명하겠다.

```js
// gseokController.ts 혹은 gseokController.js 파일
(function () {
    function gseokTestFunction() {
        console.log('test function called!!!');
    }

    $(document).ready(function () {
        console.log('provided content ready...');

        $('.gseokButton').on('click', () => {
            gseokTestFunction();
        });
    });
})();
```

위와 같이 버튼에 따른 이벤트 처리 로직이 있고, 해당 로직이 별도의 파일인 `gseokController.ts` 또는 `gseokController.js`로 존재한다고 한다. 이제 앞서 설명한 `TextDocumentContentProvider`을 상속받은 `CustomTextDocumentContentProvider`을 좀더 수정해 보겠다.

```js
import * as vscode from 'vscode';
import * as path from 'path';

class CustomTextDocumentContentProvider implements vscode.TextDocumentContentProvider {
    public async provideTextDocumentContent(uri: vscode.Uri, token: vscode.CancellationToken): Promise<string> {
        return `
            <head>
                <script src="${this.getPath(path.join('jquery.min.js'))}"></script>
                <script>
                    console.log('script run possible!!!!');
                </script>
                <script src="${this.getPath('gseokControler.js')}">
                </script>
            </head>
            <body>
                <div class='gseokDiv'>
                    test....
                </div>
                <br/>
                <button class='gseokButton' onClick='$(".gseokDiv").text("div text changed");'>Click</button>
            </body>
        `;
    }

    private getPath(resourceName: string): string {
        let p = vscode.Uri.file(path.join(__dirname, './', resourceName)).toString();
        return p;
    }
}
```

위에서 설명한 `gseokController.js` 파일과 `jquery.min.js` 파일이 `CustomTextDocumentContentProvider` 파일과 동일한 폴더에 위치하고 있다고 가정하였다. 이제 `providerTextDocumentContent`의 리턴값에, head부분에 script load 와 scrdipt run 코드를 추가하였다. 그리고, jquery와 controller을 정상 로딩 할 수 있도록 vscode.Uri.file을 통해 path을 정확한 Uri 주소로 변경해주는 유틸 getPath 함수를 작성하였다.

위와 같은 형태로 작성후, VSCode에서 구동해 보면, 정상적으로 이벤트 핸들링이 가능하고, 에디터 상에 test 로 나타나고 있던 문자열이, div text changed 문자열로 변경되는 것을 확인 할 수 있다. css역시 동일한 방법을 사용해서, 내부 content의 스타일을 정의 할 수 있다.

결론적으로, VSCode에서 제공하는 API를 사용해서, Editor영역에 자신만의 View를 구성 할 수 있고, 해당 View 내부에서는 이벤트 처리 및 외부 라이브러리 로딩이 가능하다.

