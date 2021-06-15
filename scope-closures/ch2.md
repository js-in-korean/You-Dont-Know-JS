# You Don't Know JS Yet: Scope & Closures - 2nd Edition
# 2장: 렉시컬 스코프<sub>Lexical Scope</sub> 설명하기

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

## 친구 사이의 대화

변수와 스코프가 어디서부터 왔는지 분석하는 과정을 위한 유용한 메타포는 코드가 처리되고 실행되는 동안에 엔진 안에서 일어나는 다양한 대화를 상상하는 것이다. 우리는 스코프가 어떻게 동작하는지 더 나은 개념적 기반을 얻기 위해 이 대화를 "엿듣을" 수 있다.

프로그램을 처리하는 동안 대화를 할 JS 엔진의 구성원을 지금 만나보자:

* *엔진*: JavaScript 프로그램의 시작부터 완료까지 컴파일과 실행을 담당한다.

* *컴파일러*: *엔진*의 친구 중 하나. 구문 분석과 코드 생성의 더러운 작업을 모두 처리한다(이전 섹션 참고).

* *스코프 매니저*: *엔진*의 다른 친구. 선언된 모든 변수/식별자의 룩업 목록을 수집하고 유지하며, 현재 실행하는 코드에서 접근 가능한 방법에 대한 규칙 집합을 적용한다.

JavaScript가 작동하는 방식을 *완전히 이해*하려면 *엔진*(및 친구)처럼 *생각*하는 것을 시작하고, 질문하고, 답할 필요가 있다.

대화를 살펴보기 위해 실행 중인 프로그램 예제를 다시 한 번 상기하자:

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

JS가 어떻게 프로그램을 처리할 것인지, 구체적으로 첫번째 구문부터 조사하자. 배열과 그것의 내용은 단지 기본적인 JS 값 표현식이다(따라서 스코프 문제에는 영향을 받지 않음). 그래서 여기서의 초점은 `var students = [ .. ]` 선언과 초기 할당 부분이 될 것이다.

우리는 일반적으로 단 하나의 구문으로서 생각하지만, 우리의 친구 *엔진*이 보는 방식은 아니다. 실제로 JS는 두 가지 개별 작업으로 취급한다. 하나는 *컴파일러*가 컴파일 중에 다룰 것이고 다른 하나는 *엔진*이 실행중에 다룰 것이다.

이 프로그램을 가지고 *컴파일러*가 할 첫번째 것은 토큰으로 나누기위해 어휘 분석<sub>lexing</sub>를 수행하고 트리(AST)로 구문 분석한다.

*컴파일러*가 코드 생성에 착수하면, 분명한 것보다 더 많은 세부 사항을 고려해야 한다. 합리적인 가정은 *컴파일러*가 "변수에 대한 메모리를 할당하고, `students` 레이블을 지정하고, 배열에 대한 참조를 이 변수에 붙인다"와 같은 첫 번째 구문에 대한 코드를 생성한다는 것이다. 하지만 이것이 전부는 아니다.

*컴파일러*가 이 구문을 다루기 위해 전개되는 단계가 있다:

1. `var students`를 접하면 *컴파일러*는 *스코프 매니저*에게 특정 스코프 양동이에 `students`라는 변수가 이미 존재하는지 물을 것이다. 만약 그렇다면, *컴파일러*는 이 선언을 무시하고 이동한다. 그렇지 않으면 *컴파일러*는 (실행 타임에) 스코프 양동이에 `students`라 불리는 새 변수를 생성하도록 *스코프 매니저*에게 요청하는 코드를 생성할 것이다.

2. *컴파일러*는  `students = []` 할당을 다루기 위해 나중에 실행할 *엔진*을 위한 코드를 생성한다. *엔진*이 실행하는 코드는 먼저 *스코프 매니저*에게 현재 스코프 양동이에서 접근할 수 있는 `students`라는 변수가 있는지 물을 것이다. 그렇지 않다면 *엔진*은 계속 다는 곳을 찾는다(아래 "중첩 스코프" 참고). *엔진*이 변수를 찾으면 `[ .. ]` 배열의 참조를 할당한다.

대화 형식으로, 프로그램을 위한 컴파일의 첫번째 단계는 아래처럼 *컴파일러*와 *스코프 매니저* 사이에서 흐를지도 모른다:

> ***컴파일러***: 안녕, (전역 스코프의) *스코프 매니저*야. `students`라 부르는 식별자에 대한 공식적인 선언을 찾았는데, 들어본 적 있니?

> ***(전역) 스코프 매니저***: 아니, 들어본 적 없어서 지금 막 만들었어.

> ***컴파일러***: 안녕, *스코프 매니저*야. `getStudentName`이라 부르는 식별자에 대한 공식적인 선언을 찾았는데, 들어본 적 있니?

> ***(전역) 스코프 매니저***: 아니, 들어본 적 없어서 지금 막 만들었어.

> ***컴파일러***: 안녕, *스코프 매니저*야. `getStudentName`는 함수를 가리켜서 새 스코프 양동이가 필요해.

> ***(함수) 스코프 매니저***: 알았어, 여기 스코프 양동이야.

> ***컴파일러***: 안녕, (함수의) *스코프 매니저*야. `studentID`이라 부르는 식별자에 대한 공식적인 선언을 찾았는데, 들어본 적 있니?

> ***(함수) 스코프 매니저***: 없어, 하지만 지금 이 스코프에 만들었어.

> ***컴파일러***: 안녕, (함수의) *스코프 매니저*야. 스코프 양동이가 필요한 `for` 반복문을 찾았어.

> ...

이 대화는 질의 응답 교환으로, **컴파일러**가 *스코프 매니저*에게 마주친 식별자 선언이 이미 발견되었는지 묻는다. "아니오"라면, *스코프 매니저*는 스코프에 변수를 생성한다. "예"라면, *스코프 매니저*가 더 이상 할 것이 없으므로 실질적으로 건너뛴다.

*컴파일러*는 함수나 블록 스코프를 지나갈 때도 새 스코프 양동이와 *스코프 매니저*가 실체화 될 수 있는 신호를 보낸다.

나중에 프로그램의 실행 단계에 왔을 때, 대화는 *엔진*과 *스코프 매니저*로 전환되고, 아래처럼 흐를지도 모른다:

> ***엔진***: 안녕, (전역 스코프의) *스코프 매니저*야. 시작하기 전에, 너는 식별자 `getStudentName`를 찾을 수 있니?(<-원문에 ?를 빼먹은 것 같음) 그렇다면 내가 그것에 함수를 할당할 수 있을까?

> ***(전역) 스코프 매니저***: 응, 여기 변수가 있어.

> ***엔진***: 안녕, *스코프 매니저*야. `students`에 대한 *타깃* 참조를 찾았는데, 들어본 적 있니?

> ***(전역) 스코프 매니저***: 응, 이 스코프에서 공식적으로 선언됐어. 여기 있어.

> ***엔진***: 고마워, `students`를 `undefined`로 초기화하는 중이야. 사용할 준비가 됐어.

> 안녕, (전역 스코프의) *스코프 매니저*야. `nextStudent`에 대한 *타깃* 참조를 찾았는데, 들어본 적 있니?

> ***(전역) 스코프 매니저***: 응, 이 스코프에서 공식적으로 선언됐어. 여기 있어.

> ***엔진***: 고마워, `nextStudent`를 `undefined`로 초기화하는 중이야. 사용할 준비가 됐어.

> 안녕, (전역 스코프의) *스코프 매니저*야. `getStudentName`에 대한 *소스* 참조를 찾았는데, 들어본 적 있니?

> ***(전역) 스코프 매니저***: 응, 이 스코프에서 공식적으로 선언됐어. 여기 있어.

> ***엔진***: 좋아, `getStudentName`의 값은 함수야. 이걸 실행할꺼야.

> ***엔진***: 안녕, *스코프 매니저*야. 지금 우리는 이 함수의 스코프를 실체화할 필요가 있어.

> ...

이 대화는 다른 질의 응답 교환으로, *엔진*이 먼저 *스코프 매니저*에게 호이스트된<sub>hoisted</sub>`getStudentName` 식별자를 찾고 함수를 연관시키기 위해 묻는다. 그 다음에 *엔진*은 *스코프 매니저*에게 `students`에 대한 *타깃* 참조 등을 묻기 위해 나아간다.

아래 두 가지 단계는 `var students = [ .. ]`같은 구문을 어떻게 처리되는지 검토하고 요약한다:

1. *컴파일러*는 (현재 스코프에서 이전에 선언된 적이 없다면) 스코프 변수의 선언을 설정한다.

2. *엔진*은 실행하는 동안, 구문의 할당 부분을 처리하고, *스코프 매니저*에게 변수 찾는 것을 요청하고, `undefined`로 초기화하여 사용할 준비를 한다. 그리고 배열 값을 할당한다.

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
