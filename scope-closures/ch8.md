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

어플리케이션을 구성하기 위한 현대 프론트엔드 프로그램의 최신 트렌드는 컴포넌트 아키텍처를 중심으로 캡슐화로 더욱 나아가고 있다. 대다수가 검색 결과 목록을 구성하는 모든 것(코드를 비롯한 보여지는 마크업과 스타일을 포함하여)을 상호작용할 수 있는 실재하는 어떤 하나의 프로그램 로직 단위로 통합하는 것이 자연스럽다고 느낀다. 그런 다음 "SearchList" 컴포넌트 컬렉션으로 지정한다.

또 다른 핵심 목표는 캡슐화된 데이터와 기능의 특정 측면에 대한 가시성을 제어하는 것이다. 6장에서 스코프 초과 노출의 여러 *위험*을 방어적으로 보호하기 위해 추가하는 *최소 노출<sub>least exposure</sub>* 원칙(POLE)을 상기하자. 변수와 함수 모두에 영향을 미친다. JS에서는 거의 대부분 렉시컬 스코프 기법으로 가시성 제어를 구현한다.

이 아이디어는 유사한 프로그램 조각을 함께 그룹화하고 *비공개* 세부 사항으로 고려되는 부분에 대한 프로그램의 접근을 선택적으로 제한한다. *비공개*로 고려되지 않는 것은 *공개*로 인식되며, 전체 프로그램에서 접근 가능하다.

이러한 노력의 자연적 효과는 더 나은 코드 구성이다. 깨끗하고 분명한 경계와 연결점을 가지고, 어디에 있는지 알 때 소프트웨어를 만들고 유지하는 것이 더 쉽다.

JS 프로그램을 모듈로 구성하는 것에는 몇 가지 주요 이점이 있다.

## What Is a Module?

A module is a collection of related data and functions (often referred to as methods in this context), characterized by a division between hidden *private* details and *public* accessible details, usually called the "public API."

A module is also stateful: it maintains some information over time, along with functionality to access and update that information.

| NOTE: |
| :--- |
| A broader concern of the module pattern is fully embracing system-level modularization through loose-coupling and other program architecture techniques. That's a complex topic well beyond the bounds of our discussion, but is worth further study beyond this book. |

To get a better sense of what a module is, let's compare some module characteristics to useful code patterns that aren't quite modules.

### Namespaces (Stateless Grouping)

If you group a set of related functions together, without data, then you don't really have the expected encapsulation a module implies. The better term for this grouping of *stateless* functions is a namespace:

```js
// namespace, not module
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

`Utils` here is a useful collection of utilities, yet they're all state-independent functions. Gathering functionality together is generally good practice, but that doesn't make this a module. Rather, we've defined a `Utils` namespace and organized the functions under it.

### Data Structures (Stateful Grouping)

Even if you bundle data and stateful functions together, if you're not limiting the visibility of any of it, then you're stopping short of the POLE aspect of encapsulation; it's not particularly helpful to label that a module.

Consider:

```js
// data structure, not module
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

Since `records` is publicly accessible data, not hidden behind a public API, `Student` here isn't really a module.

`Student` does have the data-and-functionality aspect of encapsulation, but not the visibility-control aspect. It's best to label this an instance of a data structure.

### Modules (Stateful Access Control)

To embody the full spirit of the module pattern, we not only need grouping and state, but also access control through visibility (private vs. public).

Let's turn `Student` from the previous section into a module. We'll start with a form I call the "classic module," which was originally referred to as the "revealing module" when it first emerged in the early 2000s. Consider:

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

`Student` is now an instance of a module. It features a public API with a single method: `getName(..)`. This method is able to access the private hidden `records` data.

| WARNING: |
| :--- |
| I should point out that the explicit student data being hard-coded into this module definition is just for our illustration purposes. A typical module in your program will receive this data from an outside source, typically loaded from databases, JSON data files, Ajax calls, etc. The data is then injected into the module instance typically through method(s) on the module's public API. |

How does the classic module format work?

Notice that the instance of the module is created by the `defineStudent()` IIFE being executed. This IIFE returns an object (named `publicAPI`) that has a property on it referencing the inner `getName(..)` function.

Naming the object `publicAPI` is stylistic preference on my part. The object can be named whatever you like (JS doesn't care), or you can just return an object directly without assigning it to any internal named variable. More on this choice in Appendix A.

From the outside, `Student.getName(..)` invokes this exposed inner function, which maintains access to the inner `records` variable via closure.

You don't *have* to return an object with a function as one of its properties. You could just return a function directly, in place of the object. That still satisfies all the core bits of a classic module.

By virtue of how lexical scope works, defining variables and functions inside your outer module definition function makes everything *by default* private. Only properties added to the public API object returned from the function will be exported for external public use.

The use of an IIFE implies that our program only ever needs a single central instance of the module, commonly referred to as a "singleton." Indeed, this specific example is simple enough that there's no obvious reason we'd need anything more than just one instance of the `Student` module.

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
