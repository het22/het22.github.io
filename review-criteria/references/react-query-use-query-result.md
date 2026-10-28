---
criteria: [1]
---

# react-query `UseQueryResult`

`status: 'pending' | 'error' | 'success'`가 판별자다. `success`에서만 `data: TData`가 확정되고, `error`에서만 `error: TError`가 non-null이다. `pending`에서 `data`는 `undefined`다.

```ts
const query = useQuery({ queryKey, queryFn })

if (query.status === 'pending') return <Spinner />
if (query.status === 'error') return <ErrorView error={query.error} />
return <List items={query.data} />
```

`isLoading`, `isError`, `isSuccess`는 같은 정보를 boolean으로 푼 편의 속성이다. 구조분해하면 판별자와 필드의 연결이 끊긴다.

```ts
const { isLoading, data } = useQuery({ queryKey, queryFn })
if (isLoading) return <Spinner />
return <List items={data} /> // data: TData | undefined
```

`isLoading`이 false여도 타입 체커는 `data`가 확정됐다고 보지 않는다. 라이브러리가 타입으로 막아둔 상태를 소비처가 다시 연다. 판별자로 분기하거나 구조분해를 하지 않는다.
