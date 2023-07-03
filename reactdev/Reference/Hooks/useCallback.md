# useCallback

useCallback은 리랜더 사이 함수를 캐시해주는 훅이다.

```jsx
const cachedFn = useCallback(fn, dependencies);
```

## Reference

### useCallback(fn, dependencies)

컴포넌트의 최상단에 useCallback을 호출해 리랜더 사이에 함수를 캐시한다.

```jsx
import { useCallback } from 'react';

export default function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);
```

**Parameters**

- fn: 캐시하고 싶은 함수의 값. 어떤 인자도 받을 수 있으며 어떤 값도 리턴할 수 있다. 리액트는 초기 랜더시 함수를 리턴한다. 다음 랜더에서 의존성이 변경되지 않을 경우 동일한 함수를 리턴한다. 현재 랜더에 넘긴 함수를 리턴할 것이며, 후에 재사용될 수 있다. 리액트는 함수를 호출하지 않을 것이다. 함수는 리턴되고 언제 호출할지 정할 수 있다.

- dependencies: 함수 내부 코드에서 참조되는 모든 반응값. 반응값은 컴포넌트 몸체에 선언된 프랍, 상태, 모든 변수를 포함한다. 린터가 설정되어 있다면, 맞는 의존성을 알려준다. 의존성 배열은 [dep1, dep2, dep3]와 같이 작성되어야 한다. 리액트는 Object.is 비교를 통해 의존성을 비교한다.

**Returns**

첫 랜더시, useCallback은 전달된 함수를 리턴한다.

후속 랜더시, 이전 랜더에서 이미 저장된 함수를 리턴하거나, 이번 랜더에 넘긴 함수를 리턴한다.

**Caveats**

- 리액트는 특별한 이유가 아니라면 캐시된 함수를 버리지 않는다. 예를 들어, 개발 환경 시 리액트는 컴포넌트 파일이 수정되면 캐시를 버린다. 개발, 배포 환경 시, 리액트는 초기 랜더가 정지 되면 캐시를 버린다. 후에, 리액트는 캐시를 버리는 것이 장점인 기능을 추가할 수 있다. 예를 들어, 리액트가 시각화된 빌트인 리스트를 추가한다면, 테이블 뷰포트에서 벗어나면 캐시를 버리는 것이 당연하다. useCallback 성능 최적화에 의존한다면 알맞는 기대이다. 아닐 경우, 상태나 레프가 더 알맞다.

## Usage

### Skipping re-rendering of components

랜더 최적화를 하기 위해, 자식 컴포넌트에 넘겨주는 함수를 캐시할 필요가 있다.
문법을 먼저 보고, 어떤 상황에서 유용한지 보자.

리랜더 사이에 함수를 캐시하기 위해, useCallback 훅으로 선언을 감싼다.

```jsx
import { useCallback } from 'react';

function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);
  // ...
```

useCallback은 두 가지가 필요하다.

1. 리랜더 시 캐시할 함수
2. 함수 내부에 사용되는 컴포넌트의 모든 값

초기 랜더 시, useCallback에서 리턴되는 함수는 전달한 함수이다.

이후 랜더에서, 리액트는 이전 랜더와 의존성을 비교한다. 만약 변경된 것이 없다면, useCallback은 동일한 함수를 리턴한다. 아닐 경우, 현재 랜더에 넘겨준 함수를 리턴한다.

즉, useCallback은 의존성이 변경될 때까지 캐시된 함수를 반환한다.

handleSubmit 함수를 ProductPage에서 ShippingForm으로 전달한다고 하자.

```jsx
function ProductPage({ productId, referrer, theme }) {
  // ...
  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
```

theme 프랍을 토글 하면, 앱이 잠시 멈추는 것을 경험할 것이다. 하지만 ShippingForm을 제거하면 빨라진 것을 느낄 것이다. 이것은 ShippingForm을 최적화 해야한다는 것을 뜻한다.

기본적으로, 컴포넌트가 리랜더 되었을 시, 리액트는 모든 자식을 재귀적으로 리랜더한다. theme가 변경되어 ProductPage가 리랜더 되고 ShippingForm 역시 랜더되는 이유다. 리랜더시 계산할 것이 적다면 이것은 괜찮다. 하지만, 리랜더가 느리다면, 프랍이 같다면 ShippingForm을 memo로 감싸 리랜더를 무시할 수 있다.

```jsx
import { memo } from "react";

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  // ...
});
```

이런 변화로, ShippingForm는 모든 프랍이 이전 랜더와 같을 경우 리랜더를 무시한다.
이것이 함수를 캐시하는 것이 중요한 이유다. handleSubmit이 useCallback 없이 선언되었다고 생각해보자.

```jsx
function ProductPage({ productId, referrer, theme }) {
  // Every time the theme changes, this will be a different function...
  function handleSubmit(orderDetails) {
    post("/product/" + productId + "/buy", {
      referrer,
      orderDetails,
    });
  }

  return (
    <div className={theme}>
      {/* ... so ShippingForm's props will never be the same, and it will re-render every time */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

자바스크립트에서 함수는 항상 다른 함수이다. 객체 리터럴이 항상 새로운 객체를 생성하는 것과 동일하다.
일반적으로, 이것은 문제가 되지않지만, ShippingForm의 프랍이 항상 같지 않고 memo 최적화가 동작하지 않는 것을 뜻한다.

```jsx
function ProductPage({ productId, referrer, theme }) {
  // Tell React to cache your function between re-renders...
  const handleSubmit = useCallback(
    (orderDetails) => {
      post("/product/" + productId + "/buy", {
        referrer,
        orderDetails,
      });
    },
    [productId, referrer]
  ); // ...so as long as these dependencies don't change...

  return (
    <div className={theme}>
      {/* ...ShippingForm will receive the same props and can skip re-rendering */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

handleSubmit을 useCallback으로 감싸는 것은, 리랜더 시 함수가 동일하다는 것을 보장한다.
특정한 이유가 아닌 경우 useCallback으로 함수를 감싸지 않아도 된다. 이 예에서, memo로 감싸진 컴포넌트에 전달 해, 리랜더를 무시한다.

#### Note

오직 useCallback을 성능 최적화로만 의존해야 한다. 만약, 없이 사용 시 코드가 동작하지 않는 다면, 문제를 찾고 고치는 게 먼저다. 그 후 useCallback을 다시 추가해라.

#### DEEP DIVE - How is useCallback related to useMemo?

useMemo를 자주 볼 것이다. 그들은 자식 컴포넌트를 최적화하기에 유용하다.
그들은 전달한 것을 메모이즈 해준다.

```jsx
import { useMemo, useCallback } from "react";

function ProductPage({ productId, referrer }) {
  const product = useData("/product/" + productId);

  const requirements = useMemo(() => {
    // Calls your function and caches its result
    return computeRequirements(product);
  }, [product]);

  const handleSubmit = useCallback(
    (orderDetails) => {
      // Caches your function itself
      post("/product/" + productId + "/buy", {
        referrer,
        orderDetails,
      });
    },
    [productId, referrer]
  );

  return (
    <div className={theme}>
      <ShippingForm requirements={requirements} onSubmit={handleSubmit} />
    </div>
  );
}
```

차이점은 어떤 것을 캐시할지이다.

- useMemo는 함수 호출의 결과를 캐시한다. 예에서, 이것은 computeRequirements(product)의 호출 결과를 캐시한다. product가 변경되기 전까진 변경되지 않는다. 이것은 requirements 객체를 불필요한 리랜더 없이 ShippingForm에 넘겨줄 수 있다. 필요시, 리액트는 함수를 호출하고 랜더 도중에 계산된 값을 전달한다.
- useCallback은 함수 그 자체를 캐시한다. useMemo와 다르게 이것은 제공된 함수를 호출하지 않는다. 대신에 productId 혹은 referrer가 변경되지 않을 경우 handleSubmit를 변경하지 않는다. 이것은 handleSubmit를 불필요한 리랜더 없이 ShippingForm에 넘겨줄 수 있다. 유저가 제출하기 전까진 함수가 실행되지 않는다.

#### DEEP DIVE - Should you add useCallback everywhere?

만약 당신의 앱이 공식 문서와 같이 상호작용이 조잡하다면, 메모이제이션은 불필요하다. 반면, 앱이 드로잉 에디터와 같이 상호작용이 세분화 되었다면, 메모이제이션이 매우 도움을 줄 것이다.

매우 적은 경우 useCallback으로 함수를 캐시하는 것이 유용하다.

- memo로 감싸진 컴포넌트에 프랍으로 넘길 경우. 값이 변경되지 않을 경우 리랜더를 무시하기 위해. 메모이제이션은 의존성이 변경되었을 때만, 리랜더를 한다.

- 함수를 다른 훅의 의존성으로 넘길 경우. 예를 들어, useCallback에 감싸진 다른 함수가 의존하는 경우, useEffect가 의존하는 경우

위 경우 말고 useCallback으로 감싸는 것은 어떤 이점도 없다. 또 이렇게 하는 것은 큰 해가 되지 않으므로, 일부 팀은 개별 사례을 생각하지 않고 가능한 많이 메모한다. 단점은 코드가 읽기 어렵다는 것이다. 또한, 모든 메모이제이션이 효과적인 것은 아니다. 항상 새로운 값이면, 전체 컴포넌트의 메모이제이션을 중단하기에 충분하다.

useCallback가 함수를 생성하는 것을 막는 게 아니라는 것을 명심해라. 항상 함수는 생성되지만, 리액트는 무시하고 캐시된 함수를 준다.

다음과 같은 원칙을 따르면, 굳이 메모이제이션을 하지 않아도 된다.

1. 컴포넌트가 다른 컴포넌트로 감싸져 자식을 받을 경우. 래퍼 컴포넌트가 가진 상태로 변경이 되면, 리액트는 자식의 리랜더가 불필요하다는 것을 안다.
2. 지역 상태를 선호하고 필요하지 않은 곳까지 상태를 올리지마라. 폼 같은 과도 상태 또는 전역 상태 라이브러리를 트리의 최상단에 소유하는 것을 피해라.
3. 랜더링 로직을 순수하게 유지해라. 만약 컴포넌트를 리랜더하는 것이 문제나 시각적으로 눈에띄는 인공물을 만드면, 이것은 컴포넌트에 버그가 있다는 것이다. 메모이제이션 대신 버그를 고쳐라.
4. 상태를 변경하는 불필요한 이펙트를 제거해라. 리액트의 대부분 성능 문제는 이펙트에 기반해 상태를 변경해 랜더 체인을 일으키는 것이다.
5. 불필요한 의존성을 이펙트에서 제거해라. 예를 들어, 메모이제이션 대신 함수나 객체를 이펙트 내부에 작성하는 것이 더 간단하다.

만약 특정 상호 작용이 여전히 래기하다면, 리액트 데브 툴의 프로파일러를 사용해 어떤 컴포넌트가 메모이제이션 해야하는지와 어디에 메모이제이션이 필요한지 봐라.
이 원칙은 당신의 컴포넌트를 더 디버그 하기 쉽게 해준다.

### Updating state from a memoized callback

가끔, 메모이즈드 된 콜백에서 이전 상태에 기반해 상태를 변경할 경우가 있다.
handleAddTodo 함수는 다음 todos를 todos에서 계산하기에 의존성으로 추가한다.

```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback((text) => {
    const newTodo = { id: nextId++, text };
    setTodos([...todos, newTodo]);
  }, [todos]);
  // ...
```

최대한 적은 의존성으로 함수를 메모이즈하길 원한다. 다음 상태를 위해 상태를 읽을 경우, 의존성을 지우고 업데이터 함수를 추가할 수 있다.

```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback((text) => {
    const newTodo = { id: nextId++, text };
    setTodos(todos => [...todos, newTodo]);
  }, []); // ✅ No need for the todos dependency
  // ...
```

todos를 의존성에 두고 읽는 대신, (todos => [...todos, newTodo])를 리액트에 전달해 상태를 업데이트할 수 있다.

### Preventing an Effect from firing too often

이펙트 내부에서 함수를 호출하길 원할 수 있다.

```jsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  function createOptions() {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    };
  }

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    // ...

```

이것은 문제를 일으킨다. 모든 반응 값은 이펙트의 의존성이어야한다.
하지만, createOptions를 의존성으로 선언하면, 이것은 이펙트가 챗룸을 지속적으로 재연결하도록 한다.

```jsx
useEffect(() => {
  const options = createOptions();
  const connection = createConnection();
  connection.connect();
  return () => connection.disconnect();
}, [createOptions]); // 🔴 Problem: This dependency changes on every render
// ...
```

이것을 해결하기 위해, useCallback으로 감쌀 수 있다.

```jsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const createOptions = useCallback(() => {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    };
  }, [roomId]); // ✅ Only changes when roomId changes

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]); // ✅ Only changes when createOptions changes
  // ...
```

이것은 createOptions 함수가 roomId가 같을 경우 동일한 함수임을 보장한다. 하지만, 의존성에서 함수를 제거하는 것이 더 좋은 방법이다.

```jsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    function createOptions() { // ✅ No need for useCallback or function dependencies!
      return {
        serverUrl: 'https://localhost:1234',
        roomId: roomId
      };
    }

    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ Only changes when roomId changes
  // ...
```

### Optimizing a custom Hook

커스텀 훅을 만들 경우, 어떤 함수든 useCallback으로 감싸는 것을 추천한다.

```jsx
function useRouter() {
  const { dispatch } = useContext(RouterStateContext);

  const navigate = useCallback(
    (url) => {
      dispatch({ type: "navigate", url });
    },
    [dispatch]
  );

  const goBack = useCallback(() => {
    dispatch({ type: "back" });
  }, [dispatch]);

  return {
    navigate,
    goBack,
  };
}
```

훅을 사용하는 곳에서 코드로 최적화 될 수 있다.

## Troubleshooting

### Every time my component renders, useCallback returns a different function

두 번째 인자로 의존성을 명시하는 것을 명심해라.

만약 의존성 배열을 까먹는다면, useCallback은 새로운 함수를 매번 반환할 것이다.

```jsx
function ProductPage({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }); // 🔴 Returns a new function every time: no dependency array
  // ...
```

두 번째 인자로 의존성을 추가하는 버전은 다음과 같다.

```jsx
function ProductPage({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]); // ✅ Does not return a new function unnecessarily
  // ...
```

만약 이것이 도움이 되지 않는다면, 의존성이 이전 랜더와 같지 않다는 것이다. 콘솔로 디버그할 수 있다.

### I need to call useCallback for each list item in a loop, but it’s not allowed

메모로 감싸진 차트 컴포넌트가 있다고 생각하자. 당신은 리포트리스트 컴포넌트가 리랜더될 시 차트 컴포넌트의 리랜더가 스킵되길 원한다. 하지만 useCallback을 반복문 안에서 사용할 수 없다.

```jsx
function ReportList({ items }) {
  return (
    <article>
      {items.map((item) => {
        // 🔴 You can't call useCallback in a loop like this:
        const handleClick = useCallback(() => {
          sendReport(item);
        }, [item]);

        return (
          <figure key={item.id}>
            <Chart onClick={handleClick} />
          </figure>
        );
      })}
    </article>
  );
}
```

각 개별 아이템으로 추출해 useCallback을 추가할 수 있다.

```jsx
unction ReportList({ items }) {
  return (
    <article>
      {items.map(item =>
        <Report key={item.id} item={item} />
      )}
    </article>
  );
}

function Report({ item }) {
  // ✅ Call useCallback at the top level:
  const handleClick = useCallback(() => {
    sendReport(item)
  }, [item]);

  return (
    <figure>
      <Chart onClick={handleClick} />
    </figure>
  );
```

useCallback 대신에, 리포트 컴포넌트를 memo로 감쌀 수 있다. 만약 아이템 프랍이 변경되지 않으면, 리포트는 리랜더를 무시하고, 차트 역시 리랜더를 무시한다.

---

## What I Learned

메모이제이션을 통한 최적화에 익숙하지 않아서 어떤 경우에 사용이 되어야하는지 잘 몰랐는데, 문서를 읽으며 어떤 경우에 사용 해야하는지 알게되었다.

메모된 컴포넌트에서 사용하는 것과 다른 훅에 의존성으로 넘기는 경우에 사용해야 하는 것을 알게되었다.
또 커스텀 훅의 경우 useCallback의 사용이 추천된다고 한다.
