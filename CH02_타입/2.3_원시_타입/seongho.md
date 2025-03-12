## 원시 타입
자바스크립트의 7가지 원시 값은 타입스크립트에서 원시 타입으로 존재함.

> [!NOTE]
> **_원시 값과 원시 래퍼 객체_** <br />
> js의 내장타입은 Pascal표기법으로 표기했음. 반면에 ts에서는 타입을 소문자로 표기함<br />
> 타입을 Pascal표기법으로 표시하면 js에서 이것을 원시 래퍼 객체라고 인식함.

<br />

### 1. `boolean`
```ts
const isEmpty: boolean = true;
const isLoading: boolean = false;

function isTextError(errorCode: errorCodeType): boolean {
  const errorAction = getErrorAction(erroCode);

  if (errorAction) {
    return errorAction.type === ERROR_TEXT;
  }

  return false;
}
```
오직 `true`와 `false`만 할당 가능한 `boolean`타입. js에서는 원시값은 아니지만 형 변환을 통해 true / false로 취급되는 `truthy`, `falsy`값이 존재하는데, 이들은 엄연히 `boolean`타입이 아님.

<br />

### 2. `undefined` <br />
```ts
let name: string;

type Person = {
  name: string;
  age?: number;
}
```
정의되지 않았다는 의미의 타입으로 오직 `undefined`만 할당할 수 있는 타입.<br />
`Person`타입의 `age`속성은 옵셔널로 지정되어져 있는데, 이런 경우에도 `undefined`를 할당할 수 있음.

<br />

### 3. `null`
```ts
let value: null;

value = null;
```
오직 `null`만 할당가능한 타입으로 `undefined`와 혼동하면 안됨.<br />
비슷해 보여도 엄연히 따로 존재하는 원시 값이기 때문에 서로의 타입에 할당할 수 없음. `undefined`는 값이 있을수도 있고, 없을수도 있음을 나타내기도 하지만, `null`은 명시적으로 값이 없다고 알려주는 의미가 강함.

<br />

### 4. `number`
```ts
const age: number = 27;
const min: number = 10;
const max: number = 12;
const infinity: number = +Infinity;
const notANumber: number = NaN;
```
숫자에 해당하는 모든 원시값을 할당할 수 있는 타입.<br />
Java와 같은 언어에서는 `byte`, `short`, `int`, `long`, `double`, `float등` 다양한 숫자 타입으로 구분되어져 있지만 js는 정수, 부동소수점을 구분하지 않기때문에 모두 `number`타입에 할당할 수 있음.<br />

> [!CAUTION]
> **`NaN`이나 `Infinity`도 포함된다는 것에 주의하자**

<br />

### 5. `bigInt`
```ts
const bigNumber: bigInt = BigInt(9999999999);
const bigNumber2: bigInt = 9999999999999n;
```
`ES2020`부터 도입된 데이터 타입으로 **ts 3.2버전**부터 사용이 가능함. <br />
`number`타입과는 엄연히 다른 타입이므로 호환은 불가능함.

<br />

### 6. `string`
```ts
const name: string = 'seongho';
const location: sting = 'goyang';
```
문자열을 할당할 수 있는 타입으로, 공백도 `string`타입에 해당 함.

<br />

### 7. `symbol`
```ts
const TITLE: symbol = Symbol('TITLE');
const TITLE2: symbol = Symbol('TITLE');

TITLE === TITLE2; // false
```
`ES2015`부터 도입된 데이터 타입으로 `Symbol()`을 사용하면 어떤 값과도 중복되지 않는 유일한 값을 생성할 수 있음.<br />
예시처럼 동일한 문자열을 넘겨줬음에도 불구하고, 다른 값을 갖고있음. ts에는 `symbol`과 `const`선언에서만 사용 가능한 `unique symbol`타입이라는 symbol의 하위타입도 존재함.

<br />

ts에서 `number`, `string`, `boolean`은 가장 많이 사용되면서 사용자에 따라서 다르게 사용될 여지가 적은 반면, `null`이나 `undefined`는 사용자에 따라서 다르게 사용될 여지가 많음.<br />
따라서 tsconfig의 `strictNullChecks`옵션을 활성화 하면 명시적으로 null이나 undefined를 할당해주지 않으면 null이나 undefined를 사용할 수 없음. 만약 해당 옵션을 사용하지 않으면 보통 `타입 가드`를 통해서 `null`이나 `undefined`를 걸러냄.<br />

타입 단언(`!`)을 통해서 null이나 undefined가 아니라고 보장할 수도 있지만, 보통 타입가드를 선호함.

