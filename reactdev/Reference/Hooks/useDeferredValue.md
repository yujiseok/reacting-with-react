# useDeferredValue

useDeferredValue은 UI 변경의 부분을 연기하기 위한 훅이다.

```jsx
const deferredValue = useDeferredValue(value);
```

## Reference

### useDeferredValue(value)

컴포넌트의 최상단에 useDeferredValue를 호출해 값의 지연된 버전을 얻을 수 있다.

```jsx
import { useState, useDeferredValue } from "react";

function SearchPage() {
  const [query, setQuery] = useState("");
  const deferredQuery = useDeferredValue(query);
  // ...
}
```

**Parameters**

- value: 지연시킬 값. 어떤 값이든 된다.

**Returns**
초기 랜더 시, 지연된 값은 제공된 값과 같다. 변경 시, 리액트는 우선 예전 값과 리랜더를 시도하고, 새로운 값과 백그라운드에서 리랜더를 시도한다.

**Caveats**

- useDeferredValue에 넘겨지는 값은 원시 값 혹은 랜더링 밖에서 생성된 객체여야한다. 만약 랜더 중에 새로운 객체를 생성해 useDeferredValue에 넘긴다면 불필요한, 백그라운드 리랜더를 일으킨다.
- useDeferredValue가 다른 값을 받으면, 현재 랜더에, 그것은 새로운 값으로 백그라운드 리랜더를 스케줄한다. 백그라운드 리랜더는 방해될 수 있어서, 새로운 값 변경이 생기면, 리액트는 백그라운드 리랜더를 처음부터 시작한다. 예를 들어, 만약 유저가 인풋 타이핑을 차트가 지연된 값으로 리랜더 되는 것 보다 빠르게 할 시, 차트는 타이핑이 종료된 후에 리랜더된다.
- useDeferredValue는 서스펜스와 통합된다. 만약 새로운 값으로 UI 지연이 생긴다면, 유저는 대체를 보지 못한다. 그들은 데이터가 로드될 때까지 이전의 지연된 값을 본다.
- useDeferredValue 자체로는 추가 네트워크 요청을 방지하지 못한다.
- useDeferredValue 자체로 고정된 지연은 없다. 리액트가 오리지널 리랜더를 끝내면, 리액트는 즉시 지연 값과 백그라운드 리랜더를 시작한다. 이벤트에 의한 어떤 변경도 백그라운드 리랜더를 막고 우선순위를 갖는다.
- useDeferredValue가 유발하는 백그라운드 리랜더는 화면에 커밋될 때가지 이펙트를 일으키지않는다. 만약 백그라운드 리랜더가 연기될 시, 데이터가 로드되고 UI가 변경 되었을 시 이펙트가 실행된다.

## Usage

### Showing stale content while fresh content is loading

컴포넌트의 최상단에 useDeferredValue를 호출해 UI를 지연 업데이트 해라.

```jsx
import { useState, useDeferredValue } from "react";

function SearchPage() {
  const [query, setQuery] = useState("");
  const deferredQuery = useDeferredValue(query);
  // ...
}
```

초기 랜더 시, deferred value은 전달 받은 값과 동일하다.

업데이트 시, deferred value는 전달 받은 값에 뒤쳐져있다. 리액트는 처음 리랜더시 지연된 값 없이 변경하고, 백그라운드에서 받은 새 값으로 리랜더한다.

이 예에서 검색결과 컴포넌트는 검색 결과 페칭을 기다린다.
"a"를 검색하고 결과를 기다린 후, "ab"로 수정해라. "a"의 결과가 로딩으로 대체된다.

```jsx
import { Suspense, useState } from "react";
import SearchResults from "./SearchResults.js";

export default function App() {
  const [query, setQuery] = useState("");
  return (
    <>
      <label>
        Search albums:
        <input value={query} onChange={(e) => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Loading...</h2>}>
        <SearchResults query={query} />
      </Suspense>
    </>
  );
}
```

아주 일반적인 대체 UI 패턴은 리스트를 지연 변경하는 것으로 새로운 값이 준비될 때까지 이전 값을 유지하는 것이다.
useDeferredValue를 호출해 지연된 값을 전달해라

```jsx
export default function App() {
  const [query, setQuery] = useState("");
  const deferredQuery = useDeferredValue(query);
  return (
    <>
      <label>
        Search albums:
        <input value={query} onChange={(e) => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Loading...</h2>}>
        <SearchResults query={deferredQuery} />
      </Suspense>
    </>
  );
}
```

query는 즉시 변경 되어, 인풋은 새로운 값을 보여준다. 하지만, deferredQuery는 데이터가 로드될 때까지 이전의 값을 유지한다. SearchResults가 이전 결과를 보여준다.

#### DEEP DIVE - How does deferring a value work under the hood?

두 단계로 생각할 수 있다.

1. 첫째로, 리액트는 새로운 쿼리로 리랜더 하지만 지연된 쿼리와 함께한다. 지연된 쿼리 값은, 쿼리 값보다 지연되어 있다.
2. 백그라운드에서, 리액트는 두 값으로 리랜더를 시도한다. 만약 리랜더가 성공하면, 리액트는 화면에 보여준다. 하지만, 만약 지연된다면, 리액트는 랜더 시도를 버리고 로드된 데이터로 다시 리랜더를 시도한다. 데이터가 준비될 때까지 유저는 이전 값을 본다.

지연된 백그라운드 랜더는 방해된다. 예를 들어 인풋에 타입을 하면 리액트는 그것을 버리고 새로운 값으로 재시작한다. 리액트는 항상 제공된 새로운 값을 사용한다.

타자를 칠 때마다 네트워크 요청이 있다는 것을 기억하자. 화면에 보여지는 결과만 지연시킬뿐, 네트워크 요청 자체는 아니다. 만약 유저가 지속적으로 타이핑을 하더라도 각 타자는 캐시된다. 백스페이스를 누른 것이 즉시 페치를 하지않는 이유다.

### Indicating that the content is stale

위의 예에서, 쿼리가 여전히 로딩 중이라는 지침이 없다. 이것은 새로운 값의 결과인지 유저에게 혼란을 준다.
이것을 좀 더 명백히 하기 위해, 지연된 값이 보여질 경우 시각적 지침을 추가할 수 있다.

```jsx
<div
  style={{
    opacity: query !== deferredQuery ? 0.5 : 1,
  }}
>
  <SearchResults query={deferredQuery} />
</div>
```

이 변경과 함께, 지연된 새로운 결과가 로드될 때까지 값은 살짝 딤드 처리가 될 수 있다.
css 트랜지션을 사용해 딤드 처리를 점진적으로 할 수 있다.

### Deferring re-rendering for a part of the UI

useDeferredValue를 통해 성능 최적화를 이룰 수 있다. UI의 일정 부분의 리랜더가 느릴 시 유용하다. 최적화에 쉬운 법은 없으며, UI를 막는 것을 방지하는 것도 그렇다.

매 타자 시, 리랜더하는 인풋과 차트나 리스트 컴포넌트가 있다고 생각하자.

```jsx
function App() {
  const [text, setText] = useState("");
  return (
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <SlowList text={text} />
    </>
  );
}
```

먼저 프랍이 같을 경우 SlowList를 최적화하기 위해 메모로 감싼다.

```jsx
const SlowList = memo(function SlowList({ text }) {
  // ...
});
```

하지만, 이것은 SlowList의 프랍이 이전 랜더와 동일할 경우에만 도움이 된다. 문제는 다른 값일 경우 느려지는 것과, 실제로 다른 값을 화면에 보여주길 원한다는 것이다.

구체적으로, 주요한 성능 문제의 이유는 인풋을 타입할 때마다, SlowList가 새로운 프랍을 받아 모든 트리를 리랜더하는 것이다.
이 예에서 useDeferredValue를 사용해 결과 목록을 변경하는 것보다 인풋 변경에 우선순위를 줄 수 있다.

```jsx
function App() {
  const [text, setText] = useState("");
  const deferredText = useDeferredValue(text);
  return (
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <SlowList text={deferredText} />
    </>
  );
}
```

이것은 SlowList의 리랜더를 지연시킨다. 하지만, 이것은 리액트에게 목록의 우선순위가 없다는 것을 알려주며, 이것은 인풋 입력을 막지 않는다.
목록은 지연되지만, 이전과 같이 캐치업한다. 리액트는 리스트의 변경을 가능한 빠르게 시도하고, 유저 입력을 막지않는다.

#### Pitfall

이 최적화는 SlowList가 메모로 감싸지는 것을 전제로 한다. 텍스트가 변경 될 시, 리액트가 부모 컴포넌트를 빠르게 리랜더 해야하기 때문이다.
리랜더 시, deferredText 값은 여전히 이전 값이므로, SlowList가 리랜더를 무시할 수 있다. 메모 없이 변경할 시, 항상 리랜더 되어 최적화의 관점을 위반한다.

#### DEEP DIVE - How is deferring a value different from debouncing and throttling?

두 가지의 일반적인 최적화 기술이 있다.

- 디바운싱은 목록을 변경하기 전에, 유저가 타이핑 멈추는 것을 기다리는 것이다.
- 쓰로틀링은 가끔씩 목록을 변경하는 것이다.

이런 기술은 어떤 경우 도움이 되지만, useDeferredValue를 사용하는 것이 리액트 그자체로는 더 잘 맞는다.

디바운싱과 쓰로틀링과 다르게, 이것은 고정된 지연값을 필요로 하지않는다. 만약 유저의 기기가 빠르다면, 지연된 리랜더는 거의 즉시 실행되고 눈에 띄지 않는다. 하지만, 유저의 기기가 느릴 경우, 목록은 기기의 느림과 비례할 것이다.

또한, 디바운싱과 쓰로틀링과 다르게, useDeferredValue에 의해 지연된 리랜더는 방해될 수 있다.
이것은 유저가 큰 목록의 랜더 도중 인풋 타이핑이 일어날 경우 리액트는 리랜더를 버리고, 타자치는 것을 다루고 백그라운드 리랜더를 시작한다.
반대로, 디바운싱과 쓰로틀링은 막기에 불안정한 경험을 제공한다. 랜더링의 키 입력을 막는 순간을 연기할 뿐이다.

만약, 최적화가 랜더 도중이 아니라면, 디바운싱과 쓰로틀링이 유용할 것이다.
예를 들어, 그들은 보다 적은 네트워크 요청을 보낸다. 이 기술들을 같이 사용할 수 있다.

---

## What I Learned

지연된 값을 사용해 유저의 경험을 더 좋게 해준다.
네트워크 요청 자체를 지연시킬 순 없다. 고정된 지연이 필요 없다.

디바운싱과 쓰로틀링과 다르게 방해가 가능하다.

이런 최적화의 기술에 대해 좀 더 공부를 해야겠다.
