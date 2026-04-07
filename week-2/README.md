# 원시값과 객체 비교

## 원시값

- **원시값은 불변성을 띈다.**
  재할당으로 변수가 가리키는 메모리 주소는 변경할 수 있지만, 메모리 주소에 있는 값 자체를 변경할 수 없다.
  - `var foo` 선언
    | 메모리 주소 1 | |
    |----------|--------------------------|
    | 메모리 주소 2 | `undefined` // ← 여기에 초기화 |
    | 메모리 주소 3 | |

  - `foo = 123` 할당
    | 메모리 주소 2 | `undefined` |
    |----------|------------------------|
    | 메모리 주소 3 | |
    | 메모리 주소 4 | 123 ← `foo`가 이걸 가르키게 됨 |

  - `foo = 456` 재할당
    | 메모리 주소 2 | `undefined` |
    |----------|--------------------|
    | 메모리 주소 3 | |
    | 메모리 주소 4 | 123 |
    | 메모리 주소 5 | |
    | 메모리 주소 6 | 456 ← 재할당으로 이걸 가리킴 |

- 문자열도 마찬가지로 읽기 전용이다.

```js
var str = "재윤";
str[0] = "힝";

console.log(str); // '재윤'
```

### 값에 의한 전달

```js
var original = 120;
var copy = original;

console.log(original, copy); // 120 120
console.log(original === copy); // true
```

`original` 변수에 120 이라는 값을 할당하고, `copy` 변수에 `original` 변수를 할당했다. 먼저 값을 `console.log` 로 출력했을 때 같은 값 `120` 이 나오고, 비교를 했을 때도 같은 `120` 이니 같은 값이라고 나온다.

<img width="613" height="559" alt="Image" src="https://github.com/user-attachments/assets/f0e1f905-e828-4a3c-ab55-a2cec1b6a616" />

대신에 `copy` 변수의 메모리 주소에는 `original` 변수가 담기지 않고 `120` 이라는 값이 담긴다. 앞서 설명한 것처럼 원시값을 **불변값**이기 **때문이다.** 만약 `copy` 의 메모리 주소에 `original` 변수를 담았으면, `original` 값이 바뀔 때마다 `copy` 변수 값도 바뀌므로 그것은 불변성에 위배되는 것이다.

`original` 이 재할당을 하면 기존에 있던 메모리 주소 2에 400이라는 값이 덮어씌우는 게 아니라 다른 메모리 주소에 400이라는 값이 주어지므로 `copy` 와 `original` 은 다른 주소를 가리키고 있게 된다.

```js
var original = 120;
var copy = original;

original = 400;

console.log(copy); // 400이 아니라 200
console.log(original === copy); // false
```

<img width="602" height="583" alt="Image" src="https://github.com/user-attachments/assets/3ad3ac09-812d-45ba-8f36-673a9318a264" />

## 객체 (참조에 의한 전달)

객체는 원시값과 달리 변할 수 있다. 값을 변경할 수도 있고, 추가할 수도, 심지어 삭제할 수도 있다. 불변성이 아니라 **가변성(mutable value)**이다.

변수에 원시값을 할당하면 메모리 주소에 값 자체가 들어가지만, 객체를 할당하면 값이 아닌 또다른 메모리 주소(= 참조값)가 들어간다. 따라서 **객체를 할당한 변수는 재할당 없이 객체를 직접 변경할 수 있다.** (주소는 그대로, 주소가 가리키는 값만 바꾸면 되니까)

<img width="831" height="601" alt="Image" src="https://github.com/user-attachments/assets/e1a910f7-57c1-486c-91a3-a7d4673a5c9f" />

<img width="823" height="621" alt="Image" src="https://github.com/user-attachments/assets/07a3f355-83cf-4af1-a72e-ba205036d558" />

하지만 이런 객체는 치명적인 단점이 있다. 여러 개의 식별자가 하나의 객체를 공유할 수도 있다는 점이다.

<img width="681" height="678" alt="Image" src="https://github.com/user-attachments/assets/2dc154b1-c0ea-4108-a733-b803bf25bcc6" />

```js
obj1.isGirl = false;

console.log(obj2.isGirl); // false
```

같은 메모리 주소를 가리키고, 그 메모리 주소는 또 같은 메모리 주소(객체가 담긴)를 보고 있기 때문에 값이 공유가 된다.

그래서 얕은 복사, 깊은 복사라는 게 있다.

### 얕은 복사

객체를 복사한다. 대신 **얕은**이라는 말처럼 1 depth까지만 복사한다.

```js
const obj = {
  name: "재윤",
  inner: {
    isGirl: false,
  },
};

const obj2 = { ...obj };

obj2.name = "연욱";
obj2.inner.isGirl = true;

console.log(obj.name); // "재윤" -> obj와 obj2는 개별적 (복사한 거니까)
console.log(obj.inner.isGirl); // true -> 얕은 복사라서 중첩 객체까지는 복사가 안 되고 공유
```

### 깊은 복사

중첩된 객체까지 모두 복사한다. (= 개별적으로 작동한다)

```js
const _ = require("lodash");

const obj3 = _.cloneDeep(obj);

console.log(obj === obj3); // false (개별적)
```
