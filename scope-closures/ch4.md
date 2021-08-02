# You Don't Know JS Yet: Scope & Closures - 2nd Edition
# 4장: 전역 스코프<sub>Global Scope</sub> 살펴보기

3장에서는 "전역 스코프"에 대해 여러 번 언급했지만, 여전히 모던 JS에서 프로그램의 가장 바깥쪽 스코프가 왜 그렇게 중요한지 궁금할 수 있다. 이제는 대부분의 작업이 전역이 아닌 함수 및 모듈 내부에서 이루어고 있다.

"전역 스코프 사용을 피하라"라고 말하는 것으로 충분한가?

JS 프로그램의 전역 스코프는 여러분이 상상하는 것보다 훨씬 더 많은 효용과 뉘앙스를 가진 풍부한 주제이다. 이 장에서는 먼저 전역 스코프가 현재 JS 프로그램 작성에 얼마나 유용하고 관련성이 있는지 살펴본 다음, 다양한 JS 환경에 따라 어디에서 그리고 *어떻게* 전역 스코프에 접근하는지 그 차이점에 대해 살펴볼 것이다.

렉시컬 스코프를 사용하여 프로그램을 구성하는 데 있어 전역 스코프를 완전히 이해하는 것은 매우 중요하다.

## 왜 전역 스코프가 중요할까?

대부분의 애플리케이션이 여러 개의 개별 JS 파일로 구성된 것은 여러분에게 놀라운 일이 아닐 것이다. 그렇다면 JS 엔진을 통해 단일 런타임 컨텍스트에서 모든 개별 파일을 정확히 어떻게 연결할 수 있을까?

브라우저에서 실행되는 애플리케이션에서는 크게 세 가지 방법이 있다.

첫째, ES 모듈을 직접 사용하는 경우(다른 모듈 번들 형식으로 변환하지 않음), 이러한 파일은 JS 환경에 의해 개별적으로 로드된다. 그런 다음 각 모듈들은 접근해야 하는 다른 모듈을 'import'한다. 분리된 모듈 파일들은 공유된 바깥쪽 스코프없이 전적으로 공유된 import만을 사용한다.

둘째, 빌드 프로세스에서 번들러를 사용하는 경우 일반적으로 브라우저와 JS 엔진으로 전송하기 전에 모든 파일이 연결되고, 그 결과 브라우저와 JS엔진은 하나의 큰 파일만 처리한다. 애플리케이션의 모든 부분이 단일 파일에 함께 위치하더라도 각 부분이 다른 부분에서 참조할 *이름*을(를) 등록하는 메커니즘과 해당 접근이 가능하도록 하기 위한 기능이 필요하다.

일부 빌드 설정에서는 파일의 전체 내용이 감싸는 함수<sub>wrapper function</sub>, 범용 모듈(UMD—부록 A 참조) 등과 같은 하나의 닫힌 스코프<sub>enclosing scope</sub>로 감싸집니다. 각 부분은 해당 공유 스코프에서 로컬 변수를 사용하여 다른 부분에서 접근 할 수 있도록 스스로를 등록할 수 있다. 예를 들어, 다음과 같다.

```js
(function wrappingOuterScope(){
    var moduleOne = (function one(){
        // ..
    })();

    var moduleTwo = (function two(){
        // ..

        function callModuleOne() {
            moduleOne.someMethod();
        }

        // ..
    })();
})();
```

위와 같이 `wrappingOuterScope()` 기능 범위 내 `moduleOne`과 `moduleTwo` 지역 변수를 선언하여 서로 접근할 수 있도록 했다.

`wrappingOuterScope()`의 범위는 전체 환경 전역 스코프가 아니라 함수이지만, 최상위 식별자를 모두 저장할 수 있는 양동이로 작동하며(실제로는 전역 스코프에 저장하는 것은 아니지만) "애플리케이션 스코프<sub>application-wide scope</sub>" 역할을 한다. 그런 점에서 전역 스코프를 대신하는 역할을 한다.

마지막 세번째 방법으로, 번들러 도구가 애플리케이션에 사용되지 않거나, (ES 모듈이 아닌) 파일이 단순히 브라우저에 개별적으로 로드가 되지 않는('스크립트' 태그 또는 기타 동적 JS 리소스 로딩을 통해)것과 상관없이 이러한 모든 부분을 둘러싸는 하나의 스코프가 없으면  **전역 스코프**가 유일하게 연결할 수 있는 방법이다.

이러한 종류의 번들 파일은 종종 다음과 같이 작성할 수 있다.

```js
var moduleOne = (function one(){
    // ..
})();
var moduleTwo = (function two(){
    // ..

    function callModuleOne() {
        moduleOne.someMethod();
    }

    // ..
})();
```

여기서는 둘러싸고 있는 함수 스코프가 없기 때문에 이 `moduleOne`과 `moduleTwo` 선언은 단순히 글로벌 스코프에 포함된다. 이는 파일이 연결되어 있지 않고 개별적으로 로드된 경우와 동일하다:

module1.js:

```js
var moduleOne = (function one(){
    // ..
})();
```

module2.js:

```js
var moduleTwo = (function two(){
    // ..

    function callModuleOne() {
        moduleOne.someMethod();
    }

    // ..
})();
```

이러한 파일들이 브라우저 환경에서 일반 독립 실행형 .js 파일로 별도로 로드되는 경우, 각 파일들의 최상위 변수 선언이 전역 변수가 된다. 왜냐하면 JS엔진에서는 전역 스코프만이 별개의 파일들(-독립적인 프로그램들)이 공유하는 유일한 리소스이기 떄문이다.

런타임 동안 애플리케이션의 코드가 존재하는 위치와 각 부분이 다른 부분에 접근하여 협력할 수 있는 방법을 설명하는 것 외에도, 전역 스코프는 다음에서도 찾아볼 수 있다:

* JS가 제공하는 빌트인<sub>built-ins</sub>:

    - primitives: `undefined`, `null`, `Infinity`, `NaN`
    - natives: `Date()`, `Object()`, `String()`, etc.
    - global functions: `eval()`, `parseInt()`, etc.
    - namespaces: `Math`, `Atomics`, `JSON`
    - friends of JS: `Intl`, `WebAssembly`

* JS을 호스팅하는 환경에서 제공하는 빌트인:

    - `console` (and its methods)
    - the DOM (`window`, `document`, etc)
    - timers (`setTimeout(..)`, etc)
    - web platform APIs: `navigator`, `history`, geolocation, WebRTC, etc.

이는 여러분의 프로그램이 상호작용할 수많은 *글로벌* 중 일부에 불과하다.

| 비고: |
| :--- |
| Node도 "전역적으로" 여러 요소를 노출하지만 기술적으로는 `전역(global)` 스코프에 있지는 않다. : `require()`, `__dirname`, `module`, `URL`, 등등. |

대부분의 개발자는 전역 스코프가 단순히 애플리케이션의 모든 변수를 떠넘기는 장소가 되어서는 안 된다는 데 동의한다. 엉망진창의 버그들이 기다리고 있다. 그러나 전역 스코프가 실질적으로 모든 JS 애플리케이션에 중요한 *접착제*라는 사실도 부인할 수 없다.

## 전역 스코프는 정확히 어디에 존재하는가?

전역 스코프가 파일의 가장 바깥쪽 부분, 즉 함수나 다른 블록 내부에 있지 않는 것 처럼 보일 수 있다. 하지만 그렇게 간단하지는 않다.

JS 환경마다 프로그램의 스코프, 특히 전역 스코프를 다르게 처리한다. JS 개발자가 자신도 모르는 사이에 잘못된 개념을 가지고 있는 것은 매우 흔한 일이다.

### 브라우저 "Window" 

전역 스코프 처리와 관련하여, JS를 실행할 수 있는 가장 *순수한(pure)* 환경은 브라우저의 웹 페이지 환경에 로드된 독립 실행형 .js 파일이다. 자동으로 아무것도 추가되지 않는다는 의미에서 "순수(pure)"하다고 말한 것은 아니다-많은 항목이 추가될 수 있다! 그보다는 코드에 대한 최소한의 침입이나 예상되는 전역 스코프 동작에 대한 간섭을 나타내기 위해서다.

아래의 .js file을 살펴보자:

```js
var studentName = "Kyle";

function hello() {
    console.log(`Hello, ${ studentName }!`);
}

hello();
// Hello, Kyle!
```

이 코드는 인라인 '<script> 태그나, 마크업에 있는 스크립트 태그 '<script src=...>' 또는 동적으로 생성되는 `<script>` DOM 엘리먼트를 통해 로드될 수 있다. 이 세 가지 경우 모두에서, `studentName`과 `hello` 식별자는 전역 스코프에서 선언된다.

즉, 전역 객체(일반적으로 브라우저의 `window`)에 접근하면 거기에서 같은 이름을 가진 속성을 찾을 수 있다.

```js
var studentName = "Kyle";

function hello() {
    console.log(`Hello, ${ window.studentName }!`);
}

window.hello();
// Hello, Kyle!
```

이는 JS 스펙을 읽었을 때 예상되는 기본 동작이다. 바깥 스코프는 *전역 범위*이며 스펙에 따라 'studentName'은 전역 변수로 생성된다.

그게 바로 여기서 말하는 *순수*의 의미다. 그러나 안타깝게도 이러한 상황이 모든 JS 환경에 적용되는 것은 아니며, 그래서 JS 개발자들을 종종 놀라게 한다.

#### 전역을 섀도잉하는 전역

3장의 섀도잉(및 전역 언섀도잉)에 대한 내용을 떠올려보자. 하나의 변수 선언이 바깥 스코프에서 동일한 이름의 선언에 대한 접근을 재정의하고 차단할 수 있었다.

전역 변수와 동일한 이름의 전역 속성 간의 차이로 인해 발생하는 비정상적인 결과는 전역 스코프 자체 내에서 전역 객체 속성이 전역 변수에 의해 가려질 수 있다는 것이다.

```js
window.something = 42;

let something = "Kyle";

console.log(something);
// Kyle

console.log(window.something);
// 42
```

`let` 선언은 `something` 전역 변수를 추가하지 전역 객체 속성은 추가하지 않는다(3장 참조). 그 효과는  `something` 렉시컬 식별자가 `something` 전역 객체 속성을 섀도잉하는 것이다.

전역 객체와 전역 스코프 간의 차이를 만드는 것은 확실히 좋지 않은 생각이다. 코드를 읽는 사람들은 거의 틀림없이 실수할 것이다.

전역 선언을 통해 이러한 상황을 피할 수 있는 간단한 방법은 전역에 항상 'var'를 사용하는 것이다. 블록 스코프에서는 'let'과 'const'을 사용해라(6장의 "블록 스코프 지정" 참조).

#### DOM 전역

브라우저가 호스팅하는 JS 환경은 가장 *순수*한 전역 스코프 동작들을 가진다고 말했다. 그러나 완벽하게 *순수*한 것은 아니다.

브라우저 기반 JS  애플리케이션에서 발생할 수 있는 한 가지 놀라운 동작: 'id' 속성을 가진 DOM 요소는 이를 참조하는 전역 변수를 자동으로 생성한다.

아래 마크업을 살펴보자:    

```text
<ul id="my-todo-list">
   <li id="first">Write a book</li>
   ..
</ul>
```

그리고 그 페이지의 JS는 다음을 포함한다:    

```js
first;
// <li id="first">..</li>

window["my-todo-list"];
// <ul id="my-todo-list">..</ul>
```

`id` 값이 유효한 렉시컬 이름(`first`와 같이)이면 렉시컬 변수를 생성햔다. 그렇지 않은 경우 전역 객체를 통해서만 전역에 접근할 수 있다(`window[..]`).    

모든 `id`가 포함된 DOM 요소를 전역 변수로 자동 등록하는 것은 오래된 브라우저 레거시이며, 그럼에도 불구하고 많은 오래된 사이트들이 여전히 이 요소에 의존하고 있기 때문에 유지되어야 한다. 이러한 전역 변수가 생성되어있더라도 절대 사용하지 않는 것이 좋다.    

#### (Window) Name 안에는 무엇이 있나?

브라우저 기반 JS의 또 다른 전역 특이성:

```js
var name = 42;

console.log(name, typeof name);
// "42" string
```

`window.name`은 브라우저 컨텍스트에서 미리 정의된 "전역"이며 전역 객체의 속성이기 때문에 일반적인 전역 변수처럼 보인다.(그러나 "일반"이 아니다).
    
선언에 'var'를 사용했는데, 사전 정의된 `name` 전역 속성을 ** 가리지 ** 않는다. 즉, 'var' 선언은 무시된다. 해당 이름의 전역 스코프 객체 속성이 이미 있기 때문이다. 앞서 논의했듯이 `let name`을 사용했다면 `window.name`에 별도의 전역 `name` 변수로 객체 속성을 가렸을 것이다.

그런데 정말 놀라운 동작은 `42`라는 숫자를 `name`(따라서 `window.name`)에 붙였음에도 불구하고 그 값을 회수하면 `"42"`라는 문자열이 붙는다는 것이다. 이는 `window` 객체에 미리 정의된 getter/setter가 그 값을 string으로 만들기 때문이다. 으악!    
    
DOM 엘리먼트 ID 및 `window.name`과 같은 일부 드문 경우를 제외하고, 브라우저 페이지에서 독립 실행형 파일로 실행되는 JS는 가장 *완전한* 전역 스코프 동작을 수행한다.    

### 웹 워커

웹 워커는 기본 JS 프로그램을 실행하는 스레드와 완전히 다른 스레드(운영 체제 기준)에서 JS 파일을 실행할 수 있는 브라우저-JS 동작 위에 있는 웹 플랫폼 익스텐션이다.    

이러한 웹 워커 프로그램은 별도의 스레드에서 실행되기 때문에 애플리케이션의 메인 스레드와의 통신에서 제한되어 경쟁 상태 및 기타 문제를 방지/제한한다. 예를 들어 웹 워커 코드는 DOM에 대한 접근 권한이 없다. 그러나 `navigator`와 같이 일부 웹 API는 워커가 사용할 수 있다.    

웹 워커는 완전히 별개의 프로그램으로 취급되기 때문에 메인 JS 프로그램과 전역 스코프를 공유하지 않는다. 그러나 브라우저의 JS 엔진에서 여전히 코드가 실행 중이므로 전역 스코프 동작에 대해 유사한 *순수함*을 기대할 수 있다. DOM 접근이 없기 때문에 전역 스코프에 대한 `window` 별칭이 없다.    

웹 워커에서 전역 객체 참조는 일반적으로 'self'를 사용하여 이루어진다:  

```js
var studentName = "Kyle";
let studentID = 42;

function hello() {
    console.log(`Hello, ${ self.studentName }!`);
}

self.hello();
// Hello, Kyle!

self.studentID;
// undefined
```

주요 JS 프로그램과 마찬가지로 'var' 및 'function' 선언은 글로벌 객체(일명 `self`)에 미러링된 속성을 생성한다. 다른 선언들(`let`, etc)은 그렇게 하지 못한다.   

여기서 확인한 전역 스코프 동작은 JS 프로그램을 실행할 때 만큼 *순수*하다: 문제를 해결할 DOM이 없기 때문에 더 *순수*할 수 있다.

### 개발자 도구 Console/REPL    

1장에서 개발자 도구가 완전히 종속된 JS 환경을 만들지 않는다는 점을 기억해라. 이들은 JS 코드를 처리하지만, 또한 UX 인터렉션이 개발자에게 가장 친숙하도록(일명 개발자 경험 또는 DX)하는 것을 지향한다.    

경우에 따라 전체 JS 프로그램을 처리하는 데 필요한 일반적인 엄격한 단계보다 짧은 JS 스니펫으로 입력할 때 DX를 선호하면 프로그램과 도구 간에 코드 동작에서 관찰 가능한 차이가 발생한다. 예를 들어, 개발 도구에 코드를 입력할 때 JS 프로그램에 적용되는 특정 오류 조건이 완화되어 표시되지 않을 수 있다.    

위에서 말한 스코프와 관련하여 관찰할 수 있는 동작의 차이는 다음과 같다.    

* 전역 스코프의 동작    

* 호이스팅 (5장 참고)

* 가장 바깥 스코프에서 사용될 때 Block-scoping 선언 (`let` / `const`, 6장 참고)

console/REPL을 사용할 때 가장 바깥쪽 스코프의 입력문이 실제 전역 스코프위에서 처리되고 있는 것처럼 보일 수 있지만, 이는 정확하지는 않다. 이러한 툴은 일반적으로 엄격한 준수가 아닌 에뮬레이션의 전역 스코프 위치를 어느 정도 에뮬레이션한다. 이러한 툴 환경은 개발자 편의성을 우선시하며, 이는 (스코프에 대한 현재 논의와 같이) 관찰된 동작이 JS 스펙에서 벗어날 수 있음을 의미한다.    

단점은 개발자 도구는 다양한 개발자 활동에 편리하고 유용하도록 최적화되었지만 실제 JS 프로그램 컨텍스트의 명시적이고 미묘한 동작을 확인하기에 적합한 환경은 *아니라는* 점이다.    

### ES Modules (ESM)

ES6는 모듈 패턴에 대한 일급 객체 지원을 도입했다(8장에서 다룬다). ESM 사용의 가장 분명한 영향 중 하나는 파일에서 관측 가능한 최상위 스코프의 동작을 변경하는 방법이다.    

이전 버전의 이 코드 스니펫을 기억해봐라(`export` 키워드를 사용하여 ESM 형식으로 조정할 예정이다):    

```js
var studentName = "Kyle";

function hello() {
    console.log(`Hello, ${ studentName }!`);
}

hello();
// Hello, Kyle!

export hello;
```

ES 모듈로 로드된 파일에 해당 코드가 있으면 그대로 실행할 것이다. 그러나 전체적인 적용 관점에서 관측 가능한 효과는 다를 것이다.
    
(모듈) 파일의 최상위 수준에서 선언되더라도 가장 바깥쪽의 명백한 스코프의 `studentName`과 `hello`는 전역 스코프가 아니다. 대신 모듈 범위<sub>module-wide</sub>의 또는 "module-global"이다.    

그러나 모듈에는 선언이 비모듈 JS 파일의 최상위 수준에 나타나는 것처럼 이러한 최상위 선언을 속성으로 추가할 내포된 "모듈 범위 스코프 객체"가 없다. 전역 변수가 존재할 수 없거나 이러한 프로그램에 접근할 수 없다는 뜻은 아니다. 단지 모듈의 최상위 범위에서 변수를 선언한다고 전역 변수가 *생성*되는 것은 아니다.    

모듈의 최상위 스코프는 전역 스코프에서 내려온 것으로, 모듈의 전체 내용이 함수에 싸여 있는 것처럼 보인다. 따라서 전역 스코프에 존재하는 모든 변수(전역 객체에 있든 없든)는 모듈의 범위 내에서 렉시컬 식별자로 사용할 수 있다.    

ESM은 현재 모듈이 작동하는 데 필요한 모든 모듈을 가져올 수 있는 전역 스코프에 대한 의존도를 최소화할 것을 권장한다. 따라서 전역 스코프 또는 전역 객체의 사용 빈도가 낮아진다.    

그러나 앞에서 언급한 바와 같이 대다수의 JS와 웹 전역은 여전히 전역 스코프에서 계속 접근할 수 있다.    

### Node

One aspect of Node that often catches JS developers off-guard is that Node treats every single .js file that it loads, including the main one you start the Node process with, as a *module* (ES module or CommonJS module, see Chapter 8). The practical effect is that the top level of your Node programs **is never actually the global scope**, the way it is when loading a non-module file in the browser.
JS 개발자의 허를 찌르는 Node의 한 가지 측면은 Node 프로세스를 시작하는 기본 파일을 포함하여 로드되는 모든 .js 파일을 *module*(ES 또는 CommonJS 모듈, 8장 참조)로 처리한다는 것이다. 실제적인 효과는 브라우저에서 모듈이 아닌 파일을 로드할 때처럼 Node 프로그램의 최상위 레벨이 실제로는 **전역 스코프**가 아니라는 것이다.    

As of time of this writing, Node has recently added support for ES modules. But additionally, Node has from its beginning supported a module format referred to as "CommonJS", which looks like this:
Node의는 최근 ES 모듈 지원을 추가했다. 그러나 Node는 처음부터 "CommonJS"라고 하는 모듈 형식을 지원했으며, 다음과 같다.

```js
var studentName = "Kyle";

function hello() {
    console.log(`Hello, ${ studentName }!`);
}

hello();
// Hello, Kyle!

module.exports.hello = hello;
```

Before processing, Node effectively wraps such code in a function, so that the `var` and `function` declarations are contained in that wrapping function's scope, **not** treated as global variables.
프로세싱 전에, Node는 이러한 코드를 함수에 효과적으로 감싸서 'var' 및 'function' 선언이 전역 변수로 처리되지 않는 함수의 범위에 포함되도록 한다.    

Envision the preceding code as being seen by Node as this (illustrative, not actual):
Node에서 이전 코드는 다음과 같은 모습을 가진다(실제가 아니라 설명을 위한 것이다):    

```js
function Module(module,require,__dirname,...) {
    var studentName = "Kyle";

    function hello() {
        console.log(`Hello, ${ studentName }!`);
    }

    hello();
    // Hello, Kyle!

    module.exports.hello = hello;
}
```

Node then essentially invokes the added `Module(..)` function to run your module. You can clearly see here why `studentName` and `hello` identifiers are not global, but rather declared in the module scope.
그런 다음 Node는 기본적으로 추가된 `Module(..)` 기능을 호출하여 모듈을 실행한다. `studentName`과 `hello` 식별자가 전역이 아닌 모듈 스코프에서 선언되는 이유를 여기서 명확히 알 수 있다.    

As noted earlier, Node defines a number of "globals" like `require()`, but they're not actually identifiers in the global scope (nor properties of the global object). They're injected in the scope of every module, essentially a bit like the parameters listed in the `Module(..)` function declaration.
앞에서 설명한 것처럼 Node는 'require()'와 같은 여러 "전역"을 정의하지만 실제로는 전역 스코프(전역 객체의 속성도 아님)에서 식별자가 아니다. 이것들은 모든 모듈의 스코프에 주입됩니다. 기본적으로 `Module(..)` 함수 선언에 나열된 매개 변수와 약간 유사하다.    

So how do you define actual global variables in Node? The only way to do so is to add properties to another of Node's automatically provided "globals," which is ironically called `global`. `global` is a reference to the real global scope object, somewhat like using `window` in a browser JS environment.
그러면 Node의 실제 전역 변수는 어떻게 정의해야 할까? 유일한 방법은 자동으로 제공되는 Node 중 다른 "전역"에 속성을 추가하는 것인데, 이는 아이러니하게도 `global`이라고 불린다. `global`은 브라우저 JS 환경에서 `window`를 사용하는 것처럼 실제 전역 스코프 객체를 가리키는 말이다.    

Consider:
살펴보자:
    
```js
global.studentName = "Kyle";

function hello() {
    console.log(`Hello, ${ studentName }!`);
}

hello();
// Hello, Kyle!

module.exports.hello = hello;
```

Here we add `studentName` as a property on the `global` object, and then in the `console.log(..)` statement we're able to access `studentName` as a normal global variable.
여기서 `studentName`을 `global` 객체의 속성으로 추가한 다음 `console.log(..)`에서 정상적인 전역 변수로 '`studentName`에 접근할 수 있다.    

Remember, the identifier `global` is not defined by JS; it's specifically defined by Node.
식별자 `global`은 JS에 의해 정의되지 않으며 Node에 의해 특별히 정의된다.    

## Global This

Reviewing the JS environments we've looked at so far, a program may or may not:
지금까지 살펴본 JS 환경을 살펴보면, 프로그램은 거의 다음과 같이 동작한다.   

* Declare a global variable in the top-level scope with `var` or `function` declarations—or `let`, `const`, and `class`.
* `var` 또는 `function` 선언으로 최상위 범위에 글로벌 변수를 선언하거나, 'let', 'const', 'class'로 선언합니다.

* Also add global variables declarations as properties of the global scope object if `var` or `function` are used for the declaration.

* Refer to the global scope object (for adding or retrieving global variables, as properties) with `window`, `self`, or `global`.

I think it's fair to say that global scope access and behavior is more complicated than most developers assume, as the preceding sections have illustrated. But the complexity is never more obvious than in trying to nail down a universally applicable reference to the global scope object.

Yet another "trick" for obtaining a reference to the global scope object looks like:

```js
const theGlobalScopeObject =
    (new Function("return this"))();
```

| NOTE: |
| :--- |
| A function can be dynamically constructed from code stored in a string value with the `Function()` constructor, similar to `eval(..)` (see "Cheating: Runtime Scope Modifications" in Chapter 1). Such a function will automatically be run in non-strict-mode (for legacy reasons) when invoked with the normal `()` function invocation as shown; its `this` will point at the global object. See the third book in the series, *Objects & Classes*, for more information on determining `this` bindings. |

So, we have `window`, `self`, `global`, and this ugly `new Function(..)` trick. That's a lot of different ways to try to get at this global object. Each has its pros and cons.

Why not introduce yet another!?!?

As of ES2020, JS has finally defined a standardized reference to the global scope object, called `globalThis`. So, subject to the recency of the JS engines your code runs in, you can use `globalThis` in place of any of those other approaches.

We could even attempt to define a cross-environment polyfill that's safer across pre-`globalThis` JS environments, such as:

```js
const theGlobalScopeObject =
    (typeof globalThis != "undefined") ? globalThis :
    (typeof global != "undefined") ? global :
    (typeof window != "undefined") ? window :
    (typeof self != "undefined") ? self :
    (new Function("return this"))();
```

Phew! That's certainly not ideal, but it works if you find yourself needing a reliable global scope reference.

(The proposed name `globalThis` was fairly controversial while the feature was being added to JS. Specifically, I and many others felt the "this" reference in its name was misleading, since the reason you reference this object is to access to the global scope, never to access some sort of global/default `this` binding. There were many other names considered, but for a variety of reasons ruled out. Unfortunately, the name chosen ended up as a last resort. If you plan to interact with the global scope object in your programs, to reduce confusion, I strongly recommend choosing a better name, such as (the laughably long but accurate!) `theGlobalScopeObject` used here.)

## Globally Aware

The global scope is present and relevant in every JS program, even though modern patterns for organizing code into modules de-emphasizes much of the reliance on storing identifiers in that namespace.

Still, as our code proliferates more and more beyond the confines of the browser, it's especially important we have a solid grasp on the differences in how the global scope (and global scope object!) behave across different JS environments.

With the big picture of global scope now sharper in focus, the next chapter again descends into the deeper details of lexical scope, examining how and when variables can be used.
