# useMemo

useMemo 랜더 사이에 계산된 값을 캐시 해주는 훅이다.

```jsx
const cachedValue = useMemo(calculateValue, dependencies);
```

## Reference

### useMemo(calculateValue, dependencies)

useMemo를 컴포넌트의 최상단에 호출해 랜더 사이에 계산을 캐시한다.

```jsx
import { useMemo } from "react";

function TodoList({ todos, tab }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

**Parameters**

- calculateValue: 캐시하고 싶은 값을 계산하는 함수. 이것은 순수해야 하며, 인자를 받지 않고, 어떤 값을 리턴해야한다. 리액트는 초기 랜더에 함수를 호출한다. 그 다음 랜더 시, 의존성이 변하지 않았다면 같은 값을 반환한다. 이것은 계산된 값을 호출하고 결과를 반환해, 나중에 재사용할 수 있도록 저장한다.
- dependencies: 값을 계산하는 코드 내부에서 사용되는 모든 반응 값.

**Returns**

첫 랜더 시, useMemo는 계산된 값을 인자 없이 호출한 결과를 반환한다.

다음 랜더 시, 이것은 지난 랜더에 저장한 값을 반환할 것이다. 또는 계산된 값을 재호출해 그 결과를 반환한다.

## Usage

### Skipping expensive recalculations

랜더 사이에 계산을 캐시하기 위해, useMemo를 컴포넌트 최상단에 호출해 감싼다.

```jsx
import { useMemo } from "react";

function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

useMemo에 두 가지를 전달해야한다.

1. 계산 함수, ()=>와 같이 인자를 받지 않고 계산 값을 반환하는 함수.
2. 의존성 배열, 계산에서 사용되는 컴포넌트의 모든 값.

첫 랜더 시, useMemo로 얻는 값은 계산 함수의 결과이다.

그 이후 랜더부터, 리액트는 의존성을 이전 랜더의 의존성과 비교한다. 만약 의존성이 변경되지 않았다면, useMemo는 전에 계산된 결과를 반환한다. 그게 아니라면, 리액트는 계산을 재실행하고 새로운 값을 반환한다.

즉 useMemo는 의존성이 바뀌기 전까지 리랜더 사이의 계산을 캐시한다.

언제 유용한지 살펴보자.

기본적으로, 리액트는 컴포넌트가 리랜더될 시 모든 몸체를 재실행한다. 예를 들어, 투두리스트의 상태 또는 프랍이 변경되면, filterTodos 함수는 재실행된다.

```jsx
function TodoList({ todos, tab, theme }) {
  const visibleTodos = filterTodos(todos, tab);
  // ...
}
```

보통, 계산은 매우 빠르기에 문제가 되지 않는다. 하지만, 큰 배열의 필터링 또는 변경, 비싼 연산일 때 데이터가 변하지 않으면 계산을 무시하고 싶다. 만약 todos와 tab이 이전 랜더와 같다면, useMemo로 감싸 이미 계산된 visibleTodos 재사용할 수 있다.

이런 타입의 캐싱을 메모이제이션이라고 한다.

#### Note

useMemo는 성능 최적화에만 사용해야한다. 만약 이것 없이는 코드가 실행되지 않으면 우선 고친뒤 useMemo로 다시 감싸라.

#### DEEP DIVE - How to tell if a calculation is expensive?

일반적으로, 수천 개가 넘는 객체를 만들거나 반복하는 것은 비싸지 않다.
만약 좀 더 확신을 얻고 싶다면, 콘솔로그로 시간의 소요를 잴 수 있다.

```jsx
console.time("filter array");
const visibleTodos = filterTodos(todos, tab);
console.timeEnd("filter array");
```

측정 중인 상호 작용을 실행해라. 그러면 아마 filter array: 0.15ms와 같은 것을 로그에서 볼 것이다.
만약 로그의 평균이 상당할 경우(1ms 또는 그 이상) 계산을 메모하는 것이 맞다.
실험으로, useMemo로 감싸 실행 시간이 줄었는지 확인할 수 있다.

```jsx
console.time("filter array");
const visibleTodos = useMemo(() => {
  return filterTodos(todos, tab); // Skipped if todos and tab haven't changed
}, [todos, tab]);
console.timeEnd("filter array");
```

useMemo는 첫 랜더를 빠르게 하지 않는다. 이것은 오직 불필요한 동작만을 무시한다.

당신의 머신이 유저보다 빠르다는 것을 명심하고, 인공적으로 성능을 늦춰서 테스트 해야한다. 예를 들어, 크롬은 cpu throttling 옵션을 제공한다.

또 개발 환경에서는 정확한 결과를 제공하지 않는다. 정확한 결과를 얻기 위해선 배포 환경에서 테스트를 진행해라.

### Skipping re-rendering of components

몇 경우에서, useMemo는 자식 컴포넌트의 리랜더를 최적화 해준다.
투두리스트 컴포넌트가 visibleTodos를 프랍으로 리스트 컴포넌트에 전달한다고 생각해보자.

```jsx
export default function TodoList({ todos, tab, theme }) {
  // ...
  return (
    <div className={theme}>
      <List items={visibleTodos} />
    </div>
  );
}
```

theme 프랍을 토글하는 것이 앱을 잠시 멈춘다는 것을 느낄것 이다. 하지만 리스트 컴포넌트를 없앤다면 빠르게 느껴진다. 이것은 리스트 컴포넌트를 최적화하는 게 효과가 있다는 뜻이다.

기본적으로, 컴포넌트가 리랜더 되면, 리액트는 모든 자식을 재귀적으로 리랜더한다.
이것은 투두리스트가 다른 테마로 리랜더할 시, 리스트 컴포넌트도 리랜더 되는 이유다.
리랜더에 많은 연산이 들지 않는 컴포넌트는 괜찮다. 하지만, 리랜더가 느리다는 것을 확인하면, 프랍이 같으면 리랜더를 무시하기 위해 memo로 감쌀 수 있다.

```jsx
import { memo } from "react";

const List = memo(function List({ items }) {
  // ...
});
```

이 변경으로, 리스트는 지난 랜더와 프랍이 같을 경우 리랜더를 무시한다.
이것이 계산이 중요한 이유다. useMemo 없이 visibleTodos를 계산한다고 생각해보자.

```jsx
export default function TodoList({ todos, tab, theme }) {
  // Every time the theme changes, this will be a different array...
  const visibleTodos = filterTodos(todos, tab);
  return (
    <div className={theme}>
      {/* ... so List's props will never be the same, and it will re-render every time */}
      <List items={visibleTodos} />
    </div>
  );
}
```

위의 예에서, filterTodos 함수는 객체 리터럴이 항상 새로운 객체를 생성하는 것 처럼 항상 다른 배열을 생성한다. 일반적으로, 이것은 문제가 되지 않지만, 리스트 프랍이 항상 다르다는 것이고 meme 최적화가 동작하지 않는다는 것이다. 이것은 useMemo가 유용한 이유다.

```jsx
export default function TodoList({ todos, tab, theme }) {
  // Tell React to cache your calculation between re-renders...
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab] // ...so as long as these dependencies don't change...
  );
  return (
    <div className={theme}>
      {/* ...List will receive the same props and can skip re-rendering */}
      <List items={visibleTodos} />
    </div>
  );
}
```

visibleTodos를 useMemo로 감싸는 것으로, 의존성이 바뀌기 전까지 같은 값임을 보장한다.
특별한 이유가 아니면 useMemo로 계산을 감쌀 필요가 없다. 이 예에서, memo로 감싸진 컴포넌트에 전달하고 리랜더를 무시하기 위해 사용했다.

#### DEEP DIVE - Memoizing individual JSX nodes

리스트를 memo로 감싸는 대신, 리스트 jsx 노드 자체를 useMemo로 감쌀 수 있다.

```jsx
export default function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  const children = useMemo(() => <List items={visibleTodos} />, [visibleTodos]);
  return <div className={theme}>{children}</div>;
}
```

동작은 동일하다. 만약 visibleTodos가 변경되지 않는 다면, 리스트는 리랜더 하지않느다.

<List items={visibleTodos} />와 같은 jsx 노드는 { type: List, props: { items: visibleTodos } }와 같은 객체이다.
객체를 생성하는 것은 매우 저렴하지만, 리액트는 그 컨텐츠가 이전과 같은지 알지 못한다. 이것이 기본적으로 리액트가 리스트 컴포넌트를 리랜더하는 이유다.

하지만, 리액트가 이전과 동일한 jsx를 본다면, 컴포넌트 리랜더를 일으키지 않는다.
이것은 jsx 노드가 불변하기 때문이다. jsx 노드 객체가 변경되지 않았기에, 리액트는 리랜더를 무시해도 안전하다는 것을 안다. 하지만, 이 경우 단지 코드가 동일한 것이 아닌 노드 그 자체가 동일한 객체이다.
이것이 useMemo가 예에서 하는 일이다.

jsx 노드를 useMemo로 감싸는 것은 불편하다. 예를 들어, 이것을 조건부로 처리할 수 없다. 그래서 컴포넌트를 memo로 감사는 것이 일반적인 이유다.

### Memoizing a dependency of another Hook

컴포넌트 몸체에서 직접적으로 계산되는 객체가 있다고 생각하자.

```jsx
function Dropdown({ allItems, text }) {
  const searchOptions = { matchMode: 'whole-word', text };

  const visibleItems = useMemo(() => {
    return searchItems(allItems, searchOptions);
  }, [allItems, searchOptions]); // 🚩 Caution: Dependency on an object created in the component body
  // ...
```

객체에 의존하는 것은 메모이제이션의 관점을 어기는 것이다. 컴포넌트가 리랜더 되면 모든 컴포넌트 몸체는 재실행 된다. searchOptions 객체를 생성하는 코드 역시 항상 재실행 된다. searchOptions가 useMemo의 의존성이기에, 호출 되고, 매번 다르다. 리액트는 의존성이 다르다는 것을 알기에 searchItems을 매번 재계산한다.

이것을 고치기 위해, searchItems 그 자체를 의존성에 전달하기 전에 메모이제이션할 수 있다.

```jsx
function Dropdown({ allItems, text }) {
  const searchOptions = useMemo(() => {
    return { matchMode: 'whole-word', text };
  }, [text]); // ✅ Only changes when text changes

  const visibleItems = useMemo(() => {
    return searchItems(allItems, searchOptions);
  }, [allItems, searchOptions]); // ✅ Only changes when allItems or searchOptions changes
  // ...
```

위의 예에서, text가 변경되지 않으면 searchOptions 역시 변경되지 않는다.
하지만, 더 좋은 방법은 searchOptions을 useMemo 계산 함수 내부에서 선언하는 것이다.

```jsx
function Dropdown({ allItems, text }) {
  const visibleItems = useMemo(() => {
    const searchOptions = { matchMode: 'whole-word', text };
    return searchItems(allItems, searchOptions);
  }, [allItems, text]); // ✅ Only changes when allItems or text changes
  // ...
```

이제 계산은 text에 의존한다.

### Memoizing a function

memo로 감싸진 폼 컴포넌트가 있다고 생각하자. 함수를 프랍으로 전달하고 싶다.

```jsx
export default function ProductPage({ productId, referrer }) {
  function handleSubmit(orderDetails) {
    post("/product/" + productId + "/buy", {
      referrer,
      orderDetails,
    });
  }

  return <Form onSubmit={handleSubmit} />;
}
```

{}가 다른 객체를 생성하는 것처럼, 함수 선언 또는 표현 역시 매 리랜더시 다른 함수이다. 함수를 만드는 그 자체는 문제가 되지 않는다. 이것은 피할 것이 아니다. 하지만, 폼 컴포넌트가 메모이즈드 되었다면, 아마도 프랍이 변경되지 않으면 리랜더를 무시하고 싶을 것이다.
항상 다른 프랍은 메모이제이션의 관점에 부합하지 않는다.

useMemo로 함수를 메모하기 위해선, 계산 함수가 다른 함수를 리턴해야한다.

```jsx
export default function Page({ productId, referrer }) {
  const handleSubmit = useMemo(() => {
    return (orderDetails) => {
      post("/product/" + productId + "/buy", {
        referrer,
        orderDetails,
      });
    };
  }, [productId, referrer]);

  return <Form onSubmit={handleSubmit} />;
}
```

이것은 투박해 보인다. 함수를 메모하는 것은 리액트의 빌트인 훅으로 충분하다.
useMemo 대신 useCallback으로 함수를 감싸 중첩된 함수를 피해라.

```jsx
export default function Page({ productId, referrer }) {
  const handleSubmit = useCallback(
    (orderDetails) => {
      post("/product/" + productId + "/buy", {
        referrer,
        orderDetails,
      });
    },
    [productId, referrer]
  );

  return <Form onSubmit={handleSubmit} />;
}
```

위 예는 정확히 동일하다. useCallback의 장점은 중첩된 함수 작성을 피하게 해준다는 것이다.

---

## What I Learned

메모이제이션이 항상 헷갈리고 익숙하지 않은 부분인데, 문서를 읽다보니 보다 명확해진 것 같다.
보통의 경우 메모이제이션은 필요 없고, 메모로 감싸진 컴포넌트가 최적화를 위해 리랜더를 스킵하고 싶을 시에 useMemo로 값을, useCallback으로 함수를 캐시해 리랜더를 스킵한다.
