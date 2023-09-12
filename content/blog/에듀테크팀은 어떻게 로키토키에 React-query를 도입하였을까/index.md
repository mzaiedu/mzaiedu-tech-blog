---
title: 에듀테크팀은 어떻게 [로키토키]에 React-query를 도입하였을까?
date: "2023-09-01T08:30:37.121Z"
---

**[ 로키토키 ] 에 React-query를 도입하여 마주한 문제들**
by Dongyeon


여느 다른 웹 서비스와 비슷하게 [ Rockie Talkie ] 에도 많은 서버 데이터를 이용하고 있습니다. 저희 에듀테크 프런트엔드팀은 **서버 데이터를 효율적이고 유연하게 관리**하기 위해 👉[React-query](https://tanstack.com/query/v4/docs/react/overview) 를 도입하였습니다. 에듀테크 프런트엔드팀이 [ Rockie Talkie ] 에 어떻게 React-query를 도입하였는지 풀어보도록 하겠습니다.

  

## 도입 배경


React의 전역 상태 관리를 위해 Redux를 채택했던 에듀테크팀에게는 큰 고민이 있었습니다. 

> *”클라이언트 데이터와 서버 데이터가 혼합되어 로직 관리가 어려워요* 😅*”*
*”RTK를 사용하고 있지만 그래도 보일러 플레이트가 너무 장황해요* 😖*”
”서버 데이터를 캐싱하기 너무 복잡해요* 😵*”*
> 

이러한 고민들은 **서버 데이터를 효율적이고 유연하게 관리할 수 있는 방식**에 대한 고민으로 이어졌습니다. 여러가지 대안 중 최종적으로 **React-query**와 **SWR**으로 선택지가 좁혀졌고, 다음과 같은 이유로 결국 React-query의 손을 들어주었습니다.

- 아직은 React-query의 다운로드 횟수가 더 높았고, 레퍼런스도 풍부했습니다.
- React-query의 쿼리에만 있는 유용한 옵션들이 있었습니다(예: select).
- React-query는 데이터 최적화와 가비지 컬렉팅을 지원하고 있었습니다.
- React-query은 공식적인 [🔗Devtools](https://tanstack.com/query/v4/docs/react/devtools) 을 지원하고 있었습니다.

작은 프로젝트에 React-query를 먼저 적용하여 사용해보았고, [ Rockie Talkie ] 에서 충분히 활용할 수 있을 것 같다는 판단에 React-query 도입을 최종 결정하였습니다.

React-query를 도입하며 기대한 효과는 다음과 같습니다.

1. **서버 데이터 비동기 처리를 기존의 RTK, Redux-Saga 조합보다 간편하게 처리한다.**
2. **서버 데이터를 더 간편하게 캐싱하여 빠르게 데이터를 가져온다.**
3. **서버 데이터와 간편하게 동기화한다.**

하지만 너무 빠르게 React-query를 도입하였던 걸까요. 저희 팀은 기존의 서버 데이터 처리 로직을 React-query로 교체하면서 아래와 같은 **문제 상황**들과 마주하였고, 해결해야 했습니다.

## 부딪히고 해결했던 문제들

React-query를 도입하면서 다양한 문제 상황들을 해결해야 했는데요, 크게 세 가지 문제 상황으로 정리해 보았습니다.

### 1. 캐싱의 함정

---

💥 문제: 캐싱해야 하는, 캐싱하지 않아야 하는 데이터의 구분, 캐싱 기한을 지정할 수 없는 상황

💡 해결: Infinity stale(cache)Time, remove(invalidate)Queries 활용, Query Key의 별도 관리

---

[ Rockie Talkie ] 에서 여러 종류의 서버 데이터를 사용하고 있었지만, 항상 클라이언트와 서버 데이터를 동기화해야 하는 것은 아니었습니다. 일례로 **사용자의 학습 단계에서는 서버 데이터와 클라이언트 데이터가 동기화되지 않아야 했습니다.** 사용자는 학습 단계에서 서비스를 사용하며 서버에 여러 데이터를 쌓게 되는데, 해당 데이터 중에는 실시간으로 이루어지는 학습 과정 자체에 영향을 줄 수 있는 요소들이 포함되어 있었습니다(예: 오늘의 학습량). 따라서 특정 시간동안은 데이터가 변경되지만 학습 과정 자체에 영향을 주지 않도록 두 데이터가 동기화되지 않았어야 했습니다.

쿼리를 통해 받아온 데이터를 캐싱해두면 서버와 동기화되지 않은(쿼리를 실행한 시점에 받아온) 데이터를 완벽하게 격리할 수 있었습니다. 하지만 관건은 **“얼마나 오래 캐싱해둘 것인지”** 였습니다.

React-query의 쿼리는 **staleTime과 cacheTime으로 데이터를 얼마나 오래 유지할 지를 결정**합니다. ms 단위로 값을 넘겨주는 거죠. 이렇게요!

```jsx
const { data: pageData } = usePageQuery({ pageId: pageListData?.page[index]?.pageId! }, {
        suspense: true,
        cacheTime: 60 * 60 * 1000,
        staleTime: 60 * 60 * 1000,
});
```

하지만 저희는 **사용자가 얼마나 오랫동안 학습을 진행할 지 알 턱이 없었습니다!** 😐

플로우 차트에 그 해답이 있었습니다: 시간은 모르지만 공간은 알 수 있다! 즉, 특정한 데이터가 얼마나 오래 캐시되어야 할지는 알 수 없지만(시간), 어느 영역(지점)에서 데이터가 동기화되어야 하는지(공간)는 알 수 있었습니다. 저희는 **cacheTime과 staleTime에 Infinity 를 넘기고, 해당 지점에서 removeQueries() 혹은 invalidateQueries()를 사용하여 서버 데이터와 동기화**하는 대안을 세웠습니다. 성공적이었습니다! **데이터를 쉽게 식별할 수 있도록 쿼리키를 별도로 관리했던 것**이 큰 도움이 되었습니다.

```jsx
// 쿼리 키를 별도로 관리했어요.
export const PageKeys = {
    all: ['pages'] as const,
    page: (filters: PK) => [...PageKeys.all, 'page', filters] as const,
    ...
}; 

// cacheTime과 staleTime을 Infinity로 설정하였어요.
const { data: pageData } = usePageQuery({ pageId: pageListData?.page[index]?.pageId! }, {
        suspense: true,
        cacheTime: Infinity,
        staleTime: Infinity,
});

// 이렇게 동기화했어요.
client.invalidateQueries(PageKeys.page({ pageId });
```
<br/>

### 2. 쿼리의 호출 여부와 순서가 중요하다면

---

💥 문제: 분기에 따라 쿼리의 호출 여부가 결정되는 상황, 의존적인 쿼리

💡 해결: enabled와 select 옵션 유기적 활용

---

[ Rockie Talkie ] 의 플로우차트는 복잡했고, 시나리오 별로 다양한 분기점이 있었습니다.

![플로우차트](./FlowChart.png)


분기점은 서버 데이터 관리에 있어 치명적인 빌런이었습니다. **조건에 따라 서버 데이터의 호출 순서가 결정되기도 했고, 아예 서버 데이터를 호출하지 않아야 하는 경우가 있었기 때문입니다!** 분기를 별도의 컴포넌트로 나눌 수 있으면 좋겠지만 항상 그런 것은 아니었기 때문에 이에 대한 해결책이 필요했습니다. 

저희는 쿼리 옵션에서 그 해답을 찾았습니다.  React-query에서 제공하는 useQuery에는 *enabled* 옵션을 통해 *쿼리를 비활성화*할 수 있었습니다. 해당 옵션을 이용하면 쿼리문을 호출하지 않을 수 있었고, 심지어는 데이터의 호출 순서도 제어할 수 있었습니다. **특정 쿼리의 enabled 옵션에 이전 쿼리의 데이터(결과)를 넣어주는 형태**로 말이죠!

```jsx
const { data: 데이터a } = use데이터AQuery({ ... });
const { data: 데이터b } = use데이터BQuery({ ... }, {
    enabled: !!데이터a.canBeCalled,
});
```

데이터의 후 가공이 필요한 경우에는 select 옵션을 이용하기도 했습니다. 뿐만 아니라 클라이언트에서 적합하게 사용할 수 있도록 서버 데이터를 후처리할 수 있다는 점은 유사한 **서버 데이터와 클라이언트의 데이터의 형태를 독립적으로 분리할 수 있다는 점에서 데이터의 의존성을 낮추기**도 하였습니다!

```jsx
const { data: is조건A } = use데이터AQuery({ ... }, {
		select: data => data.data.contains("특정키"),
});

// 쿼리에서 데이터를 가공해요.
const { data: 데이터b } = use데이터BQuery({ ... }, {
    enabled: is조건A,
		select: data => data.data.map((datum, idx) => {
				return ({
						id: `datum_${idx}`,
						속성1: Math.floor(datum.value),
				});
		});
});
```

React-query의 query와 mutation에는 이외에도 정말 다양한 옵션들과 리턴값이 존재했습니다(디테일하게 떠먹여주는 React-query의 배려심이란… 😂). 

<br/>

### 3. 어떻게 관리할 것인가

---

💥 문제: API 증가에 따른 쿼리 및 뮤테이션 관리 복잡성

💡 해결: 폴더 구조 개편, 쿼리 유연화

---

서버 측 데이터와 API가 적었던 초기에 저희는 쿼리들을 이렇게 관리하고 있었습니다. 

- 엔티티를 기반으로 폴더를 구분하였습니다.
- *index.ts*에는 QueryKey, Fetch 함수(Axios), Fetch 함수와 관련된 Type을 작성하였습니다.
- *queries.ts* 파일에는 *index.ts*에서 선언한 것들을 사용하여 커스텀 쿼리를 작성하였습니다.

이렇게 말이죠. (각 폴더를 누르면 내용물을 확인할 수 있습니다.)

- 📁 api
    - 📁 activities
        - index.ts
            
            ```jsx
            // index.ts
            // key
            export const 키 = {
                all: ['activity'] as const,
                PK키: (pk: number) => [...키.all, { PK키: pk }] as const,
            }
            
            // fetch
            export type 리퀘스트파라미터 = {
                파람: number,
            }
            export const Fetch함수 = (param: 리퀘스트파라미터) => {
                return API().post<리턴>('/api', param);
            		// API()는 Axios 인스턴스예요.
            }
            ```
            
        - queries.ts
            
            ```jsx
            // queries.ts
            export const 커스텀한useQuery= ({
                파람,
            }: 리퀘스트파라미터): UseQueryResult<리턴>, AxiosError> => {
                return useQuery({
                    queryKey: 키들.PK키(파람),
                    queryFn: () => Fetch함수({ 파람 }),
            				...
                })
            }
            ```
            
    - 📁 books
        - index.ts
        - queries.ts

**Poc(Proof of concept)를 마치고 본격 개발이 시작되면서 API 수가 기하급수적으로 증가하였습니다.** API수가 증가하면서 쿼리와 뮤테이션, 쿼리 키와 타입들을 **체계적으로 관리해야 할 필요성**을 느꼈고, 기존의 관리 구조를 몇 차례 뒤집었습니다. 해당 방식의 단점이 명확했기 때문이죠.

- **가독성의 문제**: 엔티티에 API가 추가되는 경우 원하는 쿼리를 찾기가 너무 어렵고, 관련된 타입과 Fetch 함수와 Type과의 연관성이 명확하게 읽히지 않았습니다.
- **관리의 문제**: 중복된 fetch함수와 Type을 작성하는 일이 많아졌습니다.
- **활용의 문제**: 커스텀 쿼리의 자유도가 낮아 다양하게 활용이 어려웠습니다.
<br/>

> 💡 그럼 어떻게 관리하지?😕
> 
<br/>

👉 **1. 구조를 바꿨습니다.**

- 📁 api
    - 📁 books
        - 📁 getBooks
            - api.ts
            - hook.ts
            - type.ts
        - 📁 postBooks
            - api.ts
            - hook.ts
            - type.ts
        - key.ts
        - types.ts

- **엔티티는 그대로 유지했습니다.** 다만, REST API와 HTTP API가 혼재되어 데이터의 속성에 따라서 자체적으로 구분점을 잡아야 했습니다.
- **쿼리 한 개를 관리하는 폴더(*{메서드}{엔티티})*를 생성하였습니다.**
- 최상위단의 key.ts에서는 해당 엔티티의 쿼리 키를 관리했습니다.
- 최상위단의 types.ts에서는 해당 엔티티에서 공통적으로 사용하는 타입을 관리했습니다
    
    ```jsx
    // api/books/types.ts
    export type Book {
    	bookId: number;
    	title: string;
    	... 
    }
    ```
    
- **각 쿼리 폴더에서 api.ts 와 hook.ts를 통해 각각 fetch함수와 훅을 관리하였습니다.**
- **fetch함수가 사용하는 RequestType과 ResponseType은 하위단의 type.ts에서 관리하였습니다.**
    
    ```jsx
    // api/books/getBooks/types.ts
    export type RequestType = 
    	Pick<Book, 'bookId'>;
    
    export type ResponseType = 
    ResponseWithCode<{
        book: Book;
    }>;
    // ResponseWithCode: 커스텀 타입
    ```
    

👉 **2. 커스텀 쿼리의 자유도를 높이기 위해 옵션을 분리했습니다.**

- **QueryOptions.ts**

```jsx
export type QueryOptions<R = any> = 
	UseQueryOptions<AxiosResponse<R>, AxiosError, R>;
```

- **QueryOptions은 외부에서 전달해요.**

```jsx
// api/books/getBooks/hook.ts
export const 커스텀쿼리 = (
    params: RequestType,
    **options?: QueryOption<ResponseType>**,
): UseQueryResult<ResponseType> => {
    return useQuery({
        queryKey: 키.PK(params),
        queryFn: () => Fetch함수(params),
        select: data => data.data,
        enabled: parser(params),
        **...options,**
    });
};
```

- **공통적인 옵션은 client에서 관리해요.**

```jsx
// queryClient.ts
const client = new QueryClient({
    defaultOptions: {
        queries: {
            retry: 0,
            refetchOnWindowFocus: true,
        },
    },
});
export default client;
```

두 가지 방향성에서 개선 작업을 실시했고, 성공적으로 구조를 개선했습니다! 가독성, 관리, 활용의 세 꼭지에서 이전보다 훨씬 발전되고 안정적으로 React-query를 사용할 수 있게 되었습니다.

## 앞으로는?

[ Rockie Talkie ] 의 일부 서비스에 Next.js를 도입하면서 다시 고민이 생겼습니다. 

> ‘*서버 컴포넌트로 만들면 React-query… 필요하지 않을 수도 있겠는데…?* 🙄*’*
> 

데이터를 서버에서 요청한다면(data fetching) React-query는 필요하지 않겠죠. 실제로 이 🗞️[아티클](https://tkdodo.eu/blog/you-might-not-need-react-query)에서도 그 핵심을 지적하고 있습니다. 

> *If you're starting a new application, and you're using a mature framework like [Next.js](https://nextjs.org/) or [Remix](https://remix.run/) that has a good story around data fetching and mutations, **you probably don't need React Query**.*
> 

에듀테크팀은 과연 [ Rockie Talkie ] 서비스를 완전히 Next.js로 마이그레이션 하게 될까요? 그 시점에 저희의 React-query는 생존할 수 있을까요? 🤪 **아직은 잘 모르겠습니다**. 🤪 마이그레이션을 고민하고는 있지만 항상 트레이드오프는 있으니까요.

![트레이드오프는 있어요](./TradeOff.jpeg)
