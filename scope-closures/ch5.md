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

1장에서 설명했듯이 컴파일 시간 동안 모든 식별자가 해당 스코프에 등록된다. 또한 모든 식별자는 **자신이 속한 스코프가 시작 될때마다** 해당 스코프의 시작 부분에 *만들어진다*.

변수 선언이 스코프 내에서 더 아래쪽으로 나타나더라도 변수를 해당 스코프의 처음부터 볼 수 있는 경우를 주로 **호이스팅<sub>hoisting</sub>**라고 명칭한다.

하지만 호이스팅만으로는 질문에 충분한 답이 되지 않는다. 우리는 스코프 처음부터 `greeting`이라는 식별자를 볼 수 있는데, 왜 우리는 `greeting` 함수가 선언되기 전에 `greeting`' 함수 **호출**를할 수 있을까?

다시 말해, 스코프가 실행되기 시작하는 순간부터, 어떻게 변수 `greeting`는 어떤 값(함수 참조)을 가지게 될까? 정답은 *함수 호이스팅*이라는 공식`function` 선언의 특성 때문이다. `function` 선언의 이름 식별자가 해당 스코프의 맨 위에 등록되면 해당 함수 참조값으로 자동 초기화된다. 그렇기 때문에 함수를 전체 스코프에서 호출할 수 있다!

한 가지 중요한 세부 사항은 *함수 호이스팅*과 `var` 가 취하는 *변수 호이스팅* 모두 이름 식별자를 블록 스코프가 아닌 가장 가까운 **함수 스코프**(이것이 없는 경우는 전역 스코프)에 등록한다는 것이다.

| 비고: |
| :--- |
| `let`과 `const`가 포함된 선언문은 여전히 호이스팅(이 장 뒷부분의 TDZ 설명 참조)을 수행한다. 그러나 이 두 가지 선언 양식은 단순히 `var`와 `function` 선언처럼 이를 감싸는 함수가 아닌 감싸는 블록에 등록한다. 자세한 내용은 6장의 "블록으로 스코프 지정"을 참조해라. |

### 호이스팅: 선언 vs. 표현식

*함수 호이스팅*은 공식 `function` 선언(특히 함수 선언이 있는 블록 (6장 "FiB" 참고) 외부에 나타나는)에만 적용되며, `function` 표현식 할당에는 적용되지 않는다. 다음 코드를 살펴보자:

```js
greeting();
// TypeError

var greeting = function greeting() {
    console.log("Hello!");
};
```

1행(`greeting();`)은 오류를 발생시킨다. 그러나 발생한 오류의 *종류*는 매우 중요하다. `TypeError`는 허용되지 않는 값을 사용하여 작업을 시도하고 있음을 의미한다. JS 환경에 따라 "'undefined'는 함수가 아니다" 또는 더욱 자세히 "'greeting'"는 함수가 아니다."와 같은 오류 메시지가 표시된다.

위의 오류는  `ReferenceError`가 *아니다*. JS는 스코프에서 식별자로서의 `greeting`을 찾지 못했다고 말하지 않고 있다. `greeting`이 발견됐지만 그 시점에 함수 참조가 없다는 얘기다. 함수만 호출할 수 있으므로 일부 비함수 값을 호출하려고 하면 오류가 발생한다.

함수 참조가 아니라면, `greeting`이 가진 값은 무엇일까?

호이스팅 외에도, `var`로 선언된 변수도 스코프,가장 가까운 감싸는 함수 또는 전역의 시작에서 자동으로 `undefined`으로 초기화된다. 초기화되면 전체 스코프에서 사용할 수 있다(할당, 검색 등).

즉 첫 번째 줄에 `greeting`이 존재하지만, 이것은 기본 '`undefined` 값만 가지고 있다. 4행에서야 `greeting`'이 기능 참조를 할당받는다.

여기에서 구별에 주의하라. `function` 선언은 호이스팅되고 **해당 함수 값으로 초기화**된다(*함수 호이스팅*이라고 함). 'var' 변수도 호이스팅된 다음 `undefined`으로 자동 초기화된다. 런타임 실행 중에 할당이 처리되기 전에는 해당 변수에 대한 후속 `function` 식 할당이 수행되지 않는다.

두 경우 모두 식별자의 이름이 호이스탕된다. 그러나 식별자가 공식 `function` 선언에서 생성되지 않는 한 함수 참조 연결은 초기화 시(스코프 시작) 처리되지 않는다.

### 변수 호이스팅

*변수 호이스팅*의 다른 예를 살펴보자.

```js
greeting = "Hello!";
console.log(greeting);
// Hello!

var greeting = "Howdy!";
```

5행까지는 `greeting`이 선언되지 않지만, 1행에서 할당이 가능하다. 그 이유는 무엇일까?

설명에는 두 가지 필요한 부분이 있다.

* 식별자가 호이스트 되었다.
* ** 그리고 ** 식별자는 스코프의 최상단에서 `undefined`으로 초기화되었다.

| 비고: |
| :--- |
| 이러한 종류의 *변수 호이스팅*을 사용하는 것은 아마도 부자연스럽게 느껴질 것이고, 여러분들은 당연히 당신의 프로그램에서 그것에 의존하는 것을 피하고 싶어할 것이다. 그러나 모든 호이스팅(*함수 호이스팅* 포함)을 피해야 할까? 우리는 호이스팅에 대한 이러한 다양한 관점에 대해 부록 A에서 자세히 살펴볼 것이다. |

## 호이스팅: 또다른 비유

제2장은 비유들로 가득 차 있었지만(스코프를 설명하기 위해서), 여기서 우리는 또 다른 비유에 마주하게 되었다: 호이스팅 그 자체말이다. 호이스팅은 JS 엔진이 수행하는 구체적인 실행 단계이기보다는 프로그램 **실행 전**을 설정하기 위해 JS가 수행하는 다양한 작업의 시각화라고 생각하면 더 유용할 것이다.

호이스팅의 의미에 대한 일반적인 주장: 식별자를 *리프팅*(무거운 중량을 위로 들어올리는 것) 하여 스코프의 맨 위까지 올린다. 종종 JS 엔진이 실행 전에 해당 프로그램을 실제로 *재작성*하므로 다음과 더 비슷해 보인다는 주장이 제기되기도 한다.

```js
var greeting;           // hoisted declaration
greeting = "Hello!";    // the original line 1
console.log(greeting);  // Hello!
greeting = "Howdy!";    // `var` is gone!
```

호이스팅 비유법은 실행 전에 모든 선언이 각각의 스코프의 맨 위로 이동되도록 원래 프로그램을 사전 처리하고 약간 다시 정렬할 것을 제안한다. 게다가, `function` 선언 전체가 각 스코프의 최상위에 올려져 있다고 호스팅 비유는 주장한다. 다음을 살펴보자:

```js
studentName = "Suzy";
greeting();
// Hello Suzy!

function greeting() {
    console.log(`Hello ${ studentName }!`);
}
var studentName;
```

호이스팅 비유법의 "규칙"은 함수 선언을 먼저 올린 다음 변수를 모든 함수 뒤에 바로 올리는 것이다. 따라서, 호이스팅 사례는 다음과 같이 보이는 JS 엔진에 의해 프로그램이 *재편성*되었음을 시사한다.

```js
function greeting() {
    console.log(`Hello ${ studentName }!`);
}
var studentName;

studentName = "Suzy";
greeting();
// Hello Suzy!
```

이 호이스팅 비유법은 편리하다. 그 이점은 스코프 깊숙이 파묻혀 있는 모든 선언을 찾아 어떻게든 상단으로 이동(호이스트)하는 데 필요한 전 처리를 마법처럼 미리 보게 해준다. 프로그램을 **단일 패스**, 하향식으로 실행하는 것처럼 생각하면 된다.

1장의 2단계 처리 방식 보다 단일 패스가 더 간단해 보인다.

코드 순서 재조정 메커니즘으로 호이스팅하는 것이 간단하다는 점에서 매력적일 수는 있지만 정확하지는 않을 수 있다. JS 엔진은 실제로 코드를 다시 정렬하지 않는다. 마술적으로 선언을 미리 보고 찾을 수는 없다. 코드를 파싱하는 것이 선언을 정확히 찾는 유일한 방법이다.

파싱이 뭔지 아는가? 2단계 처리의 첫 번째 단계! 그 사실을 피할 수 있는 마법의 멘탈 체조는 없다.

그렇다면 만약 호이스트 비유법이 (기껏해야) 부정확하다면, 우리는 이 용어를 어떻게 해야 할까? 나는 그것이 여전히 유용하다고 생각한다. 실제로 TC39의 회원들도 정기적으로 사용하고 있다!—하지만 소스 코드를 실제로 다시 배열한 것이라고 주장할 필요는 없다고 생각한다.

| 주의: |
| :--- |
| 부정확하거나 불완전한 멘탈 모델은 종종 우연한 정답으로 이어질 수 있기 때문에 여전히 충분해 보인다. 그러나 장기적으로는 여러분의 생각이 JS 엔진의 작동 방식과 특별히 일치하지 않을 경우 결과를 정확하게 분석하고 예측하는 것이 더 어렵다. |

나는 호이스팅을 **꼭** 사용하여 해당 스코프가 시작될 때마다 해당 스코프 시작 시 변수의 자동 등록을 위한 런타임 명령을 생성하는 **컴파일 시간 작업**을(를) 나타내야 한다고 주장한다.

이는 런타임 동작으로서의 호이스팅에서 컴파일 시간 작업 사이의 적절한 위치로 미묘하지만 중요한 전환이다.

## 재선언?

변수가 동일한 스코프에서 두 번 이상 선언되면 어떻게 된다고 생각하는가? 다음을 살펴보자:

```js
var studentName = "Frank";
console.log(studentName);
// Frank

var studentName;
console.log(studentName);   // ???
```

두 번째 메시지는 무엇을 출력할까? 대부분은 두 번째 `var studentName`이 변수를 다시 선언(따라서 '재설정')했다고 생각하기 때문에 `undefined`이 출력될 것으로 예상할 것이다.

하지만 변수가 같은 스코프에서 "재선언"될 수 있을까? 아니다.

호스팅 비유법 관점에서 이 프로그램을 고려할 경우, 코드는 실행 목적으로 다음과 같이 다시 배열된다.

```js
var studentName;
var studentName;    // clearly a pointless no-op!

studentName = "Frank";
console.log(studentName);
// Frank

console.log(studentName);
// Frank
```

실제로 호이스팅은 스코프의 시작 부분에 변수를 등록하는 것이기 때문에 원래 프로그램이 두 번째 'var studentname' 문구를 가지고 있던 스코프 중간에 수행할 수 있는 작업이 없다. 그것은 어떠한 동작도 하지않으며 무의미한 진술일 뿐이다.

| TIP: |
| :--- |
| 2장의 대화 서술 방식으로 설명하자면, *컴파일러*는 두 번째 `var` 선언문을 찾아 *스코프 매니저*에게 `studentName` 식별자를 이미 보았는지 물어본다; 스코프 매니저는 이를 보았기 때문에 더이상 할 것은 없다. |

대부분이 생각하는 것처럼 `var studentName;`이  `var studentName = undefined`을 의미하는 것은 아니라는 점도 중요하다. 프로그램의 이러한 변형을 고려하여 이 둘이 다르다는 것을 증명해 보자.

```js
var studentName = "Frank";
console.log(studentName);   // Frank

var studentName;
console.log(studentName);   // Frank <--- still!

// let's add the initialization explicitly
var studentName = undefined;
console.log(studentName);   // undefined <--- see!?
```

명시적으로 `= undefined` 초기화한 것과 이것이 암묵적으로 생략되었을 때는 어떻게 다른 결과를 낳을까? 다음 섹션에서는 선언에서의 변수 초기화 주제를 다시 살펴보자.

스코프에서 동일한 식별자 이름을 반복적으로 'var' 선언하는 것은 사실상 아무것도 하지 않는 작업이다. 다음은 같은 이름의 함수가 반복될 때에 대한 설명이다.

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

첫 번째 `greeting` 선언은 식별자를 스코프에 등록하며, `var`이기 때문에 자동 초기화는 `undefined`을 할당한다. `function` 선언은 식별자를 다시 등록할 필요가 없지만 *함수 호이스팅* 때문에 함수 참조를 사용하도록 자동 초기화를 재정한다. 이미 `greeting`이 식별자이고 자동초기화에는 *함수 호이스팅*이 우선이었기 때문에 두 번째 `greeting`만으로는 아무것도 할 수 없다.

실제로 '`"Hello!"`을 `greeting`에 할당하면 첫 함수인 `greeting()`에서 문자열로 값이 바뀌지만, `var` 자체는 아무런 효과가 없다.

스코프 내에서 `let`이나 `const`를 이용해 선언을 반복하는 것은 어떨까.

```js
let studentName = "Frank";

console.log(studentName);

let studentName = "Suzy";
```

이 프로그램은 실행되지 않고 즉시 `SyntaxError`를 발생시킨다. JS 환경에 따라 "studentName이(가) 이미 선언되었습니다."와 같은 오류 메시지가 표시된다. 즉, 시도된 "재선언"이 명시적으로 허용되지 않는 경우이다!

단순히 `let`을 포함한 이 두 선언이 이런 오류를 낳는 것은 아니다. 두 선언 중 하나가 `let`을 사용할 경우 다른 선언은 `let` 또는 `var`가 될 수 있으며, 다음 두 변형에서 볼 수 있듯이 오류가 계속 발생한다.

```js
var studentName = "Frank";

let studentName = "Suzy";
```

그리고:

```js
let studentName = "Frank";

var studentName = "Suzy";
```

두 경우 모두 *두 번째* 선언에 `SyntaxError`가 발생한다. 즉, 변수를 '재선언'하는 유일한 방법은 선언의 전부(2개 이상)에 'var'를 사용하는 것이다.

그런데 왜 그걸 허용하지 않을까? 오류의 원인은 기술적인 것이 아니라 `var` "재선언"이 항상 허용되어 왔기 때문이다. `let`은 허용되지 않는다.

이것은 정말로 "사회 공학"에 더 가까운 문제이다. TC39 본문 등 일부에서는 변수 "재선언"이 프로그램 버그로 이어질 수 있는 악습으로 보고 있다. 그래서 ES6는 `let`을 도입할 때 오류로 "재선언"을 막기로 했다.

| 비고: |
| :--- |
| 이것은 물론 형식적인 의견이지 기술적인 주장은 아니다. 많은 개발자들이 이 의견에 동의하고 있으며, 그 때문에 TC39이 그것을 오류로 포함한 것이다(`const`와 같이 동작하는 `let`도 마찬가지로). 그러나 일부 경우는 이 전에 사용된`var` 일관성을 유지하는 것이 더 중요했고, 그러한 경우는 린터 같은 도구를 채택하는 것이 최선일 수 있다. 부록 A에서는 `var`(그리고 "재선언"과 같은 관련 동작)이 여전히 현대 JS에서 유용할 수 있는지 여부를 살펴보겠다. |

*컴파일러*가 선언에 대해 *스코프 매니저*에게 질문할 때, 해당 식별자가 이미 선언되었는지 여부 및 둘 중 하나가 'let'으로 지정된 경우 오류가 발생합니다. 개발자에게 보내는 신호는 "허술한 재선언에 의존하지 말라!"이다.

### 상수

`const` 키워드는 `let`보다 더 제약이 많다. `let`처럼 같은 스코프의 동일 식별자로 `const`를 반복할 수 없다. 그러나 형식적인 이유로 "재선언"을 허용하지 않는 `let`와는 달리, 그러한 종류의 "재선언"이 허용되지 않는 중요한 기술적 이유가 실제로 있다.

`const` 키워드는 변수를 초기화해야 하므로 선언에서 할당을 생략하면 `SyntaxError`가 발생합니다.

```js
const empty;   // SyntaxError
```

`const` 선언은 다시 할당할 수 없는 변수를 만든다.

```js
const studentName = "Frank";
console.log(studentName);
// Frank

studentName = "Suzy";   // TypeError
```

`studentName` 변수는 `const`로 선언되어 있으므로 재할당할 수 없습니다.

| 주의: |
| :--- |
| `studentName`을 재할당할 때 발생하는 오류는 `TypeError`이지 `SyntaxError`가 아니다. 여기서의 미묘한 차이는 사실 꽤 중요하지만 불행하게도 놓치기 너무 쉽다. SyntaxError는 실행조차 시작하지 못하게 하는 프로그램의 결함을 나타낸다. TypeError는 프로그램 실행 중에 발생하는 장애를 나타낸다. 앞의 스니펫에서는 `studentName` 재할당을 진행하기 전에 `"Frank"`가 출력되어 오류가 발생한다. |

그러므로 만일 `const` 선언을 재할당할 수 없고 `const` 선언이 항상 할당을 요구한다면, `const`가 재선언을 허용하지없는 분명한 기술적 이유가 있다: 어떤 `const` 재선언은 반드시 `const` 재할당이다

```js
const studentName = "Frank";

// obviously this must be an error
const studentName = "Suzy";
```

TC39는 (이러한 기술적 이유로) `const` '재선언'을 불허해야 하기 때문에 일관성을 위해 `let` '재선언'도 불허해야 한다고 느꼈다. 이게 최선의 선택이었는지는 논란의 여지가 있지만, 적어도 우리는 그 결정의 이면에 있는 이유를 가지고 있다.

### 루프

따라서 이전 논의에서 우리는 JS는 동일한 스코프 내에서 변수를 "재선언"하는 것을 원하지 않는 것을 알 수 있다. 루프에서 선언문을 반복적으로 실행하는 것이 무엇을 의미하는지 고려하기 전까지는 이 말은 간단한 훈계처럼 보일 것이다. 다음을 살펴보자:

```js
var keepGoing = true;
while (keepGoing) {
    let value = Math.random();
    if (value > 0.5) {
        keepGoing = false;
    }
}
```

`value`가 이 프로그램에서 반복적으로 '재선언'되고 있는 것일까? 오류가 발생할까? 아니다.

스코프의 모든 규칙(`let` 생성 변수의 "재선언" 포함)은 *스코프 인스턴스당* 적용된다. 즉, 실행 중에 스코프를 시작될 때마다 모든 항목이 재설정된다.

각 루프 반복은 고유한 새 스코프 인스턴스이며 각 스코프 인스턴스 내에서 '값'은 한 번만 선언된다. 따라서 "재선언" 시도도 없고, 따라서 오류도 없다. 다른 루프 형식을 고려하기 전에 이전 스니펫의 '값' 선언이 'var'로 변경되면 어떻게 될까?

```js
var keepGoing = true;
while (keepGoing) {
    var value = Math.random();
    if (value > 0.5) {
        keepGoing = false;
    }
}
```

특히 'var'가 허용한다는 것을 알고 있기 때문에 여기서 `value`가 '재선언'되는 것일까? 아니다. 'var'는 블록 스코프 지정 선언(6장 참조)으로 취급되지 않기 때문에 전역 스코프에 등록된다. 따라서 `value` 변수는 `keepGoing`(이 경우 전역 스코프)와 같은 스코프 안에 딱 하나 있을 뿐이다. 여기도 "재선언"은 없다!

이 모든 것을 바로잡을 수 있는 한 가지 방법은 실행이 시작될 때쯤 코드로부터 `var`, `let`, `const` 키워드가 효과적으로 *제거된다는 것*을 기억하는 것이다. 그것들은 전적으로 컴파일러에 의해 처리된다.

선언자 키워드를 개념적으로 지운 다음 코드를 처리하려고 할 경우, 선언이 발생할 수 있는지 여부와 시기를 결정하는 데 도움이 된다.

`for` 루프와 같은 다른 루프 형식의 "재선언"은 어떨까?

```js
for (let i = 0; i < 3; i++) {
    let value = i * 10;
    console.log(`${ i }: ${ value }`);
}
// 0: 0
// 1: 10
// 2: 20
```

스코프 인스턴스당 선언된 '값'이 하나뿐임을 분명히 해야 한다. 하지만 'i'는 어떤가? "재선언"되는 건가?

이에 답하려면 'i'가 어느 스코프에 속하는지 생각해 보아야 한다. 외부(이 경우 전역) 스코프에 있을 것으로 보이지만 그렇지 않다. `value`처럼 `for` 루프 본체 스코프 안에 있다. 이 루프를 조금더 풀어서 작성하여 살펴보자.

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

이제 명확하다. `i` 변수와 `value` 변수는 모두 **스코프 인스턴스당** 정확하게 한 번 선언된다. 여기엔 "재선언"이 없다.

다른 `for` 루프 형태는 어떤가?

```js
for (let index in students) {
    // this is fine
}

for (let student of students) {
    // so is this
}
```

`for..in`과`for..of` 도 마찬가지다: 선언된 변수는 루프 본체 내부에서 처리되므로, 반복마다(일명 스코프 인스턴스별로) 처리됩니다. "재선언"은 없다.

지금 내 목소리가 고장난 레코드 같다고 생각할 것이다. 그러나 `const`가 이러한 루프 구조물에 어떤 영향을 미치는지 알아보자. 다음을 살펴보자:

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

앞서 살펴본 이 프로그램의 `let` 변수처럼 `const`가 루프 반복마다 정확히 한 번씩 실행되기 때문에 '재선언' 문제로부터 안전하다. 그러나 우리가 `for` 루프 이야기를 할 때 상황은 더 복잡해진다.

`for..in` 과 `for..of`는 `const`와 함께 사용되어도 문제 없다:

```js
for (const index in students) {
    // this is fine
}

for (const student of students) {
    // this is also fine
}
```

그러나 일반적인 `for` 루프는 아니다:

```js
for (const i = 0; i < 3; i++) {
    // oops, this is going to fail with
    // a Type Error after the first iteration
}
```

무엇이 문제인가? 우리는 이 구조에서 그냥 `let`을 사용할 수도 있고, 각각의 루프 반복 스코프에 대해 새로운 'i'를 만들 수 있다고 주장했기 때문에 '재선언'도 아닌 것 같다.

전에 했던 것처럼 이 루프를 개념적으로 "확자"해서 살펴보자:

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

당신은 문제를 발견했나? 우리의 `i`는 정말 루프 안에서 한 번 만들어질 뿐이다. 그게 문제가 아니다. 문제는 '$i++' 표현으로 매번 늘려야 하는 개념적인 '$i'다. 상수에 허용되지 않는 **재할당**("재선언"이 아님)이다.

이 "확장" 양식은 문제의 원인을 직관하는 데 도움이 되는 개념적 모델일 뿐이다. JS가 'const $i = 0'을 'let $ii = 0'으로 효과적으로 만들 수 있었는지 궁금할 것이다. 그렇다면 'const'가 우리의 고전적인 'for' 루프와 함께 동작할 수 있을까? 그럴 수도 있지만, 그렇다면 잠재적으로 `for` 루프에 대해 놀랄만한 예외를 도입했을 수도 있다.

예를 들어, `for` 루프 헤더에서 `i++`가 'constant' 할당의 엄격함을 회피할 수 있도록 허용한 것은 다소 자의적인(그리고 혼동될 가능성이 있는) 뉘앙스에서 예외일 수 있지만, 루프 반복 내에서 `i`의 다른 재할당은 때로는 유용하지 않을 수 있다.

분명한 대답은 `const`는 필수 재할당 때문에 고전적인 `for` 루프 형식과 함께 사용할 수 없다는 것이다.

흥미롭게도 재할당을 하지 않으면 유효하다.

```js
var keepGoing = true;

for (const i = 0; keepGoing; /* nothing here */ ) {
    keepGoing = (Math.random() > 0.5);
    // ..
}
```

이 코드는 작동은 하지만 무의미하다. 해당 위치의 변수의 목적은 **반복문을 계속 진행시키기 위해 사용되는 것이기** 때문에 `const`를 사용하여 해당 위치에 `i`를 선언할 이유가 없다. 그냥 `while` 루프와 같은 다른 루프 형식을 사용하거나 `let`을 사용해라!

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
