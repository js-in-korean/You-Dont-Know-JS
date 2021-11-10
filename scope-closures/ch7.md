# You Don't Know JS Yet: Scope & Closures - 2nd Edition
# 챕터 7: 클로저 사용하기

지금까지, 렉시컬 스코프의 내부와 외부가 프로그램의 변수 구성과 사용에 어떤 영향을 미치는지에 초점을 맞추었다.

우리의 관심은 역사적으로 다소 어려운 주제인 클로저로 추상적으로 다시 확대된다. 걱정하지 마라! 이를 이해하기 위해 고급 컴퓨터 공학 학위가 필요하지는 않다. 이 책의 큰 목표는 단지 스코프를 이해하는 것이 아니라, 프로그램의 구조에 스코프를 더 효과적으로 사용하는 것이다; 클로저는 그 노력의 핵심이다.

챕터6의 결론을 상기해라: "*최소 노출* 원리<sub>Least Exposure</sub>"(POLE)는 변수의 스코프 노출을 제한하기 위해서 블록(혹은 함수)스코프 사용을 장려한다.이는 코드를 이해하고 유지보수하기 쉽게 도와주고 스코프 함정(i.e. 네이밍 충돌 등)을 피하는데 도움이 된다.

클로저는 이런 접근 방식을 기반으로 한다: 시간이 지나도 사용해야하는 변수의 경우, 더 큰 외부 스코프에 변수들을 위치시키는 대신, 함수 내부에서의 접근은 보존해서 (더 좁은 스코프로) 더 넓은 사용이 가능하게끔 캡슐화 할 수 있다. 함수들은 클로저를 통해 참조된 변수의 스코프를 *기억한다.*

이미 이전 챕터(챕터6의 `factorial(..)`)에서 이러한 종류의 클로저 예시를 본 적이 있고, 아마 프로그램에서 이미 사용해봤을 것이다. 스코프 외부에서 변수를 접근하는 콜백을 작성해본 경험이 있다면... 이게 뭘까!? 바로 클로저다.

클로저는 프로그래밍에서 발명된 가장 중요한 언어 특성 중 하나다. 클로저는 FP(함수형 프로그래밍<sub>Functional Programming</sub>), 모듈 및 클래스 지향 설계의 일부분을 포함한 주요 프로그래밍 패러다임의 기반이 된다. 클로저에 익숙해지는 것은 JS를 마스터하고 코드에서 많은 중요한 디자인 패턴을 효과적으로 활용하는 데 필요하다.

클로저의 모든 측면을 다루려면 이 장 전체에 걸쳐 토론과 코드의 험난한 여정이 필요하다. 다음 단계로 넘어가기 전에 시간을 갖고 각 부분들에 익숙해졌는지 확인해라.

## 클로저 살펴보기

클로저는 원래 수학적인 개념으로 람다 대수에서 유래했다. 그러나 수학적인 공식을 나열하거나 이를 정의하기 위해 많은 표기법이나 전문용어는 사용하지는 않을 것이다.

대신, 실용적인 관점으로 살펴볼 것이다. 클로저가 JS에 없다고 가정한 상황과 비교해서 프로그램의 다른 동작에서 관찰할 수 있는 관점에서 클로저를 정의하는 것으로 시작할 것이다. 그러나 이 장의 뒷부분에서 *대안적 관점*에서 살펴보기 위해 클로저를 뒤집을 것이다.

클로저는 함수의 동작이며 함수일 뿐이다. 함수를 다루지 않는다면 클로저가 적용되지 않는다. 객체에는 클로저가 있을 수 없으며 클래스에도 클로저가 없다(해당 함수/메서드가 있을 수 있음). 오직 함수에만 클로저가 있다.

클로저를 관찰하려면 함수가 반드시 실행되어야 하고, 원래 정의된 스코프 체인과 다른 분기의 스코프에서 실행되어야 한다. 정의된 동일한 스코프에서 실행되는 함수는 클로저가 가능한지 여부에 관계없이 관찰 가능한 다른 동작이 없다. 관찰적 관점과 정의에 따르면, 그것은 클로저가 아닙니다.

관련있는 스코프 버블색들이 주석으로 달린 코드를 보자(챕터2 참고):

```js
// 외부/글로벌 스코프: 빨강(1)

function lookupStudent(studentID) {
    // 함수 스코프: 파랑(2)

    var students = [
        { id: 14, name: "Kyle" },
        { id: 73, name: "Suzy" },
        { id: 112, name: "Frank" },
        { id: 6, name: "Sarah" }
    ];

    return function greetStudent(greeting){
        // 함수 스코프: 초록(3)

        var student = students.find(
            student => student.id == studentID
        );

        return `${ greeting }, ${ student.name }!`;
    };
}

var chosenStudents = [
    lookupStudent(6),
    lookupStudent(112)
];

// 함수명 접근:
chosenStudents[0].name;
// greetStudent

chosenStudents[0]("Hello");
// Hello, Sarah!

chosenStudents[1]("Howdy");
// Howdy, Frank!
```

이 코드에서 가장 먼저 주목해야 할 것은 `lookupStudent(..)` 외부 함수가 `greetStudent(..)`라는 내부 함수를 생성하고 반환한다는 것이다. `lookupStudent(..)`는 두 번 호출되어 내부 `greetStudent(..)` 함수의 두 개의 개별 인스턴스를 생성하며 둘 다 `chosenStudents` 배열에 저장된다.

`chosenStudents[0]`에 저장된 반환된 함수의 `.name` 속성을 확인하여 이 경우인지 확인하고, 실제로 이는 안에 있는 `greetStudent(..)`의 인스턴스이다.

`lookupStudent(..)`에 대한 각 호출이 완료된 후 모든 내부 변수가 삭제되고 GC 되는 (garbage collected) 것 처럼 보인다. 내부 함수가 반환되고 보존되는 유일한 것으로 보인다. 그러나 여기에서 우리가 관찰하기 시작할 수 있는 다른 동작이 나온다.

`greetStudent(..)`는 `greeting`이라는 매개변수로 하나의 인수를 받지만 이를 포함하는 `lookupStudent(..)` 스코프에서 오는 식별자인 `students`와 `studentID`를 모두 참조한다. 내부 함수에서 외부 스코프의 변수에 대한 각 참조를 *클로저*라고 한다. 학문적 용어로 쓰면, `greetStudent(..)`의 각 인스턴스는 외부 변수 `students`와 `studentID`를 *클로즈 오버*(closes over)한다고 한다.

그렇다면 이러한 클로저들에서 구체적이고 관찰 가능한 의미는 무엇일까?

클로저를 사용하면 `greetStudent(..)`의 외부 스코프가 완료된 후에도 외부 변수에 계속 접근할 수 있다(`lookupStudent(..)`에 대한 각 호출이 완료될 때). `students` 와 `studentID`의 인스턴스가 GC되지 않고 메모리에 남아있다. 나중에 `greetStudent(..)` 함수의 인스턴스 중 하나가 호출될 때 해당 변수는 현재 값을 유지하면서 여전히 존재한다.

JS 함수에 클로저가 없다면 각 `lookupStudent(..)` 호출이 완료되면 해당 스코프가 즉시 해제되고 `students`와 `studentID` 변수가 GC된다. 나중에 `greetStudent(..)` 함수 중 하나를 호출하면 어떻게 될까?

`greetStudent(..)`가 파랑(2) 구슬이라고 생각한 것에 액세스하려고 시도하지만 그 구슬이 실제로 존재하지 않는다면(더 이상), 합리적인 가정은 `ReferenceError`가 발생하는 것이다, 맞지?

하지만 오류는 발생하지 않는다. `chosenStudents[0]("Hello")` 이 동작하고 "Hello, Sarah!" 메시지를 반환한다는 사실은 여전히 `students`와 `studentID` 변수에 액세스할 수 있음을 의미한다. 이것은 클로저의 직접적인 관찰이다!

### 클로저에 집중하기

사실 이전 논의에서 약간 세세한 부분들은 넘겼는데 아마 많은 분들이 놓쳤을 것이다!

`=>` 화살표 함수 문법의 간결성때문에 이 또한 스코프를 만든다는 것을 잊기 쉽다(챕터 3의 "화살표 함수" 부분에서 언급). `student => student.id == studentID` 화살표 함수는 `greetStudent(..)` 함수 스코프 안에 또 다른 스코프 버블을 생성한다.

챕터 2의 구슬과 양동이를 색칠하면서 비유했던 것에 기반하여 이 코드에 대한 색 다이어그램을 만들어본다면, 중첩 레벨의 가장 안쪽에 4번째 스코프가 있게 되고, 4번째 색이 필요하게 된다; 이 스코프를 위해 오렌지색(4)을 추가할 것이다.

```js
var student = students.find(
    student =>
        // 함수 스코프: 오렌지(4)
        student.id == studentID
);
```

파랑(2) `studentID`의 참조는 초록(3) 스코프 `greetStudent(..)`가 아닌 오렌지(4) 스코프 내부에 있다.; 또한, 화살표 함수의 `student` 인자는 오렌지(4)고, 초록(3) `student`를 섀도잉한다.

여기서 중요한 점은 `greetStudent(..)`가 이 클로저를 들고있다기보다는 배열의 `find(..)` 메소드 콜백으로 넘겨진 화살표 함수가 `studentID`를 클로즈 오버 해야한다는 것이다. 예상대로 잘 동작하니 큰 문제는 아니다. 그저 작은 화살표 함수도 클로저 모임에 참여할 수 있다는 사실을 모른체하지만 않으면 된다.

### 클로저 추가하기

클로저에 대해 자주 인용되는 표준 예 중 하나를 살펴보자.

```js
function adder(num1) {
    return function addTo(num2){
        return num1 + num2;
    };
}

var add10To = adder(10);
var add42To = adder(42);

add10To(15);    // 25
add42To(9);     // 51
```

각 인스턴스의 내부 `addTo(..)` 함수는 각각의 `num1` 변수를 클로즈 오버 하고있다. 그래서 `num1`은 `adder(..)`가 종료되어도 사라지지 않는다. 이후 `add10To(15)` 호출같이 내부 `addTo(..)` 인스턴스들 중 하나를 실행시킬 때 클로즈 오버된 `num1` 변수는 아직 존재하고 아직 원래 값 `10`을 가지고 있다. 따라서 `10 + 15`를 수행하고 `25` 답을 반환할 수 있다.

중요한 세부 사항을 이전 단락에서 쉽게 지나쳤을 수 있으므로, 이를 보강해보자: 클로저는 함수의 단일 렉시컬 선언보다는 함수의 인스턴스와 관련이 있다. 앞의 내용에 따르면, `adder(..)` 내부에 오직 하나의 내부 함수 `addTo(..)`가 정의되어있는데, 이 때문에 단일 클로저가 적용되는 것 처럼 보일 것이다.

그러나 사실, 바깥 `adder(..)` 함수가 실행될 때마다, *새로운* 내부 `addTo(..)` 함수 인스턴스가 각각 새 인스턴스와 새 클로저로 생성된다. 그래서 각 내부 함수 인스턴스(우리 프로그램에서 `add10To(..)`와 `add42To(..)`로 라벨링된)는 `adder(..)` 실행으로부터 각각의 스코프 환경 인스턴스를 클로즈 오버한다.

비록 클로저가 컴파일 시 관리되는 렉시컬 스코프를 기반으로 하지만, 클로저는 함수 인스턴스들의 런타임 특성으로 관찰된다.

### 스냅샷이 아닌 실시간 연결

이전 섹션의 두 예시에서, **변수에서 값을 읽고** 이 값은 클로저에 저장되었다. 이는 클로저가 마치 어떤 순간의 값 스냅샷인 것 처럼 느껴질 수 있다. 사실 이는 흔한 오해다.

클로저는 사실 전체 변수 그 자체에 대한 접근을 보존하는 실시간 연결이다. 우리는 값을 읽는 것에만 국한되지 않고 클로즈 오버된 변수는 업데이트(재할당)될 수도 있다! 함수 내부의 변수를 클로즈 오버 함으로써, 이 함수의 주소가 프로그램에 존재하거나 우리가 원하는 어디서든 함수를 실행해 변수를 계속 사용(읽기와 쓰기)할 수 있다. 이게 클로저가 강력한 기술로써 프로그래밍의 많은 영역에 걸쳐서 사용되는 이유이다!

그림 4에서 함수 인스턴스들과 스코프 링크를 묘사한다.

<figure>
    <img src="images/fig4.png" width="400" alt="Function instances linked to scopes via closure" align="center">
    <figcaption><em>그림 4: 클로저 시각화</em></figcaption>
    <br><br>
</figure>

그림 4에서 보여지듯, 각 `adder(..)` 호출들은 `num1` 변수를 포함하는 새로운 파랑(2) 스코프를 생성한다. 새로운 파랑 스코프는 초록(3) 스코프인 `addTo(..)` 함수의 새로운 인스턴스이다. 함수 인스턴스들(`addTo10(..)`과 `addTo42(..)`)은 빨강(1) 스코프로부터 실행되고 존재하고 있는 부분을 명심해라.

이제 클로즈 오버된 변수가 업데이트되는 예시를 살펴보자:

```js
function makeCounter() {
    var count = 0;

    return function getCurrent() {
        count = count + 1;
        return count;
    };
}

var hits = makeCounter();

// 나중에

hits();     // 1

// 나중에

hits();     // 2
hits();     // 3
```

`count` 변수는 내부 `getCurrent()` 함수에 의해 클로즈 오버되고, GC 대상이 되지 않고 유지된다. `hits()` 함수는 이 변수에 대한 접근과 업데이트를 하고, 증가된 숫자를 매번 반환한다.

클로저의 감싸는 스코프는 일반적으로 함수에서 있지만 필수사항은 아니다. 외부 스코프 내부에 내부 함수만 있으면 된다.

```js
var hits;
{   // 바깥 스코프 (함수는 아닌)
    let count = 0;
    hits = function getCurrent(){
        count = count + 1;
        return count;
    };
}
hits();     // 1
hits();     // 2
hits();     // 3
```

| 비고: |
| :--- |
| 여기서 의도적으로 `getCurrent()`를 `함수` 선언식 대신 `함수` 표현식으로 선언하였다. 이것은 클로저에 대한 것은 아니지만, FiB(Function Declarations in Block)의 위험한 단점이 있다(챕터 6) |

클로저를 변수 지향적이 아닌 값 지향적으로 착각하는 것이 매우 일반적이기 때문에 개발자는 때때로 클로저를 사용하여 특정 순간의 값을 스냅샷으로 보존하려고 시도하는 실수를 한다. 고려하다:

```js
var studentName = "Frank";

var greeting = function hello() {
    // `studentName`을 클로즈 오버,
    // "Frank"가 아닌
    console.log(
        `Hello, ${ studentName }!`
    );
}

// 이후

studentName = "Suzy";

// 이후

greeting();
// Hello, Suzy!
```

`greeting()` (`hello()`라고도 하는) 함수를 선언하면서 `studentName`은 `"Frank"`라는 값을 가지고 있는데(`"Suzy"`가 재할당되기 전에), 잘못된 가정은 클로저가 `"Frank"`를 저장한다는 것이다. 그러나 `greeting()`은 `studnetName` 변수를 클로즈 오버하지 값을 클로즈 오버하는건 아니다. 언제든지 `greeting()`이 실행되면, 현재의 변수(여기서는 `"Suzy"`)를 반영한다.

이 실수의 전형적인 예로는 반복문 안에 함수를 정의하는 것이다.

```js
var keeps = [];

for (var i = 0; i < 3; i++) {
    keeps[i] = function keepI(){
        // `i`가 클로즈 오버됨
        return i;
    };
}

keeps[0]();   // 3 -- 왜!?
keeps[1]();   // 3
keeps[2]();   // 3
```

| 비고: |
| :--- |
| 이러한 클로저 형식은 전형적으로 `setTimeout(..)`나 이벤트 핸들러같은 다른 콜백을 반복문 안에서 사용한다. 여기서는 함수 주소를 배열에 저장하는걸로 예시를 간소화해서 비동기 타이밍은 고려할 필요가 없었다. 클로저 원칙은 역시 동일하다. |

아마 `keeps[0]()` 실행 결과가 `0`을 반환하는 것을 기대했을 것이다. 함수가 반복문의 첫 반복 `i`가 0일때 생성되었기 때문이다. 하지만 이 가정은 클로저가 변수 지향적인게 아니라 값 지향적으로 생각한 것에서 생긴 생각이다.

`for` 반복문 구조에서 각 반복마다 새로운 `i` 변수를 가진다고 속일 수도 있다. 사실 이 프로그램에서는 `var`로 선언되었기 때문에 오직 하나의 `i`만 가진다.

각 저장된 함수들은 `3`을 반환하는데, 반복문 끝에서 이 하나의 `i` 변수는 `3`으로 할당되기 때문이다. `keeps` 배열의 각 세 함수들은 각각의 클로저를 가지고는 있지만, 이들 모두가 공유된 `i` 변수를 클로즈 오버하고 있다.

물론, 단일 변수는 특정 시간에 하나의 값만 가질 수 있다. 그래서 만약 여러 변수들을 저장하고 싶으면, 각각 다른 변수들이 필요하다.

반복문 구문에서 이를 어떻게 할 수 있을까? 각 반복마다 새로운 변수를 만들어보자.

```js
var keeps = [];

for (var i = 0; i < 3; i++) {
    // 새로운 `j`를 각 반복마다 생성하고,
    // 이 순간 `i`의 값을 복사해 저장한다.
    let j = i;

    // 여기서 `i`는 클로즈 오버되지 않아서,
    // 각 반복마다의 현재 값을
    // 즉시 사용해도 괜찮다
    keeps[i] = function keepEachJ(){
        // 클로즈 오버된 `j` 사용, `i`가 아님!
        return j;
    };
}
keeps[0]();   // 0
keeps[1]();   // 1
keeps[2]();   // 2
```

모든 변수들이 `j`로 네이밍 되어있음에도 불구하고, 이제 각 함수는 매 반복마다 각각의 (새로운) 변수를 클로즈 오버한다. 그리고 각 `j`는 반복문의 시점마다의 `i` 값을 복사한다; `j`는 전혀 재할당되지 않는다. 그래서 이제 세 함수들 모두 예상된 값을 반환한다: `0`, `1`, 그리고 `2`!

다시 강조한다. 이 프로그램에서 내부의 `keepEachJ()`가 `setTimeout(..)`이나 다른 이벤트 핸들러 구독같은 비동기한 방식으로 사용한다고 해도, 같은 방식의 클로저 동작을 관찰할 수 있다.

챕터 5의 "반복문" 섹션을 상기해보자. `for` 반복문에서 `let` 선언을 설명할 때 사실 반복문에 대해서 오직 한 변수만 생성하는게 아니라, *각 반복*마다 새로운 변수를 생성한다고 했다. 이러한 트릭이자 기이한 특징은 클로저 반복문에서 정확하게 필요한 것이다.

```js
var keeps = [];

for (let i = 0; i < 3; i++) {
    // `let i`는 새로운 `i`를 준다
    // 각 반복마다, 자동적으로!
    keeps[i] = function keepEachI(){
        return i;
    };
}
keeps[0]();   // 0
keeps[1]();   // 1
keeps[2]();   // 2
```

`let`을 사용함으로써, 각 반복마다 하나씩 총 3개의 `i`가 생성되고, 그래서 세 클로저들은 예상했던대로 *그냥 동작한다.*

### Common Closures: Ajax and Events

Closure is most commonly encountered with callbacks:

```js
function lookupStudentRecord(studentID) {
    ajax(
        `https://some.api/student/${ studentID }`,
        function onRecord(record) {
            console.log(
                `${ record.name } (${ studentID })`
            );
        }
    );
}

lookupStudentRecord(114);
// Frank (114)
```

The `onRecord(..)` callback is going to be invoked at some point in the future, after the response from the Ajax call comes back. This invocation will happen from the internals of the `ajax(..)` utility, wherever that comes from. Furthermore, when that happens, the `lookupStudentRecord(..)` call will long since have completed.

Why then is `studentID` still around and accessible to the callback? Closure.

Event handlers are another common usage of closure:

```js
function listenForClicks(btn,label) {
    btn.addEventListener("click",function onClick(){
        console.log(
            `The ${ label } button was clicked!`
        );
    });
}

var submitBtn = document.getElementById("submit-btn");

listenForClicks(submitBtn,"Checkout");
```

The `label` parameter is closed over by the `onClick(..)` event handler callback. When the button is clicked, `label` still exists to be used. This is closure.

### What If I Can't See It?

You've probably heard this common adage:

> If a tree falls in the forest but nobody is around to hear it, does it make a sound?

It's a silly bit of philosophical gymnastics. Of course from a scientific perspective, sound waves are created. But the real point: *does it matter* if the sound happens?

Remember, the emphasis in our definition of closure is observability. If a closure exists (in a technical, implementation, or academic sense) but it cannot be observed in our programs, *does it matter?* No.

To reinforce this point, let's look at some examples that are *not* observably based on closure.

For example, invoking a function that makes use of lexical scope lookup:

```js
function say(myName) {
    var greeting = "Hello";
    output();

    function output() {
        console.log(
            `${ greeting }, ${ myName }!`
        );
    }
}

say("Kyle");
// Hello, Kyle!
```

The inner function `output()` accesses the variables `greeting` and `myName` from its enclosing scope. But the invocation of `output()` happens in that same scope, where of course `greeting` and `myName` are still available; that's just lexical scope, not closure.

Any lexically scoped language whose functions didn't support closure would still behave this same way.

In fact, global scope variables essentially cannot be (observably) closed over, because they're always accessible from everywhere. No function can ever be invoked in any part of the scope chain that is not a descendant of the global scope.

Consider:

```js
var students = [
    { id: 14, name: "Kyle" },
    { id: 73, name: "Suzy" },
    { id: 112, name: "Frank" },
    { id: 6, name: "Sarah" }
];

function getFirstStudent() {
    return function firstStudent(){
        return students[0].name;
    };
}

var student = getFirstStudent();

student();
// Kyle
```

The inner `firstStudent()` function does reference `students`, which is a variable outside its own scope. But since `students` happens to be from the global scope, no matter where that function is invoked in the program, its ability to access `students` is nothing more special than normal lexical scope.

All function invocations can access global variables, regardless of whether closure is supported by the language or not. Global variables don't need to be closed over.

Variables that are merely present but never accessed don't result in closure:

```js
function lookupStudent(studentID) {
    return function nobody(){
        var msg = "Nobody's here yet.";
        console.log(msg);
    };
}

var student = lookupStudent(112);

student();
// Nobody's here yet.
```

The inner function `nobody()` doesn't close over any outer variables—it only uses its own variable `msg`. Even though `studentID` is present in the enclosing scope, `studentID` is not referred to by `nobody()`. The JS engine doesn't need to keep `studentID` around after `lookupStudent(..)` has finished running, so GC wants to clean up that memory!

Whether JS functions support closure or not, this program would behave the same. Therefore, no observed closure here.

If there's no function invocation, closure can't be observed:

```js
function greetStudent(studentName) {
    return function greeting(){
        console.log(
            `Hello, ${ studentName }!`
        );
    };
}

greetStudent("Kyle");

// nothing else happens
```

This one's tricky, because the outer function definitely does get invoked. But the inner function is the one that *could* have had closure, and yet it's never invoked; the returned function here is just thrown away. So even if technically the JS engine created closure for a brief moment, it was not observed in any meaningful way in this program.

A tree may have fallen... but we didn't hear it, so we don't care.

### Observable Definition

We're now ready to define closure:

> Closure is observed when a function uses variable(s) from outer scope(s) even while running in a scope where those variable(s) wouldn't be accessible.

The key parts of this definition are:

* Must be a function involved

* Must reference at least one variable from an outer scope

* Must be invoked in a different branch of the scope chain from the variable(s)

This observation-oriented definition means we shouldn't dismiss closure as some indirect, academic trivia. Instead, we should look and plan for the direct, concrete effects closure has on our program behavior.

## The Closure Lifecycle and Garbage Collection (GC)

Since closure is inherently tied to a function instance, its closure over a variable lasts as long as there is still a reference to that function.

If ten functions all close over the same variable, and over time nine of these function references are discarded, the lone remaining function reference still preserves that variable. Once that final function reference is discarded, the last closure over that variable is gone, and the variable itself is GC'd.

This has an important impact on building efficient and performant programs. Closure can unexpectedly prevent the GC of a variable that you're otherwise done with, which leads to run-away memory usage over time. That's why it's important to discard function references (and thus their closures) when they're not needed anymore.

Consider:

```js
function manageBtnClickEvents(btn) {
    var clickHandlers = [];

    return function listener(cb){
        if (cb) {
            let clickHandler =
                function onClick(evt){
                    console.log("clicked!");
                    cb(evt);
                };
            clickHandlers.push(clickHandler);
            btn.addEventListener(
                "click",
                clickHandler
            );
        }
        else {
            // passing no callback unsubscribes
            // all click handlers
            for (let handler of clickHandlers) {
                btn.removeEventListener(
                    "click",
                    handler
                );
            }

            clickHandlers = [];
        }
    };
}

// var mySubmitBtn = ..
var onSubmit = manageBtnClickEvents(mySubmitBtn);

onSubmit(function checkout(evt){
    // handle checkout
});

onSubmit(function trackAction(evt){
    // log action to analytics
});

// later, unsubscribe all handlers:
onSubmit();
```

In this program, the inner `onClick(..)` function holds a closure over the passed in `cb` (the provided event callback). That means the `checkout()` and `trackAction()` function expression references are held via closure (and cannot be GC'd) for as long as these event handlers are subscribed.

When we call `onSubmit()` with no input on the last line, all event handlers are unsubscribed, and the `clickHandlers` array is emptied. Once all click handler function references are discarded, the closures of `cb` references to `checkout()` and `trackAction()` are discarded.

When considering the overall health and efficiency of the program, unsubscribing an event handler when it's no longer needed can be even more important than the initial subscription!

### Per Variable or Per Scope?

Another question we need to tackle: should we think of closure as applied only to the referenced outer variable(s), or does closure preserve the entire scope chain with all its variables?

In other words, in the previous event subscription snippet, is the inner `onClick(..)` function closed over only `cb`, or is it also closed over `clickHandler`, `clickHandlers`, and `btn`?

Conceptually, closure is **per variable** rather than *per scope*. Ajax callbacks, event handlers, and all other forms of function closures are typically assumed to close over only what they explicitly reference.

But the reality is more complicated than that.

Another program to consider:

```js
function manageStudentGrades(studentRecords) {
    var grades = studentRecords.map(getGrade);

    return addGrade;

    // ************************

    function getGrade(record){
        return record.grade;
    }

    function sortAndTrimGradesList() {
        // sort by grades, descending
        grades.sort(function desc(g1,g2){
            return g2 - g1;
        });

        // only keep the top 10 grades
        grades = grades.slice(0,10);
    }

    function addGrade(newGrade) {
        grades.push(newGrade);
        sortAndTrimGradesList();
        return grades;
    }
}

var addNextGrade = manageStudentGrades([
    { id: 14, name: "Kyle", grade: 86 },
    { id: 73, name: "Suzy", grade: 87 },
    { id: 112, name: "Frank", grade: 75 },
    // ..many more records..
    { id: 6, name: "Sarah", grade: 91 }
]);

// later

addNextGrade(81);
addNextGrade(68);
// [ .., .., ... ]
```

The outer function `manageStudentGrades(..)` takes a list of student records, and returns an `addGrade(..)` function reference, which we externally label `addNextGrade(..)`. Each time we call `addNextGrade(..)` with a new grade, we get back a current list of the top 10 grades, sorted numerically descending (see `sortAndTrimGradesList()`).

From the end of the original `manageStudentGrades(..)` call, and between the multiple `addNextGrade(..)` calls, the `grades` variable is preserved inside `addGrade(..)` via closure; that's how the running list of top grades is maintained. Remember, it's a closure over the variable `grades` itself, not the array it holds.

That's not the only closure involved, however. Can you spot other variables being closed over?

Did you spot that `addGrade(..)` references `sortAndTrimGradesList`? That means it's also closed over that identifier, which happens to hold a reference to the `sortAndTrimGradesList()` function. That second inner function has to stay around so that `addGrade(..)` can keep calling it, which also means any variables *it* closes over stick around—though, in this case, nothing extra is closed over there.

What else is closed over?

Consider the `getGrade` variable (and its function); is it closed over? It's referenced in the outer scope of `manageStudentGrades(..)` in the `.map(getGrade)` call. But it's not referenced in `addGrade(..)` or `sortAndTrimGradesList()`.

What about the (potentially) large list of student records we pass in as `studentRecords`? Is that variable closed over? If it is, the array of student records is never getting GC'd, which leads to this program holding onto a larger amount of memory than we might assume. But if we look closely again, none of the inner functions reference `studentRecords`.

According to the *per variable* definition of closure, since `getGrade` and `studentRecords` are *not* referenced by the inner functions, they're not closed over. They should be freely available for GC right after the `manageStudentGrades(..)` call completes.

Indeed, try debugging this code in a recent JS engine, like v8 in Chrome, placing a breakpoint inside the `addGrade(..)` function. You may notice that the inspector **does not** list the `studentRecords` variable. That's proof, debugging-wise anyway, that the engine does not maintain `studentRecords` via closure. Phew!

But how reliable is this observation as proof? Consider this (rather contrived!) program:

```js
function storeStudentInfo(id,name,grade) {
    return function getInfo(whichValue){
        // warning:
        //   using `eval(..)` is a bad idea!
        var val = eval(whichValue);
        return val;
    };
}

var info = storeStudentInfo(73,"Suzy",87);

info("name");
// Suzy

info("grade");
// 87
```

Notice that the inner function `getInfo(..)` is not explicitly closed over any of `id`, `name`, or `grade` variables. And yet, calls to `info(..)` seem to still be able to access the variables, albeit through use of the `eval(..)` lexical scope cheat (see Chapter 1).

So all the variables were definitely preserved via closure, despite not being explicitly referenced by the inner function. So does that disprove the *per variable* assertion in favor of *per scope*? Depends.

Many modern JS engines do apply an *optimization* that removes any variables from a closure scope that aren't explicitly referenced. However, as we see with `eval(..)`, there are situations where such an optimization cannot be applied, and the closure scope continues to contain all its original variables. In other words, closure must be *per scope*, implementation wise, and then an optional optimization trims down the scope to only what was closed over (a similar outcome as *per variable* closure).

Even as recent as a few years ago, many JS engines did not apply this optimization; it's possible your websites may still run in such browsers, especially on older or lower-end devices. That means it's possible that long-lived closures such as event handlers may be holding onto memory much longer than we would have assumed.

And the fact that it's an optional optimization in the first place, rather than a requirement of the specification, means that we shouldn't just casually over-assume its applicability.

In cases where a variable holds a large value (like an object or array) and that variable is present in a closure scope, if you don't need that value anymore and don't want that memory held, it's safer (memory usage) to manually discard the value rather than relying on closure optimization/GC.

Let's apply a *fix* to the earlier `manageStudentGrades(..)` example to ensure the potentially large array held in `studentRecords` is not caught up in a closure scope unnecessarily:

```js
function manageStudentGrades(studentRecords) {
    var grades = studentRecords.map(getGrade);

    // unset `studentRecords` to prevent unwanted
    // memory retention in the closure
    studentRecords = null;

    return addGrade;
    // ..
}
```

We're not removing `studentRecords` from the closure scope; that we cannot control. We're ensuring that even if `studentRecords` remains in the closure scope, that variable is no longer referencing the potentially large array of data; the array can be GC'd.

Again, in many cases JS might automatically optimize the program to the same effect. But it's still a good habit to be careful and explicitly make sure we don't keep any significant amount of device memory tied up any longer than necessary.

As a matter of fact, we also technically don't need the function `getGrade()` anymore after the `.map(getGrade)` call completes. If profiling our application showed this was a critical area of excess memory use, we could possibly eek out a tiny bit more memory by freeing up that reference so its value isn't tied up either. That's likely unnecessary in this toy example, but this is a general technique to keep in mind if you're optimizing the memory footprint of your application.

The takeaway: it's important to know where closures appear in our programs, and what variables are included. We should manage these closures carefully so we're only holding onto what's minimally needed and not wasting memory.

## An Alternative Perspective

Reviewing our working definition for closure, the assertion is that functions are "first-class values" that can be passed around the program, just like any other value. Closure is the link-association that connects that function to the scope/variables outside of itself, no matter where that function goes.

Let's recall a code example from earlier in this chapter, again with relevant scope bubble colors annotated:

```js
// outer/global scope: RED(1)

function adder(num1) {
    // function scope: BLUE(2)

    return function addTo(num2){
        // function scope: GREEN(3)

        return num1 + num2;
    };
}

var add10To = adder(10);
var add42To = adder(42);

add10To(15);    // 25
add42To(9);     // 51
```

Our current perspective suggests that wherever a function is passed and invoked, closure preserves a hidden link back to the original scope to facilitate the access to the closed-over variables. Figure 4, repeated here for convenience, illustrates this notion:

<figure>
    <img src="images/fig4.png" width="400" alt="Function instances linked to scopes via closure" align="center">
    <figcaption><em>Fig. 4 (repeat): Visualizing Closures</em></figcaption>
    <br><br>
</figure>

But there's another way of thinking about closure, and more precisely the nature of functions being *passed around*, that may help deepen the mental models.

This alternative model de-emphasizes "functions as first-class values," and instead embraces how functions (like all non-primitive values) are held by reference in JS, and assigned/passed by reference-copy—see Appendix A of the *Get Started* book for more information.

Instead of thinking about the inner function instance of `addTo(..)` moving to the outer RED(1) scope via the `return` and assignment, we can envision that function instances actually just stay in place in their own scope environment, of course with their scope-chain intact.

What gets *sent* to the RED(1) scope is **just a reference** to the in-place function instance, rather than the function instance itself. Figure 5 depicts the inner function instances remaining in place, pointed to by the RED(1) `addTo10` and `addTo42` references, respectively:

<figure>
    <img src="images/fig5.png" width="400" alt="Function instances inside scopes via closure, linked to by references" align="center">
    <figcaption><em>Fig. 5: Visualizing Closures (Alternative)</em></figcaption>
    <br><br>
</figure>

As shown in Figure 5, each call to `adder(..)` still creates a new BLUE(2) scope containing a `num1` variable, as well as an instance of the GREEN(3) `addTo(..)` scope. But what's different from Figure 4 is, now these GREEN(3) instances remain in place, naturally nested inside of their BLUE(2) scope instances. The `addTo10` and `addTo42` references are moved to the RED(1) outer scope, not the function instances themselves.

When `addTo10(15)` is called, the `addTo(..)` function instance (still in place in its original BLUE(2) scope environment) is invoked. Since the function instance itself never moved, of course it still has natural access to its scope chain. Same with the `addTo42(9)` call—nothing special here beyond lexical scope.

So what then *is* closure, if not the *magic* that lets a function maintain a link to its original scope chain even as that function moves around in other scopes? In this alternative model, functions stay in place and keep accessing their original scope chain just like they always could.

Closure instead describes the *magic* of **keeping alive a function instance**, along with its whole scope environment and chain, for as long as there's at least one reference to that function instance floating around in any other part of the program.

That definition of closure is less observational and a bit less familiar-sounding compared to the traditional academic perspective. But it's nonetheless still useful, because the benefit is that we simplify explanation of closure to a straightforward combination of references and in-place function instances.

The previous model (Figure 4) is not *wrong* at describing closure in JS. It's just more conceptually inspired, an academic perspective on closure. By contrast, the alternative model (Figure 5) could be described as a bit more implementation focused, how JS actually works.

Both perspectives/models are useful in understanding closure, but the reader may find one a little easier to hold than the other. Whichever you choose, the observable outcomes in our program are the same.

| NOTE: |
| :--- |
| This alternative model for closure does affect whether we classify synchronous callbacks as examples of closure or not. More on this nuance in Appendix A. |

## Why Closure?

Now that we have a well-rounded sense of what closure is and how it works, let's explore some ways it can improve the code structure and organization of an example program.

Imagine you have a button on a page that when clicked, should retrieve and send some data via an Ajax request. Without using closure:

```js
var APIendpoints = {
    studentIDs:
        "https://some.api/register-students",
    // ..
};

var data = {
    studentIDs: [ 14, 73, 112, 6 ],
    // ..
};

function makeRequest(evt) {
    var btn = evt.target;
    var recordKind = btn.dataset.kind;
    ajax(
        APIendpoints[recordKind],
        data[recordKind]
    );
}

// <button data-kind="studentIDs">
//    Register Students
// </button>
btn.addEventListener("click",makeRequest);
```

The `makeRequest(..)` utility only receives an `evt` object from a click event. From there, it has to retrieve the `data-kind` attribute from the target button element, and use that value to lookup both a URL for the API endpoint as well as what data should be included in the Ajax request.

This works OK, but it's unfortunate (inefficient, more confusing) that the event handler has to read a DOM attribute each time it's fired. Why couldn't an event handler *remember* this value? Let's try using closure to improve the code:

```js
var APIendpoints = {
    studentIDs:
        "https://some.api/register-students",
    // ..
};

var data = {
    studentIDs: [ 14, 73, 112, 6 ],
    // ..
};

function setupButtonHandler(btn) {
    var recordKind = btn.dataset.kind;

    btn.addEventListener(
        "click",
        function makeRequest(evt){
            ajax(
                APIendpoints[recordKind],
                data[recordKind]
            );
        }
    );
}

// <button data-kind="studentIDs">
//    Register Students
// </button>

setupButtonHandler(btn);
```

With the `setupButtonHandler(..)` approach, the `data-kind` attribute is retrieved once and assigned to the `recordKind` variable at initial setup. `recordKind` is then closed over by the inner `makeRequest(..)` click handler, and its value is used on each event firing to look up the URL and data that should be sent.

| NOTE: |
| :--- |
| `evt` is still passed to `makeRequest(..)`, though in this case we're not using it anymore. It's still listed, for consistency with the previous snippet. |

By placing `recordKind` inside `setupButtonHandler(..)`, we limit the scope exposure of that variable to a more appropriate subset of the program; storing it globally would have been worse for code organization and readability. Closure lets the inner `makeRequest()` function instance *remember* this variable and access whenever it's needed.

Building on this pattern, we could have looked up both the URL and data once, at setup:

```js
function setupButtonHandler(btn) {
    var recordKind = btn.dataset.kind;
    var requestURL = APIendpoints[recordKind];
    var requestData = data[recordKind];

    btn.addEventListener(
        "click",
        function makeRequest(evt){
            ajax(requestURL,requestData);
        }
    );
}
```

Now `makeRequest(..)` is closed over `requestURL` and `requestData`, which is a little bit cleaner to understand, and also slightly more performant.

Two similar techniques from the Functional Programming (FP) paradigm that rely on closure are partial application and currying. Briefly, with these techniques, we alter the *shape* of functions that require multiple inputs so some inputs are provided up front, and other inputs are provided later; the initial inputs are remembered via closure. Once all inputs have been provided, the underlying action is performed.

By creating a function instance that encapsulates some information inside (via closure), the function-with-stored-information can later be used directly without needing to re-provide that input. This makes that part of the code cleaner, and also offers the opportunity to label partially applied functions with better semantic names.

Adapting partial application, we can further improve the preceding code:

```js
function defineHandler(requestURL,requestData) {
    return function makeRequest(evt){
        ajax(requestURL,requestData);
    };
}

function setupButtonHandler(btn) {
    var recordKind = btn.dataset.kind;
    var handler = defineHandler(
        APIendpoints[recordKind],
        data[recordKind]
    );
    btn.addEventListener("click",handler);
}
```

The `requestURL` and `requestData` inputs are provided ahead of time, resulting in the `makeRequest(..)` partially applied function, which we locally label `handler`. When the event eventually fires, the final input (`evt`, even though it's ignored) is passed to `handler()`, completing its inputs and triggering the underlying Ajax request.

Behavior-wise, this program is pretty similar to the previous one, with the same type of closure. But by isolating the creation of `makeRequest(..)` in a separate utility (`defineHandler(..)`), we make that definition more reusable across the program. We also explicitly limit the closure scope to only the two variables needed.

## Closer to Closure

As we close down a dense chapter, take some deep breaths let it all sink in. Seriously, that's a lot of information for anyone to consume!

We explored two models for mentally tackling closure:

* Observational: closure is a function instance remembering its outer variables even as that function is passed to and **invoked in** other scopes.

* Implementational: closure is a function instance and its scope environment preserved in-place while any references to it are passed around and **invoked from** other scopes.

Summarizing the benefits to our programs:

* Closure can improve efficiency by allowing a function instance to remember previously determined information instead of having to compute it each time.

* Closure can improve code readability, bounding scope-exposure by encapsulating variable(s) inside function instances, while still making sure the information in those variables is accessible for future use. The resultant narrower, more specialized function instances are cleaner to interact with, since the preserved information doesn't need to be passed in every invocation.

Before you move on, take some time to restate this summary *in your own words*, explaining what closure is and why it's helpful in your programs. The main book text concludes with a final chapter that builds on top of closure with the module pattern.
