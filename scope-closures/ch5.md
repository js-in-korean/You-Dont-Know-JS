# You Don't Know JS Yet: Scope & Closures - 2nd Edition
# 5장: 변수의 비밀 라이프사이클

이제 여러분은 전역 스코프에서 하위의 프로그램의 스코프를 호출하는 스코프들의 중첩에 대해 이제 어느정도 이해하고 있을 것이다.

그러나 변수가 어떤 스코프로부터 오는지에 대해서는 잘 모를 것이다. 만약 변수 선언이 스코프 첫 번째 문<sub>statement</sub> 이후에 선언되었다면, 선언 전에 해당 식별자에 대한 참조는 어떻게 동작할까? 만약 동일한 변수를 스코프안에 두 번 선언한 경우는?

JS의 렉시컬 스코프에 대한 특별한 특징은 어떻게 그리고 언제 변수가 프로그램에 존재하게 되고 사용가능한지에 대한 다양한 뉘앙스를 가진다.

## 언제 변수를 사용할 수 있을까?

변수는 언제 그것의 스코프 내에서 사용할 수 있게 될까? 명백한 답이 있는 것 같기도 하다: 변수가 *선언/작성된 후* 말이다. 그럴까? 하지만 이것은 충분한 답변이 아니다.

다음 코드를 살펴보자:

```js
greeting();
// Hello!

function greeting() {
    console.log("Hello!");
}
```

이 코드는 잘 작동한다. 전에도 이런 코드를 보거나 작성해 본 적이 있을 것이다. 하지만 그게 어떻게, 왜 작동하는지 궁금해 한 적 있는가? 구체적으로, 4행까지는 `greeting` 함수 선언이 일어나지 않는데 왜 1행(함수 참조 검색 및 실행)에서 `greeting` 식별자에 접근할 수 있는가?

1장에서 설명했듯이 컴파일 시간 동안 모든 식별자가 해당 범위에 등록된다. 또한 모든 식별자는 **자신이 속한 스코프가 시작 될때마다** 해당 스코프의 시작 부분에 *만들어진다*.

변수 선언이 스코프 내에서 더 아래쪽으로 나타나더라도 변수를 해당 스코프의 처음부터 볼 수 있는 경우를 주로 **호이스팅<sub></sub>**라고 명칭한다.

하지만 호이스팅만으로는 질문에 충분한 답이 되지 않는다. 우리는 스코프 처음부터 `greeting`라는 식별자를 볼 수 있는데, 왜 우리는 `greeting` 함수가 선언되기 전에 `greeting`' 함수 **호출**할 수 있을까?

다시 말해, 스코프가 실행되기 시작하는 순간부터, 어떻게 변수 `greeting`는 어떤 값(함수 참조)을 가지게 될까? 정답은 *함수 호이스팅*이라는 공식`function` 선언의 특수한 특성 때문이다. `function` 선언의 이름 식별자가 해당 범위의 맨 위에 등록되면 해당 함수 참조값으로 자동 초기화된다. 그렇기 때문에 기능을 전체 범위에서 호출할 수 있다!

한 가지 중요한 세부 사항은 *함수 호이스팅*과 `var` 가 취하는 *변수 호이스팅* 모두 이름 식별자를 블록 스코프가 아닌 가장 가까운 **함수 스코프**(이것이 없는 경우는 스코프 범위)에 부착한다는 것이다.

| 비고: |
| :--- |
| `let`과 `const`가 포함된 선언문은 여전히 호이스트(이 장 뒷부분의 TDZ 설명 참조)를 수행한다. 그러나 이 두 가지 선언 양식은 단순히 `var`와 `function` 선언처럼 이를 감싸는 함수가 아닌 감싸는 블록에 부착된다. 자세한 내용은 6장의 "블록으로 범위 지정"을 참조해라. |

### 호이스팅: 선언 vs. 표현식

*함수 호이스팅*은 공식 `function` 선언(특히 함수 선언이 있는 블록 "Fib"(6장 참고) 외부에 나타나는)에만 적용되며, `function` 표현식 할당에는 적용되지 않는다. 다음 코드를 살펴보자:

```js
greeting();
// TypeError

var greeting = function greeting() {
    console.log("Hello!");
};
```

1행(`greeting();`)는 오류를 발생시킨다. 그러나 발생한 오류의 *종류*는 매우 중요하다. `TypeError`는 허용되지 않는 값을 사용하여 작업을 시도하고 있음을 의미한다. JS 환경에 따라 "'undefined'는 함수가 아니다" 또는 더욱 자세히 "'greeting'"는 함수가 아니다."와 같은 오류 메시지가 표시된다.

위의 오류는  `ReferenceError`가 *아니다*. JS는 스코프에서 식별자로서의 `greeting`를 찾지 못했다고 말하지 않고 있다. `greeting`이 발견됐지만 그 시점에 함수 참조가 없다는 얘기다. 함수만 호출할 수 있으므로 일부 비함수 값을 호출하려고 하면 오류가 발생한다.

함수 참조가 아니라면, `greeting`이 가진 값은 무엇일까?

호이스팅 외에도, `var`로 선언된 변수도 스코프,가장 가까운 감싸는 함수 또는 전역의 시작에서 자동으로 `undefined`으로 초기화된다. 초기화되면 전체 범위에서 사용할 수 있다(할당, 검색 등).

즉 첫 번째 줄에 `greeting`가 존재하지만, 이것은 기본 '`undefined` 값만 가지고 있다. 4행에서야 `greeting`'가 기능 참조를 할당받는다.

여기에서 구별에 주의하라. `function` 선언은 호이스팅되고 **해당 함수 값으로 초기화**된다(*함수 호이스팅*이라고 함). 'var' 변수도 호이스트된 다음 `undefined`으로 자동 초기화된다. 런타임 실행 중에 할당이 처리되기 전에는 해당 변수에 대한 후속 `function` 식 할당이 수행되지 않는다.

두 경우 모두 식별자의 이름이 호이스트된다. 그러나 식별자가 공식 `function` 선언에서 생성되지 않는 한 함수 참조 연결은 초기화 시(스코프 시작) 처리되지 않는다.

### 변수 호이스팅

*변수 호이스팅*의 다른 예를 살펴보자.

```js
greeting = "Hello!";
console.log(greeting);
// Hello!

var greeting = "Howdy!";
```

5행까지는 `greeting`가 선언되지 않지만, 1행에서 할당이 가능하다. 그 이유는 무엇일까?

설명에는 두 가지 필요한 부분이 있다.

* 식별자가 호이스트 되었다.
* ** 그리고 *** 식별자는 스코프의 최상단에서 `undefined`으로 초기화되었다.

| 비고: |
| :--- |
| 이러한 종류의 *변수 호이스팅*를 사용하는 것은 아마도 부자연스럽게 느껴질 것이고, 여러분들은 당연히 그들의 프로그램에서 그것에 의존하는 것을 피하고 싶어할 것이다. 그러나 모든 호이스팅(*함수 호스팅* 포함)은 피해야 할까? 우리는 호이스팅에 대한 이러한 다양한 관점에 대해 부록 A에서 자세히 살펴볼 것이다. |

## 호이드스팅: 또다른 비유

제2장은 비유들로 가득 차 있었지만(스코프를 설명하기 위해서), 여기서 우리는 또 다른 비유에 마주하게 되었다: 호이스팅 그 자체말이다. 호이스팅은 JS 엔진이 수행하는 구체적인 실행 단계이기보다는 프로그램 ** 실행 전**을 설정하기 위해 JS가 수행하는 다양한 작업의 시각화라고 생각하면 더 유용할 것이다.

호이스팅의 의미에 대한 일반적인 주장: 식별자를 *리프팅*-무거운 중량을 위로 들어올리는 것)- 하여 스코프의 맨 위까지 올린다. 종종 JS 엔진이 실행 전에 해당 프로그램을 실제로 *재작성*하므로 다음과 더 비슷해 보인다는 주장이 제기되기도 한다.

```js
var greeting;           // hoisted declaration
greeting = "Hello!";    // the original line 1
console.log(greeting);  // Hello!
greeting = "Howdy!";    // `var` is gone!
```

호이스팅(비유)은 실행 전에 모든 선언이 각각의 범위의 맨 위로 이동되도록 원래 프로그램을 사전 처리하고 약간 다시 정렬할 것을 제안한다. 게다가, `function` 선언 전체가 각 범위의 최상위에 올려져 있다고 호스팅 비유는 주장한다. 다음을 살펴보자:

```js
studentName = "Suzy";
greeting();
// Hello Suzy!

function greeting() {
    console.log(`Hello ${ studentName }!`);
}
var studentName;
```

호이스팅 비유의 "규칙"은 함수 선언을 먼저 올린 다음 변수를 모든 함수 뒤에 바로 올리는 것입니다. 따라서, 호이스팅 사례는 다음과 같이 보이는 JS 엔진에 의해 프로그램이 *재편성*되었음을 시사한다.

```js
function greeting() {
    console.log(`Hello ${ studentName }!`);
}
var studentName;

studentName = "Suzy";
greeting();
// Hello Suzy!
```

이 호이스팅 비유법은 편리하다. 그 이점은 스코프 깊숙이 파묻혀 있는 모든 선언을 찾아 어떻게든 상단으로 이동(호이스트)하는 데 필요한 전 처리를 마법처럼 미리 보는 것으로 쉽게 이해시킨다. 프로그램을 **단일 패스**, 하향식으로 실행하는 것처럼 생각하면 된다.

1장의 2단계 처리 방식 보다 단일 패스가 더 간단해 보인다.

코드 순서 재조정 메커니즘으로 호스팅하는 것이 간단하다는 점에서 매력적일 수는 있지만 정확하지는 않을 수 있다. JS 엔진은 실제로 코드를 다시 정렬하지 않는다. 마술적으로 선언을 미리 보고 찾을 수는 없다. 코드를 파싱하는 것이 선언을 정확히 찾는 유일한 방법이다.

파싱이 뭔지 아는가? 2단계 처리의 첫 번째 단계! 그 사실을 피할 수 있는 마법의 멘탈 체조는 없다.

그렇다면 만약 호이스틍 비유가 (기껏해야) 부정확하다면, 우리는 이 용어를 어떻게 해야 할까? 나는 그것이 여전히 유용하다고 생각한다. 실제로 TC39의 회원들도 정기적으로 사용하고 있다!—하지만 소스 코드를 실제로 다시 배열한 것이라고 주장할 필요는 없다고 생각한다.

| 주의: |
| :--- |
| 부정확하거나 불완전한 멘탈 모델은 종종 우연한 정답으로 이어질 수 있기 때문에 여전히 충분해 보인다. 그러나 장기적으로는 여러분의 생각이 JS 엔진의 작동 방식과 특별히 일치하지 않을 경우 결과를 정확하게 분석하고 예측하는 것이 더 어렵다. |


나는 호이스팅을 **꼭** 사용하여 해당 스코프가 시작될 때마다 해당 스코프 시작 시 변수의 자동 등록을 위한 런타임 명령을 생성하는 **컴파일 시간 작업**을(를) 참조해야 한다고 주장한다.

이는 런타임 동작으로서의 호이스팅에서 컴파일 시간 작업 사이의 적절한 위치로 미묘하지만 중요한 전환이다.

## Re-declaration?

What do you think happens when a variable is declared more than once in the same scope? Consider:

```js
var studentName = "Frank";
console.log(studentName);
// Frank

var studentName;
console.log(studentName);   // ???
```

What do you expect to be printed for that second message? Many believe the second `var studentName` has re-declared the variable (and thus "reset" it), so they expect `undefined` to be printed.

But is there such a thing as a variable being "re-declared" in the same scope? No.

If you consider this program from the perspective of the hoisting metaphor, the code would be re-arranged like this for execution purposes:

```js
var studentName;
var studentName;    // clearly a pointless no-op!

studentName = "Frank";
console.log(studentName);
// Frank

console.log(studentName);
// Frank
```

Since hoisting is actually about registering a variable at the beginning of a scope, there's nothing to be done in the middle of the scope where the original program actually had the second `var studentName` statement. It's just a no-op(eration), a pointless statement.

| TIP: |
| :--- |
| In the style of the conversation narrative from Chapter 2, *Compiler* would find the second `var` declaration statement and ask the *Scope Manager* if it had already seen a `studentName` identifier; since it had, there wouldn't be anything else to do. |

It's also important to point out that `var studentName;` doesn't mean `var studentName = undefined;`, as most assume. Let's prove they're different by considering this variation of the program:

```js
var studentName = "Frank";
console.log(studentName);   // Frank

var studentName;
console.log(studentName);   // Frank <--- still!

// let's add the initialization explicitly
var studentName = undefined;
console.log(studentName);   // undefined <--- see!?
```

See how the explicit `= undefined` initialization produces a different outcome than assuming it happens implicitly when omitted? In the next section, we'll revisit this topic of initialization of variables from their declarations.

A repeated `var` declaration of the same identifier name in a scope is effectively a do-nothing operation. Here's another illustration, this time across a function of the same name:

```js
var greeting;

function greeting() {
    console.log("Hello!");
}

// basically, a no-op
var greeting;

typeof greeting;        // "function"

var greeting = "Hello!";

typeof greeting;        // "string"
```

The first `greeting` declaration registers the identifier to the scope, and because it's a `var` the auto-initialization will be `undefined`. The `function` declaration doesn't need to re-register the identifier, but because of *function hoisting* it overrides the auto-initialization to use the function reference. The second `var greeting` by itself doesn't do anything since `greeting` is already an identifier and *function hoisting* already took precedence for the auto-initialization.

Actually assigning `"Hello!"` to `greeting` changes its value from the initial function `greeting()` to the string; `var` itself doesn't have any effect.

What about repeating a declaration within a scope using `let` or `const`?

```js
let studentName = "Frank";

console.log(studentName);

let studentName = "Suzy";
```

This program will not execute, but instead immediately throw a `SyntaxError`. Depending on your JS environment, the error message will indicate something like: "studentName has already been declared." In other words, this is a case where attempted "re-declaration" is explicitly not allowed!

It's not just that two declarations involving `let` will throw this error. If either declaration uses `let`, the other can be either `let` or `var`, and the error will still occur, as illustrated with these two variations:

```js
var studentName = "Frank";

let studentName = "Suzy";
```

and:

```js
let studentName = "Frank";

var studentName = "Suzy";
```

In both cases, a `SyntaxError` is thrown on the *second* declaration. In other words, the only way to "re-declare" a variable is to use `var` for all (two or more) of its declarations.

But why disallow it? The reason for the error is not technical per se, as `var` "re-declaration" has always been allowed; clearly, the same allowance could have been made for `let`.

It's really more of a "social engineering" issue. "Re-declaration" of variables is seen by some, including many on the TC39 body, as a bad habit that can lead to program bugs. So when ES6 introduced `let`, they decided to prevent "re-declaration" with an error.

| NOTE: |
| :--- |
| This is of course a stylistic opinion, not really a technical argument. Many developers agree with the position, and that's probably in part why TC39 included the error (as well as `let` conforming to `const`). But a reasonable case could have been made that staying consistent with `var`'s precedent was more prudent, and that such opinion-enforcement was best left to opt-in tooling like linters. In Appendix A, we'll explore whether `var` (and its associated behavior, like "re-declaration") can still be useful in modern JS. |

When *Compiler* asks *Scope Manager* about a declaration, if that identifier has already been declared, and if either/both declarations were made with `let`, an error is thrown. The intended signal to the developer is "Stop relying on sloppy re-declaration!"

### Constants?

The `const` keyword is more constrained than `let`. Like `let`, `const` cannot be repeated with the same identifier in the same scope. But there's actually an overriding technical reason why that sort of "re-declaration" is disallowed, unlike `let` which disallows "re-declaration" mostly for stylistic reasons.

The `const` keyword requires a variable to be initialized, so omitting an assignment from the declaration results in a `SyntaxError`:

```js
const empty;   // SyntaxError
```

`const` declarations create variables that cannot be re-assigned:

```js
const studentName = "Frank";
console.log(studentName);
// Frank

studentName = "Suzy";   // TypeError
```

The `studentName` variable cannot be re-assigned because it's declared with a `const`.

| WARNING: |
| :--- |
| The error thrown when re-assigning `studentName` is a `TypeError`, not a `SyntaxError`. The subtle distinction here is actually pretty important, but unfortunately far too easy to miss. Syntax errors represent faults in the program that stop it from even starting execution. Type errors represent faults that arise during program execution. In the preceding snippet, `"Frank"` is printed out before we process the re-assignment of `studentName`, which then throws the error. |

So if `const` declarations cannot be re-assigned, and `const` declarations always require assignments, then we have a clear technical reason why `const` must disallow any "re-declarations": any `const` "re-declaration" would also necessarily be a `const` re-assignment, which can't be allowed!

```js
const studentName = "Frank";

// obviously this must be an error
const studentName = "Suzy";
```

Since `const` "re-declaration" must be disallowed (on those technical grounds), TC39 essentially felt that `let` "re-declaration" should be disallowed as well, for consistency. It's debatable if this was the best choice, but at least we have the reasoning behind the decision.

### Loops

So it's clear from our previous discussion that JS doesn't really want us to "re-declare" our variables within the same scope. That probably seems like a straightforward admonition, until you consider what it means for repeated execution of declaration statements in loops. Consider:

```js
var keepGoing = true;
while (keepGoing) {
    let value = Math.random();
    if (value > 0.5) {
        keepGoing = false;
    }
}
```

Is `value` being "re-declared" repeatedly in this program? Will we get errors thrown? No.

All the rules of scope (including "re-declaration" of `let`-created variables) are applied *per scope instance*. In other words, each time a scope is entered during execution, everything resets.

Each loop iteration is its own new scope instance, and within each scope instance, `value` is only being declared once. So there's no attempted "re-declaration," and thus no error. Before we consider other loop forms, what if the `value` declaration in the previous snippet were changed to a `var`?

```js
var keepGoing = true;
while (keepGoing) {
    var value = Math.random();
    if (value > 0.5) {
        keepGoing = false;
    }
}
```

Is `value` being "re-declared" here, especially since we know `var` allows it? No. Because `var` is not treated as a block-scoping declaration (see Chapter 6), it attaches itself to the global scope. So there's just one `value` variable, in the same scope as `keepGoing` (global scope, in this case). No "re-declaration" here, either!

One way to keep this all straight is to remember that `var`, `let`, and `const` keywords are effectively *removed* from the code by the time it starts to execute. They're handled entirely by the compiler.

If you mentally erase the declarator keywords and then try to process the code, it should help you decide if and when (re-)declarations might occur.

What about "re-declaration" with other loop forms, like `for`-loops?

```js
for (let i = 0; i < 3; i++) {
    let value = i * 10;
    console.log(`${ i }: ${ value }`);
}
// 0: 0
// 1: 10
// 2: 20
```

It should be clear that there's only one `value` declared per scope instance. But what about `i`? Is it being "re-declared"?

To answer that, consider what scope `i` is in. It might seem like it would be in the outer (in this case, global) scope, but it's not. It's in the scope of `for`-loop body, just like `value` is. In fact, you could sorta think about that loop in this more verbose equivalent form:

```js
{
    // a fictional variable for illustration
    let $$i = 0;

    for ( /* nothing */; $$i < 3; $$i++) {
        // here's our actual loop `i`!
        let i = $$i;

        let value = i * 10;
        console.log(`${ i }: ${ value }`);
    }
    // 0: 0
    // 1: 10
    // 2: 20
}
```

Now it should be clear: the `i` and `value` variables are both declared exactly once **per scope instance**. No "re-declaration" here.

What about other `for`-loop forms?

```js
for (let index in students) {
    // this is fine
}

for (let student of students) {
    // so is this
}
```

Same thing with `for..in` and `for..of` loops: the declared variable is treated as *inside* the loop body, and thus is handled per iteration (aka, per scope instance). No "re-declaration."

OK, I know you're thinking that I sound like a broken record at this point. But let's explore how `const` impacts these looping constructs. Consider:

```js
var keepGoing = true;
while (keepGoing) {
    // ooo, a shiny constant!
    const value = Math.random();
    if (value > 0.5) {
        keepGoing = false;
    }
}
```

Just like the `let` variant of this program we saw earlier, `const` is being run exactly once within each loop iteration, so it's safe from "re-declaration" troubles. But things get more complicated when we talk about `for`-loops.

`for..in` and `for..of` are fine to use with `const`:

```js
for (const index in students) {
    // this is fine
}

for (const student of students) {
    // this is also fine
}
```

But not the general `for`-loop:

```js
for (const i = 0; i < 3; i++) {
    // oops, this is going to fail with
    // a Type Error after the first iteration
}
```

What's wrong here? We could use `let` just fine in this construct, and we asserted that it creates a new `i` for each loop iteration scope, so it doesn't even seem to be a "re-declaration."

Let's mentally "expand" that loop like we did earlier:

```js
{
    // a fictional variable for illustration
    const $$i = 0;

    for ( ; $$i < 3; $$i++) {
        // here's our actual loop `i`!
        const i = $$i;
        // ..
    }
}
```

Do you spot the problem? Our `i` is indeed just created once inside the loop. That's not the problem. The problem is the conceptual `$$i` that must be incremented each time with the `$$i++` expression. That's **re-assignment** (not "re-declaration"), which isn't allowed for constants.

Remember, this "expanded" form is only a conceptual model to help you intuit the source of the problem. You might wonder if JS could have effectively made the `const $$i = 0` instead into `let $ii = 0`, which would then allow `const` to work with our classic `for`-loop? It's possible, but then it could have introduced potentially surprising exceptions to `for`-loop semantics.

For example, it would have been a rather arbitrary (and likely confusing) nuanced exception to allow `i++` in the `for`-loop header to skirt strictness of the `const` assignment, but not allow other re-assignments of `i` inside the loop iteration, as is sometimes useful.

The straightforward answer is: `const` can't be used with the classic `for`-loop form because of the required re-assignment.

Interestingly, if you don't do re-assignment, then it's valid:

```js
var keepGoing = true;

for (const i = 0; keepGoing; /* nothing here */ ) {
    keepGoing = (Math.random() > 0.5);
    // ..
}
```

That works, but it's pointless. There's no reason to declare `i` in that position with a `const`, since the whole point of such a variable in that position is **to be used for counting iterations**. Just use a different loop form, like a `while` loop, or use a `let`!

## Uninitialized Variables (aka, TDZ)

With `var` declarations, the variable is "hoisted" to the top of its scope. But it's also automatically initialized to the `undefined` value, so that the variable can be used throughout the entire scope.

However, `let` and `const` declarations are not quite the same in this respect.

Consider:

```js
console.log(studentName);
// ReferenceError

let studentName = "Suzy";
```

The result of this program is that a `ReferenceError` is thrown on the first line. Depending on your JS environment, the error message may say something like: "Cannot access studentName before initialization."

| NOTE: |
| :--- |
| The error message as seen here used to be much more vague or misleading. Thankfully, several of us in the community were successfully able to lobby for JS engines to improve this error message so it more accurately tells you what's wrong! |

That error message is quite indicative of what's wrong: `studentName` exists on line 1, but it's not been initialized, so it cannot be used yet. Let's try this:

```js
studentName = "Suzy";   // let's try to initialize it!
// ReferenceError

console.log(studentName);

let studentName;
```

Oops. We still get the `ReferenceError`, but now on the first line where we're trying to assign to (aka, initialize!) this so-called "uninitialized" variable `studentName`. What's the deal!?

The real question is, how do we initialize an uninitialized variable? For `let`/`const`, the **only way** to do so is with an assignment attached to a declaration statement. An assignment by itself is insufficient! Consider:

```js
let studentName = "Suzy";
console.log(studentName);   // Suzy
```

Here, we are initializing the `studentName` (in this case, to `"Suzy"` instead of `undefined`) by way of the `let` declaration statement form that's coupled with an assignment.

Alternatively:

```js
// ..

let studentName;
// or:
// let studentName = undefined;

// ..

studentName = "Suzy";

console.log(studentName);
// Suzy
```

| NOTE: |
| :--- |
| That's interesting! Recall from earlier, we said that `var studentName;` is *not* the same as `var studentName = undefined;`, but here with `let`, they behave the same. The difference comes down to the fact that `var studentName` automatically initializes at the top of the scope, where `let studentName` does not. |

Remember that we've asserted a few times so far that *Compiler* ends up removing any `var`/`let`/`const` declarators, replacing them with the instructions at the top of each scope to register the appropriate identifiers.

So if we analyze what's going on here, we see that an additional nuance is that *Compiler* is also adding an instruction in the middle of the program, at the point where the variable `studentName` was declared, to handle that declaration's auto-initialization. We cannot use the variable at any point prior to that initialization occuring. The same goes for `const` as it does for `let`.

The term coined by TC39 to refer to this *period of time* from the entering of a scope to where the auto-initialization of the variable occurs is: Temporal Dead Zone (TDZ).

The TDZ is the time window where a variable exists but is still uninitialized, and therefore cannot be accessed in any way. Only the execution of the instructions left by *Compiler* at the point of the original declaration can do that initialization. After that moment, the TDZ is done, and the variable is free to be used for the rest of the scope.

A `var` also has technically has a TDZ, but it's zero in length and thus unobservable to our programs! Only `let` and `const` have an observable TDZ.

By the way, "temporal" in TDZ does indeed refer to *time* not *position in code*. Consider:

```js
askQuestion();
// ReferenceError

let studentName = "Suzy";

function askQuestion() {
    console.log(`${ studentName }, do you know?`);
}
```

Even though positionally the `console.log(..)` referencing `studentName` comes *after* the `let studentName` declaration, timing wise the `askQuestion()` function is invoked *before* the `let` statement is encountered, while `studentName` is still in its TDZ! Hence the error.

There's a common misconception that TDZ means `let` and `const` do not hoist. This is an inaccurate, or at least slightly misleading, claim. They definitely hoist.

The actual difference is that `let`/`const` declarations do not automatically initialize at the beginning of the scope, the way `var` does. The *debate* then is if the auto-initialization is *part of* hoisting, or not? I think auto-registration of a variable at the top of the scope (i.e., what I call "hoisting") and auto-initialization at the top of the scope (to `undefined`) are distinct operations and shouldn't be lumped together under the single term "hoisting."

We've already seen that `let` and `const` don't auto-initialize at the top of the scope. But let's prove that `let` and `const` *do* hoist (auto-register at the top of the scope), courtesy of our friend shadowing (see "Shadowing" in Chapter 3):

```js
var studentName = "Kyle";

{
    console.log(studentName);
    // ???

    // ..

    let studentName = "Suzy";

    console.log(studentName);
    // Suzy
}
```

What's going to happen with the first `console.log(..)` statement? If `let studentName` didn't hoist to the top of the scope, then the first `console.log(..)` *should* print `"Kyle"`, right? At that moment, it would seem, only the outer `studentName` exists, so that's the variable `console.log(..)` should access and print.

But instead, the first `console.log(..)` throws a TDZ error, because in fact, the inner scope's `studentName` **was** hoisted (auto-registered at the top of the scope). What **didn't** happen (yet!) was the auto-initialization of that inner `studentName`; it's still uninitialized at that moment, hence the TDZ violation!

So to summarize, TDZ errors occur because `let`/`const` declarations *do* hoist their declarations to the top of their scopes, but unlike `var`, they defer the auto-initialization of their variables until the moment in the code's sequencing where the original declaration appeared. This window of time (hint: temporal), whatever its length, is the TDZ.

How can you avoid TDZ errors?

My advice: always put your `let` and `const` declarations at the top of any scope. Shrink the TDZ window to zero (or near zero) length, and then it'll be moot.

But why is TDZ even a thing? Why didn't TC39 dictate that `let`/`const` auto-initialize the way `var` does? Just be patient, we'll come back to explore the *why* of TDZ in Appendix A.

## Finally Initialized

Working with variables has much more nuance than it seems at first glance. *Hoisting*, *(re)declaration*, and the *TDZ* are common sources of confusion for developers, especially those who have worked in other languages before coming to JS. Before moving on, make sure your mental model is fully grounded on these aspects of JS scope and variables.

Hoisting is generally cited as an explicit mechanism of the JS engine, but it's really more a metaphor to describe the various ways JS handles variable declarations during compilation. But even as a metaphor, hoisting offers useful structure for thinking about the life-cycle of a variable—when it's created, when it's available to use, when it goes away.

Declaration and re-declaration of variables tend to cause confusion when thought of as runtime operations. But if you shift to compile-time thinking for these operations, the quirks and *shadows* diminish.

The TDZ (temporal dead zone) error is strange and frustrating when encountered. Fortunately, TDZ is relatively straightforward to avoid if you're always careful to place `let`/`const` declarations at the top of any scope.

As you successfully navigate these twists and turns of variable scope, the next chapter will lay out the factors that guide our decisions to place our declarations in various scopes, especially nested blocks.
