# You Don't Know JS Yet: Scope & Closures - 2nd Edition
# 부록 A: 더 살펴보기

우리는 이제 이 책의 본문에서 다룬 많은 주제를 중심으로 여러 뉘앙스와 가장자리를 탐구할 것이다. 이 부록은 선택사항이고 참고자료이다.

어떤 사람들은 미묘한 구석 사례를 너무 깊이 들어간다는 것을 알아챈다. 다양한 의견은 소음과 산만함을 야기할 뿐이다. 개발자는 일반적으로 행해지는 길을 고수하면서 더 큰 이득을 얻는다. 나의 접근방식은 비실용적이고 비생산적이라는 비판을 받아왔다. 그것을 공유하지는 않더라도 그런 관점을 이해하고 감사하게 생각한다.

단지 추측과 호기심 부족으로 세부사항을 얼버무리는 것보다 일이 어떻게 돌아가는지에 대한 지식에 의해 힘을 얻는 것이 더 낫다고 믿는다. 궁극적으로 당신은 당신이 탐험하지 않은 조각에서 무언가가 거품을 내는 상황에 직면하게 될 것 이다. 다시 말해서 여러분은 모든 시간을 부드러운 *행복한 길*을 가는데 쓸 수 없을 것이다. 거친 길의 피할 수 없는 돌발에 대비하는 것이 낫지 않을까?

이러한 논의도 본문보다 나의 의견에 더 강하게 영향을 받을 것이다. 그러므로 이를 염두에 두고 내용을 고민해야 한다. 이 부록은 다양한 책 주제를 바탕으로 한 미니 블로그 글 모음집이다. 길고 깊은 잡초이기 때문에 천천히 하고 서두르지 마라.

## 암묵적 스코프<sub>scopes</sub>

스코프는 때때로 분명하지 않은 장소에 만들어진다. 실제로 이러한 암묵적 스코프는 프로그램 동작에 자주 영향을 미치지는 않지만, 실제로 발생한다는 것을 아는 것은 아주 도움이 된다. 깜짝 놀랄만한 스코프를 아래에서 살펴보자.

* 와 일치시키면, <sub>parameter</sub> 스코프
* 함수 이름 스코프

### 와 일치시키면,  스코프

2장에서의 대화 비유<sub>metaphor</sub>는 함수 와 일치시키면, 가 기본적으로 함수 스코프에 지역적으로 선언된 변수와 같다는 것을 암시한다. 하지만 항상 그렇지는 않다.

아래를 보자.

```js
// 외부/전역 스코프: 빨강(1)

function getStudentName(studentID) {
    // 함수 스코프: 파랑(2)

    // ..
}
```

여기서 `studentID`는 "단순" 와 일치시키면, 로 보인다. 그래서 그것은 파랑(2) 함수 스코프의 멤버로 행동한다. 그러나 만약 우리가 그것을 단순하지 않은 매개변수로 바꾼다면 엄밀히 더 이상 그렇지 않다. 단순하지 않은 매개변수 형식은 기본값이 있는 매개변수, (`...`를 사용하는) 나머지 매개변수<sub>rest parameters</sub> 그리고 구조 분해 할당된<sub>destructured</sub> 매개변수가 있다.

아래를 보자.

```js
// 외부/전역 스코프: 빨강(1)

function getStudentName(/*파랑(2)*/ studentID = 0) {
    // 함수 스코프: 초록(3)

    // ..
}
```

여기서 매개변수 목록은 본질적으로 자체 스코프가 되며 함수의 스코프는 *그* 스코프안으로 중첩된다.

왜? 어떤 차이가 그렇게 만들지? 단순하지 않은 매개변수 형식은 다양한 코너 케이스<sub>corner cases</sub>가 있기 때문에 매개변수 목록은 자체 스코프가 되어 그것들을 더 효과적으로 처리할 수 있다.

아래를 보자.

```js
function getStudentName(studentID = maxID, maxID) {
    // ..
}
```

왼쪽에서 오른쪽으로 수행되는 연산을 가정하면 `studentID` 매개변수의 기본값 `= maxID`는 `maxID`가 이미 존재하길(그리고 초기화도 된) 기대한다. 이 코드는 TDZ 에러(5장)를 발생시킨다. `maxID`가 매개변수 스코프에 선언되었지만 매개변수 순서상 아직 초기화되지 않았기 때문이다. 만약 매개변수 순서가 바뀐다면 TDZ 에러는 발생하지 않는다.

```js
function getStudentName(maxID,studentID = maxID) {
    // ..
}
```

기본 매개변수 자리에 함수 표현식을 도입하면 복잡성은 *어려운 환경속에서* 더욱 커진다. 그러면 암묵적 매개변수 스코프 범위에서 매개변수에 대한 자체 클로저(7장)을 생성할 수 있다.

```js
function whatsTheDealHere(id,defaultID = () => id) {
    id = 5;
    console.log( defaultID() );
}

whatsTheDealHere(3);
// 5
```

`defaultID()` 화살표 함수는 `id` 매개변수/변수를 클로즈 오버<sub>closes over</sub>하고, `5`로 재할당하기 때문에 이 코드는 말이 된다. 하지만 지금은 함수 스코프에서 `id`를 가리는 정의를 넣어보자.

```js
function whatsTheDealHere(id,defaultID = () => id) {
    var id = 5;
    console.log( defaultID() );
}

whatsTheDealHere(3);
// 3
```

워우! `var id = 5`는 `id` 매개변수를 가리고 있다. 그러나 `defaultID()` 함수의 클로저는 함수 본체의 가리고 있는 변수가 아닌 매개변수를 가진다. 이건 매개변수 목록을 둘러싼 스코프 버블이 있다는 것을 증명한다.

하지만 그것보다 더 미친짓을 보도록 하자!

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

여기서 이상한 점은 첫번째 콘솔 메세지이다. 이 순간에 가리고 있는 `id` 지역 변수는 그냥 선언된 `var id`이다. 5장의 논점은 보통 스코프의 상단에서 `undefined`로 자동 초기화된다는 것이다. 왜 `undefined`를 출력하지 않는가?

이런 특정한 코너 케이스에서(오래된 호환성을 이유로), JS는 `id`를 `undefined`로 자동 초기화하지 않지만 `id` 매개변수의 값(`3`)으로 대신한다.

이 순간에 2개의 `id`가 하나의 변수처럼 보일지라도, 실제로는 아직 분리(그리고 분리된 스코프로)되고 있다. `id = 5` 할당은 관찰할 수 있는 분기를 만드는데, `id` 매개변수는 `3`으로 유지하고 지역 변수는 `5`가 된다.

이런 이상한 뉘앙스에 물리는 걸 피하기 위한 내 충고는 아래와 같다.

* 지역 변수를 가진 매개변수를 가리지말라.

* 매개변수가 어떤 것이라도 클로즈 오버하는 기본 매개변수 함수를 사용하는 것을 피하라.

적어도 지금은 매개변수 목록이 매개변수가 어떤 것이라도 단순하지 않다면 자체 스코프라는 사실에 대해 깨닫고 조심할 수 있다.

### 함수 이름 스코프

3장의 "함수 이름 스코프" 섹션에서는 함수 표현식의 이름은 함수 자체 스코프에 추가된다고 주장했다. 상기해보자.

```js
var askQuestion = function ofTheTeacher(){
    // ..
};
```

`ofTheTeacher`는 둘러싸인 스코프(`askQuestion`이 선언된 곳)에 추가되지 않는다는 사실이다. 하지만 짐작할 수 있는 방식으로 함수의 스코프에 *그냥* 추가되는 것도 아니다. 암묵적 스코프의 또다른 이상한 코너 케이스이다.

함수 표현식의 이름 식별자는 둘러싸인 스코프와 안쪽의 함수 스코프 사이에 중첩된 자신의 암묵적 스코프 안에 있다.

만약 `ofTheTeacher`가 함수의 스코프안에 있었다면 아래와 같은 에러가 예상된다.

```js
var askQuestion = function ofTheTeacher(){
    // 왜 중복 선언 에러가 아니지?
    let ofTheTeacher = "Confused, yet?";
};
```

`let` 선언 형식은 재선언을 허락하지 않는다(5장 참고). 하지만 이건 완전히 유효한 가리기이고, 재선언이 아니다. 2개의 `ofTheTeacher` 식별자는 분리된 스코프에 있기 때문이다.

함수의 이름 식별자 스코프가 문제되는 경우는 거의 없을 것이다. 하지만 다시, 이런 메커니즘이 어떻게 동작하는지 아는 것은 좋다. 물리는 것을 피하기 위해서는 함수 이름 식별자를 가리지 말아라.

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

`let`과 `const`은 호이스팅 되더라도, TDZ에서 그 변수들을 사용할 수 없다(5장을 참고해라). 그래서 다음 논의는 오직 `var`에만 적용된다. 진행에 앞서, 대부분의 경우에 *변수 호이스팅*은 나쁜 아이디어라는 것에 완전히 찬성한다는 사실을 밝힌다.

```js
pleaseDontDoThis = "bad idea";

// 훨씬 후에
var pleaseDontDoThis;
```

역순으로 두는 것이 *함수 호이스팅*에서는 유용한 반면, 변수의 경우 그것은 코드를 추론하는 것을 더 어렵게 한다.

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

그러나 이 값들을 위에 배치해야 하는 소수의 드문 경우가 있긴하다. 모듈의 퍼블릭 API를 내보내기전에 말이다. 예를 들어:

```js
// public API
var publicAPI = Object.assign(module.exports,{
    getStudents,
    addStudents,
    refreshData: refreshData.bind(null,cache)
});
```

`cache` 값이 퍼블릭 API(`.bind(...)` 부분 응용 프로그램)의 초기화에 사용되므로 변수에 이미 값이 할당되어 있어야 한다.

`var cache = {..}`만 이동하면 될까? 이 퍼블릭 API 초기화 위에? 글쎄, 아마도. 그러나 이제는 `var cache`가 *private* 구현 세부 정보라는 것이 명확하지 않다. 이것에 대한 절충안은 다음과 같다.

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

*변수 호이스팅*이 보이나? `cache`를 그것이 속하는 아래쪽에 선언하였다. 하지만 아주 드물게 코드 위쪽에 초기화가 필요한 영역에 사용하고 있다. 코드 코멘트에 `cache`에 할당된 값에 대한 힌트를 남겼다.

*변수 호이스팅*을 활용하여 변수의 선언보다 초기에 변수를 할당한 사례는 이 정도밖에 없다. 하지만 이런 예외에 호이스팅을 신중하게 사용하는 것은 합리적이라고 생각한다. 

## `var` 케이스

*변수 호이스팅*을 다루기 위해, `var`에 대해서 샅샅히 까발려보자, 악당 개발자는 JS 개발이 가진 많은 고민들을 탓하는 것을 좋아한다. 5장에서, 우리는 `let`/`const`를 다뤘고 그리고 `var`가 잘 작동하는 곳에서 다시 이 챕터에 대해 이야기해보기로 했다.

케이스를 정리했으니, 놓치지 말자:

* `var` 는 절대 고장난 적이 없었다.
* `let` 은 당신의 친구
* `const` 는 제한된 사용상을 가진다.
*  양쪽 세계에서의 최고의 경우: `var` *그리고* `let`

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

함정을 만드는 것은 안 될 것 같아 내가 유일하게 `const`를 사용하는 때는 '42'나 '안녕, 친구들!'처럼 이미 불변의 값을 할당하고 있을 때, 그리고 그것이 문자 그대로의 가치, 의미적 목적으로 이름 붙여진 플레이스홀더<sub>placeholder</sub>라는 의미에서 분명히 '상수'일 때이다. const가 가장 잘 쓰이는 이유다. 내 코드에서 그런 경우는 드물다.

변수 재할당이 큰 문제라면, `const`가 더 유용하다. 하지만 변수 재할당은 버그를 일으키는 측면에서 그리 큰 문제가 아니다. 프로그램에 버그를 일으키는 많은 것들이 있지만, "우발적인 재할당"은 그 목록에서 훨씬 아래쪽에 있다.

`const`(그리고 `let`)는 블록에 사용되어야 하고 블록은 짧아야 한다는 사실과 결합하면 const 선언을 적용할 수 있는 매우 작은 영역의 코드를 가질 수 있다. 10번째 줄 블록 중 1줄에 있는 `const`는 다음 9줄에 대한 내용만 알려준다. 그리고 이 9줄의 코드를 내려다보면 이미 알 수 있다: 변수는 절대 '='의 왼쪽에 있지 않으며, 재할당되지 않는다.

그게 `const`가 진짜 하는 전부다. 그 외에는 별로 쓸모가 없다. 값 vs 할당 불변성의 중대한 혼돈에 맞서서 `const`는 많은 빛을 잃는다.

재할당되지 않는 `let` (또는 `var`!)은 비록 컴파일러가 보장하지 않더라도 이미 상수이다. 대부분의 경우 그것으로 충분하다.

### `var` *그리고* `let`

내 생각에 `const`는 쓸모가 거의 없는데, let과 var의 두 마리 토끼 싸움에 불과하다. 하지만 꼭 한 명의 우승자가 있어야 하는 것은 아니기 때문에, 그것은 또한 진정한 경주가 아니다. 둘 다 이길 수 있다... 다른 경주에서.

사실 프로그램에서는 `var`와 `let`을 모두 사용해야 한다. `let`이 호출되는 곳에 `var`를 사용해서는 안 되지만 `var`가 가장 적절한 곳에 `let`을 사용해서는 안 된다.

그렇다면 `var`를 어디에 사용해야 할까? 어떤 상황에서 그것이 `let`보다 나은 선택일까?

한 가지 예로, 나는 함수의 시작, 중간, 끝에 상관없이 모든 함수의 최상위 스코프에서 항상 'var'를 사용한다. 글로벌 스코프의 사용을 최소화하려고 노력하지만 글로벌 스코프에서도 var를 사용한다.

함수 스코프에 `var`를 사용하는 이유는 무엇일까? 그것이 바로 `var`가 하는 일이기 때문이다. 말 그대로 선언을 스코핑하는 함수에서 25년 동안 정확히 해온 선언자보다 더 좋은 도구는 없다.

이 최상위 스코프에서, `let`을 *사용할 수 있지만*, 해당 작업에 가장 적합한 툴은 아니다. 또 어디서나 `let`을 사용하면 어떤 선언이 지역화 되도록 되어 있는지, 어떤 선언이 기능 전반에 사용되도록 되어 있는지 잘 알 수 없다.

반대로 블록 안에서 `var`를 사용하는 경우는 거의 없다. 그래서 `let`이 있는 것이다. 작업에 가장 적합한 도구를 사용해라. `let`이 보이면 지역 선언을 다루고 있음을 알려준다. `var`가 표시되면 함수 차원의 선언을 다루고 있음을 나타낸다. 그렇게 간단하다.

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

`studentRecords` 변수는 전체 함수에서 사용하기 위한 것이다. `var`'는 독자들에게 그것을 말해주는 최고의 선언자이다. 반대로 `record`와 `id`는 루프 반복의 좁은 범위에서만 사용하기 때문에 `let`이 해당 작업에 가장 적합한 도구이다.

이 *최상의 도구* 주장에 더하여, `var`는 특정 제한된 상황에서 더 강력하게 만드는 몇 가지 다른 특성을 가지고 있다.

예를 들어, 루프가 변수를 배타적으로 사용하고 있지만 조건 절은 이터레이션 내에서 블록 스코프 선언을 볼 수 없는 경우를 들 수 있다.

```js
function commitAction() {
    do {
        let result = commit();
        var done = result && result.code == 1;
    } while (!done);
}
```

여기서, `result`는 블록 안에서만 사용되므로 `let`을 사용한다. 그러나 `done`은 조금 다르다. 이것은 루프에만 유용하지만 `while` 절에서는 루프 내부에 나타나는 `let` 선언을 볼 수 없다. 그래서 우리는 타협하고 `var`를 사용하여 `done`이 보이는 외부 범위로 올라가도록 한다.

루프 밖에서 `done`을 선언하는 대안은 처음 사용된 곳에서 분리하여 할당하기 위해 기본값을 선택해야 하거나, 더 나쁘게는 할당되지 않은 상태로 있어 독자에게 모호하게 보일 수 있다. 나는 루프 내부의 `var`가 더 좋다고 생각한다.

`var`의 또 다른 유용한 특징은 의도하지 않은 블록 내부에 선언이 있다는 것이다. 의도하지 않은 블록은 구문에 블록이 필요하기 때문에 생성되는 블록이지만 개발자의 의도는 현지화된 범위를 만드는 것이 아니다. 의도하지 않은 범위의 가장 좋은 예는 `try..catch` 이다:

```js
function getStudents() {
    try {
        // 실제로는 블록 스코프가 아니다.
        var records = fromCache("students");
    }
    catch (err) {
        // 웁스 디폴트가 되었다.
        var records = [];
    }
    // ..
}
```

이 코드를 구성하는 다른 방법이 있다. 하지만 여러 가지 단점을 고려할 때 이것이 *가장 좋은* 방법이라고 생각한다.

`records`(`var` 또는 `let`이 있는)를 `try` 블록 밖으로 선언하고 나서, 어느 한 블록 또는 양쪽 블록으로 할당하고 싶지 않다. 초기 선언은 가능한 한 변수의 첫 번째 사용에 가까운(이상적으로 같은 행) 것이 좋다. 이 간단한 예에서는 몇 개의 회선 거리에 불과하지만 실제 코드에서는 더 많은 회선까지 늘어날 수 있다. 갭이 클수록 할당하는 스코프에서 변수를 파악하는 것이 어려워진다. 실제 할당에서 사용되는 `var`를 사용하면 덜 모호해진다.

또, `try`블록과 `catch`블록 양쪽에서 `var`를 사용하고 있는 것에 주의해라. 어떤 길을 가더라도 `records`는 반드시 선언된다는 것을 독자에게 알리고 싶기 때문이다. 엄밀히 말하면 `var`는 함수 스코프로 한 번 호이스트된다. 그러나 그것은 여전히 독자들에게 `var`가 보장하는 것을 상기시키는 좋은 의미 신호이다. 한 블록에서만 `var`를 사용하고 다른 블록만 읽고 있다면 레코드가 어디서 왔는지 쉽게 알 수 없을 것이다.

이것은 내가 보기에 `var`의 작은 초능력이다. 의도하지 않은 `try..catch`블록에서 벗어날 수 있을 뿐만 아니라, 함수의 스코프에는 여러 번 표시될 수 있다. `let`으로는 그럴 수 없다. 나쁘지 않다. 사실 조금 도움이 되는 기능이다. `var`는 변수 출처를 알려주는 선언적 주석과 비슷하다. "아하, 맞아, 함수 전체에 속하지."

이 반복 주석 슈퍼파워는 다른 경우에 유용하다.

```js
function getStudents() {
    var data = [];

    // data 를 다룬다.
    // .. 50줄 이상의 코드 ..

    // var data를 상기시키는 순수한 주석


    // data를 다시 사용한다.
    // ..
}
```

두 번째 `var data`는 `data`를 다시 선언하는 것이 아니라 `data`가 함수 전체의 선언이라는 독자들의 이익을 위해 주석을 다는 것이다. 이렇게 하면 리더는 초기 선언을 찾기 위해 50줄 이상의 코드를 스크롤할 필요가 없다.

함수 범위 전체에서 변수를 여러 목적으로 재사용하는 것은 문제 없다. 두 개의 변수를 여러 줄의 코드로 구분하는 것도 문제 없다. 어느 경우든, var를 사용하여 안전하게 재선언(주석)할 수 있으면 함수의 어디에 있든 데이터 출처를 알 수 있다.

다시 말하지만 슬프게도 `let`은 이것을 할 수 없다.

var가 어느 정도 도움을 주는 것으로 판명될 때 다른 뉘앙스와 시나리오가 있지만 더 이상 강조하지는 않을 것이다. 중요한 점은 var가 let(가끔 const)와 함께 프로그램에서 유용하게 쓰일 수 있다는 점이다. 독자들에게 더 풍부한 이야기를 들려주기 위해 JS 언어가 제공하는 도구를 창의적으로 사용할 의향이 있습니까?

더 이상 멋지지 않다고 치욕감을 줬다고 해서 `var` 같은 유용한 도구를 그냥 버리지 마라. 몇 년 전에 한 번 헷갈렸다고 해서 `var`를 피하지 마라. 이러한 툴을 학습하고, 각각의 툴을 자신의 가장 뛰어난 기능에 사용할 수 있다.

## TDZ는 무엇을 다루는가?

TDZ(일시 데드존)는 5장에서 설명되었다. 우리는 그것이 어떻게 일어나는지 설명했지만, 애초에 소개가 필요한 *이유*에 대한 설명은 대충 훑어보았다. TDZ의 동기에 대해 간단히 살펴보겠다.

TDZ의 유래 스토리에 기재되어 있는 부차적 네비게이션:

* `const`은 절대 변경되면 안된다.
*  이것은 모두 시간에 관한 것이다.
*  `let`은 `const` 와 `var`처럼 작동해야 하나?

### 어디에서 이것은 시작되었는가

TDZ는 실제로 `const`로 부터 왔다.

TC39는 초기 ES6 개발 과정에서 `const`(및 `let`)를 블록의 맨 위로 끌어올릴지 여부를 결정해야 했다. 그들은 이 선언들이 `var`와 같이 호이스팅하는 것을 결정했다. 그렇지 않다면, 다음과 같은 중간 스코프 섀도잉과의 혼동에 대한 때문이였다고 생각한다.

```js
let greeting = "Hi!";

{
    // 무엇을 여기에 출력해야 하나
    console.log(greeting);

    // .. 엄청 긴 코드 ..

    // `greeting` 변수를 섀도잉한다.
    let greeting = "Hello, friends!";

    // ..
}
```

`console.log(..)` 구문을 어떻게 해야 할까?? JS 개발자들은 이것이 "Hi!"를 출력 하는 것을 이해할까? 블록의 전반부이 아닌 후반부에만 섀도우킥을 넣는 것은 감쪽같다. 이것은 매우 직관적이지 않고 JS와 같은 동작이다. 따라서 `let`와 `const`는 최상단으로 호이스트되서 전체에서 접근가능하여야 한다.

그러나 `let`과 `const` 블록의 맨 위로 호이스팅 된다면(`var`가 함수의 맨 위로 호이스팅된 것 처럼) `var`처럼 `let`과 `const`가 자동 초기화(`undefined`로)하는 것은 어떨까. 주요 우려 사항은 다음과 같다.

```js
{
    // 무엇을 여기에 출력해야 하나
    console.log(studentName);

    // 후에

    const studentName = "Frank";

    // ..
}
```

`studentName`이 이 블록의 맨 위로 호이스팅 되었을 뿐만 아니라 `undefined`으로 자동 초기화되었다고 가정하자. 블록의 전반부에서, `studentName`은 `console.log(..)`와 같이 `undefined` 가지고 있는 것을 확인할 수 있다. 일단 `const studentName = ..` 구문에 도달하면 `studentName`에 `"Frank"가 할당된다. 그 이후로는 `studentName`은 재할당할 될 수 없다.

그러나 상수가 관측 가능한 두 개의 다른 값, 즉 처음에는 `undefined`, 그 다음에는 `"Frank"`를 갖는다는 것이 이상하거나 놀랍지 않은가? 그것은 우리가 생각하는 '상수'의 의미와 배치되는 것으로 보인다. 즉, 하나의 값만 가질 수 있어야 한다.

그래서... 이제 문제가 생겼다. `studentName`을 `undefined`(또는 그 외의 값)로 자동 초기화할 수 없다. 그러나 변수는 전체 스코프에 걸쳐 존재해야 한다. 처음 존재한 시점(스코프의 시작)부터 값이 할당된 시점까지 우리는 무엇을 해야할까?

이 기간을 '일시적 사각지대'(TDZ)과 같이 '사각지대'이라고 부른다. 혼란을 방지하기 위해 TDZ에 있는 변수의 모든 종류의 접근은 허용되지않으며 이는 TDZ 오류를 야기하도록 결정되었었다.

오케이, 그 논리도 일리가 있다, 인정한다.

### 누가 TDZ에 `let`을 허용할까 

하지만 그건 `const`일 뿐이다. `let`은 어떠한가?

TC39는 `const`는 TDZ가 필요하니 `let`도 TDZ가 있어야 한다는 결정을 내렸다. 사실 TDZ를 만들면 못생긴 변수 호이스팅을 시도하는 사람들을 단념시킬 수 있다. 그래서 개발자들의 행동을 변화시키기 위한 일관성 있는 관점과 약간의 사회 공학이 있었다.

내 반론은 일관성을 선호한다면 `const`가 아닌 `var`로 일관하라는 것이다. `let`은 `const`보다 `var`에 더 가깝다는 것이다. 특히 모든 것을 최상의 스코프로 호이스팅하는 일에서 `var`로 일관성을 택한 터라 더욱 그렇다. `const`를 TDZ의 고유의 대상으로 하여, TDZ를 순수하게 하고: 항상 스코프 상위에 상수를 선언함으로써 TDZ를 피하는 것이다. 이게 더 합리적이었을 것 같다.

하지만 안타깝게도, 그것은 해결방법이 아니다. `const`는 TDZ가 필요하기 때문에 `let`은 TDZ를 가진다, `let`과 `const`는 (블록)스코프의 최상단으로 호이스팅한다는 점에서 `var`를 흉내내기 때문이다. 자, 다 끝났다. 너무 빙빙 돌렸나? 몇 번 더 읽어봐라.

## 동기식 콜백은 여전히 클로저일까?

7장에서 클로저를 해결하기 위해 2가지 다른 모델을 제시했다.

* 클로저는 함수가 전달되어 바깥 스코프에서 **호출해도** 외부 변수를 기억하는 함수 인스턴스다.

* 클로저는 함수 인스턴스와 해당 스코프 환경이 제자리에 유지되는 동안 이에 대한 참조가 전달되고 다른 스코프에서 **호출**된다.

이러한 모델들은 매우 다양하지는 않지만 다른 관점에서 접근한다. 그리고 이 다른 관점들은 우리가 클로저를 식별하는 것을 바꾼다.

클로저와 콜백에 대한 여담을 잘 따라와보길 바란다.

* 무엇을(또는 어디에서) 다시 호출할까?
* "동기식 콜백"이 최고의 명명이 아닐 수도 있다.
* ***IIF*** 함수는 이동하지 않는다. 왜 클로저가 필요한가?
* 시간이 지남에 따라 지연시키는 것이 클로저의 열쇠다.

### 콜백이란 무엇인가?

클로저를 다시 살펴보기 전에 "콜백"이라는 단어에 대해 잠시 살펴보자. "콜백"이 *비동기 콜백* 및 *동기 콜백*과 동의어라는 것은 일반적으로 받아들여지는 규범이다. 나는 이것이 좋은 생각은 아닌 것 같다. 그래서 그 이유를 설명하고 다른 용어로 바꿀 것을 제안한다.

먼저 *나중* 시점에 호출될 함수 참조인 *비동기 콜백*을 살펴보자. 이 경우 "콜백"은 무엇을 의미할까?

이는 현재 코드가 완료되었거나 일시 중지되었으며 자체적으로 일시 중단되었고 해당 함수가 나중에 호출될 때 실행이 일시 중단된 프로그램으로 돌아가 다시 시작한다는 것을 의미한다. 구체적으로 재진입 지점은 함수 참조에 래핑된 코드다.

```js
setTimeout(function waitForASecond(){
    // 타이머가 경과했을 때
    // js가 프로그램을 다시 호출해야하는 곳
},1000);

// 현재 프로그램이 종료되는 곳
// 혹은 정지되는 곳
```

이러한 맥락에서 "콜백"은 많은 의미가 있다. JS 엔진은 특정 위치에서 *호출*하여 일시 중단된 프로그램을 다시 시작한다. 그렇다. 콜백은 비동기식이다.

### 동기식 콜백이란?

하지만 *동기식 콜백*은 어떨까? 다음 코드를 살펴보자:

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

콜백으로 `formatIDLabel(..)`을 참조해야 할까? `map(..)` 유틸리티가 실제로 우리가 제공한 함수를 호출하여 우리 프로그램으로 *콜백*할까?

프로그램이 일시 중지되거나 종료되지 않았기 때문에 그 자체로 *다시 돌아가 호출*할 것이 없다. 프로그램의 한 부분에서 프로그램의 다른 부분으로 함수(참조)를 전달하면 즉시 호출된다.

우리가 하고있는 일과 일치하는 다른 정해진 용어가 있다. 프로그램의 다른 부분이 우리를 대신하여 호출할 수 있도록 함수(참조)를 전달하는 것, 이것은 *의존성 주입<sub>Dependency Injection</sub>*(DI) 또는 *제어 역전<sub>Inversion of Control</sub>*(IoC)이라고 생각할 수 있다.

DI는 기능의 필요한 부분을 프로그램의 다른 부분으로 전달하여 작업을 완료하기 위해 호출할 수 있는 것으로 요약할 수 있다. 위의 `map(..)` 호출에 대한 적절한 설명이다. 그렇지 않나? `map(..)` 유틸리티는 목록의 값을 반복하는 것은 알고 있지만 해당 값으로 *무엇을 해야 하는지는* 알지 못한다. 이것이 `formatIDLabel(..)` 함수를 전달하는 이유다. 의존성을 전달한다.

IoC는 매우 유사하고 연관있는 개념이다. 제어 역전이란 무슨 일이 일어나고 있는지 제어하는 프로그램의 현재 영역 대신 프로그램의 다른 부분에 제어를 넘겨주는 것을 의미한다. 함수 `formatIDLabel(..)`에서 레이블 문자열을 계산하기 위한 논리를 래핑한 다음 호출 제어를 `map(..)` 유틸리티에 넘겼다.

특히 Martin Fowler는 프레임워크와 라이브러리의 차이점으로 IoC를 인용한다. 라이브러리에서는 당신이 함수를 호출한다. 프레임워크는 너의 함수를 호출한다. [^fowlerIOC]

논의의 맥락에서 DI 또는 IoC는 *동기식 콜백*에 대한 다른 이름이 될 수 있다.

하지만 다른 제안이 있다. *동기식 콜백*을 *교차 실행 함수<sub>inter-invoked functions</sub>*(IIFs)라고 부르자. 그렇다, 정확히는 IIFE를 이야기하고 있다. 이러한 종류의 함수는 *교차 실행*된다. 즉, 자체적으로 즉시 호출하는 IIFE와 달리 다른 엔티티가 호출한다.

*비동기 콜백*과 교차 실행 함수 사이의 관계는 무엇일까? *비동기 콜백*은 동기 대신 비동기적으로 호출되는 교차 실행 함수다.

### 동기식 클로저란?

이제 *동기식 콜백*에 교차 실행 함수라는 이름을 다시 지정했으므로 주요 질문으로 돌아갈 수 있다. 교차 실행 함수가 클로저의 예인가? 분명히 교차 실행 함수는 클로저가 되려면 외부 스코프의 변수를 참조해야 한다. 이전의 `formatIDLabel(..)` 교차 실행 함수는 자체 스코프 밖의 변수를 참조하지 않으므로 확실히 클로저가 아니다.

외부 참조가 있는 교차 실행 함수는 클로저인가?

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

내부 `renderLabel(..)` 교차 실행 함수는 둘러싸는 스코프의 `list`를 참조하므로 클로저가 *있을 수 있는* 교차 실행 함수다. 그러나 클로저를 위해 선택한 정의/모델은 다음과 같다.

* `renderLabel(..)`이 **다른 곳에서 전달되는 함수**이고 그 함수가 호출되면, 물론 `renderLabel(..)`는 원래 스코프 체인에 대한 접근을 보존하기 위해 클로저를 실행하고 있는 것이다.

* 그러나 7장의 대안 모델에서와 같이 `renderLabel(..)`이 제자리에 있고 이에 대한 참조만 `forEach(..)`로 전달되면 `renderLabel(..)`의 스코프 체인이 자체 스코프 안에서 동기적으로 실행되는 동안 클로저가 필요할까?

아니다. 이는 그냥 일반적인 렉시컬 스코프다.

이유를 이해하려면 `printLabels(..)`의 대안 형태를 봐보자:

```js
function printLabels(labels) {
    var list = document.getElementByID("labelsList");

    for (let label of labels) {
        // 자체 스코프에서 정상적으로 함수 호출이 되고있다. 맞나?
        // 이는 실제로 클로저가 아니다!
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

이 두 버전의 `printLabels(..)`은 본질적으로 동일하다.

후자는 적어도 유용하거나 관찰 가능한 의미에서 클로저의 예가 아니다. 그것은 단지 렉시컬 스코프다. 함수 참조를 호출하는 `forEach(..)`가 있는 이전 버전은 본질적으로 동일하다. 이 또한 클로저가 아니라 그냥 평범한 렉시컬 스코프 함수 호출이다.

### 클로저로 지연시키기

그건 그렇고, 7장에서 부분 적용<sub>partial application</sub>과 커링<sub>currying</sub>에 대해서 간략하게 언급했다(이것도 클로저에 의존한다!). 이것은 정석적으로 커링이 사용되는 흥미로운 시나리오다.

```js
function printLabels(labels) {
    var list = document.getElementByID("labelsList");
    var renderLabel = renderTo(list);

    // 이번엔 완벽히 클로저다!
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

우리가 `renderLabel`에 할당한 내부 함수 `createLabel(..)`은 `list` 에 대해 클로즈 오버 되어있으므로 클로저는 확실히 활용되고 있다.

클로저는 `renderTo(..)` 호출에서 `createLabel(..)` 교차 실행 함수의 후속 `forEach(..)` 호출로 실제 레이블 생성 논리 실행을 지연시키는 동안 `list`를 나중을 위해 기억할 수 있게 해준다. 여기에서는 짧은 순간일 수 있지만 클로저가 호출에서 호출로 연결되기 때문에 많은 시간이 지나갈 수도 있다.

## 클래식 모듈 변형

8장에서 설명한 클래식 모듈 패턴은 아래와 같다.

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

의존성으로서 `Student`(다른 모듈 인스턴스)를 전달하고 있는 것을 주목하라. 그러나 당신이 마주칠만한 이 모듈 형태에는 많은 쓸만한 변형이 있다. 이러한 변형을 알아채기 위한 몇 가지 힌트는 아래와 같다.

* 모듈이 자신의 API를 알고 있는가?
* 복잡한 모듈 로더를 사용하더라도 단지 클래식 모듈일 뿐이다.
* 어떤 모듈은 전체적으로 작동할 필요가 있다.

### 내 API는 어디에?

먼저 대부분의 클래식 모듈은 아래 코드에서 보여지는 방법처럼 `publicAPI`를 정의하지 않고 사용한다. 보통 아래와 같다.

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

여기서 유일한 차이점은 모듈을 위한 공개 API로서 제공하는 객체를 직접적으로 반환하고 있다. 내부의 `publicAPI` 변수에 먼저 저장하는 것과 반대이다. 이것이 단연코 대부분의 클래식 모듈이 정의되는 방법이다.

그러나 나는 이전 `publicAPI` 형태를 강하게 선호하고 항상 사용한다. 두 가지 이유가 있다.

* `publicAPI`는 객체의 목적을 명확하게 함으로써 가독성에 초점을 맞춘 의미적인 서술어이다.

* 반환되는 외부의 공개 API 객체를 참조하는 내부의 `publicAPI` 변수를 저장하는 것은, 만약 모듈의 생명주기 동안 API에 접근하거나 수정할 필요가 있다면 유용할 수 있다.

    예를 들어, 모듈 안에서 공개적으로 노출된 함수 중의 하나를 호출 할 수 있다. 또는 특정한 조건에 의존적인 메서드를 추가하거나 삭제하길 원하거나, 노출된 속성의 값을 수정하길 원할 수도 있다.

    무슨 경우이던지 간에, 자신의 API에 접근하기 위한 참조를 유지하지 *않을* 것이라는 것은 다소 바보 같은 생각이다. 그렇지?

### 비동기 모듈 정의 (AMD)

클래식 모듈 형식의 다른 변형은 AMD 스타일 모듈이고(지난 몇 년간 인기있던), RequireJS 유틸리티에 의해 아래처럼 지원되었다.

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

`StudentList(..)`를 자세히 보면, 클래식 모듈 팩토리 함수이다. (RequireJS가 제공하는) `define(..)` 장치 안에서 실행되고, 의존성으로 선언된 다른 모듈 인스턴스를 전달한다. 반환값은 모듈의 공개 API를 나타내는 객체이다.

이것은 우리가 클래식 모듈로 탐구했던 것과 정확하게 (클로저가 동작하는 방식을 포함하여) 같은 원리에 기반한다.

### 유니버설 모듈 (UMD)

마지막으로 살펴볼 변형은 UMD이다. UMD는 구체적이고, 정확한 형식이 아니라 매우 비슷한 형태의 모음이다. 브라우저, AMD 스타일 로더, Node에서 로드할 수 있는 모듈을 위한 더 나은 (어떤 빌드툴 변환 없이) 상호운용성을 만들기 위해 설계되었다. 나는 개인적으로 여전히 UMD 형식을 사용하여 대부분의 내 유틸리티 라이브러리를 배포한다.

UMD의 일반적인 구조는 아래와 같다.

```js
(function UMD(name,context,definition){
    // AMD 스타일 로더에 의해 로드되었는가?
    if (
        typeof define === "function" &&
        define.amd
    ) {
        define(definition);
    }
    // Node에서?
    else if (
        typeof module !== "undefined" &&
        module.exports
    ) {
        module.exports = definition(name,context);
    }
    // 독립적인 브라우저 스크립트로 가정
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

조금 특이해 보일지라도, UMD는 정말로 IIFE일 뿐이다.

다른 점은 IIFE의 (상단에 있는) 메인 `function` 표현식 부분이 모듈이 로드되는 세 가지 지원 환경을 알아내기 위해 `if..else if` 구문 시리즈를 가지고 있다는 점이 차이점이다.

보통 때는 IIFE를 실행하는 마지막 `()`는 세 가지 인수 `"StudentsList"`, `this`와 다른 `function` 표현식을 전달하게 된다. 이러한 인수를 매개변수와 일치시키면, `name`, `context`와 `definition`을 볼 것이다. `"StudentList"` (`name`)는 모듈을 위한 이름 라벨이다. 주로 전역 변수로 정의된 경우이다. `this` (`context`)는 일반적으로 이름으로 모듈을 정의하기 위한 `window`(전역 객체, 4장 참고)이다.

`definition(..)`은 모듈의 정의를 실제로 조회하기 위해 실행된다. 그러면 충분히 알아차릴 것이다. 클래식 모듈 형식일 뿐이다!

ESM(ES 모듈)이 빠르게 대중화되고 퍼지고 있다는 것은 이 글을 쓰는 현재 의심의 여지가 없다. 그러나 지난 20년 넘게 작성된 수백만개의 모듈이 모두 클래식 모듈의 이전 ESM 변형을 사용하고 있다. 이것들을 접했을 때 읽고 이해할 수 있는 것은 여전히 매우 중요하다.

[^fowlerIOC]: *Inversion of Control*, Martin Fowler, https://martinfowler.com/bliki/InversionOfControl.html, 2005년 6월 26일.
