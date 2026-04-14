# 생성자 함수에 의한 객체 생성

## Object 생성자 함수

`new` 연산자와 함께 `Object` 생성자 함수를 호출하면 빈 객체를 만들 수 있다.

## 객체 생성 방법 비교

### 객체 리터럴

`{ key: value }` 문법으로 단일 객체를 생성할 수 있다. 하지만 같은 프로퍼티를 가지고 있는 여러 객체를 만들 때에는 반복하여 작성해야 한다.

### 생성자 함수

`new` 연산자와 함께 호출하여 객체(인스턴스)를 생성하는 함수.
`String`, `Number`, `Boolean`, `Function`, `Array`, `Date`, `Math` 등의 빌트인 생성자 함수를 제공한다.

> `this` > `this`는 객체 자신의 프로퍼티나 메서드를 참조하기 위한 자기 참조 변수다. 함수 호출 방식에 따라 동적으로 결정된다.

| 함수 호출 방식       | this가 가리키는 값(this 바인딩) |
| -------------------- | ------------------------------- |
| 일반 함수로서 호출   | 전역 객체 (`window`)            |
| 메서드로서 호출      | 메서드를 호출한 객체            |
| 생성자 함수로서 호출 | 생성자 함수가 생성할 인스턴스   |

```js
// 객체 리터럴 - 매번 반복해야 함
const obj1 = {
  name: "전희재",
};

const obj2 = {
  name: "진재윤",
};

// 생성자 함수 - 붕어빵 틀처럼 재사용 가능
function Person(name) {
  this.name = name;
  this.greeting = function () {
    console.log(`제 이름은 ${this.name}입니다`);
  };
}

const person1 = new Person("전희재");
const person2 = new Person("진재윤");
```

## 생성자 함수의 인스턴스 생성 과정

생성자 함수의 역할은,

- 생성자 함수는 붕어빵 틀처럼 구조가 동일한 인스턴스(객체)를 생성하기 위해
- 생성된 인스턴스(객체)를 초기화

`new` 연산자와 함께 생성자 함수를 호출하면
→ 암묵적으로 인스턴스(객체)를 생성
→ 인스턴스를 초기화
→ 인스턴스를 반환

### 1. 인스턴스 생성과 `this` 바인딩

생성자 함수를 `new`와 함께 호출하면 **자바스크립트 엔진에 의하여 암묵적으로 빈 객체가 생성**된다. 그리고 이 빈 객체가 **인스턴스**다. 그리고 이 인스턴스는 **`this`에 바인딩된다.**

```js
function Person(name) {
  console.log(this); // 1. Person {} <- this에 바인딩

  this.name = name;
  this.greeting = function () {
    console.log(`제 이름은 ${this.name}입니다.`);
  };
}
```

### 2. 인스턴스 초기화

생성자 함수에 담겨져 있는 코드가 한 줄씩 실행되며 `this`에 바인딩되어 있는 **인스턴스를 초기화한다.** 또, 생성자 함수가 인수로 전달받은 초기값을 인스턴스 프로퍼티에 할당하여 **초기화하거나 고정값에 할당한다.**

```js
function Person(name) {
  this.name = name; // 2. 인스턴스 초기화
  this.greeting = function () {
    console.log(`제 이름은 ${this.name}입니다.`);
  };
}
```

### 3. 인스턴스 반환 (this return)

생성자 함수 내부에서 모든 처리가 끝나면 바인딩된 `this`를 암묵적으로 `return`한다.
`this`를 반환하는 것이라면 `return`문을 생략해도 되지만, 임의적으로 다른 객체를 반환할 수 있다.
하지만 원시값 반환은 **암묵적으로 무시**되며 `this` 반환으로 대체된다.

- `this`가 암묵적으로 반환되는 경우

```js
function Person(name) {
  console.log(this); // 1. Person {} <- this에 바인딩

  this.name = name; // 2. 인스턴스 초기화
  this.greeting = function () {
    console.log(`제 이름은 ${this.name}입니다.`);
  };

  // return을 생략해도 암묵적으로 this가 반환된다
}

const person = new Person("고다솜");
console.log(person); // Person { name: "고대솜", greeting: f }
```

- `this`가 아닌 객체를 리턴하거나 원시값을 리턴하는 경우

```js
function Person(name) {
  console.log(this); // 1. Person {} <- this에 바인딩

  this.name = name; // 2. 인스턴스 초기화
  this.greeting = function () {
    console.log(`제 이름은 ${this.name}입니다.`);
  };

  return {}; // this가 아닌 객체를 리턴한다면 this 반환은 무시되고, 객체를 리턴한다
}

function Person(name) {
  console.log(this); // 1. Person {} <- this에 바인딩

  this.name = name; // 2. 인스턴스 초기화
  this.greeting = function () {
    console.log(`제 이름은 ${this.name}입니다.`);
  };

  return 100; // 원시값 반환은 무시되며, 기존처럼 this가 반환된다.
}
```

## 내부 메서드 `[[Call]]`과 `[[Construct]]`

**함수라고 해서 다 `new`로 호출할 수 있는 것은 아니다.** 자바스크립트에서 **함수는 객체**다. 일반 객체와 다른 점은 호출이 가능하다는 것인데, 이 호출이 가능함을 내부적으로 `[[Call]]`과 `[[Construct]]`로 일반 함수인지 생성자 함수인지 판별한다.

```js
function foo () {
  ...
}

foo() // 일반 함수로서 호출 -> [[Call]]이 호출

new foo() // 생성자 함수로서 호출 -> [[Construct]]가 호출
```

함수가 일반 함수로서 호출되면 `[[Call]]`이, 생성자 함수로서 호출되면 `[[Construct]]`가 호출된다.

`[[Construct]]`가 있으면 `constructor`, 가지고 있지 않으면 `non-constructor`라고 부른다.

`non-constructor`는 **화살표 함수**와 `{ method() {} }`와 같은 **메서드 축약형**이다. 즉, `new` 키워드로 생성자 함수를 만들 수 없다는 것이다.

```js
const arrow = () => {};
const obj = { method() {} };

new arrow(); // Error -> non-constructor
new obj.method(); // Error -> non-constructor
```

## `new.target`

생성자 함수를 `new` 없이 호출하는 실수를 방지하기 위한 메타 프로퍼티. `new`로 호출하면 `new.target`은 함수 자신, 일반 호출이면 `undefined`를 반환한다. 이를 통해 방어 코드를 작성할 수 있다.

```js
function Person(name) {
  if (!new.target) {
    return new Person(name); // 실수로 new를 빠뜨려도 재귀함수로 생성자 함수를 호출할 수 있게
  }

  this.name = name;
}
```

# 함수와 일급 객체

## 일급 객체를 충족하는 4가지 조건

**1. 무명 리터럴로 생성 가능, 런타임에 변수 할당 가능**

```js
const foo = function () {
  return "hello";
}; // 익명 함수 리터럴을 변수에 할당
```

**2. 다른 함수의 인수로 전달 가능**

```js
[1, 2, 3].map(function (number) {
  return number * 2;
});
```

**3. 다른 함수의 반환값으로 사용 가능**

```js
function makeAdder(x) {
  return function (y) {
    return x + y;
  }; // 함수를 반환
}

const add5 = makeAdder(5);
add(3); // 5 + 3 = 8
```

**4. 객체/배열 같은 자료구조에 저장 가능**

```js
const actions = {
  fn1: function () {},
  fn2: function () {},
};
```

## 함수 객체만의 고유 프로퍼티

일반 객체에는 없고 함수만 갖고 있는 프로퍼티다.

**1. `arguments`**

- 함수 내부에서만 접근 가능한 유사 배열 객체 (`map`, `reduce` 같은 배열 메서드 사용 불가)
- 매개변수의 개수를 모를 때 가변 인수 처리에 사용
- ES6 이후부터는 \*\*rest 파라미터(`...args`)가 권장

```js
function sum() {
  console.log(arguments); // Arguments [1, 2, 3]
  return Array.from(arguments).reduce((a, b) => a + b, 0); // 배열 메서드 사용 불가로 Array.from 사용
}

sum(1, 2, 3); // 6
```

**2. length**

- 선언한 매개변수의 개수를 알 수 있음
- 함수의 `length`와 `arguments`의 `length`는 다름

```js
function foo(a, b) {}
foo.length; // 2

function bar(a, b, c) {}
bar(1, 2);

bar.length; // 3
arguments.length; // 2 (실제로 전달된 인수 개수)
```

**3. `name`**

- 함수 이름
- ES5에서는 익명 함수는 빈 문자열로 되었지만, ES6에서는 변수명으로 추론해 줌

```js
const named = function foo() {};
named.name; // foo

const anon = function () {};
anon.name; // ES5에서는 '', ES6에서는 'anon'

const arrow = () => {};
arrow.name; // arrow
```

**4. `prototype`**

- `new` 연산자로 호출 가능한(constructor) 함수만 보유
- 생성된 인스턴스의 프로토타입 객체를 가르킴

```js
function Person(name) {
  this.name = name;
}
Person.prototype; // { constructor: f } <- 생성자 함수만 소유

const obj = {};
obj.prototype; // undefined <- 일반 객체는 없음

const arrowFn = () => {};
arrowFn.prototype; // undefined <- 화살표 함수도 없음 (non-constructor)
```
