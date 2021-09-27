# You Don't Know JS Yet: Scope & Closures - 2nd Edition
# Chapter 6: 스코프 노출을 제한하기

지금까지 스코프와 변수가 어떻게 작동하는지 그 동작 방식을 설명하는 것에 집중했다. 이러한 기초 지식이 확고히 자리 잡으면서 우리의 관점은 프로그램 전체에 적용되는 결정과 패턴까지 확장되며, 이전보다 높은 수준으로 상승하게 된다.

먼저, 프로그램 변수를 스코프의 과다한 노출을 줄이는 방식으로 조직하기 위해 다른 스코프 레벨(함수와 블록)을 사용해야 하는 이유와 방법에 대해 알아볼 것이다.

## 노출의 최소화

함수가 자체 스코프를 정의하다는 것은 이해가 될 것이다. 하지만, 스코프를 생성하기 위해 다시 블록이 필요한 이유는 무엇일까?

소프트웨어 엔지니어링에서는 일반적으로 소프트웨어 보안에 적용되는 기본 원칙으로 "최소 권한 원칙<sub>The Principle of Least Privilege</sub>"(POLP). [^POLP]을 명시하고 있다. 그리고 현재 논의하고 있는 것에 적용되는 이 원칙의 변형으로 흔히 "최소 노출<sub>Least Exposure</sub"(POLE)이란 것이 있다.

POLP는 소프트웨어 아키텍처에 대한 방어태새를 나타낸다. 시스템의 구성 요소는 최소 권한, 최소 접근, 최소 노출로 작동하도록 설계되어야 한다. 각 부분이 최소한의 필요 기능으로 연결되어 있어야 하나의 부분이 손상되거나 실패하더라도 시스템의 나머지 부분에 미치는 영향이 최소화되기 때문에 보안 측면에서 전체 시스템이 더 강력해진다.

POLP가 시스템 수준의 구성요소 설계에 초점을 맞춘다면 *POLE*은 더 낮은 수준에 초점을 맞춘다. 여기서는 스코프가 서로 상호 작용하는 방식에 적용할 것이다.

다음 POLE에서 노출을 최소화하려는 것은 무엇일까? 단순히 각 스코프에 등록된 변수이다.

이렇게 생각해보자. 프로그램의 모든 변수를 글로벌 스코프에 배치한다면 어떨까? 당장은 나쁜 아이디어라고 느껴지겠지만, 왜 그런지 생각해 볼 필요가 있다. 프로그램의 한 부분에서 사용하는 변수가 스코프를 통해 프로그램의 다른 부분에 노출되는 경우 다음과 같은 세 가지 주요 위험이 발생한다.

* **네이밍 충돌**: 프로그램의 서로 다른 두 부분에서 공통적이고 유용한 변수/함수 이름을 사용하지만 식별자가 하나의 (전역 스코프처럼) 공유된 스코프에서 온 경우 이름 충돌이 발생한다. 이런 경우, 다른 부분의 코드에서 예상하지 못한 방식으로 변수/함수를 사용할 때 버그가 발생하기 쉽다.

    예를 들어, 모든 반복문에서 하나의 전역 변수의 이름으로 'i'를 사용했을때, 어떤 함수에서 반복문 내부를 반복하는 동안 다른 함수의 반복문이 실행되어 공유된 'i' 변수가 예기치 않은 값을 얻게 될 것을 생각해보라.

* **예상치 않은 동작**: 용도가 *프라이빗*한 변수/함수를 프로그램의 일부에 노출하면 다른 개발자가 의도하지 않은 방식으로 사용할 수 있으므로 예상된 동작을 위반하고 버그를 발생시킬 수 있다.

    예를 들어, 당신의 프로그램 한 부분은 어떤 배열이 숫자만 포함되어 있을것으로 추정하고 있었는데 다른 사용자의 코드는 그 배열이 참/거짓 값과 문자열을 포함하도록 수정할 수 있다. 이 경우, 예기치 않은 방식으로 코드가 잘못 동작할 수 있다.

    게다가 *프라이빗*한 세부 정보가 노출되면 의도하지 않은 사용자가 당신이 도입한 제약 사항을 침범하려 시도하게 되고 소프트웨어 내부의 허용되지 않는 부분을 사용해 작업을 수행할 것이다.

* **의도하지 않은 종속성**: 변수/함수를 불필요하게 노출하면 다른 개발자가 *프라이빗*한 부분을 사용하고 의존하게 된다. 지금 바로 프로그램이 중단되는 것은 아니지만 향후 리팩토링 위험을 발생시킨다. 왜냐하면 이제 제어하지 않는 소프트웨어의 일부를 잠재적으로 손상시키지 않고서 변수나 기능을 쉽게 리팩토링할 수 없기 때문이다.

    예를 들어, 숫자 배열에 의존하는 코드를 사용하다가 배열 대신 다른 데이터 구조를 사용하는 것이 낫다고 판단한 경우, 이제 당신은 소프트웨어에서 영향받는 다른 부분을 조정할 책임을 져야 한다.

POLE은 변수/함수 스코프 지정에 적용하는 것으로 기본적으로 필요한 최소한의 노출을 기본 설정하며 다른 모든 것은 최대한 비공개로 유지한다. 또한 모든 항목을 전역(또는 외부 함수) 스코프에 배치하지 말고 가능한 작고 깊게 중첩된 스코프에서 변수를 선언한다.

위의 조언대로 소프트웨어를 설계할 경우 이러한 세 가지 위험을 피할(적어도 최소화할) 수 있는 가능성이 훨씬 커진다.

다음 코드를 살펴보자:

```js
function diff(x,y) {
    if (x > y) {
        let tmp = x;
        x = y;
        y = tmp;
    }

    return y - x;
}

diff(3,7);      // 4
diff(7,5);      // 2
```

위의 'diff(..)' 함수에서 우리는 'y'가 'x'보다 크거나 같도록 하여 ('y - x') 뺄 때 결과가 '0' 이상이 되도록 하고자 한다. 처음에 'x'가 더 크면(결과는 음수가 된다!), 'tmp' 변수를 사용해서 'x'와 'y'를 교환하고 결과를 양수로 유지한다.

위 단순한 예시에서, `tmp`가 `if` 블록 안에 있는지, 함수 스코프에 속하는지 여부는 중요하지 않아 보인다. 물론 전역 변수가 되어서는 안 됩니다! 그러나 POLE 원칙에 따라 `tmp`는 가능한한 스코프 내부에 숨겨야한다. 그래서 (`let`을 사용하여) `tmp`의 스코프를`if` 블록까지로 차단한다.

## 평범한(함수) 스코프에 숨기기

이제, 변수 와 함수 선언을 가능한 가장 (깊게 중첩된) 아래의 스코프로 숨기는 것이 중요한 이유를 확실히 알게 되었다. 그러나 이것은 어떻게 하는 것일까?

이미 블록 스코프의 선언문인 `let`과 `const` 키워드를 다루었고, 조만간 다시 자세히 살펴보겠다. 그 전에 먼저 `var`나 `function` 선언을 스코프 내부에 숨기는 것은 어떨까? 선언문을 `function` 스코프로 감싸서 쉽게 숨길 수 있다.

`function` 스코프 지정이 유용할 수 있는 예를 들어보자.

수학 연산자인 "팩토리얼"("6!"으로 표기)은 주어진 정수부터 `1`까지 내려가며 연속으로 곱한 값이다. 사실 `1`을 곱하는 것은 의미가 없으므로 `2`에서 멈출 수 있다. 즉, "6!"은 "6 * 5!"와 같고 "6 * 5 * 4!" 등과 같다. 관련된 수학적 특성 때문에, 주어진 정수의 팩토리얼(예: "4!") 값은 달라지지 않으므로 한 번만 계산하고 다시 계산할 필요가 없다.

따라서 아무런 계획 없이 `6`에 대한 팩토리얼을 계산한 다음 `7`에 대한 팩토리얼을 계산한다면 2에서 6까지 모든 정수의 팩토리얼을 불필요하게 다시 계산할 수도 있다. 만약 메모리와 속도를 교환할 의향이 있다면 계산 시 각 정수의 팩토리얼을 캐싱하여 불필요한 연산을 아낄 수 있다.

```js
var cache = {};

function factorial(x) {
    if (x < 2) return 1;
    if (!(x in cache)) {
        cache[x] = x * factorial(x - 1);
    }
    return cache[x];
}

factorial(6);
// 720

cache;
// {
//     "2": 2,
//     "3": 6,
//     "4": 24,
//     "5": 120,
//     "6": 720
// }

factorial(7);
// 5040
```

위 코드에서는 계산한 팩토리얼 값을 `cache`에 저장하여 여러 번 `factorial(...)`을 호출하더라도 이전에 계산한 값을 유지하도록 한다. 하지만 `cache` 변수는 `factorial(..)` 동작 방식에 관련된 상당히 *프라이빗*한 세부사항으로, 특히 글로벌 스코프에는 노출하지 말아야 하는 요소이다.

| NOTE: |
| :--- |
| 이 `factorial(..)`은 내부에서 자신을 다시 호출하기 때문에 재귀적인데, 코드를 단지 간결하게 하기 위함이었다. 재귀적이지 않게 구현하더라도 `cache`와 관련된 스코프에 대한 분석은 동일할 것이다. |

그러나 이러한 과노출 문제를 해결하는 것은 `cache` 변수를 `factorial(..)` 안에 숨기는 것만으로 끝나지 않는다. 여러번 호출하더라도 `cache`를 유지해야 하므로 해당 함수의 바깥 스코프에 두어야 한다. 그렇다면 이 문제를 어떻게 해결할 수 있을까?

`cache`가 위치할 다른 중간 스코프(외부/전역 스코프와 `factorial(..)`의 내부 사이)를 정의하자.

```js
// 외부/전역 스코프

function hideTheCache() {
    // `cache`를 숨길 "중간 스코프"
    var cache = {};

    return factorial;

    // **********************

    function factorial(x) {
        // 내부 스코프
        if (x < 2) return 1;
        if (!(x in cache)) {
            cache[x] = x * factorial(x - 1);
        }
        return cache[x];
    }
}

var factorial = hideTheCache();

factorial(6);
// 720

factorial(7);
// 5040
```

`hideTheCache()` 함수는 `factorial(..)`을 여러 번 호출하더라도 `cache`가 값을 유지할 수 있도록 하는 스코프를 만드는 것 외에 다른 목적이 없다. 그러나 `factorial(..)`이 `cache`에 접근하기 위해서는 같은 스코프 내부에 `factorial(..)`을 정의해야 한다. 그 다음엔 함수 참조를 `hideTheCache()`의 값으로 반환하고 `factorial`이라는 외부 스코프 변수에 저장한다. 이제 (여러 번) `factorial(..)`를 호출해도 유지되는 `cache`는 숨겨진 채로 남게 되지만 `factorial(..)`에서만 접근할 수 있다!

좋다, 하지만... 변수나 함수를 숨겨야 할 일이 있을 때마다 매번 `hideTheCache(..)`함수 스코프(와 이름)를 정의하는 것은 지루할 것이다. 특히 각 항목에 고유한 이름을 지정하여 이 함수와 이름이 충돌하지 않도록 해야 하기 때문이다. 으윽

| NOTE: |
| :--- |
| 동일한 입력으로 호출하는 것이 반복될 것으로 예상되는 경우, 성능을 최적화하기 위해 함수에서 계산한 결과를 캐싱하는 이 기법은 함수형 프로그래밍 세계에서는 매우 일반적이다. 이 기법은 "메모이제이션"으로 불리며 클로저(7창 참조)에 의존한다. 또한 메모리 사용량 문제가 있다. (부록 B의 "메모리에 관련된 단어"에서 다룰 예정) 함수형 프로그래밍 라이브러리는 주로 함수의 메모이제이션에 최적화된 기능을 제공하며, 이 기능은 `hideTheCache(..)`를 대체한다. 메모이제이션은 지금 논의하고 있는 내용의 *스코프* (의도한 말장난!)를 벗어난다. 자세한 내용이 궁금하다면 *Functional-Light JavaScript* 책을 참조하라. |

이렇게 변수를 숨기기 위한 목적만으로 스코프를 생성해야 하는 상황이 발생할 때마다 고유한 이름을 붙여서 새로운 함수를 정의하는 대신 다음과 같은 함수 표현식을 사용하는 것이 더 나을 수 있다.

```js
var factorial = (function hideTheCache() {
    var cache = {};

    function factorial(x) {
        if (x < 2) return 1;
        if (!(x in cache)) {
            cache[x] = x * factorial(x - 1);
        }
        return cache[x];
    }

    return factorial;
})();

factorial(6);
// 720

factorial(7);
// 5040
```

잠깐! 이 함수는 여전히 `cache`를 숨기기 위한 스코프를 만드는 목적으로 함수를 사용하고 있으며 위 경우에 함수의 이름은 여전히 `hideTheCache`이다. 그러면 이 문제는 어떻게 해결될까?

"함수 이름 스코프"(3장)를 참조하여 `function` 표현식에서 이름 식별자가 어떻게 되는지 확인하라. `hideTheCache(..)`를 `function`' 선언 대신 `function` 표현식으로 정의했기 때문에 이 이름은 외부/전역 스코프가 아닌 자체 스코프, 기본적으로 `cache`와 동일한 스코프에에 있다.

즉, 이렇게 함수 표현식을 사용한 부분마다 내부에서 함수의 이름을 동일하게 지정할 수 있으며, 충돌이 발생하지 않는다. 좀 더 적절하게, 숨기려는 것이 무엇이든 간에 각각의 함수에 의미론적으로 이름을 지정할 수 있고, 어떤 이름을 선택하든 프로그램의 다른 `function` 표현식의 스코프와 충돌할 것이라는 걱정은 하지 않아도 된다.

실제로 이름 전체를 *생략할 수도 있고*, 따라서 그 대신 "익명의 `function` 표현식"을 정의할 수 있다. 그러나 부록 A에서는 그러한 스코프 전용 함수에 대해서도 이름의 중요성에 대해 설명할 것이다.

### 즉시 실행 함수 표현식

위에서 다룬 팩토리얼 재귀 함수 코드에 지나치기 쉬운 중요한 부분이 있다. 바로 `function` 표현식의 마지막 줄에 있는 `})();` 부분이다.

`function` 표현식 전체를 `( .. )`로 감싸고 있다는 것에 주목하라. 마지막 부분에 두 번째로 `()` 괄호 한 쌍을 추가했는데 이 부분이 방금 `function` 표현식으로 정의한 함수를 호출하는 것이다. 이 때, 함수 표현식을 감싸는 첫 번째 `( .. )` 괄호가 꼭 필요한 것은 아니지만 가독성을 위해 사용한다.
 
즉, 위 코드는 즉시 실행되는 `function` 표현식을 정의하고 있다. 흔히 사용하는 이 패턴의 (매우 기발한) 이름은 바로 즉시 실행 함수 표현식<sub>Immediately Invoked Function Expression</sub>(IIFE)이다.

IIFE는 변수/함수를 숨기기 위해 새 스코프를 생성해야 할 때 유용하다. 단지 표현식이기 때문에, 표현식을 사용할 수 있는 JS 프로그램의 **모든** 위치에서 사용할 수 있다. IIFE에는 `hideTheCache()` 같은 이름을 붙여 줄 수도 있고 (좀 더 일반적으로) 익명으로 사용할 수도 있다. 또, 이 표현식은 독립적으로 사용할 수도 있고 다른 구문의 일부로 사용할 수도 있다. 위 코드에서는 `hideTheCache()`가 `factorial()` 함수 참조를 반환하는데 이것을 `=`을 통해 `factorial` 변수에 할당하고 있다.

IIFE를 독립적으로 사용하는 예제와 비교해보자:

```js
// 외부 스코프

(function(){
    // 안으로 숨긴 스코프
})();

// 또 다른 외부 스코프
```

앞서 살펴본 `hideTheCache()`에서는 외부를 둘러싸는 `(..)`가 겉모습을 위한 선택지에 불과했지만, IIFE를 독립적으로 사용하는 이 예제에서는 **필수적**인 것이 되었다. `function`을 구문이 아닌 표현식으로 구별해야 하기 때문이다. 물론, 일관성을 위해 IIFE `function`은 항상  `( .. )`로 감싸야 한다. 

| 비고: |
| :--- |
| 기술적인 관점으로 보면, `( .. )`로 감싸는 기법만이 JS 파서가 IIFE 내부의 `function`를 함수 표현식으로 취급하도록 하는 유일한 구문론적 방법은 아니다. 이 외의 방법은 부록 A에서 살펴볼 것이다. |

#### Function Boundaries

Beware that using an IIFE to define a scope can have some unintended consequences, depending on the code around it. Because an IIFE is a full function, the function boundary alters the behavior of certain statements/constructs.

For example, a `return` statement in some piece of code would change its meaning if an IIFE is wrapped around it, because now the `return` would refer to the IIFE's function. Non-arrow function IIFEs also change the binding of a `this` keyword—more on that in the *Objects & Classes* book. And statements like `break` and `continue` won't operate across an IIFE function boundary to control an outer loop or block.

So, if the code you need to wrap a scope around has `return`, `this`, `break`, or `continue` in it, an IIFE is probably not the best approach. In that case, you might look to create the scope with a block instead of a function.

## Scoping with Blocks

You should by this point feel fairly comfortable with the merits of creating scopes to limit identifier exposure.

So far, we looked at doing this via `function` (i.e., IIFE) scope. But let's now consider using `let` declarations with nested blocks. In general, any `{ .. }` curly-brace pair which is a statement will act as a block, but **not necessarily** as a scope.

A block only becomes a scope if necessary, to contain its block-scoped declarations (i.e., `let` or `const`). Consider:

```js
{
    // not necessarily a scope (yet)

    // ..

    // now we know the block needs to be a scope
    let thisIsNowAScope = true;

    for (let i = 0; i < 5; i++) {
        // this is also a scope, activated each
        // iteration
        if (i % 2 == 0) {
            // this is just a block, not a scope
            console.log(i);
        }
    }
}
// 0 2 4
```

Not all `{ .. }` curly-brace pairs create blocks (and thus are eligible to become scopes):

* Object literals use `{ .. }` curly-brace pairs to delimit their key-value lists, but such object values are **not** scopes.

* `class` uses `{ .. }` curly-braces around its body definition, but this is not a block or scope.

* A `function` uses `{ .. } ` around its body, but this is not technically a block—it's a single statement for the function body. It *is*, however, a (function) scope.

* The `{ .. }` curly-brace pair on a `switch` statement (around the set of `case` clauses) does not define a block/scope.

Other than such non-block examples, a `{ .. }` curly-brace pair can define a block attached to a statement (like an `if` or `for`), or stand alone by itself—see the outermost `{ .. }` curly brace pair in the previous snippet. An explicit block of this sort—if it has no declarations, it's not actually a scope—serves no operational purpose, though it can still be useful as a semantic signal.

Explicit standalone `{ .. }` blocks have always been valid JS syntax, but since they couldn't be a scope prior to ES6's `let`/`const`, they are quite rare. However, post ES6, they're starting to catch on a little bit.

In most languages that support block scoping, an explicit block scope is an extremely common pattern for creating a narrow slice of scope for one or a few variables. So following the POLE principle, we should embrace this pattern more widespread in JS as well; use (explicit) block scoping to narrow the exposure of identifiers to the minimum practical.

An explicit block scope can be useful even inside of another block (whether the outer block is a scope or not).

For example:

```js
if (somethingHappened) {
    // this is a block, but not a scope

    {
        // this is both a block and an
        // explicit scope
        let msg = somethingHappened.message();
        notifyOthers(msg);
    }

    // ..

    recoverFromSomething();
}
```

Here, the `{ .. }` curly-brace pair **inside** the `if` statement is an even smaller inner explicit block scope for `msg`, since that variable is not needed for the entire `if` block. Most developers would just block-scope `msg` to the `if` block and move on. And to be fair, when there's only a few lines to consider, it's a toss-up judgement call. But as code grows, these over-exposure issues become more pronounced.

So does it matter enough to add the extra `{ .. }` pair and indentation level? I think you should follow POLE and always (within reason!) define the smallest block for each variable. So I recommend using the extra explicit block scope as shown.

Recall the discussion of TDZ errors from "Uninitialized Variables (TDZ)" (Chapter 5). My suggestion there was: to minimize the risk of TDZ errors with `let`/`const` declarations, always put those declarations at the top of their scope.

If you find yourself placing a `let` declaration in the middle of a scope, first think, "Oh, no! TDZ alert!" If this `let` declaration isn't needed in the first half of that block, you should use an inner explicit block scope to further narrow its exposure!

Another example with an explicit block scope:

```js
function getNextMonthStart(dateStr) {
    var nextMonth, year;

    {
        let curMonth;
        [ , year, curMonth ] = dateStr.match(
                /(\d{4})-(\d{2})-\d{2}/
            ) || [];
        nextMonth = (Number(curMonth) % 12) + 1;
    }

    if (nextMonth == 1) {
        year++;
    }

    return `${ year }-${
            String(nextMonth).padStart(2,"0")
        }-01`;
}
getNextMonthStart("2019-12-25");   // 2020-01-01
```

Let's first identify the scopes and their identifiers:

1. The outer/global scope has one identifier, the function `getNextMonthStart(..)`.

2. The function scope for `getNextMonthStart(..)` has three: `dateStr` (parameter), `nextMonth`, and `year`.

3. The `{ .. }` curly-brace pair defines an inner block scope that includes one variable: `curMonth`.

So why put `curMonth` in an explicit block scope instead of just alongside `nextMonth` and `year` in the top-level function scope? Because `curMonth` is only needed for those first two statements; at the function scope level it's over-exposed.

This example is small, so the hazards of over-exposing `curMonth` are pretty limited. But the benefits of the POLE principle are best achieved when you adopt the mindset of minimizing scope exposure by default, as a habit. If you follow the principle consistently even in the small cases, it will serve you more as your programs grow.

Let's now look at an even more substantial example:

```js
function sortNamesByLength(names) {
    var buckets = [];

    for (let firstName of names) {
        if (buckets[firstName.length] == null) {
            buckets[firstName.length] = [];
        }
        buckets[firstName.length].push(firstName);
    }

    // a block to narrow the scope
    {
        let sortedNames = [];

        for (let bucket of buckets) {
            if (bucket) {
                // sort each bucket alphanumerically
                bucket.sort();

                // append the sorted names to our
                // running list
                sortedNames = [
                    ...sortedNames,
                    ...bucket
                ];
            }
        }

        return sortedNames;
    }
}

sortNamesByLength([
    "Sally",
    "Suzy",
    "Frank",
    "John",
    "Jennifer",
    "Scott"
]);
// [ "John", "Suzy", "Frank", "Sally",
//   "Scott", "Jennifer" ]
```

There are six identifiers declared across five different scopes. Could all of these variables have existed in the single outer/global scope? Technically, yes, since they're all uniquely named and thus have no name collisions. But this would be really poor code organization, and would likely lead to both confusion and future bugs.

We split them out into each inner nested scope as appropriate. Each variable is defined at the innermost scope possible for the program to operate as desired.

`sortedNames` could have been defined in the top-level function scope, but it's only needed for the second half of this function. To avoid over-exposing that variable in a higher level scope, we again follow POLE and block-scope it in the inner explicit block scope.

### `var` *and* `let`

Next, let's talk about the declaration `var buckets`. That variable is used across the entire function (except the final `return` statement). Any variable that is needed across all (or even most) of a function should be declared so that such usage is obvious.

| NOTE: |
| :--- |
| The parameter `names` isn't used across the whole function, but there's no way limit the scope of a parameter, so it behaves as a function-wide declaration regardless. |

So why did we use `var` instead of `let` to declare the `buckets` variable? There's both semantic and technical reasons to choose `var` here.

Stylistically, `var` has always, from the earliest days of JS, signaled "variable that belongs to a whole function." As we asserted in "Lexical Scope" (Chapter 1), `var` attaches to the nearest enclosing function scope, no matter where it appears. That's true even if `var` appears inside a block:

```js
function diff(x,y) {
    if (x > y) {
        var tmp = x;    // `tmp` is function-scoped
        x = y;
        y = tmp;
    }

    return y - x;
}
```

Even though `var` is inside a block, its declaration is function-scoped (to `diff(..)`), not block-scoped.

While you can declare `var` inside a block (and still have it be function-scoped), I would recommend against this approach except in a few specific cases (discussed in Appendix A). Otherwise, `var` should be reserved for use in the top-level scope of a function.

Why not just use `let` in that same location? Because `var` is visually distinct from `let` and therefore signals clearly, "this variable is function-scoped." Using `let` in the top-level scope, especially if not in the first few lines of a function, and when all the other declarations in blocks use `let`, does not visually draw attention to the difference with the function-scoped declaration.

In other words, I feel `var` better communicates function-scoped than `let` does, and `let` both communicates (and achieves!) block-scoping where `var` is insufficient. As long as your programs are going to need both function-scoped and block-scoped variables, the most sensible and readable approach is to use both `var` *and* `let` together, each for their own best purpose.

There are other semantic and operational reasons to choose `var` or `let` in different scenarios. We'll explore the case for `var` *and* `let` in more detail in Appendix A.

| WARNING: |
| :--- |
| My recommendation to use both `var` *and* `let` is clearly controversial and contradicts the majority. It's far more common to hear assertions like, "var is broken, let fixes it" and, "never use var, let is the replacement." Those opinions are valid, but they're merely opinions, just like mine. `var` is not factually broken or deprecated; it has worked since early JS and it will continue to work as long as JS is around. |

### Where To `let`?

My advice to reserve `var` for (mostly) only a top-level function scope means that most other declarations should use `let`. But you may still be wondering how to decide where each declaration in your program belongs?

POLE already guides you on those decisions, but let's make sure we explicitly state it. The way to decide is not based on which keyword you want to use. The way to decide is to ask, "What is the most minimal scope exposure that's sufficient for this variable?"

Once that is answered, you'll know if a variable belongs in a block scope or the function scope. If you decide initially that a variable should be block-scoped, and later realize it needs to be elevated to be function-scoped, then that dictates a change not only in the location of that variable's declaration, but also the declarator keyword used. The decision-making process really should proceed like that.

If a declaration belongs in a block scope, use `let`. If it belongs in the function scope, use `var` (again, just my opinion).

But another way to sort of visualize this decision making is to consider the pre-ES6 version of a program. For example, let's recall `diff(..)` from earlier:

```js
function diff(x,y) {
    var tmp;

    if (x > y) {
        tmp = x;
        x = y;
        y = tmp;
    }

    return y - x;
}
```

In this version of `diff(..)`, `tmp` is clearly declared in the function scope. Is that appropriate for `tmp`? I would argue, no. `tmp` is only needed for those few statements. It's not needed for the `return` statement. It should therefore be block-scoped.

Prior to ES6, we didn't have `let` so we couldn't *actually* block-scope it. But we could do the next-best thing in signaling our intent:

```js
function diff(x,y) {
    if (x > y) {
        // `tmp` is still function-scoped, but
        // the placement here semantically
        // signals block-scoping
        var tmp = x;
        x = y;
        y = tmp;
    }

    return y - x;
}
```

Placing the `var` declaration for `tmp` inside the `if` statement signals to the reader of the code that `tmp` belongs to that block. Even though JS doesn't enforce that scoping, the semantic signal still has benefit for the reader of your code.

Following this perspective, you can find any `var` that's inside a block of this sort and switch it to `let` to enforce the semantic signal already being sent. That's proper usage of `let` in my opinion.

Another example that was historically based on `var` but which should now pretty much always use `let` is the `for` loop:

```js
for (var i = 0; i < 5; i++) {
    // do something
}
```

No matter where such a loop is defined, the `i` should basically always be used only inside the loop, in which case POLE dictates it should be declared with `let` instead of `var`:

```js
for (let i = 0; i < 5; i++) {
    // do something
}
```

Almost the only case where switching a `var` to a `let` in this way would "break" your code is if you were relying on accessing the loop's iterator (`i`) outside/after the loop, such as:

```js
for (var i = 0; i < 5; i++) {
    if (checkValue(i)) {
        break;
    }
}

if (i < 5) {
    console.log("The loop stopped early!");
}
```

This usage pattern is not terribly uncommon, but most feel it smells like poor code structure. A preferable approach is to use another outer-scoped variable for that purpose:

```js
var lastI;

for (let i = 0; i < 5; i++) {
    lastI = i;
    if (checkValue(i)) {
        break;
    }
}

if (lastI < 5) {
    console.log("The loop stopped early!");
}
```

`lastI` is needed across this whole scope, so it's declared with `var`. `i` is only needed in (each) loop iteration, so it's declared with `let`.

### What's the Catch?

So far we've asserted that `var` and parameters are function-scoped, and `let`/`const` signal block-scoped declarations. There's one little exception to call out: the `catch` clause.

Since the introduction of `try..catch` back in ES3 (in 1999), the `catch` clause has used an additional (little-known) block-scoping declaration capability:

```js
try {
    doesntExist();
}
catch (err) {
    console.log(err);
    // ReferenceError: 'doesntExist' is not defined
    // ^^^^ message printed from the caught exception

    let onlyHere = true;
    var outerVariable = true;
}

console.log(outerVariable);     // true

console.log(err);
// ReferenceError: 'err' is not defined
// ^^^^ this is another thrown (uncaught) exception
```

The `err` variable declared by the `catch` clause is block-scoped to that block. This `catch` clause block can hold other block-scoped declarations via `let`. But a `var` declaration inside this block still attaches to the outer function/global scope.

ES2019 (recently, at the time of writing) changed `catch` clauses so their declaration is optional; if the declaration is omitted, the `catch` block is no longer (by default) a scope; it's still a block, though!

So if you need to react to the condition *that an exception occurred* (so you can gracefully recover), but you don't care about the error value itself, you can omit the `catch` declaration:

```js
try {
    doOptionOne();
}
catch {   // catch-declaration omitted
    doOptionTwoInstead();
}
```

This is a small but delightful simplification of syntax for a fairly common use case, and may also be slightly more performant in removing an unnecessary scope!

## Function Declarations in Blocks (FiB)

We've seen now that declarations using `let` or `const` are block-scoped, and `var` declarations are function-scoped. So what about `function` declarations that appear directly inside blocks? As a feature, this is called "FiB."

We typically think of `function` declarations like they're the equivalent of a `var` declaration. So are they function-scoped like `var` is?

No and yes. I know... that's confusing. Let's dig in:

```js
if (false) {
    function ask() {
        console.log("Does this run?");
    }
}
ask();
```

What do you expect for this program to do? Three reasonable outcomes:

1. The `ask()` call might fail with a `ReferenceError` exception, because the `ask` identifier is block-scoped to the `if` block scope and thus isn't available in the outer/global scope.

2. The `ask()` call might fail with a `TypeError` exception, because the `ask` identifier exists, but it's `undefined` (since the `if` statement doesn't run) and thus not a callable function.

3. The `ask()` call might run correctly, printing out the "Does it run?" message.

Here's the confusing part: depending on which JS environment you try that code snippet in, you may get different results! This is one of those few crazy areas where existing legacy behavior betrays a predictable outcome.

The JS specification says that `function` declarations inside of blocks are block-scoped, so the answer should be (1). However, most browser-based JS engines (including v8, which comes from Chrome but is also used in Node) will behave as (2), meaning the identifier is scoped outside the `if` block but the function value is not automatically initialized, so it remains `undefined`.

Why are browser JS engines allowed to behave contrary to the specification? Because these engines already had certain behaviors around FiB before ES6 introduced block scoping, and there was concern that changing to adhere to the specification might break some existing website JS code. As such, an exception was made in Appendix B of the JS specification, which allows certain deviations for browser JS engines (only!).

| NOTE: |
| :--- |
| You wouldn't typically categorize Node as a browser JS environment, since it usually runs on a server. But Node's v8 engine is shared with Chrome (and Edge) browsers. Since v8 is first a browser JS engine, it adopts this Appendix B exception, which then means that the browser exceptions are extended to Node. |

One of the most common use cases for placing a `function` declaration in a block is to conditionally define a function one way or another (like with an `if..else` statement) depending on some environment state. For example:

```js
if (typeof Array.isArray != "undefined") {
    function isArray(a) {
        return Array.isArray(a);
    }
}
else {
    function isArray(a) {
        return Object.prototype.toString.call(a)
            == "[object Array]";
    }
}
```

It's tempting to structure code this way for performance reasons, since the `typeof Array.isArray` check is only performed once, as opposed to defining just one `isArray(..)` and putting the `if` statement inside it—the check would then run unnecessarily on every call.

| WARNING: |
| :--- |
| In addition to the risks of FiB deviations, another problem with conditional-definition of functions is it's harder to debug such a program. If you end up with a bug in the `isArray(..)` function, you first have to figure out *which* `isArray(..)` implementation is actually running! Sometimes, the bug is that the wrong one was applied because the conditional check was incorrect! If you define multiple versions of a function, that program is always harder to reason about and maintain. |

In addition to the previous snippets, several other FiB corner cases are lurking; such behaviors in various browsers and non-browser JS environments (JS engines that aren't browser based) will likely vary. For example:

```js
if (true) {
    function ask() {
        console.log("Am I called?");
    }
}

if (true) {
    function ask() {
        console.log("Or what about me?");
    }
}

for (let i = 0; i < 5; i++) {
    function ask() {
        console.log("Or is it one of these?");
    }
}

ask();

function ask() {
    console.log("Wait, maybe, it's this one?");
}
```

Recall that function hoisting as described in "When Can I Use a Variable?" (in Chapter 5) might suggest that the final `ask()` in this snippet, with "Wait, maybe..." as its message, would hoist above the call to `ask()`. Since it's the last function declaration of that name, it should "win," right? Unfortunately, no.

It's not my intention to document all these weird corner cases, nor to try to explain why each of them behaves a certain way. That information is, in my opinion, arcane legacy trivia.

My real concern with FiB is, what advice can I give to ensure your code behaves predictably in all circumstances?

As far as I'm concerned, the only practical answer to avoiding the vagaries of FiB is to simply avoid FiB entirely. In other words, never place a `function` declaration directly inside any block. Always place `function` declarations anywhere in the top-level scope of a function (or in the global scope).

So for the earlier `if..else` example, my suggestion is to avoid conditionally defining functions if at all possible. Yes, it may be slightly less performant, but this is the better overall approach:

```js
function isArray(a) {
    if (typeof Array.isArray != "undefined") {
        return Array.isArray(a);
    }
    else {
        return Object.prototype.toString.call(a)
            == "[object Array]";
    }
}
```

If that performance hit becomes a critical path issue for your application, I suggest you consider this approach:

```js
var isArray = function isArray(a) {
    return Array.isArray(a);
};

// override the definition, if you must
if (typeof Array.isArray == "undefined") {
    isArray = function isArray(a) {
        return Object.prototype.toString.call(a)
            == "[object Array]";
    };
}
```

It's important to notice that here I'm placing a `function` **expression**, not a declaration, inside the `if` statement. That's perfectly fine and valid, for `function` expressions to appear inside blocks. Our discussion about FiB is about avoiding `function` **declarations** in blocks.

Even if you test your program and it works correctly, the small benefit you may derive from using FiB style in your code is far outweighed by the potential risks in the future for confusion by other developers, or variances in how your code runs in other JS environments.

FiB is not worth it, and should be avoided.

## Blocked Over

The point of lexical scoping rules in a programming language is so we can appropriately organize our program's variables, both for operational as well as semantic code communication purposes.

And one of the most important organizational techniques is to ensure that no variable is over-exposed to unnecessary scopes (POLE). Hopefully you now appreciate block scoping much more deeply than before.

Hopefully by now you feel like you're standing on much more solid ground with understanding lexical scope. From that base, the next chapter jumps into the weighty topic of closure.

[^POLP]: *Principle of Least Privilege*, https://en.wikipedia.org/wiki/Principle_of_least_privilege, 3 March 2020.
