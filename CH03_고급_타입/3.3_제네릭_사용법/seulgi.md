# 제네릭 사용법

## 1) 함수의 제네릭

함수의 매개변수나 반환 값에 다양한 타입을 넣고 싶을 때 제네릭을 사용할 수 있다.

```ts
function ReadOnlyRepository<T>(
  target: ObjectType<T> | EntitySchema<T> | string
): Repository<T> {
  return getConnection("ro").getRepository(target);
}
```
