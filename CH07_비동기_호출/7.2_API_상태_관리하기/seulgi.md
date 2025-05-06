# API 상태 관리하기

실제 API를 요청하는 코드는 컴포넌트 내에서 비동기 함수를 직접 호출하지 않는다.

비동기 API를 호출하기 위해서는 API의 성공 실패에 따른 상태가 관리 되어야 하므로<br />
상태 관리 라이브러리의 액션이나 훅과 같이 재정의된 형태를 사용해야 한다.

## 1) 상태 관리 라이브러리에서 호출하기

상태 관리 라이브러리의 비동기 함수들은 서비스 코드를 사용해서<br />
비동기 상태를 변화시킬 수 있는 함수를 제공한다.

컴포넌트는 이런 함수를 사용해 상태를 구독하며, 상태가 변경될 때<br />
컴포넌트를 다시 렌더링하는 방식으로 동작한다.

### Redux의 경우

비동기 상태가 아닌 전역 상태를 위해 만들어진 라이브러리이기 때문에<br />
미들웨어라고 불리는 여러 도구를 도입해 비동기 상태를 관리한다.

보일러플레이트 코드가 많아지는 등 간편하게 비동기 상태를<br />
관리하기 어려운 상황도 발생한다.

### MobX 같은 라이브러리의 경우

이런 불편함을 개선하기 위해 비동기 콜백 함수를 분리하여<br />
액션으로 만들거나 runInAction과 같은 메소드를 사용하여 상태 변경을 처리한다.

async / await나 flow 같은 비동기 상태 관리를 위한 기능도 있어<br />
간편하게 사용 가능하다.

```ts
// stores/userStore.ts
import { makeAutoObservable, runInAction } from "mobx";

class UserStore {
  user: any = null;
  loading = false;

  constructor() {
    makeAutoObservable(this);
  }

  async fetchUser() {
    this.loading = true;

    try {
      const response = await fetch("/api/user");
      const data = await response.json();

      runInAction(() => {
        this.user = data;
        this.loading = false;
      });
    } catch (error) {
      runInAction(() => {
        this.loading = false;
      });

      console.error(error);
    }
  }
}

export const userStore = new UserStore();
```

> 모든 상태 관리 라이브러리에서 비동기 처리 함수를 호출하기 위해 액션을 추가할 때마다<br />
> 관련된 스토어나 상태가 계속 늘어난다.

> 가장 큰 문제점은 전역 상태 관리자가 모든 비동기 상태에 접근하고 변경할 수 있다는 것이다.<br />
> (2개 이상 컴포넌트가 구독하고 있는 비동기 상태는 쓸데없는 비동기 통신과 의도치 않은 상태 변경이 발생할 수 있음)

<br />
<br />

## 2) 훅으로 호출하기

react-query나 useSwr과 같은 훅은 캐시를 사용해 비동기 함수를 호출하며,<br />
상태 관리 라이브러에서 발생했던 의도치 않은 상태 변경을 방지하는 데 도움이 된다.

예를 들어, react query에서는 onSuccess 옵션의 invalidateQueries를 사용해<br />
특정 키의 API를 유효하ㅣ지 않은 상태로 설정할 수 있다.

```ts
// hooks/usePosts.ts
import { useMutation, useQuery, useQueryClient } from "@tanstack/react-query";
import { createPost, fetchPosts } from "../api/posts";

export const usePosts = () => {
  return useQuery({ queryKey: ["posts"], queryFn: fetchPosts });
};

export const useCreatePost = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: createPost,
    onSuccess: () => {
      // 성공 시, posts 쿼리를 무효화해서 자동으로 refetch
      queryClient.invalidateQueries({ queryKey: ["posts"] });
    },
  });
};
```

> 다만, 이후 컴포넌트가 반드시 최신 상태를 표현하려면 폴링이나 웹소켓 등의 방법을 써야한다.

> [!NOTE] > **폴링(polling)?** <br />
> 클라이언트가 주기적으로 서버에 요청을 보내 데이터를 업데이트하는 것이다.<br />
> 클라이언트는 일정한 시간 간격으로 서버에 요청을 보내고, 서버는 해당 요청에 대해<br />
> 최신 상태의 데이터를 응답으로 보내주는 방식을 말한다.

### 상태 관리 라이브러리의 단점?

비동기로 상태를 변경하는 코드가 점점 추가되면 전역 상태 관리 스토어가 비대해지는데,<br />
단순히 상태를 변경하는 액션이 증가하는 것뿐만 아니라 전역 상태 자체도 복잡해진다.

에러 발생, 로딩 중 등과 같은 상태는 전역으로 관리할 필요가 없다.<br />
이런 고민으로 인해 비동기 통신으로 react query를 사용해 처리하고 있다.<br />
(다만 react-query가 최적의 라이브러리는 아님, 상황에 따라 사용해야 함)

<br />
<br />
