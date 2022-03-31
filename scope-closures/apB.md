# You Don't Know JS Yet: Scope & Closures - 2nd Edition
# 부록 B: 연습

이 부록은 이 책의 주요 주제에 대한 이해를 테스트하고 확고히 하는 도전적이면서 재미있는 연습 문제를 제공하는데 초점을 맞춘다. 마지막의 솔루션으로 건너뛰는 것 보다 연습 문제들을 (실제 코드 에디터로!) 스스로 시도해보는게 좋을 것이다. 컨닝하지 마라!

이 연습문제들은 정확히 얻어야 하는 특정한 정답은 없다. 너의 접근 방법이 제시된 해결책과 일부(또는 많이!) 다를 수 있으나 괜찮다.

너의 코드를 어떻게 작성했는지 아무도 평가하지 않는다. 나는 너가 튼튼한 기반 지식으로 이러한 종류의 코딩 작업을 해결할 수 있다는 자신감을 가지고 이 책을 마무리했으면 좋겠다. 그것이 여기서 유일한 목표다. 너의 코드로 너가 행복하다면 나도 행복하다!

## 구슬 양동이

2장에서의 그림2를 기억하나?

<figure>
    <img src="images/fig2.png" width="300" alt="Colored Scope Bubbles" align="center">
    <figcaption><em>그림2 (2장): 색칠된 스코프 버블</em></figcaption>
    <br><br>
</figure>

이 연습 문제는 이러한 제약 조건을 만족하며 중첩 함수와 블록 스코프들을 포함하는 프로그램(어떤 프로그램이든!)을 작성하는 것을 묻는다.

* 모든 스코프(글로벌 스코프를 포함해서!)를 다른 색으로 색칠한다면, 적어도 6개의 색이 필요하다. 각 스코프를 색상으로 표시하는 코드 주석을 추가해라.

    보너스: 코드가 가질 수 있는 묵시적 스코프를 식별해라.

* 한 스코프는 적어도 한 식별자를 가진다.

* 적어도 2개의 함수 스코프와 적어도 2개의 블록 스코프를 포함한다.

* 적어도 바깥 스코프의 한 변수는 반드시 중첩 스코프 변수(3장)에 의해 섀도잉되어야한다.

* 적어도 하나의 변수 참조는 스코프 체인에서 적어도 2단계 이상의 변수 선언을 이행해야한다.

| 조언: |
| :--- |
| 이 연습 문제를 위해서 foo/bar/baz 같은 임시 코드를 작성할 *수도* 있으나, 적어도 합리적인 일을 하는 일종의 사소한 실제 코드를 써보도록 노력할 것을 추천한다. |

연습 문제를 스스로 시도해보고, 이 부록의 마지막에 제시된 해결방안을 확인해라.

## 클로저 (파트 1)

몇 가지 일반적인 수학 연산을 이용해서 클로저 연습을 시작해보자. 값이 소수(1과 그 자신 이외에 약수가 없는 수)인지 아닌지를 판별하고, 주어진 숫자에 대한 소수 인자(약수) 목록을 만들어보자.

예제:

```js
isPrime(11);        // true
isPrime(12);        // false

factorize(11);      // [ 11 ]
factorize(12);      // [ 3, 2, 2 ] --> 3*2*2=12
```

아래는 Math.js 패키지에서 구현한 `isPrime(..)` 코드이다: [^MathJSisPrime]

```js
function isPrime(v) {
    if (v <= 3) {
        return v > 1;
    }
    if (v % 2 == 0 || v % 3 == 0) {
        return false;
    }
    var vSqrt = Math.sqrt(v);
    for (let i = 5; i <= vSqrt; i += 6) {
        if (v % i == 0 || v % (i + 2) == 0) {
            return false;
        }
    }
    return true;
}
```

그리고 `factorize(..)`의 기본적인 구현은 다음과 같다. (6장의 `factorial(..)`과 혼동하지 말 것):

```js
function factorize(v) {
    if (!isPrime(v)) {
        let i = Math.floor(Math.sqrt(v));
        while (v % i != 0) {
            i--;
        }
        return [
            ...factorize(i),
            ...factorize(v / i)
        ];
    }
    return [v];
}
```

| 비고: |
| :--- |
| 성능을 최적화하지 않았기 때문에 이것을 기본형이라고 부르겠다. 이진 재귀(꼬리 물기 최적화를 하지 못함)이고 중간 단계에서 많은 배열 복사본을 생성한다. 또한 발견한 인자의 순서를 정하지도 못한다. 이 작업을 위한 다른 알고리즘도 많지만, 연습을 위해 짧고 쉽게 이해할 수 있는 알고리즘을 사용했다. |

위 프로그램에서 `isPrime(4327)`를 몇 번 호출하면, 매 단계 마다 수십 번 씩 비교/연산하는 것을 확인할 수 있다. `factorize(..)`를 자세히 보면, `isPrime(..)`을 여러번 호출해서 소수 목록을 만들고 있다. 그리고 대부분의 호출이 반복된 것일 가능성이 높다. 이건 낭비하는 일이다!

이 연습에서 처음으로 해야할 일은 클로저를 사용해서 `isPrime(..)`의 결과를 저장하는 캐시를 구현하여, 주어진 수의 소수 여부(`true` 또는 `false`)를 한 번만 계산하도록 하는 것이다. 힌트: 우리는 이미 6장에서 `factorial(..)`로 이러한 종류의 캐싱을 보여주었다.

`factorize(..)`는 재귀적으로 구현되어 있는데, 이는 자기 자신을 반복해서 호출한다는 걸 의미한다. 즉, 같은 숫자의 소수 여부를 계산하기 위해 낭비하게 되는 호출이 많을 수 있다. 따라서, 두 번째로 해야할 일은 `factorize(..)`에도 동일한 클로저 캐시 기법을 사용하는 것이다.

`isPrime(..)`와 `factorize(..)`의 캐싱에는 하나의 스코프에 넣지 말고 분리된 클로저를 사용하라.

직접 연습하고 나서 이 부록의 끝에 있는 권장 답안을 확인하라.

### 메모리에 대하여

클로저 캐시 기법과 이것이 애플리케이션의 성능에 미치는 영향에 대해 간단히 설명하겠다.

반복되는 호출 결과를 저장하면 계산 속도가 (경우에 따라 극적으로) 향상된다는 것을 알 수 있다. 그러나 이렇게 클로저를 사용하는 것에는 매우 주의해야 할 트레이드오프가 있다.

그것은 바로 메모리이다. 클로저 캐시 기법은 필수적으로 (메모리에서) 캐시를 무한히 늘리게 된다. 만약 문제의 함수를 수백만 번 호출하는데 대부분 고유한 입력값을 사용한다면, 매우 많은 메모리를 낭비하게 될 것이다. 이 기법은 분명히 비용을 들일 가치가 있긴 하지만, 일반적인 입력으로 반복될 가능성이 높은지 확인하고 나서 활용해야만 효율적이다.

대부분 고유한 입력값으로 호출하게 되어 캐시가 본질적으로 전혀 유용하게 *사용되지* 않는 경우, 이 기법은 사용하기 부적절하다.

용량을 제한하는 LRU(least recently used) 캐시처럼 정교한 캐싱 방식을 사용하는 것도 좋은 방법이다. LRU는 최대 용량에 가까워지면 최근에 가장 적게 사용한 값을 메모리에서 제거한다.

이 때, 단점은 LRU가 그 자체의 구현이 쉽지 않다는 것이다. 제대로 최적화된 LRU를 사용하고 싶다면, 문제 상황에서 발생할 수 있는 모든 단점을 빠짐없이 알고 있어야 한다.

## 클로저 (파트 2)

이번 연습에서는 클로저로 값 토글러를 전달해주는 `toggle(..)` 유틸리티를 다시 정의한다.

하나 이상의 값(인자)을 `toggle(..)`에 전달하면 함수를 반환하게 된다. 반환된 함수는 전달 받았던 모든 값을 순서대로 하나씩 돌아가며 전환할 것이다.

```js
function toggle(/* .. */) {
    // ..
}

var hello = toggle("hello");
var onOff = toggle("on","off");
var speed = toggle("slow","medium","fast");

hello();      // "hello"
hello();      // "hello"

onOff();      // "on"
onOff();      // "off"
onOff();      // "on"

speed();      // "slow"
speed();      // "medium"
speed();      // "fast"
speed();      // "slow"
```

`toggle(..)`에 값을 전달하지 않는 코너 케이스는 그다지 중요하지 않다. 이런 경우, 토글러 인스턴스를 항상 `undefined`로 반환하면 되기 때문이다.

직접 연습한 다음, 이 부록의 끝에 있는 권장 해결책을 확인하라.

## 클로저 (파트 3)

마지막 세 번째 연습에서는 기본적인 계산기를 구현해보자. `calculator()` 함수는 다음과 같은 함수 형태로 자체의 상태를 유지하는 계산기 인스턴스를 생성한다(`calc(..)`, 아래를 참고하라).

```js
function calculator() {
    // ..
}

var calc = calculator();
```

매번 `calc(..)`를 호출할때, 누른 계산기의 버튼을 표현하는 문자를 하나씩 전달한다. 좀 더 알기 쉽게 하기 위해, 계산기는 숫자(0~9), 산술 연산자(+, -, \*, /)와 "="의 입력만을 지원하도록 기능을 제한할 것이다. 연산자는 입력한 순서대로만 처리하며, "( )"로 묶거나 연산자 우선순위를 고려하지 않는다.

소수점 입력은 지원하지 않지만, 나눗셈 연산의 결과는 소수점을 포함할 수 있다. 음수 입력을 지원하진 않지만, "-" 연산의 결과로 음수가 나올 수 있다. 따라서, 처음으로 산술 연산자를 입력할 때부터 연산 결과로 음수나 소수가 나오는 것이 가능하게 하고 그 값으로 계속 연산을 할 수 있게 해야한다.

`calc(..)` 호출의 반환 값은 실제 계산기에 표시되는 내용을 모방하라. 누른 값를 반영해서 보여주다가 "="를 누를 때에는 계산 결과를 보여줘야 한다. 

다음 예제를 보자:

```js
calc("4");     // 4
calc("+");     // +
calc("7");     // 7
calc("3");     // 3
calc("-");     // -
calc("2");     // 2
calc("=");     // 75
calc("*");     // *
calc("4");     // 4
calc("=");     // 300
calc("5");     // 5
calc("-");     // -
calc("5");     // 5
calc("=");     // 0
```

이 사용 예제는 아직 다듬어지지 않아서 `useCalc(..)`을 사용하고 있다. 문자열에서 하나씩 문자를 가져와 계산기를 실행하고 매번 표시할 내용을 계산한다.

```js
function useCalc(calc,keys) {
    return [...keys].reduce(
        function showDisplay(display,key){
            var ret = String( calc(key) );
            return (
                display +
                (
                  (ret != "" && key == "=") ?
                      "=" :
                      ""
                ) +
                ret
            );
        },
        ""
    );
}

useCalc(calc,"4+3=");           // 4+3=7
useCalc(calc,"+9=");            // +9=16
useCalc(calc,"*8=");            // *5=128
useCalc(calc,"7*2*3=");         // 7*2*3=42
useCalc(calc,"1/0=");           // 1/0=ERR
useCalc(calc,"+3=");            // +3=ERR
useCalc(calc,"51=");            // 51
```

이 `useCalc(..)` 헬퍼의 실질적인 사용법을 보면 항상 마지막에 문자 "="를 입력 받고 있다.

계산기에 합계를 표시할 때, 어떤 경우에는 특별한 처리가 필요하다. 아래와 같이 `formatTotal(..)` 함수를 제공하여, (`"="`을 입력받은 이후) 계산한 값을 반환될 때마다 사용하고 있다:

```js
function formatTotal(display) {
    if (Number.isFinite(display)) {
        // 최대 11자까지 표시하도록 제한한다.
        let maxDigits = 11;
        // "e+" 표기를 위한 자리를 남길 것인가?
        if (Math.abs(display) > 99999999999) {
            maxDigits -= 6;
        }
        // "-" 표기를 위한 자리를 남길 것인가?
        if (display < 0) {
            maxDigits--;
        }

        // 정수인가?
        if (Number.isInteger(display)) {
            display = display
                .toPrecision(maxDigits)
                .replace(/\.0+$/,"");
        }
        // 소수
        else {
            // "."를 위한 자리를 남기기
            maxDigits--;
            // "0"이 들어갈 자리를 남겨둬야 하는가?
            if (
                Math.abs(display) >= 0 &&
                Math.abs(display) < 1
            ) {
                maxDigits--;
            }
            display = display
                .toPrecision(maxDigits)
                .replace(/0+$/,"");
        }
    }
    else {
        display = "ERR";
    }
    return display;
}
```

`formatTotal(..)`가 어떻게 작동하는지에 대해서 너무 많이 걱정하지는 말아라. 대부분은 음수, 소수점 반복, "e+" 지수 표기법이 필요한 경우에도 최대 11자까지 표시하도록 길이를 제한하기 위한 여러가지 처리이다.

다시 말하지만, 계산기 고유의 동작을 구현하는 데에 깊게 빠지지는 말아라. 클로저를 통한 *메모리* 기능에 초점을 맞춰라.

직접 연습한 다음, 이 부록의 끝에 있는 권장 해결책을 확인하라.

## 모듈

이 연습문제는 계산기를 클로저(3장)에서 모듈로 변환한다.

계산기에 추가 기능을 넣지 않고, 인터페이스만 변경하고 있다. 단일 함수 `calc(..)`를 호출하는 대신에 계산기의 "키 입력(keypress)"마다 공개 API에서 특정 메서드를 호출할 것이다. 출력은 같다.

모듈은 싱글톤 IIFE 대신에 `calculator()`라 부르는 클래식 모듈 팩토리로서 표현하게 된다. 그래서 여러개의 계산기가 원하는 만큼 생성될 수 있다.

공개 API는 아래 메서드를 포함해야 한다.

* `number(..)` (입력: 임의의 문자/숫자를 "눌렀을 때")
* `plus()`
* `minus()`
* `mult()`
* `div()`
* `eq()`

사용법은 아래와 같다.

```js
var calc = calculator();

calc.number("4");     // 4
calc.plus();          // +
calc.number("7");     // 7
calc.number("3");     // 3
calc.minus();         // -
calc.number("2");     // 2
calc.eq();            // 75
```

`formatTotal(..)`은 이전 연습문제와 같다. 그러나 `useCalc(..)` 헬퍼는 모듈 API와 같이 동작하기 위해 조정될 필요가 있다.

```js
function useCalc(calc,keys) {
    var keyMappings = {
        "+": "plus",
        "-": "minus",
        "*": "mult",
        "/": "div",
        "=": "eq"
    };

    return [...keys].reduce(
        function showDisplay(display,key){
            var fn = keyMappings[key] || "number";
            var ret = String( calc[fn](key) );
            return (
                display +
                (
                  (ret != "" && key == "=") ?
                      "=" :
                      ""
                ) +
                ret
            );
        },
        ""
    );
}

useCalc(calc,"4+3=");           // 4+3=7
useCalc(calc,"+9=");            // +9=16
useCalc(calc,"*8=");            // *5=128
useCalc(calc,"7*2*3=");         // 7*2*3=42
useCalc(calc,"1/0=");           // 1/0=ERR
useCalc(calc,"+3=");            // +3=ERR
useCalc(calc,"51=");            // 51
```

연습문제를 풀어보고, 부록의 끝에 있는 해결 방안을 확인해라.

이 연습문제를 풀어봄으로써, 이전 연습문제의 클로저 함수 접근 방식에 비해 계산기를 모듈로서 나타낼 때의 장/단점을 고려하게 된다.

보너스: 너의 생각을 설명하는 몇 가지 문장을 써봐라.

보너스 #2: 모듈을 UMD, CommonJS, and ESM (ES 모듈)를 포함하여 다른 모듈 형식으로 변환해봐라.

## 해결 방안 예시

너가 이 부분을 읽기 전에 연습 문제를 모두 시도해봤기를 바란다. 컨닝하지 마라!

기억해라. 각 예시 해결 방안은 각 문제에 접근하는 많은 다른 방법 중 하나일 뿐이다. 이것들이 "정답"인건 아니지만 각 연습 문제들을 합리적인 방향으로 접근하도록 설명해준다.

이 해결 방안을 읽으면서 너가 얻을 수 있는 가장 중요한 장점은 이를 너의 코드와 비교하고 왜 각각이 비슷하거나 다른 선택을 했는지를 분석하는 것이다. 사소한 것들에 대해서 보지말고 핵심 주제에 집중하도록 해라.

### 해결 방안: 구슬 양동이

*구슬 양동이 연습 문제*는 이렇게 해결될 수 있다:

```js
// 빨강(1)
const howMany = 100;

// 에라토스테네스의 체
function findPrimes(howMany) {
    // 파랑(2)
    var sieve = Array(howMany).fill(true);
    var max = Math.sqrt(howMany);

    for (let i = 2; i < max; i++) {
        // 초록(3)
        if (sieve[i]) {
            // 오렌지(4)
            let j = Math.pow(i,2);
            for (let k = j; k < howMany; k += i) {
                // 보라(5)
                sieve[k] = false;
            }
        }
    }

    return sieve
        .map(function getPrime(flag,prime){
            // 분홍(6)
            if (flag) return prime;
            return flag;
        })
        .filter(function onlyPrimes(v){
            // 노랑(7)
            return !!v;
        })
        .slice(1);
}

findPrimes(howMany);
// [
//    2, 3, 5, 7, 11, 13, 17,
//    19, 23, 29, 31, 37, 41,
//    43, 47, 53, 59, 61, 67,
//    71, 73, 79, 83, 89, 97
// ]
```

### 해결 방안: 클로저 (파트 1)

*클로저 연습 문제 (파트 1)*의 `isPrime(..)`과 `factorize(..)`은 이렇게 해결할 수 있다:

```js
var isPrime = (function isPrime(v){
    var primes = {};

    return function isPrime(v) {
        if (v in primes) {
            return primes[v];
        }
        if (v <= 3) {
            return (primes[v] = v > 1);
        }
        if (v % 2 == 0 || v % 3 == 0) {
            return (primes[v] = false);
        }
        let vSqrt = Math.sqrt(v);
        for (let i = 5; i <= vSqrt; i += 6) {
            if (v % i == 0 || v % (i + 2) == 0) {
                return (primes[v] = false);
            }
        }
        return (primes[v] = true);
    };
})();

var factorize = (function factorize(v){
    var factors = {};

    return function findFactors(v) {
        if (v in factors) {
            return factors[v];
        }
        if (!isPrime(v)) {
            let i = Math.floor(Math.sqrt(v));
            while (v % i != 0) {
                i--;
            }
            return (factors[v] = [
                ...findFactors(i),
                ...findFactors(v / i)
            ]);
        }
        return (factors[v] = [v]);
    };
})();
```

각 함수에서 사용한 일반적인 과정:

1. IIFE를 래핑하여 캐시 변수가 상주할 스코프를 정의한다.

2. 기본 호출에서 캐시를 확인하고 결과를 이미 알고 있으면 반환한다.

3. `반환`이 발생했던 각 위치에서 캐시에 할당하고 해당 할당 작업의 결과를 반환하면 된다.이것은 주로 책에서 간결함을 위한 공간 절약 트릭이다.

또한 내부 함수 이름을 `factorize(..)` 에서 `findFactors(..)`로 변경했다. 이는 기술적으로 필요한 것은 아니지만, 재귀 호출이 호출하는 함수를 더 명확하게 하는 데 도움이 된다.

### 해결 방안: 클로저 (파트 2)

*클로저 연습문제 (파트 2)* `toggle(..)` 은 이렇게 해결할 수 있다:

```js
function toggle(...vals) {
    var unset = {};
    var cur = unset;

    return function next(){
        // 목록 끝에
        // 이전 값을 저장한다.
        if (cur != unset) {
            vals.push(cur);
        }
        cur = vals.shift();
        return cur;
    };
}

var hello = toggle("hello");
var onOff = toggle("on","off");
var speed = toggle("slow","medium","fast");

hello();      // "hello"
hello();      // "hello"

onOff();      // "on"
onOff();      // "off"
onOff();      // "on"

speed();      // "slow"
speed();      // "medium"
speed();      // "fast"
speed();      // "slow"
```

### 해결 방안: 클로저 (파트 3)

*클로저 연습문제 (파트 3)* `calculator()` 는 이렇게 해결할 수 있다:

```js
// 이전 부분:
//
// function useCalc(..) { .. }
// function formatTotal(..) { .. }

function calculator() {
    var currentTotal = 0;
    var currentVal = "";
    var currentOper = "=";

    return pressKey;

    // ********************

    function pressKey(key){
        // 숫자 키인지?
        if (/\d/.test(key)) {
            currentVal += key;
            return key;
        }
        // 연산자 키인지?
        else if (/[+*/-]/.test(key)) {
            // 다중 연산인지?
            if (
                currentOper != "=" &&
                currentVal != ""
            ) {
                // 암시적으로 '=' 키를 누른다
                pressKey("=");
            }
            else if (currentVal != "") {
                currentTotal = Number(currentVal);
            }
            currentOper = key;
            currentVal = "";
            return key;
        }
        // = 키인지?
        else if (
            key == "=" &&
            currentOper != "="
        ) {
            currentTotal = op(
                currentTotal,
                currentOper,
                Number(currentVal)
            );
            currentOper = "=";
            currentVal = "";
            return formatTotal(currentTotal);
        }
        return "";
    };

    function op(val1,oper,val2) {
        var ops = {
            // 비고: 책의 간결함을 위해
            // 화살표 함수를 사용했다
            "+": (v1,v2) => v1 + v2,
            "-": (v1,v2) => v1 - v2,
            "*": (v1,v2) => v1 * v2,
            "/": (v1,v2) => v1 / v2
        };
        return ops[oper](val1,val2);
    }
}

var calc = calculator();

useCalc(calc,"4+3=");           // 4+3=7
useCalc(calc,"+9=");            // +9=16
useCalc(calc,"*8=");            // *5=128
useCalc(calc,"7*2*3=");         // 7*2*3=42
useCalc(calc,"1/0=");           // 1/0=ERR
useCalc(calc,"+3=");            // +3=ERR
useCalc(calc,"51=");            // 51
```

| 비고: |
| :--- |
| 기억해라: 이건 클로저에 대한 연습문제다. 실제 계산기의 구조에 너무 집중하지 말고, 함수 호출에 거쳐서 상태를 적절하기 *기억하는지*에 초점을 맞춰라. |

### 해결 방안: 모듈

*모듈 연습 문제* `calculator()`는 이렇게 해결할 수 있다:

```js
// 이전 부분:
//
// function useCalc(..) { .. }
// function formatTotal(..) { .. }

function calculator() {
    var currentTotal = 0;
    var currentVal = "";
    var currentOper = "=";

    var publicAPI = {
        number,
        eq,
        plus() { return operator("+"); },
        minus() { return operator("-"); },
        mult() { return operator("*"); },
        div() { return operator("/"); }
    };

    return publicAPI;

    // ********************

    function number(key) {
        // 숫자 키인지?
        if (/\d/.test(key)) {
            currentVal += key;
            return key;
        }
    }

    function eq() {
        // = 키인지?
        if (currentOper != "=") {
            currentTotal = op(
                currentTotal,
                currentOper,
                Number(currentVal)
            );
            currentOper = "=";
            currentVal = "";
            return formatTotal(currentTotal);
        }
        return "";
    }

    function operator(key) {
        // 다중 연산인지?
        if (
            currentOper != "=" &&
            currentVal != ""
        ) {
            // 암시적으로 '='를 누른다.
            eq();
        }
        else if (currentVal != "") {
            currentTotal = Number(currentVal);
        }
        currentOper = key;
        currentVal = "";
        return key;
    }

    function op(val1,oper,val2) {
        var ops = {
            // 비고: 책의 간결성을 위해서
            // 화살표 함수를 사용하였다.
            "+": (v1,v2) => v1 + v2,
            "-": (v1,v2) => v1 - v2,
            "*": (v1,v2) => v1 * v2,
            "/": (v1,v2) => v1 / v2
        };
        return ops[oper](val1,val2);
    }
}

var calc = calculator();

useCalc(calc,"4+3=");           // 4+3=7
useCalc(calc,"+9=");            // +9=16
useCalc(calc,"*8=");            // *5=128
useCalc(calc,"7*2*3=");         // 7*2*3=42
useCalc(calc,"1/0=");           // 1/0=ERR
useCalc(calc,"+3=");            // +3=ERR
useCalc(calc,"51=");            // 51
```

이 책은 여기까지다. 마무리한 것을 축하한다! 준비가 되면 3권, *객체 및 클래스*로 이동해라.

[^MathJSisPrime]: *Math.js: isPrime(..)*, https://github.com/josdejong/mathjs/blob/develop/src/function/utils/isPrime.js, 3 March 2020.
