## 아이템 28. 유효한 상태만 표현하는 타입을 지향하기
> 효과적으로 타입을 설계하려면, 유효한 상태만 표현할 수 있는 타입을 만드는 것이 중요하다.

### 무효한 상태 
``` tsx
interface State {
	pageText: string;
	isLoading: boolean;
	error?: string;
}

function renderPage(state: State) {
	if (state.error)  {
		return `Error! Unable to load ${currentPage}: ${state.error}`;
	}
	else if (state.isLoading) {
		return `Loading ${currentPage}...`;
	}
	return `<h1>${currentPage}</h1>\n${state.pageText}`;
}
```

- 에러 상태와 로딩 상태를 정확히 구분하기 어렵고, 두 속성이 충돌할 수 있다. 
- isLoading이 true이면서 error 값이 존재하면 로딩 중인지 오류 발생 상태인지 명확히 구분할 수 없다. 
- 이는 정확히 구분이 어려운 무효한 상태다. 유효한 상태와 무효한 상태를 모두 표현하는 타입은 혼란을 초래하고 오류를 유발하기 쉽다.

### 유효한 상태
- 다음과 같이 무효한 상태를 허용하지 않도록 제대로 표현할 수 있다. 
- 모든 요청은 하나의 상태만 가진다.


``` ts
interface RequestPending {
	state: 'pending';
}

interface RequestError {
	state: 'error';
	error: string;
}

interface RequestSuccess {
	state: 'ok';
	pageText: string;
}

type RequestState = RequestPending | RequestError | RequestSuccess;

interface State {
	currentPage: string;
	requests: { [page: string]: RequestState };
}

function renderPage(state: State) {
	const { currentPage } = state;
	const requestState = state.requests[currentPage];
	switch (requestState.state) {
		case 'pending':
			return `Loading ${currentPage}...`;
		case 'error':
			return `Error! Unable to load ${currentPage}: ${requestState.error}`;
		case 'ok':
			return `<h1>${currentPage}</h1>\n${state.pageText}`;
	}
}
```

### tanstack query에서 타입

``` ts
const {
	data,
	dataUpdatedAt,
	error,
	errorUpdateCount,
	errorUpdatedAt,
	failureCount,
	failureReason,
	fetchStatus,
	isError,
	isFetched,
	isFetchedAfterMount,
	isFetching,
	isInitialLoading,
	isLoading,
	isLoadingError,
	isPaused,
	isPlaceholderData,
	isPreviousData,
	isRefetchError,
	isRefetching,
	isStale,
	isSuccess,
	refetch,
	remove,
	status
} = useQuery({...})
```

<img width="678" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/80238096/a6e21eda-6cbf-4b65-a7a6-d8323d2566b1">

- 에러 상태와 성공 상태를 유니온으로 나타내면서 속성 간의 충돌이 발생하지 않는다.


<img width="704" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/80238096/db0c5f28-b45a-4f23-a34f-44256ce614c8">

