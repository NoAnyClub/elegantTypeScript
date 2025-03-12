## 객체 타입
앞에서 언급한 7가지 타입에 포함되지 않는 값들은 모두 객체 타입으로 분류할 수 있음.
js에서는 이런 방대한 값들을 모두 `객체`로 퉁치지만, ts에서는 개별적으로 타입을 지정할 수 있음.

<br />

### 1. `object`
js 객체의 정의에 맞게끔 이에 대응하는 ts의 타입 시스템은 `object`타입임.
그러나 object타입은 가급적 사용하지 말도록 권장 됨. **object는 객체에 해당하는 모든 값들을 할당할 수 있어서 정적 타이핑의 의미가 퇴색되기 때문임.**

<img src="../../assets/CH02/object type.jpeg" width='500px' alt='object타입 이미지'/>

<br />

### 2. `{}`
객체를 타이핑 할때 사용하는 방식으로, 중괄호 안에 객체의 프로퍼티를 지정해주는 식으로 사용함.
```ts
const obj: {
  name: string; age: number;
} = {
  name: 'seongho', age: 27,
};

const emptyObj: {} = { name: 'seongho' }; // Error!
```
그냥 `{}`타입으로 타입을 지정하면 해당 객체는 어떤 프로퍼티도 가질 수 없음. 사실 빈 객체 타입을 지정 해줄때는 `{}`보다는 `Record<string, never>`가 더 권장되는 방식임.

<br />

### 3. `array`
ts에서는 js의 객체를 세분화 해서 타입을 지정할 수 있는 타입 시스템을 가지고 있다고 했음.
js에서는 객체 외에도 `배열`, `함수`, `정규표현식` 등이 객체 범주에 속함. 그러나 ts에서는 이런 각각의 객체에 타입을 지정할 수 있음.<br />
js에서는 배열에 원소를 자유롭게 추가/제거 할수 있으며, 타입 제한 없이 다양한 값을 다룸.
```js
const arr = [1, 'srt', () => {}];
```
근데 이런 특징은 ts가 추구하는 `정적 타이핑`과는 방향이 맞지 않음.<br />
ts에서는 배열을 array라고 하는 별도의 타입으로 구분함. ts에서의 array는 하나의 타입 값만 가질 수 있지만, 원소의 갯수는 상관이 없음.
선언하는 방식은 Array키워드로 선언하거나, 대괄호(`[]`)를 사용해서 선언할 수 있음.
```ts
const numArray: number[] = [1, 2, 3, 4, 5];
const numArray2: number[] = []; // 빈 배열도 가능
const strArray: Array<string> = ['a', 'b', 'c'];
const strArray2: Array<string> = ['a', 'b', 1]; // Error!
```

> [!TIP]
> **튜플 타입** <br />
> ts에서는 튜플 타입이라는게 존재하고, 튜플 타입 또한 대괄호를 사용해서 선언 함. <br />
> 튜플은 선언 시점에 지정해준 타입 값을 선언시점에 지정한 원소의 갯수 만큼만 할당할 수 있음<br />
> ```ts
> const tuple: ['A', 'B'] = ['A', 'B'];
> const tuple2: ['A', 'B'] = ['A', 'B', 'C']; // Error!
> ```

<br />

### 4. `type`과 `interface` 키워드
ts에서 객체를 타이핑 하기 위해서는 ts의 독자적인 키워드를 사용하는게 일반적임. 흔히 객체를 타이핑 하기 위해서 자주 사용되는 키워드는 `type`과 `interface`가 있음.
```ts
type Person = {
  name: string;
  age: number;
};

interface IPerson {
  name: string;
  age: number;
}

const person1: Person = { name: 'seongho', age: 27 };
const person2: IPerson = { name: 'seongho2', age: 27 };
```

<br />

### 5. `function`
js에서는 함수도 객체로 취급하지만, `typeof` 연산자를 통해서 함수는 `function`이라는 별도 타입으로 분류 되는것을 알 수 있음.
```js
typeof function() {}; // 'function'
```
ts에서도 함수를 별도의 타입으로 지정할 수 있음. 이때 주의할 점이 몇가지 있는데
1. **js에서 `typeof` 연산자로 확인한 `'function'` 이라는 키워드를 타입으로 사용하지 않는 다는 것.**
2. **매개변수도 별도 타입으로 지정해야 한다는 것.**
3. **함수가 반환하는 값이 있다면 반환값에 대한 타이핑도 필요함.**

```ts
function add(a: number, b: number): number {
  return a + b;
}
```

<br />

만약, 함수 자체의 타입을 지정하고 싶다면 `호출 시그니처`를 정의해야 함.
_(타입 영역에서의 `typeof` 키워드로 함수의 타입을 받았을 때 반환하는 것이 `호출 시그니처`임)_
<img src="../../assets/CH02/typeof function.jpeg" width="400px" alt="호출 시그니처 예시 사진" />

<br />

```ts
type Add = (a: number, b: number) => number; // 호출 시그니처

const add2:Add = (c, d) => {
  return c + d; // ✅
};

const add3: Add = (e, f) => {
  return e + "b"; // Error!!
}

```
