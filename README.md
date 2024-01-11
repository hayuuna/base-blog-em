# React-Query

💡 **서버: JSONPlacehoder**

- 블로그 서버를 시뮬레이션하는 엔드포인트 존재
- 게시물, 댓글 가져오기 가능
- 업데이트 가능(실제 서버 데이터 변경 X → 데이터가 변경된 것처럼 응답)


💡 **React-Query 개념 학습**

- 데이터 페칭(Fetch)
- 로딩, 오류 상태 관리(사용자에게 알림)
- 개발자 도구(쿼리에게 무슨 일이 일어나고 있는지 알 수 있음)
- Pagenation
- Pagenation의 Prefetching(다음 페이지를 미리 가져오면 사용자가 다음 페이지를 클릭할 때 데이터를 즉시 가져옴 → 매끄럽게 처리)
- 서버 데이터를 변경하는 변이

### 1. useQuery로 쿼리 생성하기

```jsx
import { QueryClient, QueryClientProvider } from "react-query";
```

- **QueryClient**: 서버와 데이터 연결을 관리하고 캐시를 관리
- **QueryClientProvider**: 모든 자녀 컴포넌트가 클라이언트를 사용할 수 있게 함

```jsx
const queryClient = new QueryClient();
```

공급자가 클라이언트를 소품으로 사용하게 되면서 클라이언트가 가지고 있는 캐시와 모든 기본 옵션을 공급자의 자녀 컴포넌트도 사용할 수 있게 된다.

```jsx
function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <div className="App">
        <h1>Blog &apos;em Ipsum</h1>
        <Posts />
      </div>
    </QueryClientProvider>
  );
}
```

```jsx
import { useQuery } from "react-query";
```

- **useQuery**: 서버에서 데이터를 가져올 때 사용

데이터는 하드코딩된 배열이 아닌, 반환 객체에서 useQusery로 데이터 속성을 구조 분해

useQuery는 다양한 속성을 가진 객체 반환

```jsx
const { data } = useQuery("posts", fetchPosts);
```

1. 첫번째 인자: 쿼리 키(쿼리이름)
2. 두번째 인자: 함수(쿼리에 대한 데이터를 가져오는 비동기 함수)

이렇게 하게되면 오류 → map을 읽을 수 없음

fetchPosts가 비동기식이기 때문에 데이터를 불러오기 전 데이터를 할당 해 데이터의 할당 항목을 알 수 없음

**간단한 해결 방법:**

```jsx
if (!data) return <div> </div>;
```



- data가 없으면 div만 반환하고 fetchPosts가 해결될 때 까지 데이터는 거짓이 되고
- 해결이 되면 데이터에 배열이 포함
- 컴포넌트가 다시 렌더링 되어 매핑이 가능(조기 반환 수행 X)

### 2. 로딩상태와 에러 처리하기
데이터가 정의되지 않으면 오류가 되게 두지 않고 **적절한 조치**를 취함

- **isLoading**: 데이터 로딩 여부를 알려줌(boolean)
- **isError**: 데이터를 가져올 때 오류가 있는지 여부를 알려줌(boolean)

```jsx
const { data, isLoading, isError } = useQuery("posts", fetchPosts);
```

- 구조분해하여 데이터가 거짓인지 확인

```jsx
// if (!data) return <div> </div>;

if (isLoading) return <h3>is Loading...</h3>;
```

- 빈 div를 반환하는 대신 로드 중인지 확인 가능
- 로딩중일 때 로딩 표시기로 조기 반환
- 더이상 로딩하지 않으면 isLoading이 false가 됨
- 해당 게시물 제목과 페이지의 나머지 부분을 반환

- **isFetchng**: 비동기 쿼리가 해결되지 않음(아직 fetching을 완료하지 않았음을 의미, 쿼리가 Axios 호출 또는 Graph 호출일 수 있음)


| isFetching | isLoading |
| --- | --- |
| 비동기 쿼리가 해결되지 않음(데이터를 가져오는 중) | isFetching의 하위집합 |
| fetching을 완료하지 않음 | 가져오는 상태에 있음 |
| isLoading → isFetching으로 바꾸면 캐싱과 상관없이 나타남 | 쿼리 함수가 아직 해결되지 않음 |
| 캐시된 데이털르 표시하면서 배경에서는 데이터의 업데이트 여부를 조용히 서버를 확인 → 업데이트된 경우 해당 데이터를 페이지에 보여줌 | 쿼리에 표시할 캐시 된 데이터 없음 |

→ Pagination을 할 때 캐시된 데이터가 있을 때와 없을 때를 구분해야 함

```jsx
if (isError) return <h3>Oops, something went wrong</h3>;
```

- 로딩을 포기하기 전까지 기본값으로 세번 시도 후 데이터를 가져올 수 없다고 결정(시도횟수는 변경 가능)
- 네트워크탭에서 확인 가능
- 시도 중: 로딩 중 표시
- 완료: 오류 메시지 표시

```jsx
const { data, isLoading, isError, error } = useQuery("posts", fetchPosts);
```

- 반환 객체에는 쿼리 함수에서 전달하는 오류(error)도 있음

```jsx
if (isError)
    return (
      <>
        <h3>Oops, something went wrong</h3>
        <p>{error.toString()}</p>
      </>
    );
```

- 어떤 오류인지도 확인 가능

### 3. 개발자 도구

- 앱에 추가할 수 있는 컴포넌트로 개발 중인 **모든 쿼리의 상태를 표시**함
- 예상대로 작동하지 않는 경우 문제 해결에 도움
    - 쿼리 키로 쿼리를 표시해 주고 **활성, 비활성, 만료** 등 모든 쿼리의 상태를 알려줌
    - 마지막에 업데이트된 **타임스탬프**도 알려줌
    - 쿼리에 의해 반환된 데이터를 확인할 수 있는 **데이터 탐색기** 존재
    - 쿼리를 볼 수 있는 **쿼리 탐색기** 존재
- 개발자 도구는 프로덕션 변들에 포함되지 않음
- NODE_ENV 변수에 따라 프로덕션 환경에 있는지 여부 결정
- Create React App은 npm run build를 실행할 때만 이 NODE_ENV 변수를 프로덕션으로 설정
- 그렇지 않으면 development 또는 testing으로 설정
- 개발중일 때는 개발자 도구가 표시, 프로덕션일 때 표시하지 않음

**개발자 도구 사용 방법:**

```jsx
import { ReactQueryDevtools } from "react-query/devtools";
```

```jsx
function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <div className="App">
        <h1>Blog &apos;em Ipsum</h1>
        <Posts />
      </div>
      <ReactQueryDevtools />
    </QueryClientProvider>
  );
} 
```

![image](https://github.com/hayuuna/base-blog-em/assets/144312023/90bf0d14-4816-4b3c-8f2e-fcdc956e5b41)


쿼리와 어떤 데이터가 있는지 탐색할 수 있음

- **Refetch**: 새 데이터 확인
- **Invalidate**: 무효화, Refetch가 트리거
- **Remove**: 캐시에서 완전히 제거(Refetching전까지 쿼리가 보이지 않음)

**Stale Data**

- 데이터 리페칭은 만료된 데이터에서만 실행
- 여러 트리거 존재 (컴포넌트가 다시 마운트 될 때, 윈도우가 다시 포커스 될 때)

fetching → stale(만료)로 자동 바뀜

- **staleTime**: 데이터가 만료됐다고 판단하기 전까지 허용하는 시간, 데이터가 사용 가능한 상태로 유지되는 시간, 서버로 돌아가 데이터가 여전히 정확한지 확인하는 시점까지
- 예) 웹사이트에 표시된 데이터가 10초까지는 그대로여도 괜찮다면 staleTime을 10초로 설정

```jsx
const { data, isLoading, isError, error } = useQuery("posts", fetchPosts, {
	staleTime: 2000,
});
```

세번째 인자로 옵션을 줄 수 있음

staleTime을 2000으로 설정하면 2초마다 데이터가 만료됨

**staleTime 기본값이 0인 이유:** 

- 업데이트가 왜 안되죠? 보다 데이터가 어떻게 늘 최신상태를 유지하나요?가 더 나은 질문이다.
- 데이터는 항상 만료상태이므로 서버에서 다시 가져와야 한다고 가정한다는 의미이다.
- 이렇게 하면 클라이언트에게 실수도 만료된 데이터를 제공할 가능성이 줄어든다.

**cacheTime**

- 데이터가 비활성화 된 이후 남아 있는 시간
- 캐시된 데이터는 쿼리를 다시 실행했을 때 사용

| staleTime | cacheTime |
| --- | --- |
| 리페칭 할 때 고려사항 | 나중에 다시 필요 할 수 있는 데이터 |
|  | 특정 쿼리에 대한 활성 useQuery가 없는 경우 해당 데이터는 콜드 스토리지로 이동 |
|  | 구성된 cacheTime이 지나면 캐시의 데이터가 만료(유효시간의 기본값: 5분) |
|  | cacheTime이 관찰하는 시간의 양은 특정 쿼리에 대한 useQuery가 활성화 된 후 경과한 시간 |
|  | 캐시가 만료되면 가비지 컬렉션이 실행되고 클라이언트는 데이터를 사용할 수 없음 |
|  | 데이터가 캐시에 있는 동안에는 Fetching할  때 사용될 수 있음 |

데이터가 페칭을 중지하지 않으므로 서버의 최신 상태로 새로고침이 가능함

계속해서 빈 페이지만 보는 경우가 생김 → 새로운 데이터를 수집하는 동안오래된 데이터를 보여주는게  빈 페이지보다 나을 수 있음, 만료된 데이터가 위험한 경우에는 cacheTime을 0으로 설정

## 4. Post.id에 맞는 comments를 fetch할 때 refresh가 되지 않는 이유

- 모든 쿼리키가 comments 쿼리키를 동일하게 사용하고 있기 때문
- comments와 같이 알려진 쿼리 키가 있을 때는 어떠한 트리거(이벤트)가 있어야만 데이터를 다시 가져옴
    - 리페칭 트리거 예시)
        - 컴포넌트 재마운트
        - 윈도우 다시 포커스
        - useQuery에서 반환되어 수동으로 리페칭 실행
        - 지정된 간격으로 리페칭 자동 실행
        - 변이 생성 후 쿼리를 무효화할 때
        - 클라이언트의 데이터가 서버의 데이터와 불일치할 때

→ 새 블로그 게시물 제목 클릭시 트리거가 일어나지 않음: 데이터가 만료되어도 새 데이터를 가져오지 않음

**해결 방법:**

- 새 블로그 게시물 제목을 클릭할 때 마다 데이터를 무효화 시킴
    - 쉽지 않고 간단하지 않음
- 데이터를 제거하면 안됨(게시물 2의 댓글에 대한  쿼리를 만들 때 캐시에서 게시물 1의 댓글을 제거하면 안됨)
    - 같은 쿼리를 실행시키는 것이 아님
    - 같은 캐시 공간을 차지하지 않음
- 쿼리에 게시물 ID를 포함
    - 쿼리별로 캐시를 남길 수 있음
    - coments 쿼리에 대한 캐시를 공유하지 않아도 됨
    - 각 쿼리에 해당하는 캐시를 가지게 됨
- 각 게시물에 대한 쿼리에 라벨 설정
    - 쿼리 키에 문자열 대신 배열 전달
    
    ```jsx
    ['comments', post.id]
    ```
    
    - 배열의 첫번째 요소로 문자열 “comments”를 가지고 두번째 요소로 post.id를 가짐
    - 쿼리 키를 쿼리에 대한 의존성 배열로 취급하게 됨(쿼리키가 변경되면 즉, post.id가 업데이트 되면 React Query가 새 쿼리를 생성해서 staleTime과 casheTime을 가지게 되고 의존성 배열이 다르면 완전히 다른 것으로 간주)
    - 데이터를 가져올 때 사용하는 쿼리 함수에 있는 값이 쿼리 키에 포함되어야 함 → 모든 comments 쿼리가 같은 쿼리로 간주되는 상황을 막고 각 다른 쿼리로 다뤄짐

```jsx
const { data, isLoading, isError, error } = useQuery(
    ["comments", post.id],
    () => fetchComments(post.id)
  );
```

- post.id을 배열로 추가하면 의존성 배열로 작용하여 문자열 “comments”에 식별자가 추가됨

![image](https://github.com/hayuuna/base-blog-em/assets/144312023/128a4b71-1334-415a-8bfe-fd925d3533d1)


- 다른 게시물을 클릭하면 이전 comments 쿼리는 비활성화가 됨 → 아직 캐시에 남아 있음, 가비지로 수집되기 전까지 캐시에 남아 있음(cacheTime동안 사용되지 않을 때)

### 5. Pagination

- 컴포넌트의 상태, currentPage상태를 통해 현재 페이지를 파악하는 페이지 매김
- 페이지마다 다른 쿼리키가 필요함 → 쿼리 키를 배열로 업데이트해서 가져오는 페이지 번호를 포함
    
    ```jsx
    ["posts", currentPage]
    ```
    
- 다음 혹은 이전 페이지 버튼을 누르면 currentPage상태를 업데이트
    - 버튼 클릭 시 업데이트가 일어남
    - Reacr Query가 바뀐 쿼리 키 감지 새로운 쿼리키 실행 → 새 페이지 표시

```jsx
const [currentPage, setCurrentPage] = useState(1);
```

- 현재 페이지 상태를 표시

```jsx
export async function fetchPosts(pageNum) {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/posts?_limit=10&_page=${pageNum}`
  );
  return response.json();
}
```

```jsx
const { data, isLoading, isError, error } = useQuery(
    ["posts", currentPage],
    () => fetchPosts(currentPage)
  );
```

- currentPage를 의존성 배열로 작동
- 배열이 바뀌면 함수도 바뀜 → 데이터도 바뀜
- 쿼리 키가 바뀌면 useQuery에 새로운 쿼리를 알려 데이터를 다시 가져오도록 만듦

```jsx
<button
	disabled={currentPage <= 1}
	onClick={() => {
		setCurrentPage((previousValue) => previousValue - 1);
  }}
>
	Previous page
</button>
```

- 이전 버튼에는 1과 같거나 작으면 disabled처리
- 클릭시 이전 페이지에서 -1로 상태 관리

```jsx
<button
	disabled={currentPage >= maxPostPage}
	onClick={() => {
		setCurrentPage(previousValue) => previousValue + 1;
  }}
>
	Next page
</button>
```

- data로 최대 페이지 수를 찾을 수 있음 → 다음 페이지 유무애 대한 프로퍼티를 가지고 있을 때 버튼을 활성화 할지 판단하는 지표로 사용

→ 이렇게 하면 다음 페이지로 넘어갈 때 isLoading이 발생해 사용자 경험을 방해: 페이지에 캐시가 없기 때문

**해결 방법:**

- 다음 페이지 결과를 Prefetching해서 바로 보이도록 함(데이터를 가져올 동안 사용자가 기달릴 필요 X)

### 6. 데이터 Prefetching

- 데이터를 캐시에 추가
- 기본값으로 만료 상태
- 데이터를 사용하고자 할 때 만료 상태에서 데이터를 다시 가져옴
    - 데이터를 다시 가져오는 중에는 캐시에 있는 데이터를 이용해  앱에 나타냄
    - 캐시가 만료되지 않았다는 가정(사용자가 cacheTime보다 오래 머물수도 있음)
- 추후 사용자가 사용할 법한 모든 데이터에 프리페칭을 사용함
    - 사용자들이 통계적으로 특정 탭을 누를 확률이 높다면 특정 데이터를 미리 가져옴
    

```jsx
import { useQueryClient } from "react-query";
```

prefetch 쿼리는 queryClient의 메서드 → useQuery 훅을 통해 queryClient를 가져올 수 있음

```jsx
const queryClient = useQueryClient();
```

이 훅을 기능 컴포넌트에 실행해야 함

**실행시 유의 사항**

- 다음페이지로 onClick시 실행하는건 좋은 생각이 아님
    - 상태 업데이트가 비동기식으로 일어나기 때문에 이미 업데이트가 진행됐는지 알 방법이 없음
    - 현재 페이지가 무엇인지 알 수 있는 확실한 방법이 없음

해결 방법:

- useEffect사용

```jsx
useEffect(() => {
    if (currentPage < maxPostPage) {
      const nextPage = currentPage + 1;
      queryClient.prefetchQuery(["posts", nextPage], () =>
        fetchPosts(nextPage)
      );
    }
  }, [currentPage, queryClient]);
```

- 현재 페이지가 변경할때마다 다음 함수를 실행할 수 있도록 의존성 배열에 currentPage 추가
- 실행될 함수는 **queryClient.prefetchQuery**
- 다음페이지가 무엇이는 데이터를 미리 가져와야 함(nextPage)
- prefetchQuery의 인수는 useQuery 인수와 흡사함
- useQuery에 사용한 쿼리 키와 똑같은 모습을 하고 있음 → React Quey가 캐시에 이미 데이터가 있는지 useQuery에서 확인함
- 알고 있는 범위 외의 데이터를 가져오지 않도록 조건(if)을 추가해야 함(maxPostPage이전이면 프리페칭, maxPostPage에 있을 때 미리 가져올 데이터가 없음 )

```jsx
const { data, isLoading, isError, error } = useQuery(
    ["posts", currentPage],
    () => fetchPosts(currentPage),
    {
      staleTime: 2000,
      keepPreviousData: true,
    }
  );
```

- 쿼리 키가 바뀔 때도 지난 데이터를 유지해서 이전 페이지로 돌아갔을 때 캐시에 해당 데이터가 있도록 만들고 싶을 때 옵션에 **keepPreviousData**를 **true**로 설정

→ 페이지가 매끄럽게 넘어감

![image](https://github.com/hayuuna/base-blog-em/assets/144312023/e18ad059-bf97-4abd-bf5f-8f5567ebff38)


- 활성화 상태 쿼리는 현재 페이지 쿼리
- 나머지 쿼리도 남아 있음
- 다음 페이지를 미리 가져옴(데이터를 캐시에 추가 → 다음 페이지로 갔을 때 페이지가 바로 보임)

### 7. Mutation(변이)

- 서버에 데이터를 업데이트하도록 서버에 네트워크 호출
    - 포스트 추가, 삭제, 제목 변경
    - JSONPlaceholder API의 제한사항은 서버를 실제로 업데이트하지 않음(변이로 생성하는 서버 호출을 전송)
    변경을 적용할 때 이루어지는 원리
- 규모가 큰 애플리케이셔에는 변경 내용을 사용자에게 보여주거나 진행된 변경 내용을 등록하여 사용자가 볼 수 있게 하는 방법이 있음
    - 긍정적 업데이트로 서버 호출에서 모든 내용이 잘 진행될 것으로 간주
    - 사실이 아닌것으로 판명될 경우 롤백 진행
    - 변이 호출을 실행할 때 서버에서 받는 데이터를 취하고 업데이트된 해당 데이터로 React Query 캐시를 업데이트 하는 것
    - 관련 쿼리를 무효화(서버에서 리페치를 개시하여 클라이언트에 있는 데이터를 서버의 데이터와 최신 상태로 유지)

**useMutation**

- useQuery와 비슷함
    - mutate 함수를 반환하는데 이 함수는 변경된 사항을 토대로 서버를 호출할 때 사용
    - 데이터를 저장하지 않으므로 쿼리키는 필요하지 않음
    - isLoading은 존재, isFetching은 없음(캐시된 항목이 없기 때문)
    - 변이에 관련된 캐시는 존재하지 않고 재시도 또한 기본값으로 존재하지 않음(재시도를 적용하고 싶다면 설정은 가능)

```jsx
import { useMutation } from "react-query";
```

```jsx
const deleteMutation = useMutation((postId) => deletePost(postId));

// const deleteMutation = useMutation(() => deletePost(post.id));
```

- 쿼리 키는 할당하지 않고 함수를 부여함
- useMutation과 useQuery의 차이
    - 인수로 전달하는 변이 함수는 그 자체도 인수를 받을 수 있음(postId를 넣어주고 postId에 대해 deletePost 실행)
    - postId는 props에서 유래, post.id라고 입력해도 무방
- 변이 함수를 반환(delete 버튼을 클릭할 때 변이 함수 실행)

```jsx
<button onClick={() => deleteMutation.mutate(post.id)}>Delete</button>
```

- 객체를 반환하는 deleteMutation과 속성 함수인 mutate를 실행 props에서 받은 propsId가 무엇이든 상관업없이 실행
- mutate안의 post.id를 가져다가 변이 함수의 delete 포스트로 전달함

```jsx
{deleteMutation.isError && (
	<p style={{ color: "red" }}>Error deleting the post</p>
)}
{deleteMutation.isLoading && (
	<p style={{ color: "purple" }}>Deleting the post</p>
)}
{deleteMutation.isSuccess && (
	<p style={{ color: "green" }}>Post has (not) been deleted</p>
)}
```

- deleteMutation의 반환 객체에서 다른 속성들도 사용해 진행중인 작업의 인디케이션을 사용자에게 제공
- isSuccess: 이 API방식을 통해 페이지에서 데이터를 변경함을 통해 성공적인 변이를 제시할 수 없음
    - 변이가 성공적이었다는건 사용자에게 나타내 줄 수 있음
    - 삭제되지 않았다고 출력됨: 이 API에서는 데이터 변경 불가능
- 변이의 주기를 다룰 수 있음
- 반환 객체에 변이 속성을 실행하여 변이의 호출 조절 가능
- 쿼리에서 진행한 것과 유사한 방식으로 주기 처리 가능

→ 실행하면 보라색 글씨 → 초록색 글씨로 바뀜
