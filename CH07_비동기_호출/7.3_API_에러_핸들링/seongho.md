비동기 API호출을 하다보면, 상태 코드에 따라서 다양한 에러를 마주하게 됨. <br />
ts에서 어떻게 에러를 처리하고, 명시할 수 있는지 알아보자

<br />

### 타입 가드 활용하기
---
`Axios`를 사용한다면 `isAxiosError`라는 타입가드를 활용할 수 있지만, 서버 에러임을 명확하게 표시하고, 서버에서 내려주는 에러 응답 객체에 대해서도 구체적으로 정의하면 더 좋음. <br />

```ts
interface ErrorResponse {
  status: string;
  serverDataTime: string;
  errorCode: string;
  errorMessage: string;
}

function isAxiosError(error: unknonw): error is AxiosError<ErrorResponse> {
  return axios.isAxiosError(error);
}
```
이렇게 작성하면 서버 에러인지, 클라이언트 에러인지 명확하게 구분이 가능함.

```ts
try {
  // API 호출 로직
} catch(error) {
  if (isAxiosError(error)) {
    // 서버 에러
    setErrorMessage(error.errorMessage);

    return;
  }

  // 클라이언트 에러
  setErrorMessage('클라이언트 에러');
}
```

<br />

### 에러 서브클래싱 하기
---
> [!NOTE]
> **서브클래싱(Subclassing)** <br />
> 기존 클래스를 확장하여 새로운 클래스를 만드는 과정을 말함. <br />
> 새로운 클래스는 상위 클래스의 모든 속성과 메서드를 상속받아서 사용할 수 있고, 추가적으로 새로운 속성과 메서드를 정의할 수도 있음.

우리가 코드를 작성하다보면 다양한 에러가 내려올 때가 있는데 이때 서브클래싱을 활용하면 어떤 에러인지 바로 확인할 수 있고, 다르게 처리가 가능함.

```ts
class OrderHttpError extends Error {
    private readonly privateResponse: AxiosResponse<ErrorResponse> | undefined;

    constructor(message: string, response?: AxiosResponse<ErrorResponse>) {
        super(message);

        this.name = 'OrderHttpError';
        this.privateResponse = response;
    }

    getResponse(): AxiosResponse<ErrorResponse> | undefined {
        return this.privateResponse;
    }
}

class NetworkError extends Error {
    constructor(message = '') {
        super(message);

        this.name = 'NetworkError';
    }
}

class UnauthorizedError extends Error {
    constructor(message = '') {
        super(message);

        this.name = 'UnauthorizedError';
    }
}
```

요런식으로 케이스를 분류해둔 뒤, 가공해서 처리할 수 있음

```ts
const enum HttpStatusCode {
    'NETWORK' = 500,
    'UNAUTHORIZED' = 401
}

// 에러 분류
const classifyErrorByType = (
  error: Error | AxiosError<Error>
) => {
  if (isAxiosError(error)) {
    const { response } = error;

    switch (response) {
      case response === HttpStatusCode.NETWORK :
        return Promise.reject(
          new NetworkError(response.data.message)
        );

      case response === HttpStatusCode.UNAUTHORIZED :
        return Promise.reject(
          new UnauthorizedError(response.data.message)
        );

      default:
        return Promise.reject(
          new OrderHttpError(
            response.data.message,
            response
          )
        );
    }
  } else {
    return Promise.reject(error);
  }
}

// 에러별 핸들링
const handleError = (error: unknown) {
  if (error instanceof UnauthorizedError) {
    return onUnauthorizedError(
      error.message,
      // ...
    );
  }

  if (error instanceof NetworkError) {
    return alert('네트워크 연결이 원활하지 않습니다.');
  }

  return onOrderHttpError(
    error.message,
    error
  );
}
```
이렇게 해두면 아래와 같이 활용할 수 있음.

```ts
const getJobList = async () => {
  try {
    // API 호출 로직
  } catch (error) {
    const classifiedError = classifyErrorByType(error); // 에러 분류
    handleError(classifiedError); // 분류된 에러 별 액션
  }
}
```

<br />

### 인터셉터를 활용한 에러 처리
---
만약 `Axios`를 사용한다면 이러한 에러 분류를 `interceptor`에 붙히는 것도 좋은 방법인 것 같음.

```ts
axios.interceptors.response.use(
  (response: AxiosResponse) => response,
  classifyErrorByType, // 인터셉터에서 에러 분류
);
```

<br />

### 리액트 쿼리를 활용한 에러 처리
---
리액트 쿼리에서는 `onError`나 `isError`와 같이 에러 핸들러나, 상태를 반환해주는 플래그가 존재하기 때문에 훨씬 에러를 관리하기가 쉬움.

> [!CAUTION]
> ***`onError는` react-query의 4버전까지만 존재하는 에러 핸들러임 (5버전에서는 삭제)*** <br />
