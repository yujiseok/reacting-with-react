# useTransition

useTransition은 UI를 블로킹 하지 않고 상태를 변경할 수 있도록 해주는 훅이다.

```jsx
const [isPending, startTransition] = useTransition();
```

## Reference

### useTransition()

컴포넌트 최상단에 useTransition을 호출해, 상태 변경에 전환을 준다.

```jsx
import { useTransition } from "react";

function TabContainer() {
  const [isPending, startTransition] = useTransition();
  // ...
}
```

**Parameters**
useTransition은 어떤 파라미터도 받지 않는다.

**Returns**

useTransition은 두 아이템을 갖는 배열을 리턴한다.

1. isPending 플래그는 대기중인 전환이 있는지 알려준다.
2. startTransition 함수로 상태 변경을 전환으로 표시할 수 있다.

### startTransition function

useTransition에서 반환되는 startTransition 함수는 상태 변경을 전환으로 표시할 수 있다.

```jsx
function TabContainer() {
  const [isPending, startTransition] = useTransition();
  const [tab, setTab] = useState("about");

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
  // ...
}
```

**Parameters**

- scope: 한 개 이상의 셋 함수를 호출해 상태를 변경하는 함수. 리액트는 즉시 스코프를 파라미터 없이 호출하고, 모든 상태 변경을 스코프 함수가 전환으로 호출되는 동안 동기로 스케줄링 한다. 이것은 논블로킹이고 원치 않는 로딩 인디케이터를 보여주지 않는다.

**Returns**
startTransition는 아무것도 리턴하지 않는다.

**Caveats**

- useTransition은 훅으로, 컴포넌트 혹은 커스텀 훅에서만 호출 되어야한다. 만약 다른 곳에서 전환을 원한다면, startTransition을 호출해라
- 상태를 셋해주는 함수에 접근할 수 있다면, 전환으로 감쌀 수 있다. 만약 프랍이나 커스텀 훅의 응답으로 전환을 시작하고 싶다면, useDeferredValue를 사용해라.
- startTransition에 넘겨진 함수는 반드시 동기적이어야한다. 리액트는 즉시 그 함수를 실행하고, 전환이 시작하면 모든 상태를 변경한다. 만약 상태 변경을 더 이후에 하고싶다면, 이것은 전환으로 마크되지 않는다.
- 전환으로 마크된 상태 변경은 다른 상태 변경에 의해 막아진다. 예를 들어, 만약 전환 내부에서 차트 컴포넌트를 변경할 시, 차트가 랜더 중간 쯤 왔을 때, 인풋에 타이핑을 할 때, 리액트는 인풋 변경을 우선 다룬 후 차트 컴포넌트를 랜더한다.
- 전환 변경은 텍스트 인풋을 컨트롤할 수 없다.
- 여러 전환이 있다면, 리액트는 배치할 것이다. 이것은 한정적이라, 리액트의 미래 릴리즈에서 제거될 것이다.

## Usage

### Marking a state update as a non-blocking transition

useTransition를 컴포넌트 최상단에 호출해 상태 변경을 논블로킹 전환으로 마크해라.

```jsx
import { useState, useTransition } from "react";

function TabContainer() {
  const [isPending, startTransition] = useTransition();
  // ...
}
```

useTransition은 두 아이템을 갖는 배열을 리턴한다.

1. isPending 플래그는 대기중인 전환이 있는지 알려준다.
2. startTransition 함수로 상태 변경을 전환으로 표시할 수 있다.

상태 변경을 다음과 같이 전환으로 마크할 수 있다.

```jsx
function TabContainer() {
  const [isPending, startTransition] = useTransition();
  const [tab, setTab] = useState("about");

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
  // ...
}
```

전환은 느린 기기에서도 UI 변경을 반응하도록 해준다.

전환과, UI는 리랜더의 중간에 머문다. 예를 들어, 유저가 탭을 클릭하고 생각을 바꿔 다른 탭을 클릭시, 그들은 첫번째 랜더가 끝나는 것을 기다릴 필요가 없다.

### Updating the parent component in a transition

useTransition를 호출해 부모 컴포넌트의 상태를 변경할 수 있다. 예를 들어, 탭버튼 컴포넌트의 온클릭 로직을 전환으로 감싼다.

```jsx
export default function TabButton({ children, isActive, onClick }) {
  const [isPending, startTransition] = useTransition();
  if (isActive) {
    return <b>{children}</b>;
  }
  return (
    <button
      onClick={() => {
        startTransition(() => {
          onClick();
        });
      }}
    >
      {children}
    </button>
  );
}
```

부모 컴포넌트의 상태 변경이 온클릭 이벤트 핸들러에서 일어나기에, 상태 변경이 전환으로 마크 된다.
이전의 예처럼, 포스트를 클릭한 후 즉시 컨택트를 클리할 수 있는 이유다.
선택된 탭을 변경하는 것이 전환으로 마크되어, 이것은 유저의 상호작용을 막지 않는다.

### Displaying a pending visual state during the transition

useTransition에서 반환된 isPending 불리언 값으로 유저에게 전환의 진전을 알려줄 수 있다.
예를 들어, 탭버트는 특별한 펜딩 값을 가질 수 있다.

```jsx
function TabButton({ children, isActive, onClick }) {
  const [isPending, startTransition] = useTransition();
  // ...
  if (isPending) {
    return <b className="pending">{children}</b>;
  }
  // ...
```

포스트를 클릭하는 것이 좀 더 반응적이 된 것을 느낄 수 있다.

### Preventing unwanted loading indicators

포스트탭 컴포넌트는 서스펜스를 통해 데이터를 페치한다.
포스트 버튼을 클릭했을 시, 포스트탭 컴포넌트가 정지되고, 대체 로딩이 보여진다.

```jsx
import { Suspense, useState } from "react";
import TabButton from "./TabButton.js";
import AboutTab from "./AboutTab.js";
import PostsTab from "./PostsTab.js";
import ContactTab from "./ContactTab.js";

export default function TabContainer() {
  const [tab, setTab] = useState("about");
  return (
    <Suspense fallback={<h1>🌀 Loading...</h1>}>
      <TabButton isActive={tab === "about"} onClick={() => setTab("about")}>
        About
      </TabButton>
      <TabButton isActive={tab === "posts"} onClick={() => setTab("posts")}>
        Posts
      </TabButton>
      <TabButton isActive={tab === "contact"} onClick={() => setTab("contact")}>
        Contact
      </TabButton>
      <hr />
      {tab === "about" && <AboutTab />}
      {tab === "posts" && <PostsTab />}
      {tab === "contact" && <ContactTab />}
    </Suspense>
  );
}
```

모든 탭 컨테이너를 로딩 인디케이터로 보여주는 것은 유저 경험 부조화를 일으킨다.
만약 useTransition을 탭버튼에 추가한다면, 탭버튼에 로딩을 보여주도록 할 수 있다.

포스트를 클릭 하는 것은 이제 모든 컨테이너를 스피너로 대체하지 않는다.

```jsx
import { useTransition } from "react";

export default function TabButton({ children, isActive, onClick }) {
  const [isPending, startTransition] = useTransition();
  if (isActive) {
    return <b>{children}</b>;
  }
  if (isPending) {
    return <b className="pending">{children}</b>;
  }
  return (
    <button
      onClick={() => {
        startTransition(() => {
          onClick();
        });
      }}
    >
      {children}
    </button>
  );
}
```

#### Note

전환은 이미 노출된 컨텐츠를 숨기지 않을 만큼만 대기한다.
만약 포스트탭에 중첩된 서스펜스 바운더리가 있다면, 전환은 그것을 기다리지 않을 것이다.

### Building a Suspense-enabled router

만약, 리액트 프레임워크나 라우터를 만든다면, 페이지 네비게이션을 전환으로 마크하는 것을 추천한다.

```jsx
function Router() {
  const [page, setPage] = useState('/');
  const [isPending, startTransition] = useTransition();

  function navigate(url) {
    startTransition(() => {
      setPage(url);
    });
  }
  // ...
```

이것은 두 이유이다.

- 전환은 중단이 가능하다. 즉 유저가 리랜더의 완성을 기다리지 않고 클릭할 수 있게 한다.
- 전환은 원치 않는 로딩을 막는다. 유저가 부조화된 네비게이션을 피할 수 있다.

## Troubleshooting

### Updating an input in a transition doesn’t work

인풋의 상태 변경을 전환으로 사용할 수 없다.

```jsx
const [text, setText] = useState("");
// ...
function handleChange(e) {
  // ❌ Can't use transitions for controlled input state
  startTransition(() => {
    setText(e.target.value);
  });
}
// ...
return <input value={text} onChange={handleChange} />;
```

전환이 논블로킹이고, 체인지 이벤트의 결과를 인풋에 변경하는 것은 동기적이다.
만약 인풋 타이핑을 전환으로 하고싶다면, 두 가지 선택지가 있다.

1. 두가지의 다른 상태 변수를 선언: 하나는 인풋 상태이고, 하나는 전환에서 업데이트 되는 상태. 이것은 인풋을 컨트롤하는 것은 동기적인 상태고, 전환에 전달된 상태는 랜더로직을 위한 상태다.
2. 하나의 상태를 가지며, useDeferredValue를 통해 진짜 값을 뒤쳐지도록 할 수 있다. 이것은 논블로킹 리랜더를 일으키며 새로운 값을 자동으로 캐치한다.

### React doesn’t treat my state update as a transition

전환으로 상태 변경을 감쌌을 시, startTransition 호출에서 일어나는 것임을 확실히 해라.

```jsx
startTransition(() => {
  // ✅ Setting state *during* startTransition call
  setPage("/about");
});
```

startTransition에 넘겨지는 함수는 반드시 동기적이어야 한다.

다음과 같이 전환을 변경할 수 없다.

```jsx
startTransition(() => {
  // ❌ Setting state *after* startTransition call
  setTimeout(() => {
    setPage("/about");
  }, 1000);
});
```

대신 다음과 같이 할 수 있다.

```jsx
setTimeout(() => {
  startTransition(() => {
    // ✅ Setting state *during* startTransition call
    setPage("/about");
  });
}, 1000);
```

비슷하게 다음과 같이 전환을 변경할 수 없다.

```jsx
startTransition(async () => {
  await someAsyncFunction();
  // ❌ Setting state *after* startTransition call
  setPage("/about");
});
```

대신 이것은 동작한다.

```jsx
await someAsyncFunction();
startTransition(() => {
  // ✅ Setting state *during* startTransition call
  setPage("/about");
});
```

### I want to call useTransition from outside a component

useTransition는 훅이기에 컴포넌트 외부에서 호출할 수 없다. 이 경우, startTransition 메서드를 사용해라. 이것은 똑같이 동작하지만, isPending을 제공하지 않는다.

### The function I pass to startTransition executes immediately

만약 이 코드를 실행하면, 1,2,3이 출력될 것이다.

```jsx
console.log(1);
startTransition(() => {
  console.log(2);
  setPage("/about");
});
console.log(3);
```

이것은 1,2,3으로 출력되는 것으로 기대된다. startTransition에 넘긴 함수는 딜레이되지 않는다. 브라우저의 셋타임아웃과 다르게, 이것은 콜백을 나중에 호출하지 않는다. 리액트는 함수를 즉시 실행한다.
하지만, 이것이 실행될 시 모든 상태 변경은 전환으로 마크되어 스케줄된다.

```jsx
// A simplified version of how React works

let isInsideTransition = false;

function startTransition(scope) {
  isInsideTransition = true;
  scope();
  isInsideTransition = false;
}

function setState() {
  if (isInsideTransition) {
    // ... schedule a transition state update ...
  } else {
    // ... schedule an urgent state update ...
  }
```

---

## What I Learned

useTransition는 리액트 18 버전에서 추가된 훅으로 알고 있다. 사실 어떤 용도로 사용이 되는지 문서를 읽어도 완전히 와닿지 않는다. 어디에 적용을 해야할지 모르겠어서 그런 것 같다.

우선 순위를 리액트에게 알려주는 훅이다. isPending을 통해 상태 변경의 보류 여부를 알 수 있으며, startTransition을 통해 상태 변경을 알릴 수 있다.
