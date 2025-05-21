리액트에 훅이 추가 되기 이전에는 클래스 컴포넌트에서만 상태를 가질 수 있었고, `componentDidMount`, `componentWillUnmount`와 같은 컴포넌트 생명주기 함수 내부에서만 상태 업데이트 로직을 실행시킬 수 있었음.


이후 리액트에 훅이 도입되면서 함수 컴포넌트에서도 클래스 컴포넌트와 같이 생명주기에 맞춰서 로직을 실행할 수 있게 되었음

<br />

### useState
---
리액트 함수 컴포넌트에서 상태를 관리하기 위해서 사용되는 대표적인 훅
```ts
function useState<S>(
  initialState: S | (() => S)
): [S, Dispatch<SetStateAction<S>>]; // 함수 오버로드의 타입 선언부

type Dispatch<A> = (value: A) => void;
type SetStateAction<S> = S | ((prevState: S) => S);
```

> [!NOTE]
> <a href="https://www.typescriptlang.org/docs/handbook/2/functions.html#function-overloads" target="_blank">함수 오버로드</a> <br />
> 동일한 이름에 매개 변수만 다른 여러 버전의 함수를 만드는 것을 함수의 오버로딩이라고 함 <br />
> 파라미터의 형태가 다양한 여러 케이스에 대응하는 같은 이름을 가진 함수를 만드는 것 <br />
> ```ts
> function makeDate(timestamp: number): Date;
> function makeDate(m: number, d: number, y: number): Date;
> function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
>   if (d !== undefined && y !== undefined) {
>     return new Date(y, mOrTimestamp, d);
>   } else {
>     return new Date(mOrTimestamp);
>   }
> }
> const d1 = makeDate(12345678);
> const d2 = makeDate(5, 5, 5);
> const d3 = makeDate(1, 3); // Error! No overload expects 2 arguments, but overloads do exist that expect either 1 or 3 arguments.
> ```
`useState`는 튜플을 반환하는데, 첫번째 요소는 제네릭 타입으로 지정한 `S`타입이고, 두번째 요소는 상태를 업데이트 할 수 있는 `Dispatch`타입의 함수임.
`useState`에 타입을 명시해주면 휴먼 에러를 방지할 수 있음
```js
const [person, setPerson] = useState([
  {
    name: 'seongho',
    age: 1
  },
  {
    name: 'seulgi',
    age: 2
  }
]);

setPerson([...person, { name: 'foo' , agee: 3}]); // agee

person.reduce((acc, cur) => acc + cur.age, 0); // NaN
```
해당 예시에 `useState`타입을 추가한다면?
```ts
interface Person {
  name: string;
  age: number;
}

const [person, setPerson] = useState<Person>([
  {
    name: 'seongho',
    age: 1
  },
  {
    name: 'seulgi',
    age: 2
  }
]);

setPerson([...person, { name: 'foo' , agee: 3}]); // 🚫 Error!

person.reduce((acc, cur) => acc + cur.age, 0);
```
이렇게 에러를 컴파일 타임에 발견할 수 있음

<br />

### 의존성 배열을 사용하는 훅
---
- ### `useEffect`와 `useLayoutEffect`
  리액트에서 렌더링 이후에 추가적인 사용자의 인터랙션 없이 수행해야 하는 로직이 있다면 `useEffect`를 사용할 수 있음
  ```ts
  function useEffect(effect: EffectCallback, deps?: DependencyList): void;

  type DependencyList: ReadonlyArray<any>;
  type EffectCallback = () => void | Destructor;
  ```

  `useEffect`의 콜백함수로는 비동기 함수가 들어갈 수 없음. 만약 비동기함수가 들어가게 되면 `경쟁 상태`를 불러일으킬 수 있기 때문임.

  <br />

  > [!NOTE]
  > **경쟁 상태** <br />
  > 서로 다른 두 요청이 서로 경쟁하여 도착 순서를 예상할수 없게되어, 프로그램 동작이 원하지 않는 방향으로 진행되는 상태

  `EffectCallback`타입은 아무것도 반환하지 않거나, `Destructor`를 반환하는데 이때 `Destructor`란 클린업 함수임. <br />
  **클린업 함수**는 해당 컴포넌트가 언마운트 될 때, 그리고 해당 `EffectCallback`이 재실행 되기 전에 실행 됨.

  <br />

  useEffect와 비슷한 역할을 하는 훅으로 `useLayoutEffect`가 있음.<br />
  해당 훅도 useEffect와 타입은 동일한데, 동작하는 방식에만 차이가 있음.

  ```ts
  function useLayoutEffect(effect: EffectCallback, deps?: DependencyList): void;

  type DependencyList: ReadonlyArray<any>;
  type EffectCallback = () => void | Destructor;
  ```
  첫 렌더링 이후에 `EffectCallback`을 실행시키는 `useEffect`와는 다르게, `useLayoutEffect`는 첫 렌더링 전에 `EffectCallback`를 실행시킴.

<br />

- ### `useMemo`와 `useCallback`
  ```ts
  type DependencyList: ReadonlyArray<any>;

  function useMemo<T>(
    factory: () => T,
    deps: DependencyList | undefined
  ): T

  function useCallback<T extends (...args: any[]) => any>(
    callback: T,
    deps: DependenctList
  ): T
  ```

  > [!TIP]
  > **모든 곳에 useMemo를 추가해야 할까?** <br />
  > `useMemo`로 최적화하는 것은 몇몇 경우에만 유용합니다.<br />
  > - ***useMemo에 입력하는 계산이 눈에 띄게 느리고 종속성이 거의 변경되지 않는 경우.***<br />
  > - ***memo로 감싸진 컴포넌트에 prop로 전달할 경우.***<br />
  >   값이 변경되지 않았다면 렌더링을 건너뛰고 싶을 것입니다. 메모이제이션을 사용하면 의존성이 동일하지 않은 경우에만 컴포넌트를 다시 렌더링할 수 있습니다. <br />
  > - ***전달한 값을 나중에 일부 Hook의 종속성으로 이용할 경우***<br />
  > 예를 들어, 다른 useMemo의 계산 값이 여기에 종속되어 있을 수 있습니다. 또는 useEffect의 값에 종속되어 있을 수 있습니다.

<br />

- ### `useRef`
  리액트로 애플리케이션을 개발하다보면 DOM을 직접 선택해야 하는 경우가 발생하기 마련임 (특정 요소 위치로 스크롤하기 등) <br />
  이때 `useRef`훅을 사용할 수 있음.

  ```tsx
  const name = useRef<string>(null); // ✅ OK
  ```
  근데 한가지 궁금한 점이 있다. <br />
  위 코드에서 나는 `useRef`의 타입을 `string`으로 명시해줬는데, 어떻게 `null`이 초깃값으로 들어갈 수 있을까?

  그 이유는 `useRef`가 `함수 오버로드`를 하기 때문임.

  ```ts
  interface RefObject<T> {
    readonly current: T;
  }

  interface MutableRefObject<T> {
    current: T;
  }

  // 함수 오버로드
  function useRef<T>(initialValue: T): MutableRefObject<T>;
  function useRef<T>(initialValue: T | null): RefObject<T>;
  function useRef<T>(initialValue: T | undefined): MutableRefObject<T | undefined>;
  ```

  만약 내가 `useRef`를
  ```tsx
  const name = useRef<string | null>(null);
  // 또는
  const name = useRef<string>('');
  ```
  이렇게 작성했다면 a는 `MutableRefObject<string>`이 되어서 `current`값을 변경할 수 있는 사이드 이펙트의 여지가 생김.

  <br />

  8장에서 보았던 것 처럼 만약 자식 컴포넌트에게 `ref`를 전달하고 싶다면 `forwardRef`를 사용해야 함

  ```tsx
  interface Props {
    profileImageUrl: string;
  }

  const Profile = forwardRef<HTMLImageElement, props>(
    (props, ref) => {
      const { profileImageUrl } = props;

      return (
        <img
          ref={ref}
          src={props}
        />
      );
    }
  );

  export default Profile
  ```
  <br />

  이때 사용하는 `forwardRef`의 타입은 아래와 같음

  ```ts
  function forwardRef<T, P = {}>(
    render: ForwardRefRenderFunction<T, P>
  ): ForwardRefExoticComponent<PropsWithoutRef<P> & RefAttributes<T>>;

  // forwardRef의 render함수타입
  interface ForwardRefRenderFunction<T, P = {}> {
    (props: P, ref: ForwardedRef<T>): ReactElement | null;
    displayName?: string;
    defaultProps?: never | undefined;
    propTypes?: never | undefined;
  }
  ```

  > [!TIP]
  > `useRef`는 DOM을 직접 접근하는 용도 외에도 다양하게 활용할 수 있음 <br />
  > - ***`useRef`로 관리되는 변수는 값이 바뀌어도 리렌더링을 촉발하지 않음.*** <br />
  > - ***리액트의 상태는 값을 업데이트하면 리렌더링 이후에 업데이트 된 상태를 조회할 수 있지만, `useRef`는 업데이트 이후 즉시 조회가 가능함*** <br />
