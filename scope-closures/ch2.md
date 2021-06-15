# You Don't Know JS Yet: Scope & Closures - 2nd Edition
# 2장: 렉시컬 스코프<sub>Lexical Scope</sub> 살펴보기

1장에서 스코프<sub>scope</sub>가 "렉시컬 스코프"라 부르는 모델로 어떻게 코드 컴파일 중에 결정되는지 살펴봤다. "렉시컬<sub>lexical</sub>" 용어는 컴파일(렉싱<sub>lexing</sub>/파싱<sub>parsing</sub>)의 첫번째 단계에 관련이 있다.

프로그램에 대한 적절한 *근거*를 위해서는 스코프가 작동하는 방식에 대한 탄탄한 개념적 기반을 가지는 것이 중요하다. 만약 추측과 직관에 의지한다면 가끔은 뜻하지 않게 정답을 얻을 수도 있지만, 대부분 크게 벗어난다. 이는 성공을 위한 비법이 아니다.

초등학교 수학 수업에서처럼, 거기에 가기 위한 올바른 단계를 보여주지 않는다면 정답을 맞추는 것만으로는 충분하지 않다. 새로운 기반으로서 정확하고 도움이 되는 멘탈 모델<sub>mental models</sub>을 만들 필요가 있다.

이 장에서는 *스코프*를 몇 가지 메타포<sub>metaphors</sub>와 함께 설명한다. 여기서의 목표는 프로그램이 어떻게 JS 엔진에 의해 처리되는지에 대해 JS 엔진이 실제로 작동하는 방식에 더 밀접하게 맞추는 방식으로 *생각하는* 것이다.

## 구슬과 양동이와 버블<sub>Bubbles</sub>... 이런!

스코프를 이해하는 데 효과적인 한 가지 메타포는 색깔있는 구슬을 같은 색을 가진 양동이에 넣어 분류하는 것이다.

우연히 구슬 더미를 발견했다고 상상해라. 그리고 모든 구슬이 빨강, 파랑 또는 초록색인 것을 주목해라. 모든 구슬을 빨간색 구슬은 빨간색 양동이에, 초록색 구슬은 초록색 양동이에 그리고 파란색 구슬은 파란색 양동이에 떨어트려서 분류하자. 분류 후에는, 나중에 초록색 구슬이 필요할 때 그것을 얻기 위해 갈 곳은 초록색 양동이라는 것을 이미 알게 된다.

이 메타포에서 구슬은 프로그램의 변수이다. 양동이는 스코프(함수와 블록)이다. 단지 개념적으로 논의 목적을 위해 개별 색상을 할당했다. 각 구슬의 색상은 이와 같이 본래 만들어진 구슬을 알게되는 *색상* 스코프에 의해서 밝혀진다.

1장의 실행 프로그램 예제에 스코프 색상 라벨로 주석을 달아보자.

```js
// 외부/전역 스코프: 빨강

var students = [
    { id: 14, name: "Kyle" },
    { id: 73, name: "Suzy" },
    { id: 112, name: "Frank" },
    { id: 6, name: "Sarah" }
];

function getStudentName(studentID) {
    // 함수 스코프: 파랑

    for (let student of students) {
        // 반복문 스코프: 초록

        if (student.id == studentID) {
            return student.name;
        }
    }
}

var nextStudent = getStudentName(73);
console.log(nextStudent);   // Suzy
```

코드 주석을 가진 세 가지 스코프 색상을 지정했다. 빨강(가장 바깥 전역 스코프), 파랑(함수 `getStudentName(..)`의 스코프), 그리고 초록(`for` 반복문의/안의 스코프)이다. 그러나 나열된 코드를 볼 때 이러한 스코프 양동이의 경계를 인식하기는 여전히 어려울 수 있다.

그림 2는 색깔있는 버블(일명 양동이)을 각각 둘러 그림으로써 스코프의 경계를 시각화하는 데 도움이 된다.

<figure>
    <img src="images/fig2.png" width="500" alt="Colored Scope Bubbles" align="center">
    <figcaption><em>그림 2: 색깔있는 스코프 버블</em></figcaption>
</figure>

1. **버블 1** (빨강)은 세 개의 식별자/변수를 가지고 있는 전역 스코프를 둘러싼다: `students` (1행), `getStudentName` (8행)과 `nextStudent` (16행).

2. **버블 2** (파랑)은 단지 하나의 식별자/변수를 가지고 있는 함수 `getStudentName(..)` (8행)의 스코프를 둘러싼다: 매개변수 `studentID` (8행).

3. **버블 3** (초록)은 단지 하나의 식별자/변수를 가지고 있는 `for` 반복문 (9행)의 스코프를 둘러싼다: `student` (9행).

| 비고: |
| :--- |
| 기술적으로 매개변수 `studentID`는 정확히 파랑(2) 스코프에 있지 않다. 우리는 부록 A의 "암묵적 스코프"에서 이런 혼동을 풀 것이다. 지금은 `studentID`에 파랑(2) 구슬이라는 딱지를 붙이는 것으로 거의 충분하다. |

스코프 버블은 스코프의 함수/블록이 작성되는 위치, 서로 중첩되는 위치 등에 따라 컴파일 중에 결정된다. 두 개의 다른 외부 스코프에 부분적으로 포함되지 않는 각 스코프 버블은 부모 스코프 버블 안에 완전히 포함된다.

각 구슬(변수/식별자)은 어떤 버블(양동이)에서 선언되었는지에 따라 색상이 지정되며, 접근 가능할 수 있는 스코프의 색상이 아니다(예: 9행의 `students`와 10행의 `studentID`).

| 비고: |
| :--- |
| 1장에서 `id`, `name`과 `log`는 모두 변수가 아닌 속성<sub>properties</sub>라고 주장한 것을 기억해라. 다른 말로 양동이 안의 구슬이 아니다. 그래서 이 책에서 논하는 어떤 규칙에 기초하여 색이 지정되지 않았다. 그러한 속성 접근이 어떻게 다뤄지는지 이해하기 위해서는 시리즈의 세번째 책 *Objects & Classes*를 보자. |

JS 엔진은 (컴파일 중에) 프로그램을 처리하고 변수 선언을 찾을 때, 근본적으로 "내가 현재 어떤 *색상* 스코프(버블 또는 양동이)에 있지?"라고 묻는다. 변수는 같은 *색상*으로 지정되며 이는 그 양동이/버블에 속함을 의미한다.

초록(3) 양동이는 파랑(2) 양동이 안에 완전히 중첩되어 있다. 그리고 마찬가지로 파랑(2) 양동이는 빨강(1) 양동이 안에 완전히 중첩되어 있다. 스코프는 보여지는 대로 프로그램이 필요한 중첩의 깊이까지 각자 내부에 중첩될 수 있다.

변수/식별자에 대한 참조(선언이 아닌)는 만약 현재 스코프 또는 현재 스코프의 상위/외부 스코프에 일치하는 선언이 있다면 허용되지만, 하위/내부 스코프의 선언은 허용되지 않는다.

빨강(1) 양동이에 있는 표현식은 빨강(1) 구슬에만 접근 권한이 있다. 파랑(2)이나 초록(3) 구슬이 **아니다**. 파랑(2) 양동이에 있는 표현식은 파랑(2)이나 빨강(1) 구슬 둘다 참조할 수 있다. 초록(3) 구슬이 **아니다**. 그리고 초록(3) 양동이에 있는 표현식은 빨강(1), 파랑(2) 그리고 초록(3) 구슬에 접근 권한이 있다.

런타임<sub>runtime</sub> 중에 이런 선언이 아닌 구슬 색상을 결정하는 처리를 룩업<sub>lookup</sub>으로 개념화할 수 있다. 9행의 `for` 반복문에서 `students` 변수 참조는 선언이 아니기 때문에 색깔이 없다. 그래서 현재 파랑(2) 스코프 양동이에 일치하는 이름을 가진 구슬이 있는지 물어본다. 없기 때문에, 룩업은 다음 외부/포함 스코프인 빨강(1)으로 계속 진행한다. 빨강(1) 양동이는 `students`라는 구슬을 가지고 있다. 그래서 반복문의 `students` 변수 참조는 빨강(1) 구슬로 결정된다.

10행의 `if (student.id == studentID)` 구문은 마찬가지로 `student`라는 초록(3) 구슬과 `studentID`라는 파랑(3) 구슬을 참조하는 것으로 결정된다.

| 비고: |
| :--- |
| JS 엔진은 일반적으로 런타임 중에 이런 구슬 색상을 결정하지 않는다. 여기서 "룩업"은 여러분이 개념을 이해할 수 있도록 도와주는 수사적인 장치이다. 컴파일 중에 대부분 또는 모든 변수 참조는 이미 알려진 스코프 양동이에 일치할 것이다. 그래서 그 색상은 이미 결정되어 있으며 프로그램이 실행될 때 불필요한 조회를 피하기 위해 각 구슬 참조와 함께 저장된다. 3장에서 이 미묘한 차이에 대해 더 알아보자. |

구슬과 양동이 (그리고 버블!)의 주요 요점 정리:

* 변수는 특정한 스코프에 선언되며, 이는 일치하는 색의 양동이로부터 온 색이 있는 구슬이라고 볼 수 있다.

* 선언된 스코프에 나타나거나 더 깊은 중첩 스코프에 나타나는 변수 참조는, 간섭 스코프가 변수 선언에 "그늘을 드리우"지 않는 한 같은 색상의 구슬로 지정될 것이다. 3장에서 "Shadowing"을 보자.

* 색이 있는 양동이와 거기 있는 구슬의 결정은 컴파일 중에 일어난다. 이 정보는 코드 실행<sub>execution</sub> 중에 변수(구슬 색상) "룩업"에 사용된다.

## A Conversation Among Friends

Another useful metaphor for the process of analyzing variables and the scopes they come from is to imagine various conversations that occur inside the engine as code is processed and then executed. We can "listen in" on these conversations to get a better conceptual foundation for how scopes work.

Let's now meet the members of the JS engine that will have conversations as they process our program:

* *Engine*: responsible for start-to-finish compilation and execution of our JavaScript program.

* *Compiler*: one of *Engine*'s friends; handles all the dirty work of parsing and code-generation (see previous section).

* *Scope Manager*: another friend of *Engine*; collects and maintains a lookup list of all the declared variables/identifiers, and enforces a set of rules as to how these are accessible to currently executing code.

For you to *fully understand* how JavaScript works, you need to begin to *think* like *Engine* (and friends) think, ask the questions they ask, and answer their questions likewise.

To explore these conversations, recall again our running program example:

```js
var students = [
    { id: 14, name: "Kyle" },
    { id: 73, name: "Suzy" },
    { id: 112, name: "Frank" },
    { id: 6, name: "Sarah" }
];

function getStudentName(studentID) {
    for (let student of students) {
        if (student.id == studentID) {
            return student.name;
        }
    }
}

var nextStudent = getStudentName(73);

console.log(nextStudent);
// Suzy
```

Let's examine how JS is going to process that program, specifically starting with the first statement. The array and its contents are just basic JS value literals (and thus unaffected by any scoping concerns), so our focus here will be on the `var students = [ .. ]` declaration and initialization-assignment parts.

We typically think of that as a single statement, but that's not how our friend *Engine* sees it. In fact, JS treats these as two distinct operations, one which *Compiler* will handle during compilation, and the other which *Engine* will handle during execution.

The first thing *Compiler* will do with this program is perform lexing to break it down into tokens, which it will then parse into a tree (AST).

Once *Compiler* gets to code generation, there's more detail to consider than may be obvious. A reasonable assumption would be that *Compiler* will produce code for the first statement such as: "Allocate memory for a variable, label it `students`, then stick a reference to the array into that variable." But that's not the whole story.

Here's the steps *Compiler* will follow to handle that statement:

1. Encountering `var students`, *Compiler* will ask *Scope Manager* to see if a variable named `students` already exists for that particular scope bucket. If so, *Compiler* would ignore this declaration and move on. Otherwise, *Compiler* will produce code that (at execution time) asks *Scope Manager* to create a new variable called `students` in that scope bucket.

2. *Compiler* then produces code for *Engine* to later execute, to handle the `students = []` assignment. The code *Engine* runs will first ask *Scope Manager* if there is a variable called `students` accessible in the current scope bucket. If not, *Engine* keeps looking elsewhere (see "Nested Scope" below). Once *Engine* finds a variable, it assigns the reference of the `[ .. ]` array to it.

In conversational form, the first phase of compilation for the program might play out between *Compiler* and *Scope Manager* like this:

> ***Compiler***: Hey, *Scope Manager* (of the global scope), I found a formal declaration for an identifier called `students`, ever heard of it?

> ***(Global) Scope Manager***: Nope, never heard of it, so I just created it for you.

> ***Compiler***: Hey, *Scope Manager*, I found a formal declaration for an identifier called `getStudentName`, ever heard of it?

> ***(Global) Scope Manager***: Nope, but I just created it for you.

> ***Compiler***: Hey, *Scope Manager*, `getStudentName` points to a function, so we need a new scope bucket.

> ***(Function) Scope Manager***: Got it, here's the scope bucket.

> ***Compiler***: Hey, *Scope Manager* (of the function), I found a formal parameter declaration for `studentID`, ever heard of it?

> ***(Function) Scope Manager***: Nope, but now it's created in this scope.

> ***Compiler***: Hey, *Scope Manager* (of the function), I found a `for`-loop that will need its own scope bucket.

> ...

The conversation is a question-and-answer exchange, where **Compiler** asks the current *Scope Manager* if an encountered identifier declaration has already been encountered. If "no," *Scope Manager* creates that variable in that scope. If the answer is "yes," then it's effectively skipped over since there's nothing more for that *Scope Manager* to do.

*Compiler* also signals when it runs across functions or block scopes, so that a new scope bucket and *Scope Manager* can be instantiated.

Later, when it comes to execution of the program, the conversation will shift to *Engine* and *Scope Manager*, and might play out like this:

> ***Engine***: Hey, *Scope Manager* (of the global scope), before we begin, can you look up the identifier `getStudentName` so I can assign this function to it?

> ***(Global) Scope Manager***: Yep, here's the variable.

> ***Engine***: Hey, *Scope Manager*, I found a *target* reference for `students`, ever heard of it?

> ***(Global) Scope Manager***: Yes, it was formally declared for this scope, so here it is.

> ***Engine***: Thanks, I'm initializing `students` to `undefined`, so it's ready to use.

> Hey, *Scope Manager* (of the global scope), I found a *target* reference for `nextStudent`, ever heard of it?

> ***(Global) Scope Manager***: Yes, it was formally declared for this scope, so here it is.

> ***Engine***: Thanks, I'm initializing `nextStudent` to `undefined`, so it's ready to use.

> Hey, *Scope Manager* (of the global scope), I found a *source* reference for `getStudentName`, ever heard of it?

> ***(Global) Scope Manager***: Yes, it was formally declared for this scope. Here it is.

> ***Engine***: Great, the value in `getStudentName` is a function, so I'm going to execute it.

> ***Engine***: Hey, *Scope Manager*, now we need to instantiate the function's scope.

> ...

This conversation is another question-and-answer exchange, where *Engine* first asks the current *Scope Manager* to look up the hoisted `getStudentName` identifier, so as to associate the function with it. *Engine* then proceeds to ask *Scope Manager* about the *target* reference for `students`, and so on.

To review and summarize how a statement like `var students = [ .. ]` is processed, in two distinct steps:

1. *Compiler* sets up the declaration of the scope variable (since it wasn't previously declared in the current scope).

2. While *Engine* is executing, to process the assignment part of the statement, *Engine* asks *Scope Manager* to look up the variable, initializes it to `undefined` so it's ready to use, and then assigns the array value to it.

## Nested Scope

When it comes time to execute the `getStudentName()` function, *Engine* asks for a *Scope Manager* instance for that function's scope, and it will then proceed to look up the parameter (`studentID`) to assign the `73` argument value to, and so on.

The function scope for `getStudentName(..)` is nested inside the global scope. The block scope of the `for`-loop is similarly nested inside that function scope. Scopes can be lexically nested to any arbitrary depth as the program defines.

Each scope gets its own *Scope Manager* instance each time that scope is executed (one or more times). Each scope automatically has all its identifiers registered at the start of the scope being executed (this is called "variable hoisting"; see Chapter 5).

At the beginning of a scope, if any identifier came from a `function` declaration, that variable is automatically initialized to its associated function reference. And if any identifier came from a `var` declaration (as opposed to `let`/`const`), that variable is automatically initialized to `undefined` so that it can be used; otherwise, the variable remains uninitialized (aka, in its "TDZ," see Chapter 5) and cannot be used until its full declaration-and-initialization are executed.

In the `for (let student of students) {` statement, `students` is a *source* reference that must be looked up. But how will that lookup be handled, since the scope of the function will not find such an identifier?

To explain, let's imagine that bit of conversation playing out like this:

> ***Engine***: Hey, *Scope Manager* (for the function), I have a *source* reference for `students`, ever heard of it?

> ***(Function) Scope Manager***: Nope, never heard of it. Try the next outer scope.

> ***Engine***: Hey, *Scope Manager* (for the global scope), I have a *source* reference for `students`, ever heard of it?

> ***(Global) Scope Manager***: Yep, it was formally declared, here it is.

> ...

One of the key aspects of lexical scope is that any time an identifier reference cannot be found in the current scope, the next outer scope in the nesting is consulted; that process is repeated until an answer is found or there are no more scopes to consult.

### Lookup Failures

When *Engine* exhausts all *lexically available* scopes (moving outward) and still cannot resolve the lookup of an identifier, an error condition then exists. However, depending on the mode of the program (strict-mode or not) and the role of the variable (i.e., *target* vs. *source*; see Chapter 1), this error condition will be handled differently.

#### Undefined Mess

If the variable is a *source*, an unresolved identifier lookup is considered an undeclared (unknown, missing) variable, which always results in a `ReferenceError` being thrown. Also, if the variable is a *target*, and the code at that moment is running in strict-mode, the variable is considered undeclared and similarly throws a `ReferenceError`.

The error message for an undeclared variable condition, in most JS environments, will look like, "Reference Error: XYZ is not defined." The phrase "not defined" seems almost identical to the word "undefined," as far as the English language goes. But these two are very different in JS, and this error message unfortunately creates a persistent confusion.

"Not defined" really means "not declared"—or, rather, "undeclared," as in a variable that has no matching formal declaration in any *lexically available* scope. By contrast, "undefined" really means a variable was found (declared), but the variable otherwise has no other value in it at the moment, so it defaults to the `undefined` value.

To perpetuate the confusion even further, JS's `typeof` operator returns the string `"undefined"` for variable references in either state:

```js
var studentName;
typeof studentName;     // "undefined"

typeof doesntExist;     // "undefined"
```

These two variable references are in very different conditions, but JS sure does muddy the waters. The terminology mess is confusing and terribly unfortunate. Unfortunately, JS developers just have to pay close attention to not mix up *which kind* of "undefined" they're dealing with!

#### Global... What!?

If the variable is a *target* and strict-mode is not in effect, a confusing and surprising legacy behavior kicks in. The troublesome outcome is that the global scope's *Scope Manager* will just create an **accidental global variable** to fulfill that target assignment!

Consider:

```js
function getStudentName() {
    // assignment to an undeclared variable :(
    nextStudent = "Suzy";
}

getStudentName();

console.log(nextStudent);
// "Suzy" -- oops, an accidental-global variable!
```

Here's how that *conversation* will proceed:

> ***Engine***: Hey, *Scope Manager* (for the function), I have a *target* reference for `nextStudent`, ever heard of it?

> ***(Function) Scope Manager***: Nope, never heard of it. Try the next outer scope.

> ***Engine***: Hey, *Scope Manager* (for the global scope), I have a *target* reference for `nextStudent`, ever heard of it?

> ***(Global) Scope Manager***: Nope, but since we're in non-strict-mode, I helped you out and just created a global variable for you, here it is!

Yuck.

This sort of accident (almost certain to lead to bugs eventually) is a great example of the beneficial protections offered by strict-mode, and why it's such a bad idea *not* to be using strict-mode. In strict-mode, the ***Global Scope Manager*** would instead have responded:

> ***(Global) Scope Manager***: Nope, never heard of it. Sorry, I've got to throw a `ReferenceError`.

Assigning to a never-declared variable *is* an error, so it's right that we would receive a `ReferenceError` here.

Never rely on accidental global variables. Always use strict-mode, and always formally declare your variables. You'll then get a helpful `ReferenceError` if you ever mistakenly try to assign to a not-declared variable.

### Building On Metaphors

To visualize nested scope resolution, I prefer yet another metaphor, an office building, as in Figure 3:

<figure>
    <img src="images/fig3.png" width="250" alt="Scope &quot;Building&quot;" align="center">
    <figcaption><em>Fig. 3: Scope "Building"</em></figcaption>
    <br><br>
</figure>

The building represents our program's nested scope collection. The first floor of the building represents the currently executing scope. The top level of the building is the global scope.

You resolve a *target* or *source* variable reference by first looking on the current floor, and if you don't find it, taking the elevator to the next floor (i.e., an outer scope), looking there, then the next, and so on. Once you get to the top floor (the global scope), you either find what you're looking for, or you don't. But you have to stop regardless.

## Continue the Conversation

By this point, you should be developing richer mental models for what scope is and how the JS engine determines and uses it from your code.

Before *continuing*, go find some code in one of your projects and run through these conversations. Seriously, actually speak out loud. Find a friend and practice each role with them. If either of you find yourself confused or tripped up, spend more time reviewing this material.

As we move (up) to the next (outer) chapter, we'll explore how the lexical scopes of a program are connected in a chain.
