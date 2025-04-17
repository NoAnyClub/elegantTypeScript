이번 장은 `Record`를 명시적으로 사용하는 방안에 대해서 다룸.

### 무한한 키를 집합으로 가지는 Record
---
아래처럼 음식 분류를 키로 사용하는 음식 배열이 담긴 객체를 만들었다.
```ts
type Category = string;
interface Food {
    name: string;
}

const FoodByCategory: Record<Category, Food[]> = {
    '한식': [{ name: '김치찌개' }, { name: '비빔밥' }],
    '일식': [{ name: '초밥' }, { name: '텐동' }],
}
```
이때 foodByCategory는 무한한 키 집합을 가지게 됨. 따라서 존재하지 않는 키값을 사용해도 ts는 에러를 뱉지 않음.
```ts
type Category = string;
interface Food {
    name: string;
}

const foodByCategory: Record<Category, Food[]> = {
    '한식': [{ name: '김치찌개' }, { name: '비빔밥' }],
    '일식': [{ name: '초밥' }, { name: '텐동' }],
}

const foodList = foodByCategory['양식']; // 없는 key값인데도 Food[]로 추론

foodByCategory['양식'].forEach((food) => food.name); // OK ✅
```
이는 컴파일 시에는 문제가 없지만, 런타임에서 코드가 동작할때는 에러를 뱉을 것임.

이럴때는 js의 `옵셔널 체이닝` 기법을 사용해서 에러를 방지할 수 있음.

> [!NOTE]
> **옵셔널 체이닝**
> 옵셔널 체이닝(`?.`) 연산자는 . 체이닝 연산자와 유사하게 작동하지만, 만약 참조가 `nullish` (null 또는 undefined)이라면, > 에러가 발생하는 것 대신에 표현식의 리턴 값은 `undefined`로 단락된다.<br/>
> 함수 호출에서 사용될 때, 만약 주어진 함수가 존재하지 않는다면, `undefined`를 리턴한다.

```ts
foodByCategory['양식']?.forEach((food) => food.name); // 런타임도 OK ✅
```

그러나 매번 옵셔널 체이닝을 사용할수는 없는 노릇. <br />
그렇다면 ts에서 개발중에 유효하지 않은 키가 사용되었는지, 또는 `undefined`일 수 있는 값이 있는지 등을 사전에 파악하려면 어떻게 해야 할까?

<br />

### 유닛 타입으로 변경하기
키가 유한한 집합이라면 `유닛 타입`을 사용할 수 있음.
> 유닛 타입은 다른 타입으로 쪼개지지 않고, 오직 하나의 정확한 값을 가지는 타입

```ts
type Category = '한식' | '일식'; // ...
interface Food {
    name: string;
}

const foodByCategory: Record<Category, Food[]> = {
    '한식': [{ name: '김치찌개' }, { name: '비빔밥' }],
    '일식': [{ name: '초밥' }, { name: '텐동' }],
}

const foodList = foodByCategory['양식']; // 에러 발생!
```
그러나 이건, 키가 무한한 상황에서는 적합하지 않음.

<br />

### Partial을 활용해서 정확한 타입 표현하기
---
객체 값이 undefined일 수 있는경우에 PartialRecord타입을 활용하고, 객체를 선언할 때 이를 활용할 수 있음.
```ts
type PartialRecord<K extends string, T> = Partial<Record<K, T>>

type Category = string;

interface Food {
    name: string;
}

const foodByCategory: PartialRecord<Category, Food[]> = {
    '한식': [{ name: '김치찌개' }, { name: '비빔밥' }],
    '일식': [{ name: '초밥' }, { name: '텐동' }],
}

const foodList = foodByCategory['양식']; // Food[] | undefined
const foodList = foodByCategory['한식']; // Food[] | undefined
```

근데 이건, `foodByCategory`내부에 존재하는 key값을 써도 `undefined`도 같이 추론되기 때문에 좋은(?) 방법인지는 모르겠다..!