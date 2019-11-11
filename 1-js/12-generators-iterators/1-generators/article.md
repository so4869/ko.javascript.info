# 제너레이터

일반 함수는 하나의 값(혹은 0개의 값)만을 반환합니다.

하지만 제너레이터(generator)를 사용하면 여러 개의 값을 필요에 따라 차례 차례 반환(yield)할 수 있습니다. 제너레이터와 [iterable](info:iterable)을 함께 사용하면 손쉽게 데이터 스트림을 만들 수 있습니다.

## 제너레이터 함수

제너레이터를 만들려면 '제너레이터 함수'라 불리는 특별한 문법 구조, `function*`이 필요합니다.

예시:

```js
function* generateSequence() {
  yield 1;
  yield 2;
  return 3;
}
```

제너레이터 함수는 일반 함수와 동작 방식이 다릅니다. 제너레이터 함수를 호출하면 코드가 실행되지 않습니다. 대신 실행 처리를 담당하는 '제너레이터 객체'라 불리는 특별한 객체를 반환합니다. 

예시를 살펴봅시다.

```js run
function* generateSequence() {
  yield 1;
  yield 2;
  return 3;
}

// '제너레이터 함수'는 '제너레이터 객체'를 생성합니다.
let generator = generateSequence();
*!*
alert(generator); // [object Generator]
*/!*
```

함수를 호출해도 함수 본문의 코드는 아직 실행되지 않습니다.

![](generateSequence-1.svg)

`next()`는 제너레이터의 주요 메서드입니다. `next()`를 호출하면 가장 가까운 `yield <value>`문을 만날 때까지 실행이 지속됩니다(`value`를 생략할 수도 있는데, 이런 경우엔 `undefined`가 됩니다). `yield <value>`문을 만나면 실행이 멈추고 산출하고자 하는(yielded) 값인 `value`가 바깥 코드에 반환됩니다.

`next()`의 결과는 항상 객체입니다. 이 객체엔 반드시 아래 두 프로퍼티가 있습니다.
- `value`: 산출 값
- `done`: 함수 코드 실행이 끝났으면 `true`, 아니라면 `false`

아래 예시에선 제너레이터를 만들고 첫 번째 산출 값을 받습니다. 

```js run
function* generateSequence() {
  yield 1;
  yield 2;
  return 3;
}

let generator = generateSequence();

*!*
let one = generator.next();
*/!*

alert(JSON.stringify(one)); // {value: 1, done: false}
```

현재로서는 첫 번째 값만 받았으므로 함수 실행은 두 번째 줄에 있는 상태입니다.

![](generateSequence-2.svg)

`generator.next()`를 다시 호출해봅시다. 코드 실행이 다시 시작되고 다음 `yield`를 반환합니다.

```js
let two = generator.next();

alert(JSON.stringify(two)); // {value: 2, done: false}
```

![](generateSequence-3.svg)

`generator.next()`를 세 번째 호출하면 실행은 `return`문에 다다르고 함수 실행이 종료됩니다.

```js
let three = generator.next();

alert(JSON.stringify(three)); // {value: 3, *!*done: true*/!*}
```

![](generateSequence-4.svg)

이제 제너레이터가 종료되었습니다. 마지막 결과인 `value:3`과 `done:true`를 통해 이를 확인할 수 있습니다. 

제너레이터가 종료되었기 때문에 지금 상태에선 `generator.next()`를 호출해도 아무 소용이 없습니다. `generator.next()`를 호출하면 동일한 객체, `{done: true}`가 반환됩니다.  

```smart header="`function* f(…)` 혹은 `function *f(…)`?"
두 문법 맞습니다.

그런데 `*`가 제너레이터 함수라는 것을 나타내므로 대개는 첫 번째 문법이 선호됩니다. `*`는 종류를 나타는 것이지 이름을 나타내는것이 아닙니다. 그러므로 `*`을 `function` 키워드에 붙이도록 합시다.  
```

## 제너레이터와 이터러블

`next()` 메서드를 보면서 짐작하셨듯이, 제너레이터는 [이터러블](info:iterable)입니다.

`for..of` 반복문을 사용해 값을 얻을 수 있죠. 

```js run
function* generateSequence() {
  yield 1;
  yield 2;
  return 3;
}

let generator = generateSequence();

for(let value of generator) {
  alert(value); // 1, 2
}
```

`.next().value`을 호출해 산출값을 얻는 것 보단 `for..of`를 사용하는게 더 나아보이네요.

그런데 위 예시를 실행하면 `1`과 `2`만 출력됩니다. 그게 다이죠. `3`은 출력되지 않습니다.

이유는 `for..of` 반복은 `done: true`일 때, 마지막 `value`를 무시하기 때문입니다. 그러므로 `for..of`를 사용했을 때 모든 값이 출력되길 원한다면 `yield`로 값을 반환해야합니다. 

```js run
function* generateSequence() {
  yield 1;
  yield 2;
*!*
  yield 3;
*/!*
}

let generator = generateSequence();

for(let value of generator) {
  alert(value); // 1, 2, 3
}
```

제너레이터는 이터러블이므로 제너레이터에도 전개 연산자(`...`)같은 이터러블 관련 기능을 모두 사용할 수 있습니다.

```js run
function* generateSequence() {
  yield 1;
  yield 2;
  yield 3;
}

let sequence = [0, ...generateSequence()];

alert(sequence); // 0, 1, 2, 3
```

위 예시에서 `...generateSequence()`는 이터러블 제너레이터 객체를 배열 요소로 바꿔줍니다. 전개 연산자에 대한 자세한 설명은 [](info:rest-parameters-spread-operator#spread-operator)에서 보실수 있습니다.

## 이터러블 대신 제너레이터 사용하기

[](info:iterable)를 다룬 챕터에서 `from..to` 사이에 있는 값을 반환하는 이터러블 객체인 `range`를 만들어 보았습니다.

코드는 아래와 같았죠.

```js run
let range = {
  from: 1,
  to: 5,

  // for..of 최초 호출 시, Symbol.iterator가 호출됩니다.
  [Symbol.iterator]() {
    // Symbol.iterator는 이터레이터 객체를 반환합니다.
    // for..of는 반환된 이터레이터 객체만을 대상으로 동작하는데, 이때 다음 값도 정해집니다.
    return {
      current: this.from,
      last: this.to,

      // for..of 반복문에 의해 각 반복마다 next()가 호출됩니다.
      next() {
        // next()는 값을 객체 {done:.., value :...}형태로 반환해야 합니다.
        if (this.current <= this.last) {
          return { done: false, value: this.current++ };
        } else {
          return { done: true };
        }
      }
    };
  }
};

// range.from부터 range.to까지의 숫자가 출력됩니다.
alert([...range]); // 1,2,3,4,5
```

제너레이터 함수를 `Symbol.iterator`처럼 사용하면 이터레이션에 제너레이터 함수를 사용할 수 있습니다.

좀 더 압축된 `range`를 살펴봅시다.

```js run
let range = {
  from: 1,
  to: 5,

  *[Symbol.iterator]() { // [Symbol.iterator]: function*()를 짧게 줄임
    for(let value = this.from; value <= this.to; value++) {
      yield value;
    }
  }
};

alert( [...range] ); // 1,2,3,4,5
```

`range[Symbol.iterator]()`는 제너레이터를 반환하고, 제너레이터 메서드는 `for..of`가 제대로 동작하는데 필요한 사항(아래 설명)을 충족시켜주므로 예시가 잘 동작합니다.
- `.next()` 메서드가 있음
- 반환 값의 형태는 `{value: ..., done: true/false}`이어야 함

이렇게 이터러블 대신 제너레이터를 사용할 수 있는것은 우연이 아닙니다. 제너레이터는 이터레이터를 쉽게 구현하는 것을 염두하며 자바스크립트에 추가되었습니다. 

제너레이터를 사용해 구현한 예시는 이터러블을 사용해 구현한 예시보다 더 간결하고 동일한 기능을 제공합니다.

```smart header="Generators may generate values forever"
In the examples above we generated finite sequences, but we can also make a generator that yields values forever. For instance, an unending sequence of pseudo-random numbers.

That surely would require a `break` (or `return`) in `for..of` over such generator. Otherwise, the loop would repeat forever and hang.
```

## 제너레이터 컴포지션

제너레이터 컴포지션(generator composition) is a special feature of generators that allows to transparently "embed" generators in each other.

먼저, 연속된 숫자를 제너레이트(생성)하는 제너레이터 함수를 만들어봅시다. 

```js
function* generateSequence(start, end) {
  for (let i = start; i <= end; i++) yield i;
}
```

그리고 위 함수를 기반으로 좀 더 복잡한 값을 연속해서 생성하는 함수를 만들어봅시다. 값 생성 규칙은 다음과 같습니다.  
- 처음엔 숫자 `0`부터 `9`를 생성합니다(문자 코드 48부터 57까지),
- 이어서 알파벳 대문자 `A`부터 `Z`까지를 생성합니다(문자 코드 65부터 90까지).
- 이어서 알파벳 소문자 `a`부터 `z`까지를 생성합니다(문자 코드 97부터 122까지).

위와 같은 규칙을 충족하는 연속 값은 비밀번호를 만들 때 응용할 수 있습니다(특수 문자도 추가 가능). 

일반 함수를 사용해 원하는 기능을 구현하려면 여러개의 함수를 만들고 그 호출 결과를 어딘가에 저장한 후 다시 그 결과들을 조합해야 할 겁니다.

하지만 제너레이터의 특수 문법 `yield*`를 사용하면 제너레이터를 다른 제너레이터에 '임베딩(embeding, composing)'할 수 있습니다.

컴포지션을 적용한 제너레이터는 다음과 같습니다.

```js run
function* generateSequence(start, end) {
  for (let i = start; i <= end; i++) yield i;
}

function* generatePasswordCodes() {

*!*
  // 0..9
  yield* generateSequence(48, 57);

  // A..Z
  yield* generateSequence(65, 90);

  // a..z
  yield* generateSequence(97, 122);
*/!*

}

let str = '';

for(let code of generatePasswordCodes()) {
  str += String.fromCharCode(code);
}

alert(str); // 0..9A..Za..z
```

`yield*` 지시자는 실행을 다른 제너레이터에 *위임합니다(delegate)*. 여기서 '위임'은 `yield* gen`이 제너레이터 `gen`을 대상으로 반복을 수행하고 그 결과를 바깥으로 전달한다는 것을 의미합니다. 따라서 마치 외부 제너레이터에 의해 값이 산출된 것처럼 동작합니다.

중첩 제너레이터를 사용해 코드를 안에 작성해도 결과는 같습니다.

```js run
function* generateSequence(start, end) {
  for (let i = start; i <= end; i++) yield i;
}

function* generateAlphaNum() {

*!*
  // yield* generateSequence(48, 57);
  for (let i = 48; i <= 57; i++) yield i;

  // yield* generateSequence(65, 90);
  for (let i = 65; i <= 90; i++) yield i;

  // yield* generateSequence(97, 122);
  for (let i = 97; i <= 122; i++) yield i;
*/!*

}

let str = '';

for(let code of generateAlphaNum()) {
  str += String.fromCharCode(code);
}

alert(str); // 0..9A..Za..z
```

제너레이터 컴포지션은 한 제너레이터의 흐름을 다른 제너레이터에 삽입하는 자연스러운 방법입니다. 제너레이터 컴포지션을 사용하면 중간 결과 저장 용도의 추가 메모리가 필요하지 않습니다.

## "yield" is a two-way road

Until this moment, generators were similar to iterable objects, with a special syntax to generate values. But in fact they are much more powerful and flexible.

That's because `yield` is a two-way road: it not only returns the result outside, but also can pass the value inside the generator.

To do so, we should call `generator.next(arg)`, with an argument. That argument becomes the result of `yield`.

Let's see an example:

```js run
function* gen() {
*!*
  // Pass a question to the outer code and wait for an answer
  let result = yield "2 + 2 = ?"; // (*)
*/!*

  alert(result);
}

let generator = gen();

let question = generator.next().value; // <-- yield returns the value

generator.next(4); // --> pass the result into the generator  
```

![](genYield2.svg)

1. The first call `generator.next()` is always without an argument. It starts the execution and returns the result of the first `yield "2+2=?"`. At this point the generator pauses the execution (still on that line).
2. Then, as shown at the picture above, the result of `yield` gets into the `question` variable in the calling code.
3. On `generator.next(4)`, the generator resumes, and `4` gets in as the result: `let result = 4`.

Please note, the outer code does not have to immediately call`next(4)`. It may take time. That's not a problem: the generator will wait.

For instance:

```js
// resume the generator after some time
setTimeout(() => generator.next(4), 1000);
```

As we can see, unlike regular functions, a generator and the calling code can exchange results by passing values in `next/yield`.

To make things more obvious, here's another example, with more calls:

```js run
function* gen() {
  let ask1 = yield "2 + 2 = ?";

  alert(ask1); // 4

  let ask2 = yield "3 * 3 = ?"

  alert(ask2); // 9
}

let generator = gen();

alert( generator.next().value ); // "2 + 2 = ?"

alert( generator.next(4).value ); // "3 * 3 = ?"

alert( generator.next(9).done ); // true
```

The execution picture:

![](genYield2-2.svg)

1. The first `.next()` starts the execution... It reaches the first `yield`.
2. The result is returned to the outer code.
3. The second `.next(4)` passes `4` back to the generator as the result of the first `yield`, and resumes the execution.
4. ...It reaches the second `yield`, that becomes the result of the generator call.
5. The third `next(9)` passes `9` into the generator as the result of the second `yield` and resumes the execution that reaches the end of the function, so `done: true`.

It's like a "ping-pong" game. Each `next(value)` (excluding the first one) passes a value into the generator, that becomes the result of the current `yield`, and then gets back the result of the next `yield`.

## generator.throw

As we observed in the examples above, the outer code may pass a value into the generator, as the result of `yield`.

...But it can also initiate (throw) an error there. That's natural, as an error is a kind of result.

To pass an error into a `yield`, we should call `generator.throw(err)`. In that case, the `err` is thrown in the line with that `yield`.

For instance, here the yield of `"2 + 2 = ?"` leads to an error:

```js run
function* gen() {
  try {
    let result = yield "2 + 2 = ?"; // (1)

    alert("The execution does not reach here, because the exception is thrown above");
  } catch(e) {
    alert(e); // shows the error
  }
}

let generator = gen();

let question = generator.next().value;

*!*
generator.throw(new Error("The answer is not found in my database")); // (2)
*/!*
```

The error, thrown into the generator at line `(2)` leads to an exception in line `(1)` with `yield`. In the example above, `try..catch` catches it and shows it.

If we don't catch it, then just like any exception, it "falls out" the generator into the calling code.

The current line of the calling code is the line with `generator.throw`, labelled as `(2)`. So we can catch it here, like this:

```js run
function* generate() {
  let result = yield "2 + 2 = ?"; // Error in this line
}

let generator = generate();

let question = generator.next().value;

*!*
try {
  generator.throw(new Error("The answer is not found in my database"));
} catch(e) {
  alert(e); // shows the error
}
*/!*
```

If we don't catch the error there, then, as usual, it falls through to the outer calling code (if any) and, if uncaught, kills the script.

## Summary

- Generators are created by generator functions `function* f(…) {…}`.
- Inside generators (only) there exists a `yield` operator.
- The outer code and the generator may exchange results via `next/yield` calls.

In modern JavaScript, generators are rarely used. But sometimes they come in handy, because the ability of a function to exchange data with the calling code during the execution is quite unique. And, surely, they are great for making iterable objects.

Also, in the next chapter we'll learn async generators, which are used to read streams of asynchronously generated data (e.g paginated fetches over a network) in `for await ... of` loops.

In web-programming we often work with streamed data, so that's another very important use case.
