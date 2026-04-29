# strict mode

## strict mode가 생겨난 이유

자바스크립트는 초기에 **개발 친화적인 이유**로 오류를 발생시킬 수 있는 것들을 **무시**하고는 했다. 이런 암묵적인 온상들이 추후 개발 성능이나 코드의 가독성을 저하시킬 수 있는 요인이 되어 **ES5부터 `strict mode`가 도입됐다.**

## strict mode가 제한하는 것

### 1. 암묵적인 전역 변수 생성 금지

```js
"use strict";

x = 10; // ReferenceError x is not defined
```

### 2. 변수, 함수, 매개변수에 `delete` 금지

```js
"use strict";

var x = 1;
delete x; // SyntaxError
```

### 3. 이름이 중복인 매개변수 금지

```js
"use strict";

function foo(x, x) {
  // SyntaxError
  return x;
}
```

### 4. `with`문 사용 금지

`with`는 스코프 체인을 교란해서 최적화를 방해하고 혼란을 일으켜 금지

## `this` 바인딩

`strict mode`에서는 일반 함수로 호출할 때 `this`는 `undefined`가 된다. 메서드로 쓰이려고 만든 함수를 실수로 일반 함수처럼 호출할 경우 의도하지 않게 전역 객체를 건드리는 버그를 방지해 주기 때문.

## strict mode 대신 ESLint를

- ES6 모듈 (import/export): 자동으로 strict mode
- 클래스 내부 코드: 자동으로 strict mode
- Vite, webpack 번들러: 보통 strict mode로 번들링
- ESLint: 'use strict' 사용을 강제하거나 이미 규칙으로 잡아줌
  -> 굳이 쓰지 않아도 된다.

# 빌트인 객체

## 객체의 종류

### 1. 표준 빌트인

- ECMAScript 표준
- 어디에서나 사용 가능
- Object, String, Number, Array, Math, Date, Promise...

### 2. 호스트 객체

- 실행 환경이 제공 (브라우저, Node)
- window, document, fetch, setTimeout...

### 3. 사용자 객체 정의

- 개발자가 직접 만듦
- class User {} / const obj = {} ...

## 래퍼 객체

`String`, `Number` 같은 표준 빌트인은 `new` 연산자 없이 사용 가능한데, 왜 `"hello".length()`가 가능한 것일까?

**1. 자바스크립트 엔진이 원시값을 감지**

```js
const str = "hello";
str.toUpperCase();
```

-> 원시값인데 프로퍼티에 접근하려고 하는 것을 JS가 감지

**2. 래퍼 객체를 자동 생성**

```js
const temp = new String("hello");
```

-> `String` 생성자로 임시 객체를 만들고, 이 객체 안에는 `length`, `slice`, `toUpperCase` 같은 메서드가 전부 들어 있음

**3. 메서드 실행**

```js
temp.toUpperCase(); // "HELLO"
```

-> `String.prototype`에 있는 메서드를 찾아서 실행

**4. 결과 반환 후 즉시 GC 대상이 됨**
임시로 만든 객체(temp)는 버리고 원래 있었던 `str`은 변형하지 않음

## 전역 객체

`window.parseInt('10')`에서 어떻게 `window`를 생략할 수 있을까?
**1. 브라우저가 켜지는 순간 전역 객체 생성**

```js
window = {
  parseInt: function() { ... },
  isNaN: function() { ... },
  console: { log: function() { ... } },
  document: { ... },
  // ...수백 개
}
```

자바스크립트가 실행되기 전에 브라우저가 `window`라는 객체를 만듦

**2. 코드에서 변수를 찾을 때 전역까지 뒤짐**

```js
parseInt("10");
```

엔진이 `parseInt`를 찾으러 스코프 체인을 타고 올라감 -> 지역 스코프에 없으면 전역까지 올라감 -> 전역의 끝이 `window`

**3. 그래서 `window`를 생략 가능**

```js
parseInt("10");
window.parseInt("10");
// 결과 동일
```

> `var`가 전역 객체게 붙는 것도 같은 맥락

```js
var a = 1;

// 엔진이 내부적으로
window.a = 1;

//그래서
console.log(window.a); // 1
```

하지만 `let`, `const`는 전역에 선언해도 `window`와 분리된 별도 공간에 저장8

## 빌트인 전역 함수

1. `parseFloat`

- `Number`가 더 범용적으로 쓰여서 잘 쓰이지 않지만
- CSS 값을 파싱할 때는 좋을 수 있다

```js
parseInt("42px"); // 42
```

2. `encodeURL` vs `encodeURLComponent`
   한글, 공백, 특수문자가 URL에 들어갈 수 없어(서버가 못 읽음) 안전한 형태로 변환

```js
const url = "https://example.com/name=하늘";
// URL 전체 인코딩
encodeURL(url); // 'https://example.com/%EC%9D%B4%EB%A6%84=%ED%95%98%EB%8A%98'

// = 포함 전체 인코딩
encodeURIComponent("이름=하늘"); // '%EC%9D%B4%EB%A6%84%3D%ED%95%98%EB%8A%98'

// 실무 패턴
const keyword = "아이폰 15 pro";
fetch(`/api/search?q=${encodeURIComponent(keyword)}`);

// 디코딩은 decodeURIComponent로
decodeURIComponent("%EC%9D%B4%EB%A6%84"); // '이름'
```

# `this`

## 생겨난 이유

- 객체는 상태(프로퍼티)와 동작(메서드)을 하나의 논리적 단위로 묶은 자료구조.
- 메서드가 자신이 속한 객체의 프로퍼티를 참조하려면 자신이 속한 객체를 가리키는 식별자를 참조할 수 있어야 한다

-> `this`는 자신이 속한 객체, 또는 자신이 생성할 인스턴스를 가리키는 **자기 참조 변수**

## `this`의 특징

- 자바스크립트 엔진에 의해 암묵적으로 생성
- 코드 어디에서든지 참조 가능
- `this` 바인딩은 **함수 호출 방식에 의해 동적으로 결정**
- `strict mode` 또한 `this` 바인딩에 영향을 준다

## 함수 호출 방식에 따른 `this` 바인딩

| 호출 방식           | this 바인딩                   |
| ------------------- | ----------------------------- |
| 일반 함수 호출      | 전역 객체 (window / global)   |
| 메서드 호출         | 메서드를 호출한 객체          |
| 생성자 함수 호출    | 생성자 함수가 생성할 인스턴스 |
| apply / call / bind | 첫 번째 인수로 전달한 객체    |

### 1. 일반 함수 호출

```js
function foo() {
  console.log(this); // window (전역 객체)
}

foo();
```

- strict mode에서는 `undefined`
- 콜백 함수, 중첩 함수도 **일반 함수로 호출되면 `this`는 전역 객체**

메서드 내부의 중첩 함수나 콜백 함수의 `this`가 전역 객체를 가리키는 문제는 아래처럼 해결할 수 있다.

```js
const obj = {
  value: 100,
  foo() {
    // ① this를 변수에 저장
    const that = this;
    setTimeout(function () {
      console.log(that.value); // 100
    }, 100);

    // ② 화살표 함수 사용 (상위 스코프의 this를 그대로 사용)
    setTimeout(() => console.log(this.value), 100); // 100
  }
};
```

### 2. 메서드 호출

```js
const person = {
  name: 'Lee',
  getName() {
    return this.name; // this === 메서드를 호출한 객체
  }
};

console.log(person.getName()); // 'Lee'
```

`this`는 메서드를 **소유한 객체**가 아니라 메서드를 **호출한 객체**에 바인딩된다.

```js
const anotherPerson = { name: 'Kim' };
anotherPerson.getName = person.getName;

console.log(anotherPerson.getName()); // 'Kim' — 호출한 객체가 달라졌기 때문
```

### 3. 생성자 함수 호출

```js
function Circle(radius) {
  this.radius = radius; // this === 생성자 함수가 생성할 인스턴스
}

const c1 = new Circle(5);
console.log(c1.radius); // 5
```

`new` 없이 호출하면 일반 함수처럼 동작해서 `this`가 전역 객체를 가리키게 된다.

### 4. `apply` / `call` / `bind`

`Function.prototype`의 메서드로, `this`를 명시적으로 지정할 수 있다.

**`apply` / `call`** — `this`를 지정하면서 함수를 즉시 호출

```js
function greet(greeting) {
  console.log(`${greeting}, ${this.name}`);
}

const user = { name: 'Lee' };

greet.apply(user, ['Hello']); // Hello, Lee  (인수를 배열로 전달)
greet.call(user, 'Hello');    // Hello, Lee  (인수를 쉼표로 구분해 전달)
```

**`bind`** — 함수를 호출하지 않고 `this`가 바인딩된 새 함수를 반환

```js
const boundGreet = greet.bind(user);
boundGreet('Hi'); // Hi, Lee
```

메서드 내부 콜백 함수의 `this` 불일치 문제를 해결할 때 유용하다.

```js
const obj = {
  value: 100,
  foo() {
    setTimeout(function () {
      console.log(this.value); // 100
    }.bind(this), 100);
  }
};
```
