# You Don't Know JS Yet: Scope & Closures - 2nd Edition
# Appendix A: Exploring Further

We will now explore a number of nuances and edges around many of the topics covered in the main text of this book. This appendix is optional, supporting material.

Some people find diving too deeply into the nuanced corner cases and varying opinions creates nothing but noise and distraction—supposedly, developers are better served by sticking to the commonly-tread paths. My approach has been criticized as being impractical and counterproductive. I understand and appreciate that perspective, even if I don't necessarily share it.

I believe it's better to be empowered by knowledge of how things work than to just gloss over details with assumptions and lack of curiosity. Ultimately, you will encounter situations where something bubbles up from a piece you hadn't explored. In other words, you won't get to spend all your time riding on the smooth *happy path*. Wouldn't you rather be prepared for the inevitable bumps of off-roading?

These discussions will also be more heavily influenced by my opinions than the main text was, so keep that in mind as you consume and consider what is presented. This appendix is a bit like a collection of mini-blog posts that elaborate on various book topics. It's long and deep in the weeds, so take your time and don't rush through everything here.

## Implied Scopes

Scopes are sometimes created in non-obvious places. In practice, these implied scopes don't often impact your program behavior, but it's still useful to know they're happening. Keep an eye out for the following surprising scopes:

* Parameter scope
* Function name scope

### Parameter Scope

The conversation metaphor in Chapter 2 implies that function parameters are basically the same as locally declared variables in the function scope. But that's not always true.

Consider:

```js
// outer/global scope: RED(1)

function getStudentName(studentID) {
    // function scope: BLUE(2)

    // ..
}
```

Here, `studentID` is a considered a "simple" parameter, so it does behave as a member of the BLUE(2) function scope. But if we change it to be a non-simple parameter, that's no longer technically the case. Parameter forms considered non-simple include parameters with default values, rest parameters (using `...`), and destructured parameters.

Consider:

```js
// outer/global scope: RED(1)

function getStudentName(/*BLUE(2)*/ studentID = 0) {
    // function scope: GREEN(3)

    // ..
}
```

Here, the parameter list essentially becomes its own scope, and the function's scope is then nested inside *that* scope.

Why? What difference does it make? The non-simple parameter forms introduce various corner cases, so the parameter list becomes its own scope to more effectively deal with them.

Consider:

```js
function getStudentName(studentID = maxID, maxID) {
    // ..
}
```

Assuming left-to-right operations, the default `= maxID` for the `studentID` parameter requires a `maxID` to already exist (and to have been initialized). This code produces a TDZ error (Chapter 5). The reason is that `maxID` is declared in the parameter scope, but it's not yet been initialized because of the order of parameters. If the parameter order is flipped, no TDZ error occurs:

```js
function getStudentName(maxID,studentID = maxID) {
    // ..
}
```

The complication gets even more *in the weeds* if we introduce a function expression into the default parameter position, which then can create its own closure (Chapter 7) over parameters in this implied parameter scope:

```js
function whatsTheDealHere(id,defaultID = () => id) {
    id = 5;
    console.log( defaultID() );
}

whatsTheDealHere(3);
// 5
```

That snippet probably makes sense, because the `defaultID()` arrow function closes over the `id` parameter/variable, which we then re-assign to `5`. But now let's introduce a shadowing definition of `id` in the function scope:

```js
function whatsTheDealHere(id,defaultID = () => id) {
    var id = 5;
    console.log( defaultID() );
}

whatsTheDealHere(3);
// 3
```

Uh oh! The `var id = 5` is shadowing the `id` parameter, but the closure of the `defaultID()` function is over the parameter, not the shadowing variable in the function body. This proves there's a scope bubble around the parameter list.

But it gets even crazier than that!

```js
function whatsTheDealHere(id,defaultID = () => id) {
    var id;

    console.log(`local variable 'id': ${ id }`);
    console.log(
        `parameter 'id' (closure): ${ defaultID() }`
    );

    console.log("reassigning 'id' to 5");
    id = 5;

    console.log(`local variable 'id': ${ id }`);
    console.log(
        `parameter 'id' (closure): ${ defaultID() }`
    );
}

whatsTheDealHere(3);
// local variable 'id': 3   <--- Huh!? Weird!
// parameter 'id' (closure): 3
// reassigning 'id' to 5
// local variable 'id': 5
// parameter 'id' (closure): 3
```

The strange bit here is the first console message. At that moment, the shadowing `id` local variable has just been `var id` declared, which Chapter 5 asserts is typically auto-initialized to `undefined` at the top of its scope. Why doesn't it print `undefined`?

In this specific corner case (for legacy compat reasons), JS doesn't auto-initialize `id` to `undefined`, but rather to the value of the `id` parameter (`3`)!

Though the two `id`s look at that moment like they're one variable, they're actually still separate (and in separate scopes). The `id = 5` assignment makes the divergence observable, where the `id` parameter stays `3` and the local variable becomes `5`.

My advice to avoid getting bitten by these weird nuances:

* Never shadow parameters with local variables

* Avoid using a default parameter function that closes over any of the parameters

At least now you're aware and can be careful about the fact that the parameter list is its own scope if any of the parameters are non-simple.

### Function Name Scope

In the "Function Name Scope" section in Chapter 3, I asserted that the name of a function expression is added to the function's own scope. Recall:

```js
var askQuestion = function ofTheTeacher(){
    // ..
};
```

It's true that `ofTheTeacher` is not added to the enclosing scope (where `askQuestion` is declared), but it's also not *just* added to the scope of the function, the way you're likely assuming. It's another strange corner case of implied scope.

The name identifier of a function expression is in its own implied scope, nested between the outer enclosing scope and the main inner function scope.

If `ofTheTeacher` was in the function's scope, we'd expect an error here:

```js
var askQuestion = function ofTheTeacher(){
    // why is this not a duplicate declaration error?
    let ofTheTeacher = "Confused, yet?";
};
```

The `let` declaration form does not allow re-declaration (see Chapter 5). But this is perfectly legal shadowing, not re-declaration, because the two `ofTheTeacher` identifiers are in separate scopes.

You'll rarely run into any case where the scope of a function's name identifier matters. But again, it's good to know how these mechanisms actually work. To avoid being bitten, never shadow function name identifiers.

## Anonymous vs. Named Functions

As discussed in Chapter 3, functions can be expressed either in named or anonymous form. It's vastly more common to use the anonymous form, but is that a good idea?

As you contemplate naming your functions, consider:

* Name inference is incomplete
* Lexical names allow self-reference
* Names are useful descriptions
* Arrow functions have no lexical names
* IIFEs also need names

### Explicit or Inferred Names?

Every function in your program has a purpose. If it doesn't have a purpose, take it out, because you're just wasting space. If it *does* have a purpose, there *is* a name for that purpose.

So far many readers likely agree with me. But does that mean we should always put that name into the code? Here's where I'll raise more than a few eyebrows. I say, unequivocally, yes!

First of all, "anonymous" showing up in stack traces is just not all that helpful to debugging:

```js
btn.addEventListener("click",function(){
    setTimeout(function(){
        ["a",42].map(function(v){
            console.log(v.toUpperCase());
        });
    },100);
});
// Uncaught TypeError: v.toUpperCase is not a function
//     at myProgram.js:4
//     at Array.map (<anonymous>)
//     at myProgram.js:3
```

Ugh. Compare to what is reported if I give the functions names:

```js
btn.addEventListener("click",function onClick(){
    setTimeout(function waitAMoment(){
        ["a",42].map(function allUpper(v){
            console.log(v.toUpperCase());
        });
    },100);
});
// Uncaught TypeError: v.toUpperCase is not a function
//     at allUpper (myProgram.js:4)
//     at Array.map (<anonymous>)
//     at waitAMoment (myProgram.js:3)
```

See how `waitAMoment` and `allUpper` names appear and give the stack trace more useful information/context for debugging? The program is more debuggable if we use reasonable names for all our functions.

| NOTE: |
| :--- |
| The unfortunate "&lt;anonymous>" that still shows up refers to the fact that the implementation of `Array.map(..)` isn't present in our program, but is built into the JS engine. It's not from any confusion our program introduces with readability shortcuts. |

By the way, let's make sure we're on the same page about what a named function is:

```js
function thisIsNamed() {
    // ..
}

ajax("some.url",function thisIsAlsoNamed(){
   // ..
});

var notNamed = function(){
    // ..
};

makeRequest({
    data: 42,
    cb /* also not a name */: function(){
        // ..
    }
});

var stillNotNamed = function butThisIs(){
    // ..
};
```

"But wait!", you say. Some of those *are* named, right!?

```js
var notNamed = function(){
    // ..
};

var config = {
    cb: function(){
        // ..
    }
};

notNamed.name;
// notNamed

config.cb.name;
// cb
```

These are referred to as *inferred* names. Inferred names are fine, but they don't really address the full concern I'm discussing.

### Missing Names?

Yes, these inferred names might show up in stack traces, which is definitely better than "anonymous" showing up. But...

```js
function ajax(url,cb) {
    console.log(cb.name);
}

ajax("some.url",function(){
    // ..
});
// ""
```

Oops. Anonymous `function` expressions passed as callbacks are incapable of receiving an inferred name, so `cb.name` holds just the empty string `""`. The vast majority of all `function` expressions, especially anonymous ones, are used as callback arguments; none of these get a name. So relying on name inference is incomplete, at best.

And it's not just callbacks that fall short with inference:

```js
var config = {};

config.cb = function(){
    // ..
};

config.cb.name;
// ""

var [ noName ] = [ function(){} ];
noName.name
// ""
```

Any assignment of a `function` expression that's not a *simple assignment* will also fail name inferencing. So, in other words, unless you're careful and intentional about it, essentially almost all anonymous `function` expressions in your program will in fact have no name at all.

Name inference is just... not enough.

And even if a `function` expression *does* get an inferred name, that still doesn't count as being a full named function.

### Who am I?

Without a lexical name identifier, the function has no internal way to refer to itself. Self-reference is important for things like recursion and event handling:

```js
// broken
runOperation(function(num){
    if (num <= 1) return 1;
    return num * oopsNoNameToCall(num - 1);
});

// also broken
btn.addEventListener("click",function(){
   console.log("should only respond to one click!");
   btn.removeEventListener("click",oopsNoNameHere);
});
```

Leaving off the lexical name from your callback makes it harder to reliably self-reference the function. You *could* declare a variable in an enclosing scope that references the function, but this variable is *controlled* by that enclosing scope—it could be re-assigned, etc.—so it's not as reliable as the function having its own internal self-reference.

### Names are Descriptors

Lastly, and I think most importantly of all, leaving off a name from a function makes it harder for the reader to tell what the function's purpose is, at a quick glance. They have to read more of the code, including the code inside the function, and the surrounding code outside the function, to figure it out.

Consider:

```js
[ 1, 2, 3, 4, 5 ].filter(function(v){
    return v % 2 == 1;
});
// [ 1, 3, 5 ]

[ 1, 2, 3, 4, 5 ].filter(function keepOnlyOdds(v){
    return v % 2 == 1;
});
// [ 1, 3, 5 ]
```

There's just no reasonable argument to be made that **omitting** the name `keepOnlyOdds` from the first callback more effectively communicates to the reader the purpose of this callback. You saved 13 characters, but lost important readability information. The name `keepOnlyOdds` very clearly tells the reader, at a quick first glance, what's happening.

The JS engine doesn't care about the name. But human readers of your code absolutely do.

Can the reader look at `v % 2 == 1` and figure out what it's doing? Sure. But they have to infer the purpose (and name) by mentally executing the code. Even a brief pause to do so slows down reading of the code. A good descriptive name makes this process almost effortless and instant.

Think of it this way: how many times does the author of this code need to figure out the purpose of a function before adding the name to the code? About once. Maybe two or three times if they need to adjust the name. But how many times will readers of this code have to figure out the name/purpose? Every single time this line is ever read. Hundreds of times? Thousands? More?

No matter the length or complexity of the function, my assertion is, the author should figure out a good descriptive name and add it to the code. Even the one-liner functions in `map(..)` and `then(..)` statements should be named:

```js
lookupTheRecords(someData)
.then(function extractSalesRecords(resp){
   return resp.allSales;
})
.then(storeRecords);
```

The name `extractSalesRecords` tells the reader the purpose of this `then(..)` handler *better* than just inferring that purpose from mentally executing `return resp.allSales`.

The only excuse for not including a name on a function is either laziness (don't want to type a few extra characters) or uncreativity (can't come up with a good name). If you can't figure out a good name, you likely don't understand the function and its purpose yet. The function is perhaps poorly designed, or it does too many things, and should be re-worked. Once you have a well-designed, single-purpose function, its proper name should become evident.

Here's a trick I use: while first writing a function, if I don't fully understand its purpose and can't think of a good name to use, I just use `TODO` as the name. That way, later when reviewing my code, I'm likely to find those name placeholders, and I'm more inclined (and more prepared!) to go back and figure out a better name, rather than just leave it as `TODO`.

All functions need names. Every single one. No exceptions. Any name you omit is making the program harder to read, harder to debug, harder to extend and maintain later.

### Arrow Functions

Arrow functions are **always** anonymous, even if (rarely) they're used in a way that gives them an inferred name. I just spent several pages explaining why anonymous functions are a bad idea, so you can probably guess what I think about arrow functions.

Don't use them as a general replacement for regular functions. They're more concise, yes, but that brevity comes at the cost of omitting key visual delimiters that help our brains quickly parse out what we're reading. And, to the point of this discussion, they're anonymous, which makes them worse for readability from that angle as well.

Arrow functions have a purpose, but that purpose is not to save keystrokes. Arrow functions have *lexical this* behavior, which is somewhat beyond the bounds of our discussion in this book.

Briefly: arrow functions don't define a `this` identifier keyword at all. If you use a `this` inside an arrow function, it behaves exactly as any other variable reference, which is that the scope chain is consulted to find a function scope (non-arrow function) where it *is* defined, and to use that one.

In other words, arrow functions treat `this` like any other lexical variable.

If you're used to hacks like `var self = this`, or if you prefer to call `.bind(this)` on inner `function` expressions, just to force them to inherit a `this` from an outer function like it was a lexical variable, then `=>` arrow functions are absolutely the better option. They're designed specifically to fix that problem.

So, in the rare cases you need *lexical this*, use an arrow function. It's the best tool for that job. But just be aware that in doing so, you're accepting the downsides of an anonymous function. You should expend additional effort to mitigate the readability *cost*, such as more descriptive variable names and code comments.

### IIFE Variations

All functions should have names. I said that a few times, right!? That includes IIFEs.

```js
(function(){
    // don't do this!
})();

(function doThisInstead(){
    // ..
})();
```

How do we come up with a name for an IIFE? Identify what the IIFE is there for. Why do you need a scope in that spot? Are you hiding a cache variable for student records?

```js
var getStudents = (function StoreStudentRecords(){
    var studentRecords = [];

    return function getStudents() {
        // ..
    }
})();
```

I named the IIFE `StoreStudentRecords` because that's what it's doing: storing student records. Every IIFE should have a name. No exceptions.

IIFEs are typically defined by placing `( .. )` around the `function` expression, as shown in those previous snippets. But that's not the only way to define an IIFE. Technically, the only reason we're using that first surrounding set of `( .. )` is just so the `function` keyword isn't in a position to qualify as a `function` declaration to the JS parser. But there are other syntactic ways to avoid being parsed as a declaration:

```js
!function thisIsAnIIFE(){
    // ..
}();

+function soIsThisOne(){
    // ..
}();

~function andThisOneToo(){
    // ..
}();
```

The `!`, `+`, `~`, and several other unary operators (operators with one operand) can all be placed in front of `function` to turn it into an expression. Then the final `()` call is valid, which makes it an IIFE.

I actually kind of like using the `void` unary operator when defining a standalone IIFE:

```js
void function yepItsAnIIFE() {
    // ..
}();
```

The benefit of `void` is, it clearly communicates at the beginning of the function that this IIFE won't be returning any value.

However you define your IIFEs, show them some love by giving them names.

## 호이스팅: 함수와 변수

5장에서는 *함수 호이스팅*과 *변수 호이스팅*에 대해서 다루었다. 호이스팅은 종종 JS 설계상의 실수로 언급되기 때문에, 왜 이 두 형태의 호이스팅이 유용하며 그리고 고려되어야 하는지를 간략하게 설명하고자 한다. 

다음의 장점들을 생각하며 호이스팅을 더 깊게 이해해보자

* 실행 가능한 코드를 먼저, 함수 선언을 나중에
* 변수 선언의 의미적 배치

### 함수 호이스팅

이 프로그램은 *함수 호이스팅* 으로 작동한다.

```js
getStudents();

// ..

function getStudents() {
    // ..
}
```

`function` 선언은 컴파일 동안 호이스팅된다. 다시 말하면 `getStudents`은 전체 스코프에 선언된 식별자라는 것이다. 게다가 `getStudents` 식별자는 스코프 시작 부분에서 함수 참조와 함께 다시 자동 초기화된다.

왜 이것이 유용할까? *function hoisting*은 *실행가능한* 코드를 상위의 어떤 스코프에나 둘 수 있고 선언들 (함수)을 더 아래에 둘 수 있다. 이것은 동작하는 코드를 찾기 쉽게 도와주는데, 스코프/함수의 끝을 알기 위해 `}`를 찾아 계속 스크롤할 필요가 없기 때문이다.

이러한 위치의 역전은 모든 스코프 레벨에서 찾을 수 있다.

```js
getStudents();

// *************

function getStudents() {
    var whatever = doSomething();

    // 다른 내용

    return whatever;

    // *************

    function doSomething() {
        // ..
    }
}
```

위 파일을 처음 열었을 때 맨 첫 번째 줄은 실행 가능한 코드이다. 그것은 매우 발견하기 쉽다! 그런 다음 `getStudents()`를 찾아 확인해야 한다면 첫 번째 줄도 실행 가능한 코드인 것이 좋다. `doSomething()`의 세부 사항을 봐야 할 경우에만 아래에 있는 정의를 찾아간다.

즉, *함수 호이스팅*은 위에서 아래로 흘러가는 점진적인 읽기 순서를 통해 코드를 보다 쉽게 읽을 수 있게 한다.

### 변수 호이스팅

*변수 호이스팅*이란 무엇일까?

`let`과 `const`은 호이스팅 되더라, TDZ에서 그 변수들을 사용할 수 없다(5장을 참고해라). 그래서 다음 논의는 오직 `var`에만 적용된다. 진행에 앞서, 대부분의 경우에 *변수 호이스팅*은 나쁜 아이디어라는 것에 완전히 찬성한다는 사실을 밝힌다.

```js
pleaseDontDoThis = "bad idea";

// 훨씬 후에
var pleaseDontDoThis;
```

역순으로 두는 것이 *함수 호이스팅*에서는 유용한 반면,  변수의 경우 그것은 코드를 추론하는 것을 더 어렵게 한다.

그러나 직접 작성한 코딩중에 드물긴 하지만 한가지 예외가 있다.  

다음은 Node에서 module 정의할 때 전형적으로 사용하는 구조이다.

```js
// dependencies
var aModuleINeed = require("very-helpful");
var anotherModule = require("kinda-helpful");

// public API
var publicAPI = Object.assign(module.exports,{
    getStudents,
    addStudents,
    // ..
});

// ********************************
// private implementation

var cache = { };
var otherData = [ ];

function getStudents() {
    // ..
}

function addStudents() {
    // ..
}
```

왜 `cache`와 `otherData`는 모듈 레이아웃의 "private" 섹션에 존재하게 되었는가? 왜냐하면 공개적으로 노출하지 않기 원했기 때문이다. 그래서 모듈을 이와같이 작성하게 되었고 그래서 모듈의 다른 숨겨진 구현들과 함께 배치되게 되었다.

그러나 이 값들을 위에 배치해야 하는 소수의 드문 경우가 있긴하다. 모듈의 퍼블릭 API 내보내기전에 말이다. 예를 들어:

```js
// public API
var publicAPI = Object.assign(module.exports,{
    getStudents,
    addStudents,
    refreshData: refreshData.bind(null,cache)
});
```

`cache` 값이 퍼블릭 API(`.bind(...)` 부분 응용 프로그램)의 초기화에 사용되므로 변수에 이미 값이 할당되어 있어야 한다.

`var cache = {..}`만 이동하면 될까? 이 공개 API 초기화 위에? 글쎄, 아마도. 그러나 이제는 `var cache`가 *private* 구현 세부 정보라는 것이 명확하지 않다. 이것에 대한 절충안은 다음과 같다.

```js
cache = {};   // 여기에서 사용되지만, 아래에 할당되어 있다.

// public API
var publicAPI = Object.assign(module.exports,{
    getStudents,
    addStudents,
    refreshData: refreshData.bind(null,cache)
});

// ********************************
// private implementation

var cache /* = {}*/;
```

*변수 호이스팅*가 보이나? `cache`를 그것이 속하는 아래쪽에 선언하였다. 하지만 코드 위쪽에 초기화가 필요한 영역 사용하고 있다. 코드 코멘트에 `cache`에 할당된 값에 대한 힌트를 남겼다.

*변수 호이스팅*을 활용하여 변수의 선언보다 초기에 변수를 할당한 사례는 이 정도밖에 없다. 하지만 이런 예외에 호이스팅을 신중하게 사용하는 것은 합리적이라고 생각한다. 

## `var` 케이스

*변수 호이스팅*을 다루기 위해, `var`에 대해서 샅샅히 까발려보자, 악당 개발자는 JS 개발이 가진 많은 고민들을 탓하는 것을 좋아한다. 5장에서, 우리는 `let`/`const`를 다뤘고 그리고 `var`가 잘 작동하는 곳에서 다시 이 챕터에 대해 이야기해보기로 했다.

케이스를 정리했으니, 놓치지 말자:


* `var` 는 절대 고장난 적이 없었다.
* `let` 은 당신의 친구
* `const` 는 제한된 사용상을 가진다.
*  두개의 세계에서의 최고: `var` *그리고* `let`

### `var`를 버리지 마라

`var` 는 괜찮다, 그리고 괜찮게 작동한다. 이것은 25년 동안 존재해왔다, 그리고 다음 25년 동안도 존재할 것이며 유용하고 기능적일 것이다. `var`는 고장나고, 권장되지 않고, 오래되었고, 위험하고 또는 잘못 디자인 되었다는 주장은 모호한 편승이론일뿐이다.

이게 'var'가 프로그램의 모든 선언에 적합한 선언자임을 의미할까? 확실히 아니다. 하지만 당신의 프로그램에는 여전히 자리하고 있다. 팀의 누군가가 'var'의 목을 조르는 공격적인 의견을 선택했다는 이유로 당신이 사용을 거부하는 것은 당신의 체면을 상하게 하기 위해 당신의 코를 자르는 것이다.


좋다, 이제 당신을 화나게 했으니, 이에 대한 입장을 설명하겠다.

참고로 나는 블록스코프 선언의 팬이다. 나는 TDZ가 정말 싫고 그것은 실수였다고 생각한다. 하지만 `let` 자체는 대단하다. 자주 쓴다. 사실 나는 아마 `var`를 사용하는 것보다 많이 혹은 더 많이 사용하고 있을 것이다.


### `const`-완전히 혼란스러운

반면에 `const`는 자주 사용하지 않는다. 그 이유를 다 파헤치지는 않겠지만, 결론은 '자신의 무게를 짊어지지 않기' 때문으로 귀결된다. 즉, 경우에 따라서는 const의 약간의 이점도 있지만, JS에 나타나기 훨씬 전에 다양한 언어의  `const `를 둘러싼 오랜 혼란의 역사가 그 이점을 능가한다.

`const`는 돌연변이가 불가능한 값을 만들수 있는 척한다 - 여러 언어의 개발자 커뮤니티에서 매우 흔한 오해이지만 실제로는 단지 재할당을 막기만할 뿐이다. 

```js
const studentIDs = [ 14, 73, 112 ];

// 후에

studentIDs.push(6);   // 후아, 잠깐... 뭐라고!?
```

변이 가능한 값(예: 배열 또는 객체)이 있는 `const`를 사용하는 것은 미래의 개발자(또는 코드를 읽는 사람)가 설정한 함정에 빠질 것을 요청하는 것이다. 즉, *값 불변성*은 *할당 불변성*과 전혀 같지 않다는 것을 몰랐거나 잊어버린 것이다.

함정을 만드는 것은 안 될 것 같아 내가 유일하게 `const`를 사용하는 때는 '42'나 '안녕, 친구들!'처럼 이미 불변의 값을 할당하고 있을 때, 그리고 그것이 문자 그대로의 가치, 의미적 목적으로 이름 붙여진 자리 표시자라는 의미에서 분명히 'constant'일 때이다. const가 가장 잘 쓰이는 이유다. 제 코드에서 그런 경우는 드물다.

If variable re-assignment were a big deal, then `const` would be more useful. But variable re-assignment just isn't that big of a deal in terms of causing bugs. There's a long list of things that lead to bugs in programs, but "accidental re-assignment" is way, way down that list.
변수 재할당이 큰 문제라면, `const`가 더 유용하다. 하지만 변수 재할당은 버그를 일으키는 측면에서 그리 큰 문제가 아니다. 프로그램에 버그를 일으키는 많은 것들이 있지만, "우발적인 재할당"은 그 목록에서 훨씬 아래쪽에 있다.

Combine that with the fact that `const` (and `let`) are supposed to be used in blocks, and blocks are supposed to be short, and you have a really small area of your code where a `const` declaration is even applicable. A `const` on line 1 of your ten-line block only tells you something about the next nine lines. And the thing it tells you is already obvious by glancing down at those nine lines: the variable is never on the left-hand side of an `=`; it's not re-assigned.
`const`(그리고 `let`)는 블록에 사용되어야 하고 블록은 짧아야 한다는 사실과 결합하면 const 선언을 적용할 수 있는 매우 작은 영역의 코드를 가질 수 있다. 10줄 블록 중 1줄에 있는 `const`는 다음 9줄에 대한 내용만 알려준다. 그리고 이 9개의 선을 내려다보면 이미 알 수 있다: 변수는 절대 '='의 왼쪽에 있지 않으며, 재할당되지 않는다.


That's it, that's all `const` really does. Other than that, it's not very useful. Stacked up against the significant confusion of value vs. assignment immutability, `const` loses a lot of its luster.
그게 `const`가 진짜 하는 전부다. 그 외에는 별로 쓸모가 없다. 값 vs 할당 불변성의 중대한 혼돈에 맞서서 `const`는 많은 빛을 잃는다.

A `let` (or `var`!) that's never re-assigned is already behaviorally a "constant", even though it doesn't have the compiler guarantee. That's good enough in most cases.
재할당되지 않는 'let' (또는 'var'!)은 비록 컴파일러가 보장하지 않더라도 이미 상수이다. 대부분의 경우 그것으로 충분하다.

### `var` *그리고* `let`

In my mind, `const` is pretty rarely useful, so this is only two-horse race between `let` and `var`. But it's not really a race either, because there doesn't have to be just one winner. They can both win... different races.
내 생각에 `const`는 쓸모가 거의 없어, let과 var의 두 마리 토끼 싸움에 불과하다. 하지만 꼭 한 명의 우승자가 있어야 하는 것은 아니기 때문에, 그것은 또한 진정한 경주가 아니다. 둘 다 이길 수 있다... 다른 경주에서.

The fact is, you should be using both `var` and `let` in your programs. They are not interchangeable: you shouldn't use `var` where a `let` is called for, but you also shouldn't use `let` where a `var` is most appropriate.
사실 프로그램에서는 `var`와 `let`을 모두 사용해야 한다. `let`이 호출되는 곳에 `var`를 사용해서는 안 되지만 `var`가 가장 적절한 곳에 `let`을 사용해서는 안 된다.

So where should we still use `var`? Under what circumstances is it a better choice than `let`?
그렇다면 `var`를 어디에 사용해야 할까? 어떤 상황에서 그것이 `let`보다 나은 선택일까?

For one, I always use `var` in the top-level scope of any function, regardless of whether that's at the beginning, middle, or end of the function. I also use `var` in the global scope, though I try to minimize usage of the global scope.
한 가지 예로, 나는 함수의 시작, 중간, 끝에 상관없이 모든 함수의 최상위 스코프에서 항상 'var'를 사용한다. 글로벌 스코프의 사용을 최소화하려고 노력하지만 글로벌 스코프에서도 var를 사용한다.

Why use `var` for function scoping? Because that's exactly what `var` does. There literally is no better tool for the job of function scoping a declaration than a declarator that has, for 25 years, done exactly that.
함수 스코프에 `var`를 사용하는 이유는 무엇일까? 그것이 바로 `var`가 하는 일이기 때문이다. 말 그대로 선언을 스코핑하는 함수에서 25년 동안 정확히 해온 선언자보다 더 좋은 도구는 없다.

You *could* use `let` in this top-level scope, but it's not the best tool for that job. I also find that if you use `let` everywhere, then it's less obvious which declarations are designed to be localized and which ones are intended to be used throughout the function.
이 최상위 스코프에서, `let`을 *사용할 수 있지만*, 해당 작업에 가장 적합한 툴은 아니다. 또 어디서나 `let`을 사용하면 어떤 선언이 지역화 되도록 되어 있는지, 어떤 선언이 기능 전반에 사용되도록 되어 있는지 잘 알 수 없다.


By contrast, I rarely use a `var` inside a block. That's what `let` is for. Use the best tool for the job. If you see a `let`, it tells you that you're dealing with a localized declaration. If you see `var`, it tells you that you're dealing with a function-wide declaration. Simple as that.
반대로 블록 안에서 `var`를 사용하는 경우는 거의 없다. 그래서 `let`이 있는 것이다. 작업에 가장 적합한 도구를 사용해라. `let`이 보이면 지역 선언을 다루고 있음을 알려준다. `var`가 표시되면 함수 차원의 선언을 다루고 있음을 나타. 그렇게 간단하다.

```js
function getStudents(data) {
    var studentRecords = [];

    for (let record of data.records) {
        let id = `student-${ record.id }`;
        studentRecords.push({
            id,
            record.name
        });
    }

    return studentRecords;
}
```

The `studentRecords` variable is intended for use across the whole function. `var` is the best declarator to tell the reader that. By contrast, `record` and `id` are intended for use only in the narrower scope of the loop iteration, so `let` is the best tool for that job.
`studentRecords` 변수는 전체 기능에서 사용하기 위한 것이다. `va`'는 독자들에게 그것을 말해주는 최고의 선언자이다. 반대로 `record`와 `id`는 루프 반복의 좁은 범위에서만 사용하기 때문에 `let` 해당 작업에 가장 적합한 도구이다.

In addition to this *best tool* semantic argument, `var` has a few other characteristics that, in certain limited circumstances, make it more powerful.
이 *최상의 도구* 주장에 더하여, `var`는 특정 제한된 상황에서 더 강력하게 만드는 몇 가지 다른 특성을 가지고 있다.

One example is when a loop is exclusively using a variable, but its conditional clause cannot see block-scoped declarations inside the iteration:
예를 들어, 루프가 변수를 배타적으로 사용하고 있지만 조건 절은 이터레이션 내에서 블록 스코프 선언을 볼 수 없는 경우를 들 수 있다.

```js
function commitAction() {
    do {
        let result = commit();
        var done = result && result.code == 1;
    } while (!done);
}
```

Here, `result` is clearly only used inside the block, so we use `let`. But `done` is a bit different. It's only useful for the loop, but the `while` clause cannot see `let` declarations that appear inside the loop. So we compromise and use `var`, so that `done` is hoisted to the outer scope where it can be seen.
여기서, `result`는 블록 안에서만 사용되므로 `let`을 사용한다. 그러나 `done`은 조금 다르다. 이것은 루프에만 유용하지만 `while` 절에서는 루프 내부에 나타나는 `let` 선언을 볼 수 없다. 그래서 우리는 타협하고 `var`를 사용하여 `done`이 보이는 외부 범위로 올라가도록 한다.

The alternative—declaring `done` outside the loop—separates it from where it's first used, and either necessitates picking a default value to assign, or worse, leaving it unassigned and thus looking ambiguous to the reader. I think `var` inside the loop is preferable here.
루프 밖에서 `done`을 선언하는 대안은 처음 사용된 곳에서 분리하여 할당하기 위해 기본값을 선택해야 하거나, 더 나쁘게는 할당되지 않은 상태로 있어 독자에게 모호하게 보일 수 있다. 나는 루프 내부의 `var`가 더 좋다고 생각한다.

Another helpful characteristic of `var` is seen with declarations inside unintended blocks. Unintended blocks are blocks that are created because the syntax requires a block, but where the intent of the developer is not really to create a localized scope. The best illustration of unintended scope is the `try..catch` statement:
`var`의 또 다른 유용한 특징은 의도하지 않은 블록 내부에 선언이 있다는 것이다. 의도하지 않은 블록은 구문에 블록이 필요하기 때문에 생성되는 블록이지만 개발자의 의도는 현지화된 범위를 만드는 것이 아니다. 의도하지 않은 범위의 가장 좋은 예는 `try..catch` 이다:


```js
function getStudents() {
    try {
        // not really a block scope
        // 실제로는 블록 스코프가 아니다.
        var records = fromCache("students");
    }
    catch (err) {
        //
        // 웁스 디폴트가 되었다.
        var records = [];
    }
    // ..
}
```

There are other ways to structure this code, yes. But I think this is the *best* way, given various trade-offs.
이 코드를 구성하는 다른 방법이 있다. 하지만 여러 가지 단점을 고려할 때 이것이 *가장 좋은* 방법이라고 생각한다.

I don't want to declare `records` (with `var` or `let`) outside of the `try` block, and then assign to it in one or both blocks. I prefer initial declarations to always be as close as possible (ideally, same line) to the first usage of the variable. In this simple example, that would only be a couple of lines distance, but in real code it can grow to many more lines. The bigger the gap, the harder it is to figure out what variable from what scope you're assigning to. `var` used at the actual assignment makes it less ambiguous.
`records`(`var` 또는 `let`이 있는)를 `try` 블록 밖으로 선언하고 나서, 어느 한 블록 또는 양쪽 블록으로 할당하고 싶지 않다. 초기 선언은 가능한 한 변수의 첫 번째 사용에 가까운(이상적으로 같은 행) 것이 좋다. 이 간단한 예에서는 몇 개의 회선 거리에 불과하지만 실제 코드에서는 더 많은 회선까지 늘어날 수 있다. 갭이 클수록 할당하는 스코프에서 변수를 파악하는 것이 어려워진다. 실제 할당에서 사용되는 `var`를 사용하면 덜 모호해진다.

Also notice I used `var` in both the `try` and `catch` blocks. That's because I want to signal to the reader that no matter which path is taken, `records` always gets declared. Technically, that works because `var` is hoisted once to the function scope. But it's still a nice semantic signal to remind the reader what either `var` ensures. If `var` were only used in one of the blocks, and you were only reading the other block, you wouldn't as easily discover where `records` was coming from.
또, `try`블록과 `catch`블록 양쪽에서 `var`를 사용하고 있는 것에 주의해라. 어떤 길을 가더라도 `records`는 반드시 선언된다는 것을 독자에게 알리고 싶기 때문이다. 엄밀히 말하면 `var`는 함수 스코프로 한 번 호이스트된다. 그러나 그것은 여전히 독자들에게 `var`가 보장하는 것을 상기시키는 좋은 의미 신호이다. 한 블록에서만 `var`를 사용하고 다른 블록만 읽고 있다면 레코드가 어디서 왔는지 쉽게 알 수 없을 것이다.

This is, in my opinion, a little superpower of `var`. Not only can it escape the unintentional `try..catch` blocks, but it's allowed to appear multiple times in a function's scope. You can't do that with `let`. It's not bad, it's actually a little helpful feature. Think of `var` more like a declarative annotation that's reminding you, each usage, where the variable comes from. "Ah ha, right, it belongs to the whole function."
이것은 내가 보기에 `var`의 작은 초능력이다. 의도하지 않은 `try..catch`블록에서 벗어날 수 있을 뿐만 아니라, 함수의 스코프에는 여러 번 표시될 수 있다. `let`으로는 그럴 수 없다. 나쁘지 않다. 사실 조금 도움이 되는 기능이다. `var`는 변수 출처를 알려주는 선언적 주석과 비슷하다. "아하, 맞아, 함수 전체에 속하지."

This repeated-annotation superpower is useful in other cases:
이 반복 주석 슈퍼파워는 다른 경우에 유용하다.

```js
function getStudents() {
    var data = [];

    // do something with data
    // .. 50 more lines of code ..
    // data 를 다룬다.
    // .. 50줄 이상의 코드 ..

    // purely an annotation to remind us
    var data;
    // var data를 상기시키는 순수한 주석

    // use data again
    // data를 다시 사용한다.
}
```

The second `var data` is not re-declaring `data`, it's just annotating for the readers' benefit that `data` is a function-wide declaration. That way, the reader doesn't need to scroll up 50+ lines of code to find the initial declaration.
두 번째 `var data`는 `data`를 다시 선언하는 것이 아니라 `data`가 함수 전체의 선언이라는 독자들의 이익을 위해 주석을 다는 것이다. 이렇게 하면 리더는 초기 선언을 찾기 위해 50줄 이상의 코드를 스크롤할 필요가 없다.

I'm perfectly fine with re-using variables for multiple purposes throughout a function scope. I'm also perfectly fine with having two usages of a variable be separated by quite a few lines of code. In both cases, the ability to safely "re-declare" (annotate) with `var` helps make sure I can tell where my `data` is coming from, no matter where I am in the function.
함수 범위 전체에서 변수를 여러 목적으로 재사용하는 것은 문제 없다. 두 개의 변수를 여러 줄의 코드로 구분하는 것도 문제 없다. 어느 경우든, var를 사용하여 안전하게 재선언(주석)할 수 있으면 함수의 어디에 있든 데이터 출처를 알 수 있다.

Again, sadly, `let` cannot do this.
다시 말하지만 슬프게도 `let`은 이것을 할 수 없다.

There are other nuances and scenarios when `var` turns out to offer some assistance, but I'm not going to belabor the point any further. The takeaway is that `var` can be useful in our programs alongside `let` (and the occasional `const`). Are you willing to creatively use the tools the JS language provides to tell a richer story to your readers?
var가 어느 정도 도움을 주는 것으로 판명될 때 다른 뉘앙스와 시나리오가 있지만 더 이상 강조하지는 않을 것이다. 중요한 점은 var가 let(가끔 const)와 함께 프로그램에서 유용하게 쓰일 수 있다는 점이다. 독자들에게 더 풍부한 이야기를 들려주기 위해 JS 언어가 제공하는 도구를 창의적으로 사용할 의향이 있습니까?

Don't just throw away a useful tool like `var` because someone shamed you into thinking it wasn't cool anymore. Don't avoid `var` because you got confused once years ago. Learn these tools and use them each for what they're best at.

## What's the Deal with TDZ?

The TDZ (temporal dead zone) was explained in Chapter 5. We illustrated how it occurs, but we skimmed over any explanation of *why* it was necessary to introduce in the first place. Let's look briefly at the motivations of TDZ.

Some breadcrumbs in the TDZ origin story:

* `const`s should never change
* It's all about time
* Should `let` behave more like `const` or `var`?

### Where It All Started

TDZ comes from `const`, actually.

During early ES6 development work, TC39 had to decide whether `const` (and `let`) were going to hoist to the top of their blocks. They decided these declarations would hoist, similar to how `var` does. Had that not been the case, I think some of the fear was confusion with mid-scope shadowing, such as:

```js
let greeting = "Hi!";

{
    // what should print here?
    console.log(greeting);

    // .. a bunch of lines of code ..

    // now shadowing the `greeting` variable
    let greeting = "Hello, friends!";

    // ..
}
```

What should we do with that `console.log(..)` statement? Would it make any sense to JS devs for it to print "Hi!"? Seems like that could be a gotcha, to have shadowing kick in only for the second half of the block, but not the first half. That's not very intuitive, JS-like behavior. So `let` and `const` have to hoist to the top of the block, visible throughout.

But if `let` and `const` hoist to the top of the block (like `var` hoists to the top of a function), why don't `let` and `const` auto-initialize (to `undefined`) the way `var` does? Here was the main concern:

```js
{
    // what should print here?
    console.log(studentName);

    // later

    const studentName = "Frank";

    // ..
}
```

Let's imagine that `studentName` not only hoisted to the top of this block, but was also auto-initialized to `undefined`. For the first half of the block, `studentName` could be observed to have the `undefined` value, such as with our `console.log(..)` statement. Once the `const studentName = ..` statement is reached, now `studentName` is assigned `"Frank"`. From that point forward, `studentName` can't ever be re-assigned.

But, is it strange or surprising that a constant observably has two different values, first `undefined`, then `"Frank"`? That does seem to go against what we think a `const`ant means; it should only ever be observable with one value.

So... now we have a problem. We can't auto-initialize `studentName` to `undefined` (or any other value for that matter). But the variable has to exist throughout the whole scope. What do we do with the period of time from when it first exists (beginning of scope) and when it's assigned its value?

We call this period of time the "dead zone," as in the "temporal dead zone" (TDZ). To prevent confusion, it was determined that any sort of access of a variable while in its TDZ is illegal and must result in the TDZ error.

OK, that line of reasoning does make some sense, I must admit.

### Who `let` the TDZ Out?

But that's just `const`. What about `let`?

Well, TC39 made the decision: since we need a TDZ for `const`, we might as well have a TDZ for `let` as well. *In fact, if we make let have a TDZ, then we discourage all that ugly variable hoisting people do.* So there was a consistency perspective and, perhaps, a bit of social engineering to shift developers' behavior.

My counter-argument would be: if you're favoring consistency, be consistent with `var` instead of `const`; `let` is definitely more like `var` than `const`. That's especially true since they had already chosen consistency with `var` for the whole hoisting-to-the-top-of-the-scope thing. Let `const` be its own unique deal with a TDZ, and let the answer to TDZ purely be: just avoid the TDZ by always declaring your constants at the top of the scope. I think this would have been more reasonable.

But alas, that's not how it landed. `let` has a TDZ because `const` needs a TDZ, because `let` and `const` mimic `var` in their hoisting to the top of the (block) scope. There ya go. Too circular? Read it again a few times.

## Are Synchronous Callbacks Still Closures?

Chapter 7 presented two different models for tackling closure:

* Closure is a function instance remembering its outer variables even as that function is passed around and **invoked in** other scopes.

* Closure is a function instance and its scope environment being preserved in-place while any references to it are passed around and **invoked from** other scopes.

These models are not wildly divergent, but they do approach from a different perspective. And that different perspective changes what we identify as a closure.

Don't get lost following this rabbit trail through closures and callbacks:

* Calling back to what (or where)?
* Maybe "synchronous callback" isn't the best label
* ***IIF*** functions don't move around, why would they need closure?
* Deferring over time is key to closure

### What is a Callback?

Before we revisit closure, let me spend a brief moment addressing the word "callback." It's a generally accepted norm that saying "callback" is synonymous with both *asynchronous callbacks* and *synchronous callbacks*. I don't think I agree that this is a good idea, so I want to explain why and propose we move away from that to another term.

Let's first consider an *asynchronous callback*, a function reference that will be invoked at some future *later* point. What does "callback" mean, in this case?

It means that the current code has finished or paused, suspended itself, and that when the function in question is invoked later, execution is entering back into the suspended program, resuming it. Specifically, the point of re-entry is the code that was wrapped in the function reference:

```js
setTimeout(function waitForASecond(){
    // this is where JS should call back into
    // the program when the timer has elapsed
},1000);

// this is where the current program finishes
// or suspends
```

In this context, "calling back" makes a lot of sense. The JS engine is resuming our suspended program by *calling back in* at a specific location. OK, so a callback is asynchronous.

### Synchronous Callback?

But what about *synchronous callbacks*? Consider:

```js
function getLabels(studentIDs) {
    return studentIDs.map(
        function formatIDLabel(id){
            return `Student ID: ${
               String(id).padStart(6)
            }`;
        }
    );
}

getLabels([ 14, 73, 112, 6 ]);
// [
//    "Student ID: 000014",
//    "Student ID: 000073",
//    "Student ID: 000112",
//    "Student ID: 000006"
// ]
```

Should we refer to `formatIDLabel(..)` as a callback? Is the `map(..)` utility really *calling back* into our program by invoking the function we provided?

There's nothing to *call back into* per se, because the program hasn't paused or exited. We're passing a function (reference) from one part of the program to another part of the program, and then it's immediately invoked.

There's other established terms that might match what we're doing—passing in a function (reference) so that another part of the program can invoke it on our behalf. You might think of this as *Dependency Injection* (DI) or *Inversion of Control* (IoC).

DI can be summarized as passing in necessary part(s) of functionality to another part of the program so that it can invoke them to complete its work. That's a decent description for the `map(..)` call above, isn't it? The `map(..)` utility knows to iterate over the list's values, but it doesn't know what to *do* with those values. That's why we pass it the `formatIDLabel(..)` function. We pass in the dependency.

IoC is a pretty similar, related concept. Inversion of control means that instead of the current area of your program controlling what's happening, you hand control off to another part of the program. We wrapped the logic for computing a label string in the function `formatIDLabel(..)`, then handed invocation control to the `map(..)` utility.

Notably, Martin Fowler cites IoC as the difference between a framework and a library: with a library, you call its functions; with a framework, it calls your functions. [^fowlerIOC]

In the context of our discussion, either DI or IoC could work as an alternative label for a *synchronous callback*.

But I have a different suggestion. Let's refer to (the functions formerly known as) *synchronous callbacks*, as *inter-invoked functions* (IIFs). Yes, exactly, I'm playing off IIFEs. These kinds of functions are *inter-invoked*, meaning: another entity invokes them, as opposed to IIFEs, which invoke themselves immediately.

What's the relationship between an *asynchronous callback* and an IIF? An *asynchronous callback* is an IIF that's invoked asynchronously instead of synchronously.

### Synchronous Closure?

Now that we've re-labeled *synchronous callbacks* as IIFs, we can return to our main question: are IIFs an example of closure? Obviously, the IIF would have to reference variable(s) from an outer scope for it to have any chance of being a closure. The `formatIDLabel(..)` IIF from earlier does not reference any variables outside its own scope, so it's definitely not a closure.

What about an IIF that does have external references, is that closure?

```js
function printLabels(labels) {
    var list = document.getElementByID("labelsList");

    labels.forEach(
        function renderLabel(label){
            var li = document.createELement("li");
            li.innerText = label;
            list.appendChild(li);
        }
    );
}
```

The inner `renderLabel(..)` IIF references `list` from the enclosing scope, so it's an IIF that *could* have closure. But here's where the definition/model we choose for closure matters:

* If `renderLabel(..)` is a **function that gets passed somewhere else**, and that function is then invoked, then yes, `renderLabel(..)` is exercising a closure, because closure is what preserved its access to its original scope chain.

* But if, as in the alternative conceptual model from Chapter 7, `renderLabel(..)` stays in place, and only a reference to it is passed to `forEach(..)`, is there any need for closure to preserve the scope chain of `renderLabel(..)`, while it executes synchronously right inside its own scope?

No. That's just normal lexical scope.

To understand why, consider this alternative form of `printLabels(..)`:

```js
function printLabels(labels) {
    var list = document.getElementByID("labelsList");

    for (let label of labels) {
        // just a normal function call in its own
        // scope, right? That's not really closure!
        renderLabel(label);
    }

    // **************

    function renderLabel(label) {
        var li = document.createELement("li");
        li.innerText = label;
        list.appendChild(li);
    }
}
```

These two versions of `printLabels(..)` are essentially the same.

The latter one is definitely not an example of closure, at least not in any useful or observable sense. It's just lexical scope. The former version, with `forEach(..)` calling our function reference, is essentially the same thing. It's also not closure, but rather just a plain ol' lexical scope function call.

### Defer to Closure

By the way, Chapter 7 briefly mentioned partial application and currying (which *do* rely on closure!). This is a interesting scenario where manual currying can be used:

```js
function printLabels(labels) {
    var list = document.getElementByID("labelsList");
    var renderLabel = renderTo(list);

    // definitely closure this time!
    labels.forEach( renderLabel );

    // **************

    function renderTo(list) {
        return function createLabel(label){
            var li = document.createELement("li");
            li.innerText = label;
            list.appendChild(li);
        };
    }
}
```

The inner function `createLabel(..)`, which we assign to `renderLabel`, is closed over `list`, so closure is definitely being utilized.

Closure allows us to remember `list` for later, while we defer execution of the actual label-creation logic from the `renderTo(..)` call to the subsequent `forEach(..)` invocations of the `createLabel(..)` IIF. That may only be a brief moment here, but any amount of time could pass, as closure bridges from call to call.

## Classic Module Variations

Chapter 8 explained the classic module pattern, which can look like this:

```js
var StudentList = (function defineModule(Student){
    var elems = [];

    var publicAPI = {
        renderList() {
            // ..
        }
    };

    return publicAPI;

})(Student);
```

Notice that we're passing `Student` (another module instance) in as a dependency. But there's lots of useful variations on this module form you may encounter. Some hints for recognizing these variations:

* Does the module know about its own API?
* Even if we use a fancy module loader, it's just a classic module
* Some modules need to work universally

### Where's My API?

First, most classic modules don't define and use a `publicAPI` the way I have shown in this code. Instead, they typically look like:

```js
var StudentList = (function defineModule(Student){
    var elems = [];

    return {
        renderList() {
            // ..
        }
    };

})(Student);
```

The only difference here is directly returning the object that serves as the public API for the module, as opposed to first saving it to an inner `publicAPI` variable. This is by far how most classic modules are defined.

But I strongly prefer, and always use myself, the former `publicAPI` form. Two reasons:

* `publicAPI` is a semantic descriptor that aids readability by making it more obvious what the purpose of the object is.

* Storing an inner `publicAPI` variable that references the same external public API object returned, can be useful if you need to access or modify the API during the lifetime of the module.

    For example, you may want to call one of the publicly exposed functions, from inside the module. Or, you may want to add or remove methods depending on certain conditions, or update the value of an exposed property.

    Whatever the case may be, it just seems rather silly to me that we *wouldn't* maintain a reference to access our own API. Right?

### Asynchronous Module Defintion (AMD)

Another variation on the classic module form is AMD-style modules (popular several years back), such as those supported by the RequireJS utility:

```js
define([ "./Student" ],function StudentList(Student){
    var elems = [];

    return {
        renderList() {
            // ..
        }
    };
});
```

If you look closely at `StudentList(..)`, it's a classic module factory function. Inside the machinery of `define(..)` (provided by RequireJS), the `StudentList(..)` function is executed, passing to it any other module instances declared as dependencies. The return value is an object representing the public API for the module.

This is based on exactly the same principles (including how the closure works!) as we explored with classic modules.

### Universal Modules (UMD)

The final variation we'll look at is UMD, which is less a specific, exact format and more a collection of very similar formats. It was designed to create better interop (without any build-tool conversion) for modules that may be loaded in browsers, by AMD-style loaders, or in Node. I personally still publish many of my utility libraries using a form of UMD.

Here's the typical structure of a UMD:

```js
(function UMD(name,context,definition){
    // loaded by an AMD-style loader?
    if (
        typeof define === "function" &&
        define.amd
    ) {
        define(definition);
    }
    // in Node?
    else if (
        typeof module !== "undefined" &&
        module.exports
    ) {
        module.exports = definition(name,context);
    }
    // assume standalone browser script
    else {
        context[name] = definition(name,context);
    }
})("StudentList",this,function DEF(name,context){

    var elems = [];

    return {
        renderList() {
            // ..
        }
    };

});
```

Though it may look a bit unusual, UMD is really just an IIFE.

What's different is that the main `function` expression part (at the top) of the IIFE contains a series of `if..else if` statements to detect which of the three supported environments the module is being loaded in.

The final `()` that normally invokes an IIFE is being passed three arguments: `"StudentsList"`, `this`, and another `function` expression. If you match those arguments to their parameters, you'll see they are: `name`, `context`, and `definition`, respectively. `"StudentList"` (`name`) is the name label for the module, primarily in case it's defined as a global variable. `this` (`context`) is generally the `window` (aka, global object; see Chapter 4) for defining the module by its name.

`definition(..)` is invoked to actually retrieve the definition of the module, and you'll notice that, sure enough, that's just a classic module form!

There's no question that as of the time of this writing, ESM (ES Modules) are becoming popular and widespread rapidly. But with millions and millions of modules written over the last 20 years, all using some pre-ESM variation of classic modules, they're still very important to be able to read and understand when you come across them.

[^fowlerIOC]: *Inversion of Control*, Martin Fowler, https://martinfowler.com/bliki/InversionOfControl.html, 26 June 2005.
