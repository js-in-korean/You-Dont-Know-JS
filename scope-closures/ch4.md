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

일부 빌드 설정에서는 파일의 전체 내용이 감싸는 함수<sub>wrapper function</sub>, 범용 모듈(UMD—부록 A 참조) 등과 같은 하나의 닫힌 스코프(enclosing scope)로 감싸집니다. 각 부분은 해당 공유 스코프에서 로컬 변수를 사용하여 다른 부분에서 접근 할 수 있도록 스스로를 등록할 수 있다. 예를 들어, 다음과 같다.

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

마지막 세번째 방법으로, 번들러 도구가 애플리케이션에 사용되는 않거나, (ES 모듈이 아닌) 파일이 단순히 브라우저에 개별적으로 로드가 되지 않는('스크립트' 태그 또는 기타 동적 JS 리소스 로딩을 통해)것과 상관없이 이러한 모든 부분을 둘러싸는 하나의 스코프가 없으면  **전역 스코프*가 유일하게 연결할 수 있는 방법이다.

이러한 종류의 번들 파일은 종종 다음과 같은 코드를 가진다.

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

실행 시간 동안 애플리케이션의 코드가 존재하는 위치와 각 부분이 다른 부분에 접근하여 협력할 수 있는 방법을 설명하는 것 외에도, 전역 스코프는 다음에서도 찾아볼 수 있다:


* JS가 제공하는 빌트인<sub>built-ins</sub>:

    - primitives: `undefined`, `null`, `Infinity`, `NaN`
    - natives: `Date()`, `Object()`, `String()`, etc.
    - global functions: `eval()`, `parseInt()`, etc.
    - namespaces: `Math`, `Atomics`, `JSON`
    - friends of JS: `Intl`, `WebAssembly`

* JS을 호스팅하는 환경에서 제공하는 빌트인<sub>built-ins</sub>:

    - `console` (and its methods)
    - the DOM (`window`, `document`, etc)
    - timers (`setTimeout(..)`, etc)
    - web platform APIs: `navigator`, `history`, geolocation, WebRTC, etc.

이는 여러분의 프로그램이 상호작용할 수많은 *글로벌* 중 일부에 불과하다.

| : |
| :--- |
| Node도 "전역적으로" 여러 요소를 노출하지만 기술적으로는 `전역(global)` 스코프에 있지는 않다. : `require()`, `__dirname`, `module`, `URL`, 등등. |

대부분의 개발자는 전역 스코프가 단순히 애플리케이션의 모든 변수를 떠넘기는 장소가 되어서는 안 된다는 데 동의한다. 엉망진창의 버그들이 기다리고 있다. 그러나 전역 스코프가 실질적으로 모든 JS 애플리케이션에 중요한 *접착제*라는 사실도 부인할 수 없다.

## Where Exactly is this Global Scope?

It might seem obvious that the global scope is located in the outermost portion of a file; that is, not inside any function or other block. But it's not quite as simple as that.

Different JS environments handle the scopes of your programs, especially the global scope, differently. It's quite common for JS developers to harbor misconceptions without even realizing it.

### Browser "Window"

With respect to treatment of the global scope, the most *pure* environment JS can be run in is as a standalone .js file loaded in a web page environment in a browser. I don't mean "pure" as in nothing automatically added—lots may be added!—but rather in terms of minimal intrusion on the code or interference with its expected global scope behavior.

Consider this .js file:

```js
var studentName = "Kyle";

function hello() {
    console.log(`Hello, ${ studentName }!`);
}

hello();
// Hello, Kyle!
```

This code may be loaded in a web page environment using an inline `<script>` tag, a `<script src=..>` script tag in the markup, or even a dynamically created `<script>` DOM element. In all three cases, the `studentName` and `hello` identifiers are declared in the global scope.

That means if you access the global object (commonly, `window` in the browser), you'll find properties of those same names there:

```js
var studentName = "Kyle";

function hello() {
    console.log(`Hello, ${ window.studentName }!`);
}

window.hello();
// Hello, Kyle!
```

That's the default behavior one would expect from a reading of the JS specification: the outer scope *is* the global scope and `studentName` is legitimately created as global variable.

That's what I mean by *pure*. But unfortunately, that won't always be true of all JS environments you encounter, and that's often surprising to JS developers.

#### Globals Shadowing Globals

Recall the discussion of shadowing (and global unshadowing) from Chapter 3, where one variable declaration can override and prevent access to a declaration of the same name from an outer scope.

An unusual consequence of the difference between a global variable and a global property of the same name is that, within just the global scope itself, a global object property can be shadowed by a global variable:

```js
window.something = 42;

let something = "Kyle";

console.log(something);
// Kyle

console.log(window.something);
// 42
```

The `let` declaration adds a `something` global variable but not a global object property (see Chapter 3). The effect then is that the `something` lexical identifier shadows the `something` global object property.

It's almost certainly a bad idea to create a divergence between the global object and the global scope. Readers of your code will almost certainly be tripped up.

A simple way to avoid this gotcha with global declarations: always use `var` for globals. Reserve `let` and `const` for block scopes (see "Scoping with Blocks" in Chapter 6).

#### DOM Globals

I asserted that a browser-hosted JS environment has the most *pure* global scope behavior we'll see. However, it's not entirely *pure*.

One surprising behavior in the global scope you may encounter with browser-based JS applications: a DOM element with an `id` attribute automatically creates a global variable that references it.

Consider this markup:

```text
<ul id="my-todo-list">
   <li id="first">Write a book</li>
   ..
</ul>
```

And the JS for that page could include:

```js
first;
// <li id="first">..</li>

window["my-todo-list"];
// <ul id="my-todo-list">..</ul>
```

If the `id` value is a valid lexical name (like `first`), the lexical variable is created. If not, the only way to access that global is through the global object (`window[..]`).

The auto-registration of all `id`-bearing DOM elements as global variables is an old legacy browser behavior that nevertheless must remain because so many old sites still rely on it. My advice is never to use these global variables, even though they will always be silently created.

#### What's in a (Window) Name?

Another global scope oddity in browser-based JS:

```js
var name = 42;

console.log(name, typeof name);
// "42" string
```

`window.name` is a pre-defined "global" in a browser context; it's a property on the global object, so it seems like a normal global variable (yet it's anything but "normal").

We used `var` for our declaration, which **does not** shadow the pre-defined `name` global property. That means, effectively, the `var` declaration is ignored, since there's already a global scope object property of that name. As we discussed earlier, had we used `let name`, we would have shadowed `window.name` with a separate global `name` variable.

But the truly surprising behavior is that even though we assigned the number `42` to `name` (and thus `window.name`), when we then retrieve its value, it's a string `"42"`! In this case, the weirdness is because `name` is actually a pre-defined getter/setter on the `window` object, which insists on its value being a string value. Yikes!

With the exception of some rare corner cases like DOM element ID's and `window.name`, JS running as a standalone file in a browser page has some of the most *pure* global scope behavior we will encounter.

### Web Workers

Web Workers are a web platform extension on top of browser-JS behavior, which allows a JS file to run in a completely separate thread (operating system wise) from the thread that's running the main JS program.

Since these Web Worker programs run on a separate thread, they're restricted in their communications with the main application thread, to avoid/limit race conditions and other complications. Web Worker code does not have access to the DOM, for example. Some web APIs are, however, made available to the worker, such as `navigator`.

Since a Web Worker is treated as a wholly separate program, it does not share the global scope with the main JS program. However, the browser's JS engine is still running the code, so we can expect similar *purity* of its global scope behavior. Since there is no DOM access, the `window` alias for the global scope doesn't exist.

In a Web Worker, the global object reference is typically made using `self`:

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

Just as with main JS programs, `var` and `function` declarations create mirrored properties on the global object (aka, `self`), where other declarations (`let`, etc) do not.

So again, the global scope behavior we're seeing here is about as *pure* as it gets for running JS programs; perhaps it's even more *pure* since there's no DOM to muck things up!

### Developer Tools Console/REPL

Recall from Chapter 1 in *Get Started* that Developer Tools don't create a completely adherent JS environment. They do process JS code, but they also lean in favor of the UX interaction being most friendly to developers (aka, developer experience, or DX).

In some cases, favoring DX when typing in short JS snippets, over the normal strict steps expected for processing a full JS program, produces observable differences in code behavior between programs and tools. For example, certain error conditions applicable to a JS program may be relaxed and not displayed when the code is entered into a developer tool.

With respect to our discussions here about scope, such observable differences in behavior may include:

* The behavior of the global scope

* Hoisting (see Chapter 5)

* Block-scoping declarators (`let` / `const`, see Chapter 6) when used in the outermost scope

Although it might seem, while using the console/REPL, that statements entered in the outermost scope are being processed in the real global scope, that's not quite accurate. Such tools typically emulate the global scope position to an extent; it's emulation, not strict adherence. These tool environments prioritize developer convenience, which means that at times (such as with our current discussions regarding scope), observed behavior may deviate from the JS specification.

The take-away is that Developer Tools, while optimized to be convenient and useful for a variety of developer activities, are **not** suitable environments to determine or verify explicit and nuanced behaviors of an actual JS program context.

### ES Modules (ESM)

ES6 introduced first-class support for the module pattern (covered in Chapter 8). One of the most obvious impacts of using ESM is how it changes the behavior of the observably top-level scope in a file.

Recall this code snippet from earlier (which we'll adjust to ESM format by using the `export` keyword):

```js
var studentName = "Kyle";

function hello() {
    console.log(`Hello, ${ studentName }!`);
}

hello();
// Hello, Kyle!

export hello;
```

If that code is in a file that's loaded as an ES module, it will still run exactly the same. However, the observable effects, from the overall application perspective, will be different.

Despite being declared at the top level of the (module) file, in the outermost obvious scope, `studentName` and `hello` are not global variables. Instead, they are module-wide, or if you prefer, "module-global."

However, in a module there's no implicit "module-wide scope object" for these top-level declarations to be added to as properties, as there is when declarations appear in the top-level of non-module JS files. This is not to say that global variables cannot exist or be accessed in such programs. It's just that global variables don't get *created* by declaring variables in the top-level scope of a module.

The module's top-level scope is descended from the global scope, almost as if the entire contents of the module were wrapped in a function. Thus, all variables that exist in the global scope (whether they're on the global object or not!) are available as lexical identifiers from inside the module's scope.

ESM encourages a minimization of reliance on the global scope, where you import whatever modules you may need for the current module to operate. As such, you less often see usage of the global scope or its global object.

However, as noted earlier, there are still plenty of JS and web globals that you will continue to access from the global scope, whether you realize it or not!

### Node

One aspect of Node that often catches JS developers off-guard is that Node treats every single .js file that it loads, including the main one you start the Node process with, as a *module* (ES module or CommonJS module, see Chapter 8). The practical effect is that the top level of your Node programs **is never actually the global scope**, the way it is when loading a non-module file in the browser.

As of time of this writing, Node has recently added support for ES modules. But additionally, Node has from its beginning supported a module format referred to as "CommonJS", which looks like this:

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

Envision the preceding code as being seen by Node as this (illustrative, not actual):

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

As noted earlier, Node defines a number of "globals" like `require()`, but they're not actually identifiers in the global scope (nor properties of the global object). They're injected in the scope of every module, essentially a bit like the parameters listed in the `Module(..)` function declaration.

So how do you define actual global variables in Node? The only way to do so is to add properties to another of Node's automatically provided "globals," which is ironically called `global`. `global` is a reference to the real global scope object, somewhat like using `window` in a browser JS environment.

Consider:

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

Remember, the identifier `global` is not defined by JS; it's specifically defined by Node.

## Global This

Reviewing the JS environments we've looked at so far, a program may or may not:

* Declare a global variable in the top-level scope with `var` or `function` declarations—or `let`, `const`, and `class`.

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
