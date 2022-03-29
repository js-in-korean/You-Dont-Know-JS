# You Don't Know JS Yet: Scope & Closures - 2nd Edition
# Appendix B: Practice

This appendix aims to give you some challenging and interesting exercises to test and solidify your understanding of the main topics from this book. It's a good idea to try out the exercises yourself—in an actual code editor!—instead of skipping straight to the solutions at the end. No cheating!

These exercises don't have a specific right answer that you have to get exactly. Your approach may differ some (or a lot!) from the solutions presented, and that's OK.

There's no judging you on how you write your code. My hope is that you come away from this book feeling confident that you can tackle these sorts of coding tasks built on a strong foundation of knowledge. That's the only objective, here. If you're happy with your code, I am, too!

## Buckets of Marbles

Remember Figure 2 from back in Chapter 2?

<figure>
    <img src="images/fig2.png" width="300" alt="Colored Scope Bubbles" align="center">
    <figcaption><em>Fig. 2 (Ch. 2): Colored Scope Bubbles</em></figcaption>
    <br><br>
</figure>

This exercise asks you to write a program—any program!—that contains nested functions and block scopes, which satisfies these constraints:

* If you color all the scopes (including the global scope!) different colors, you need at least six colors. Make sure to add a code comment labeling each scope with its color.

    BONUS: identify any implied scopes your code may have.

* Each scope has at least one identifier.

* Contains at least two function scopes and at least two block scopes.

* At least one variable from an outer scope must be shadowed by a nested scope variable (see Chapter 3).

* At least one variable reference must resolve to a variable declaration at least two levels higher in the scope chain.

| TIP: |
| :--- |
| You *can* just write junk foo/bar/baz-type code for this exercise, but I suggest you try to come up with some sort of non-trivial real'ish code that at least does something kind of reasonable. |

Try the exercise for yourself, then check out the suggested solution at the end of this appendix.

## 클로저 (PART 1)

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

이 연습의 첫 번째 부분은 클로저를 사용해서 `isPrime(..)`의 결과를 저장하는 캐시를 구현하여, 주어진 수의 소수 여부(`true` 또는 `false`)를 한 번만 계산하도록 하는 것이었다. 힌트: 우리는 이미 6장에서 `factorial(..)`로 이러한 종류의 캐싱을 보여주었다.

`factorize(..)`는 재귀적으로 구현되어 있는데, 이는 자기 자신을 반복해서 호출한다는 걸 의미한다. 즉, 같은 숫자의 소수 여부를 계산하기 위해 낭비하게 되는 호출이 많을 수 있다. 따라서, 연습의 두 번째 단계로 `factorize(..)`에도 동일한 클로저 캐시 기법을 사용하는 것이다.

`isPrime(..)`와 `factorize(..)`의 캐싱에는 하나의 스코프에 넣지 말고 분리된 클로저를 사용하라.

직접 연습하고 나서 이 부록의 끝에 있는 권장 답안을 확인하라.

### 메모리에 대하여

클로저 캐시 기법과 이것이 애플리케이션의 성능에 미치는 영향에 대해 간단히 설명하겠다.

반복되는 호출 결과를 저장하면 계산 속도가 (경우에 따라 극적으로) 향상된다는 것을 알 수 있다. 그러나 이렇게 클로저를 사용하는 것에는 매우 주의해야 할 트레이드오프가 있다.

그것은 바로 메모리이다. 클로저 캐시 기법은 필수적으로 (메모리의) 캐시를 무한히 늘리게 된다. 만약 문제의 함수를 수백만 번 호출하는데 대부분 고유한 입력값을 사용한다면, 매우 많은 메모리를 낭비하게 될 것이다. 이 기법은 분명히 비용을 들일 가치가 있긴 하지만, 일반적인 입력으로 반복될 가능성이 높은지 확인하고 나서 활용해야만 효율적이다.

대부분의 고유한 입력값으로 호출하게 되어 캐시가 본질적으로 전혀 유용하게 *사용되지* 않는 경우, 이 기법은 사용하기 부적절하다.

용량을 제한하는 LRU(least recently used) 캐시처럼 정교한 캐싱 방식을 사용하는 것도 좋은 방법이다. LRU는 최대 용량에 가까워지면 최근에 가장 적게 사용한 값을 메모리에서 제거한다.

이 때, 단점은 LRU가 그 자체의 구현이 쉽지 않다는 것이다. 제대로 최적화된 LRU를 사용하고 싶다면, 문제 상황에서 발생할 수 있는 모든 단점을 빠짐없이 알고 있어야 한다.

## 클로저 (PART 2)

이번 연습에서는 클로저로 값 토글러를 전달해주는 `toggle(..)` 유틸리티를 다시 정의한다.

하나 이상의 값(인자)을 `toggle(..)`에 전달하면 함수를 반환하게 된다. 반환된 함수는 전달 받은 모든 값을 한 번씩 순서대로 돌아가며 교체할 것이다.

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

`toggle(..)`에 값을 전달하지 않는 특이한 경우는 그다지 중요하지 않습니다. 이런 경우, 토글러 인스턴스를 항상 `undefined`로 반환하면 되기 때문이다.

직접 연습한 다음, 이 부록의 끝에 있는 권장 해결책을 확인하라.

## 클로저 (PART 3)

마지막 세 번째 연습에서는 기본적인 계산기를 구현해보자. `calculator()` 함수는 다음과 같은 함수 형태로 자체의 상태를 유지하는 계산기 인스턴스를 생성한다(`calc(..)`, below).

```js
function calculator() {
    // ..
}

var calc = calculator();
```

매번 `calc(..)`를 호출할때, 누른 계산기의 버튼을 표현하는 문자를 하나씩 전달한다. 좀 더 알기 쉽게 하기 위해, 계산기는 숫자(0~9), 산술 연산자(+, -, \*, /)와 "="의 입력만을 지원하도록 기능을 제한할 것이다. 연산자는 엽력한 순서대로만 처리하며, "( )"로 묶거나 연산자 우선순위를 고려하지 않는다.

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

이 `useCalc(..)`의 실질적인 사용법을 보면 항상 마지막에 문자 "="를 입력 받고 있다.

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
            // "."를 위한 자리를 예약
            maxDigits--;
            // "0"로 시작하여, 자리를 예약해야 하는가?
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

## Modules

This exercise is to convert the calculator from Closure (PART 3) into a module.

We're not adding any additional functionality to the calculator, only changing its interface. Instead of calling a single function `calc(..)`, we'll be calling specific methods on the public API for each "keypress" of our calculator. The outputs stay the same.

This module should be expressed as a classic module factory function called `calculator()`, instead of a singleton IIFE, so that multiple calculators can be created if desired.

The public API should include the following methods:

* `number(..)` (input: the character/number "pressed")
* `plus()`
* `minus()`
* `mult()`
* `div()`
* `eq()`

Usage would look like:

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

`formatTotal(..)` remains the same from that previous exercise. But the `useCalc(..)` helper needs to be adjusted to work with the module API:

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

Try the exercise for yourself, then check out the suggested solution at the end of this appendix.

As you work on this exercise, also spend some time considering the pros/cons of representing the calculator as a module as opposed to the closure-function approach from the previous exercise.

BONUS: write out a few sentences explaining your thoughts.

BONUS #2: try converting your module to other module formats, including: UMD, CommonJS, and ESM (ES Modules).

## Suggested Solutions

Hopefully you've tried out the exercises before you're reading this far. No cheating!

Remember, each suggested solution is just one of a bunch of different ways to approach the problems. They're not "the right answer," but they do illustrate a reasonable way to approach each exercise.

The most important benefit you can get from reading these suggested solutions is to compare them to your code and analyze why we each made similar or different choices. Don't get into too much bikeshedding; try to stay focused on the main topic rather than the small details.

### Suggested: Buckets of Marbles

The *Buckets of Marbles Exercise* can be solved like this:

```js
// RED(1)
const howMany = 100;

// Sieve of Eratosthenes
function findPrimes(howMany) {
    // BLUE(2)
    var sieve = Array(howMany).fill(true);
    var max = Math.sqrt(howMany);

    for (let i = 2; i < max; i++) {
        // GREEN(3)
        if (sieve[i]) {
            // ORANGE(4)
            let j = Math.pow(i,2);
            for (let k = j; k < howMany; k += i) {
                // PURPLE(5)
                sieve[k] = false;
            }
        }
    }

    return sieve
        .map(function getPrime(flag,prime){
            // PINK(6)
            if (flag) return prime;
            return flag;
        })
        .filter(function onlyPrimes(v){
            // YELLOW(7)
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

### Suggested: Closure (PART 1)

The *Closure Exercise (PART 1)* for `isPrime(..)` and `factorize(..)`, can be solved like this:

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

The general steps I used for each utility:

1. Wrap an IIFE to define the scope for the cache variable to reside.

2. In the underlying call, first check the cache, and if a result is already known, return.

3. At each place where a `return` was happening originally, assign to the cache and just return the results of that assignment operation—this is a space savings trick mostly just for brevity in the book.

I also renamed the inner function from `factorize(..)` to `findFactors(..)`. That's not technically necessary, but it helps it make clearer which function the recursive calls invoke.

### Suggested: Closure (PART 2)

The *Closure Exercise (PART 2)* `toggle(..)` can be solved like this:

```js
function toggle(...vals) {
    var unset = {};
    var cur = unset;

    return function next(){
        // save previous value back at
        // the end of the list
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

### Suggested: Closure (PART 3)

The *Closure Exercise (PART 3)* `calculator()` can be solved like this:

```js
// from earlier:
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
        // number key?
        if (/\d/.test(key)) {
            currentVal += key;
            return key;
        }
        // operator key?
        else if (/[+*/-]/.test(key)) {
            // multiple operations in a series?
            if (
                currentOper != "=" &&
                currentVal != ""
            ) {
                // implied '=' keypress
                pressKey("=");
            }
            else if (currentVal != "") {
                currentTotal = Number(currentVal);
            }
            currentOper = key;
            currentVal = "";
            return key;
        }
        // = key?
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
            // NOTE: using arrow functions
            // only for brevity in the book
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

| NOTE: |
| :--- |
| Remember: this exercise is about closure. Don't focus too much on the actual mechanics of a calculator, but rather on whether you are properly *remembering* the calculator state across function calls. |

### Suggested: Modules

The *Modules Exercise* `calculator()` can be solved like this:

```js
// from earlier:
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
        // number key?
        if (/\d/.test(key)) {
            currentVal += key;
            return key;
        }
    }

    function eq() {
        // = key?
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
        // multiple operations in a series?
        if (
            currentOper != "=" &&
            currentVal != ""
        ) {
            // implied '=' keypress
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
            // NOTE: using arrow functions
            // only for brevity in the book
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

That's it for this book, congratulations on your achievement! When you're ready, move on to Book 3, *Objects & Classes*.

[^MathJSisPrime]: *Math.js: isPrime(..)*, https://github.com/josdejong/mathjs/blob/develop/src/function/utils/isPrime.js, 3 March 2020.
