# You Don't Know JS Yet: Scope & Closures - 2nd Edition
# 8장: 모듈 패턴

이 장에서 우리는 모든 프로그래밍에서 가장 중요한 코드 구성 패턴의 하나인 모듈을 살펴봄으로써 책의 본문을 마무리한다. 알다시피 모듈은 앞에서 다룬 렉시컬 스코프<sub>lexical scope<sub>와 클로저<sub>closure</sub>를 학습한 노력에 대한 성과에 본질적으로 구축된다.

전역 스코프의 범위부터 중첩 블록 스코프를 거쳐 변수 생명 주기의 복잡성에 이르기 까지 렉시컬 스코프의 모든 것을 조사했다. 그리고 클로저의 모든 힘을 이해하기 위해 렉시컬 스코프를 활용했다.

잠시 시간을 내어 얼마나 멀리 이 여정까지 온 건지 되돌아 보자. JS를 더 깊이 알기 위해 큰 발걸음을 내딛었다.

이 책의 중심 테마는 스코프와 클로저를 이해하고 마스터 하는 것이, 특히 변수로 정보를 어디에 저장할지 결정하는 코드를 적절하게 구성하는 데 핵심이라는 것이었다.

이 마지막 장의 목표는 프로그램을 만드는 데 있어 추상적인 개념으로 부터 실용적인 개선으로 끌어올리면서, 모듈이 이러한 주제의 중요성을 구현하는 방법을 이해하는 것이다.

## 캡슐화 및 최소 노출 (POLE)

캡슐화는 객체지향 프로그래밍의 원칙으로 자주 인용된다. 그러나 그 보다 더 근본적이고 넓게 적용된다. 캡슐화의 목표는 공통의 목적을 함께 제공하는 정보(데이터)와 행위(함수)의 결합 또는 공동 배치<sub>co-location</sub>이다.

어떤 문법이나 코드 구조의 독립은, 캡슐화의 정신은 공통의 목적을 가진 전체 프로그램의 조각을 묶기 위해 별도의 파일을 사용하는 것으로 간단하게 실현될 수 있다. 만약 검색 결과 목록을 동작시키는 모든 것을 "search-list.js"라는 단일 파일로 묶는다면, 프로그램의 일부분을 캡슐화하고 있는 것이다.

어플리케이션을 구성하기 위한 현대 프론트엔드 프로그램의 최신 트렌드는 컴포넌트 아키텍처를 중심으로 캡슐화로 더욱 나아가고 있다. 대다수가 검색 결과 목록을 구성하는 모든 것(코드를 비롯한 보여지는 마크업과 스타일을 포함하여)을 상호작용할 수 있는 실재하는 어떤 하나의 프로그램 로직 단위로 통합하는 것이 자연스럽다고 느낀다. 그런 다음 "SearchList" 컴포넌트 모음으로 지정한다.

또 다른 핵심 목표는 캡슐화된 데이터와 기능의 특정 측면에 대한 가시성을 제어하는 것이다. 6장에서 스코프 초과 노출의 여러 *위험*을 방어적으로 보호하기 위해 추가하는 *최소 노출<sub>least exposure</sub>* 원칙(POLE)을 상기하자. 변수와 함수 모두에 영향을 미친다. JS에서는 거의 대부분 렉시컬 스코프 기법으로 가시성 제어를 구현한다.

이 아이디어는 유사한 프로그램 조각을 함께 그룹화하고 *비공개* 세부 사항으로 고려되는 부분에 대한 프로그램의 접근을 선택적으로 제한한다. *비공개*로 고려되지 않는 것은 *공개*로 인식되며, 전체 프로그램에서 접근 가능하다.

이러한 노력의 자연적 효과는 더 나은 코드 구성이다. 깨끗하고 분명한 경계와 연결점을 가지고, 어디에 있는지 알 때 소프트웨어를 만들고 유지하는 것이 더 쉽다.

JS 프로그램을 모듈로 구성하는 것에는 몇 가지 주요 이점이 있다.

## 모듈은 무엇인가?

모듈은 연관된 데이터와 함수(이 문맥에서는 흔히 메서드로 언급)의 모음이며, 숨겨진 *비공개* 세부 사항과 *공개* 접근 가능한 세부 사항 사이의 구분이 특징이며, 보통 "공개 API"라고 부른다.

모듈은 또한 상태를 가진다. 시간이 지나도 어떤 정보를, 그 정보에 접근하고 수정하는 기능과 마찬가지로 유지한다.

| 비고: |
| :--- |
| 모듈 패턴의 광범위한 관심사는 느슨한 결합과 기타 프로그램 아키텍처 기술을 통한 시스템 수준 모듈화를 완전히 수용하는 것이다. |

모듈이 무엇인지 더 잘 이해하기 위해서 일부 모듈 특징을 모듈이 아닌 쓸만한 코드 패턴과 비교해보자.

### 네임스페이스 (상태가 없는 그룹화)

데이터 없이 연관된 함수의 집합을 함께 모은다면 모듈이 암시하는 예상 캡슐화는 정말로 가질 수 없다. *상태가 없는* 함수의 그룹화에 더 적합한 용어는 네이스페이스이다.

```js
// 네임스페이스, 모듈이 아님
var Utils = {
    cancelEvt(evt) {
        evt.preventDefault();
        evt.stopPropagation();
        evt.stopImmediatePropagation();
    },
    wait(ms) {
        return new Promise(function c(res){
            setTimeout(res,ms);
        });
    },
    isValidEmail(email) {
        return /[^@]+@[^@.]+\.[^@.]+/.test(email);
    }
};
```

여기서 `Utils`는 유용한 유틸리티 모음이지만 모두 상태 독립적인 함수이다. 기능을 함께 모으는 것은 일반적으로 좋은 방법이지만 모듈이 되지 않는다. 대신 `Utils` 네임스페이스를 정의했고 그 아래에 함수를 구성했다.

### 자료 구조 (상태가 있는 그룹화)

데이터와 상태가 있는 함수를 함께 묶어도 이것의 가시성을 제한하고 있지 않는다면 캡슐화의 POLE 측면에는 미치지 못한다. 모듈로 이름 붙이기에는 그다지 도움이 되지 않는다.

아래를 보자.

```js
// 자료 구조, 모듈이 아님
var Student = {
    records: [
        { id: 14, name: "Kyle", grade: 86 },
        { id: 73, name: "Suzy", grade: 87 },
        { id: 112, name: "Frank", grade: 75 },
        { id: 6, name: "Sarah", grade: 91 }
    ],
    getName(studentID) {
        var student = this.records.find(
            student => student.id == studentID
        );
        return student.name;
    }
};

Student.getName(73);
// Suzy
```

`records`는 공개적으로 접근 가능한 데이터이고 공개 API 뒤로 숨겨져 있지 않기 때문에 `Student`는 실제로 모듈이 아니다.

`Student`는 캡슐화의 데이터 및 기능 측면을 가진다. 하지만 가시성 제어 측면은 가지고 있지 않다. 자료 구조의 인스턴스로 이름 붙이는 것이 최선이다.

### 모듈 (상태가 있는 접근 제어)

모듈 패턴의 완전한 정신을 구현하기 위해서는 그룹화 및 상태뿐만 아니라 가시성(비공개 vs 공개)을 통한 접근 제어가 필요하다.

이전 섹션의 `Student`를 모듈로 바꿔보자. 내가 "클래식 모듈"이라 부르는 형태부터 시작할 것이다. 2000년대 초에 처음 등장했을 때는 원래 "리빌링<sub>revealing</sub> 모듈"로 불렸다. 아래를 보자.

```js
var Student = (function defineStudent(){
    var records = [
        { id: 14, name: "Kyle", grade: 86 },
        { id: 73, name: "Suzy", grade: 87 },
        { id: 112, name: "Frank", grade: 75 },
        { id: 6, name: "Sarah", grade: 91 }
    ];

    var publicAPI = {
        getName
    };

    return publicAPI;

    // ************************

    function getName(studentID) {
        var student = records.find(
            student => student.id == studentID
        );
        return student.name;
    }
})();

Student.getName(73);   // Suzy
```

`Student`는 지금 모듈의 인스턴스이다. 단일 메서드 `getName(..)`을 가진 공개 API가 특징이다. 이 메서드는 비공개로 숨겨진 `records` 데이터에 접근할 수 있다.

| 경고: |
| :--- |
| 이 모듈 정의에서 코드로 넣어진 명시적인 학생 데이터는 단지 우리의 예시 목적을 위한 것임을 지적하고 싶다. 당신의 프로그램에서 보통의 모듈은 이 데이터를 바깥의 출처에서 받을 것이다. 일반적으로 데이터베이스, JSON 데이터 파일, Ajax 호출 등이다. 그런 다음 데이터는 일반적으로 모듈의 공개 API로 된 메서드를 통하여 모듈 인스턴스에 주입된다.  |

클래식 모듈 형식은 어떻게 동작하는가?

모듈의 인스턴스는 실행되는 `defineStudent()` IIFE에 의해 생성된다는 것을 주목해라. 이 IIFE는 내부 `getName(..)` 함수를 참조하는 속성이 있는 객체(`publicAPI`)를 반환한다.

`publicAPI` 객체로 이름 지은 것은 내가 선호하는 스타일이다. 이 객체는 당신이 좋아하는 어떤 이름이든 될 수 있다(JS는 신경쓰지 않는다). 또는 내부의 이름 있는 변수에 할당하지 않고 객체를 그냥 반환할 수 있다. 이 선택에 대한 자세한 내용은 부록 A에 있다.

외부에서 `Student.getName(..)`은 이 노출된 내부 함수를 호출하여, 클로저를 통해 내부 `records` 변수에 대한 접근을 유지한다.

속성 중 하나로 함수를 가지는 객체를 반환해서는 *안된다*. 그냥 객체 대신 함수를 직접 반환할 수 있다. 클랙식 모듈의 핵심 부분을 여전히 만족한다.
> 뭔 말인지...

렉시컬 스코프<sub>lexical scope</sub> 작동 방식에 따라 외부 모듈 정의 함수 안에서 변수와 함수를 정의하는 것은 모든 것을 *기본적으로* 비공개로 만든다. 오직 그 함수로 반환된 공개 API 객체에 추가된 속성만이 외부의 공개적인 사용을 위해 내보내진다.

IIFE의 사용은 프로그램이 보통 "싱글톤"이라 불리는 모듈의 단일 중앙 인스턴스를 필요로 한다는 것을 의미한다. 실제로, 이 구체적인 예제는 `Student` 모듈의 인스턴스 이상의 것이 필요할 이유가 없을 정도로 충분히 간단하다.

#### Module Factory (Multiple Instances)

But if we did want to define a module that supported multiple instances in our program, we can slightly tweak the code:

```js
// factory function, not singleton IIFE
function defineStudent() {
    var records = [
        { id: 14, name: "Kyle", grade: 86 },
        { id: 73, name: "Suzy", grade: 87 },
        { id: 112, name: "Frank", grade: 75 },
        { id: 6, name: "Sarah", grade: 91 }
    ];

    var publicAPI = {
        getName
    };

    return publicAPI;

    // ************************

    function getName(studentID) {
        var student = records.find(
            student => student.id == studentID
        );
        return student.name;
    }
}

var fullTime = defineStudent();
fullTime.getName(73);            // Suzy
```

Rather than specifying `defineStudent()` as an IIFE, we just define it as a normal standalone function, which is commonly referred to in this context as a "module factory" function.

We then call the module factory, producing an instance of the module that we label `fullTime`. This module instance implies a new instance of the inner scope, and thus a new closure that `getName(..)` holds over `records`. `fullTime.getName(..)` now invokes the method on that specific instance.

#### Classic Module Definition

So to clarify what makes something a classic module:

* There must be an outer scope, typically from a module factory function running at least once.

* The module's inner scope must have at least one piece of hidden information that represents state for the module.

* The module must return on its public API a reference to at least one function that has closure over the hidden module state (so that this state is actually preserved).

You'll likely run across other variations on this classic module approach, which we'll look at in more detail in Appendix A.

## Node CommonJS Modules

In Chapter 4, we introduced the CommonJS module format used by Node. Unlike the classic module format described earlier, where you could bundle the module factory or IIFE alongside any other code including other modules, CommonJS modules are file-based; one module per file.

Let's tweak our module example to adhere to that format:

```js
module.exports.getName = getName;

// ************************

var records = [
    { id: 14, name: "Kyle", grade: 86 },
    { id: 73, name: "Suzy", grade: 87 },
    { id: 112, name: "Frank", grade: 75 },
    { id: 6, name: "Sarah", grade: 91 }
];

function getName(studentID) {
    var student = records.find(
        student => student.id == studentID
    );
    return student.name;
}
```

The `records` and `getName` identifiers are in the top-level scope of this module, but that's not the global scope (as explained in Chapter 4). As such, everything here is *by default* private to the module.

To expose something on the public API of a CommonJS module, you add a property to the empty object provided as `module.exports`. In some older legacy code, you may run across references to just a bare `exports`, but for code clarity you should always fully qualify that reference with the `module.` prefix.

For style purposes, I like to put my "exports" at the top and my module implementation at the bottom. But these exports can be placed anywhere. I strongly recommend collecting them all together, either at the top or bottom of your file.

Some developers have the habit of replacing the default exports object, like this:

```js
// defining a new object for the API
module.exports = {
    // ..exports..
};
```

There are some quirks with this approach, including unexpected behavior if multiple such modules circularly depend on each other. As such, I recommend against replacing the object. If you want to assign multiple exports at once, using object literal style definition, you can do this instead:

```js
Object.assign(module.exports,{
   // .. exports ..
});
```

What's happening here is defining the `{ .. }` object literal with your module's public API specified, and then `Object.assign(..)` is performing a shallow copy of all those properties onto the existing `module.exports` object, instead of replacing it This is a nice balance of convenience and safer module behavior.

To include another module instance into your module/program, use Node's `require(..)` method. Assuming this module is located at "/path/to/student.js", this is how we can access it:

```js
var Student = require("/path/to/student.js");

Student.getName(73);
// Suzy
```

`Student` now references the public API of our example module.

CommonJS modules behave as singleton instances, similar to the IIFE module definition style presented before. No matter how many times you `require(..)` the same module, you just get additional references to the single shared module instance.

`require(..)` is an all-or-nothing mechanism; it includes a reference of the entire exposed public API of the module. To effectively access only part of the API, the typical approach looks like this:

```js
var getName = require("/path/to/student.js").getName;

// or alternately:

var { getName } = require("/path/to/student.js");
```

Similar to the classic module format, the publicly exported methods of a CommonJS module's API hold closures over the internal module details. That's how the module singleton state is maintained across the lifetime of your program.

| NOTE: |
| :--- |
| In Node `require("student")` statements, non-absolute paths (`"student"`) assume a ".js" file extension and search "node_modules". |

## Modern ES Modules (ESM)

The ESM format shares several similarities with the CommonJS format. ESM is file-based, and module instances are singletons, with everything private *by default*. One notable difference is that ESM files are assumed to be strict-mode, without needing a `"use strict"` pragma at the top. There's no way to define an ESM as non-strict-mode.

Instead of `module.exports` in CommonJS, ESM uses an `export` keyword to expose something on the public API of the module. The `import` keyword replaces the `require(..)` statement. Let's adjust "students.js" to use the ESM format:

```js
export { getName };

// ************************

var records = [
    { id: 14, name: "Kyle", grade: 86 },
    { id: 73, name: "Suzy", grade: 87 },
    { id: 112, name: "Frank", grade: 75 },
    { id: 6, name: "Sarah", grade: 91 }
];

function getName(studentID) {
    var student = records.find(
        student => student.id == studentID
    );
    return student.name;
}
```

The only change here is the `export { getName }` statement. As before, `export` statements can appear anywhere throughout the file, though `export` must be at the top-level scope; it cannot be inside any other block or function.

ESM offers a fair bit of variation on how the `export` statements can be specified. For example:

```js
export function getName(studentID) {
    // ..
}
```

Even though `export` appears before the `function` keyword here, this form is still a `function` declaration that also happens to be exported. That is, the `getName` identifier is *function hoisted* (see Chapter 5), so it's available throughout the whole scope of the module.

Another allowed variation:

```js
export default function getName(studentID) {
    // ..
}
```

This is a so-called "default export," which has different semantics from other exports. In essence, a "default export" is a shorthand for consumers of the module when they `import`, giving them a terser syntax when they only need this single default API member.

Non-`default` exports are referred to as "named exports."

The `import` keyword—like `export`, it must be used only at the top level of an ESM outside of any blocks or functions—also has a number of variations in syntax. The first is referred to as "named import":

```js
import { getName } from "/path/to/students.js";

getName(73);   // Suzy
```

As you can see, this form imports only the specifically named public API members from a module (skipping anything not named explicitly), and it adds those identifiers to the top-level scope of the current module. This type of import is a familiar style to those used to package imports in languages like Java.

Multiple API members can be listed inside the `{ .. }` set, separated with commas. A named import can also be *renamed* with the `as` keyword:

```js
import { getName as getStudentName }
   from "/path/to/students.js";

getStudentName(73);
// Suzy
```

If `getName` is a "default export" of the module, we can import it like this:

```js
import getName from "/path/to/students.js";

getName(73);   // Suzy
```

The only difference here is dropping the `{ }` around the import binding. If you want to mix a default import with other named imports:

```js
import { default as getName, /* .. others .. */ }
   from "/path/to/students.js";

getName(73);   // Suzy
```

By contrast, the other major variation on `import` is called "namespace import":

```js
import * as Student from "/path/to/students.js";

Student.getName(73);   // Suzy
```

As is likely obvious, the `*` imports everything exported to the API, default and named, and stores it all under the single namespace identifier as specified. This approach most closely matches the form of classic modules for most of JS's history.

| NOTE: |
| :--- |
| As of the time of this writing, modern browsers have supported ESM for a few years now, but Node's stable'ish support for ESM is fairly recent, and has been evolving for quite a while. The evolution is likely to continue for another year or more; the introduction of ESM to JS back in ES6 created a number of challenging compatibility concerns for Node's interop with CommonJS modules. Consult Node's ESM documentation for all the latest details: https://nodejs.org/api/esm.html |

## Exit Scope

Whether you use the classic module format (browser or Node), CommonJS format (in Node), or ESM format (browser or Node), modules are one of the most effective ways to structure and organize your program's functionality and data.

The module pattern is the conclusion of our journey in this book of learning how we can use the rules of lexical scope to place variables and functions in proper locations. POLE is the defensive *private by default* posture we always take, making sure we avoid over-exposure and interact only with the minimal public API surface area necessary.

And underneath modules, the *magic* of how all our module state is maintained is closures leveraging the lexical scope system.

That's it for the main text. Congratulations on quite a journey so far! As I've said numerous times throughout, it's a really good idea to pause, reflect, and practice what we've just discussed.

When you're comfortable and ready, check out the appendices, which dig deeper into some of the corners of these topics, and also challenge you with some practice exercises to solidify what you've learned.
