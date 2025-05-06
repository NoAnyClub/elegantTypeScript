FE개발을 하다보면 API가 나오기 전에 개발을 진행해야 하는 일이 종종 생김. <br />
그렇다면 이러한 상황에서는 프론트엔드 개발을 어떻게 진행할 수 있을까??

<ol>
  <li>
    <b>임시 더미 데이터를 만들어서 구현하기</b>
  </li>
    <span>
      -> 요청 응답에 따라서 각기 다른 화면을 보여줘야 할 경우 대응하기가 힘들다.
    </span>
  <li>
    <b>별도의 가짜 서버를 만들어서 운영하기</b>
  </li>
  <span>
    -> 프론트 개발 과정에서 발생하는 모든 예외를 처리하는 것은 쉽지 않다.
  </span>
</ol>

<br />

이럴 때 `모킹`이라는 방법을 활용할 수 있음. <br />
`모킹`을 활용하면 앞서 제시한 상황에서 유연하게 대처할 수 있음. 또한 서버의 영향을 받지 않고 프론트엔드 개발을 할 수 있게 됨. <br />

그렇다면 모킹에는 어떤 방법들이 있을까?

<br />

### JSON파일 불러오기
간단한 조회만 필요한 경우에는 .json파일을 만들거나, js파일안에 JSON형식의 정보를 저장하고 export해주는 방식을 사용하면 됨.<br />
이러고 get요청에 해당 파일 경로를 삽입해주면 조회 응답으로 원하는 값을 받을 수 있음.
```ts
// mock/service.ts
const MOCK = [
  {
    name: 'OSH',
    age: 27,
  }
];

export default MOCK;

// api
const getPerson = apiRequester.get('/mock/service.ts');
```

<br />

### NextApiHandler 활용하기
---
만약 `NextJS`를 사용하고 있다면, <a href='https://nextjs.org/docs/app/building-your-application/routing/route-handlers' target='_blank'>NextApiHandler</a>를 사용하는 방법도 있음.
```ts
// route.ts
const MOCK = [
  // mock data...
]

export async function GET(req: NextRequest) {
  return NextResponse.json(MOCK);
};
```

<br />

### axios-mock-adapter활용하기
---
만약, axios를 사용한다면 <a href='https://github.com/ctimmerm/axios-mock-adapter' target='_blank'>axios-mock-adaptor</a>를 사용할 수 있음.
> [!WARNING]
> `axios-mock-adapter`는 api요청을 중간에 가로채는 것 이기 때문에 실제 API 요청을 주고받지는 않음.

<br />

### 그 외 방법들
---
나는 개인적으로도 사용해본 `msw`나 `mirage`도 괜찮다고 생각함.
