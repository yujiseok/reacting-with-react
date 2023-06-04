- [Referencing Values with Refs](#referencing-values-with-refs)
  - [Adding a ref to your component](#adding-a-ref-to-your-component)
  - [Example: building a stopwatch](#example-building-a-stopwatch)
  - [Differences between refs and state](#differences-between-refs-and-state)
    - [DEEP DIVE - How does useRef work inside?](#deep-dive---how-does-useref-work-inside)
  - [When to use refs](#when-to-use-refs)
  - [Best practices for refs](#best-practices-for-refs)
  - [Refs and the DOM](#refs-and-the-dom)
  - [Recap](#recap)
  - [What I Learned](#what-i-learned)

# Referencing Values with Refs

컴포넌트가 어떤 정보를 기억하게 하고 싶지만, 새로운 랜더를 일으키고 싶지 않은 경우, 당신은 ref를 사용할 수 있습니다.

## Adding a ref to your component

`useRef`를 통해 컴포넌트에 ref를 추가할 수 있습니다.

```jsx
import { useRef } from "react";
```

`useRef` 훅을 호출한 후 초기값을 전달해 레퍼런스할 수 있습니다.
예를 들어, ref가 있고 값이 0이라면:

```jsx
const ref = useRef(0);
```

`useRef`는 다음과 같은 객체를 반환합니다.

```jsx
{
  current: 0; // The value you passed to useRef
}
```

당신은 `ref.current` 속성으로 현재 값에 접근할 수 있습니다.
이 값은 뮤터블 하므로, 당신은 읽고 쓸 수 있습니다.
이것은 리액트는 모르는 비밀 주머니입니다.

여기 `ref.current`를 매 클릭 시 증가하는 버튼이 있습니다.

```jsx
import { useRef } from "react";

export default function Counter() {
  let ref = useRef(0);

  function handleClick() {
    ref.current = ref.current + 1;
    alert("You clicked " + ref.current + " times!");
  }

  return <button onClick={handleClick}>Click me!</button>;
}
```

ref는 숫자를 가리키지만, 상태와 같이 어떤것이든 될 수 있습니다.
상태와 다르게, ref는 `current` 속성을 갖는 자바스크립트 객체입니다.

컴포넌트가 증가시에 리랜더 되지 않는 것을 기억하세요.
상태와 같이 ref는 리랜더 사이에 보존됩니다. 하지만, 상태를 설정하는 것은 컴포넌트를 리랜더합니다. ref 변경은 그렇지 않습니다.

## Example: building a stopwatch

당신은 ref와 상태를 한 컴포넌트에서 합칠 수 있습니다.
예를 들어, 유저가 멈추고 시작할 수 있는 스탑워치를 만든다고 합시다.
시작 버튼을 눌렀을 시 시간이 얼마나 지났는지 보여주기 위해, 당시는 언제 시작 버튼을 눌렀고 현재 시간은 무엇인지 추적해야합니다.
이 정보는 랜더에 필요하므로 상태로 관리해야합니다.

```jsx
const [startTime, setStartTime] = useState(null);
const [now, setNow] = useState(null);
```

시작을 눌렀을 때, 당신은 `setInterval`을 이용해 매 10 밀리초 마다 변경되도록합니다.

```jsx
import { useState } from "react";

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);

  function handleStart() {
    // Start counting.
    setStartTime(Date.now());
    setNow(Date.now());

    setInterval(() => {
      // Update the current time every 10ms.
      setNow(Date.now());
    }, 10);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>Start</button>
    </>
  );
}
```

정지 버튼을 눌렀을 경우, 당신은 now 상태를 변경하는 인터벌을 종료해야합니다.
당신은 `clearInterval`을 통해 할 수 있지만, 이전 `setInterval`에서 반환된 인터벌 ID가 필요합니다.
당신은 인터벌 ID를 어딘가 보관해야합니다. 인터벌 ID는 랜더링에 필요하지 않으므로, ref에 보관합니다.

```jsx
import { useState, useRef } from "react";

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);
  const intervalRef = useRef(null);

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current);
    intervalRef.current = setInterval(() => {
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>Start</button>
      <button onClick={handleStop}>Stop</button>
    </>
  );
}
```

랜더링에 필요한 정보면, 상태에 담으세요. 정보가 이벤트 핸들러에 필요하고 리랜더를 일으킬 필요가 없다면, ref가 더 효율적입니다.

## Differences between refs and state

아마도 당신은 ref가 덜 엄격한 상태라고 생각할 것입니다. - 상태를 설정하는 함수 대신 뮤테이트할 수 있기에.
하지만 대부분에서, 당신은 상태를 사용할 것입니다. ref는 탈출구로 당신은 대부분 필요하지 않습니다.
상태와 ref의 비교입니다.

|                          refs                           |                                        state                                         |
| :-----------------------------------------------------: | :----------------------------------------------------------------------------------: | --- |
| `useRef(initialValue)` 반환 `{ current: initialValue `  | `useState(initialValue)`는 현재 상태와 상태를 설정하는 함수 반환 `[value, setValue]` |
|             변경해도 리랜더를 일으키지 않음             |                                    변경시 리랜더                                     |
| `current` 값을 랜더 과정 밖에서 변경할 수 있음 - 뮤터블 |                상태 설정 함수를 통해서 변경해 리랜더 큐에 등록 - 불변                |     |
|          랜더 중 `current` 값을 읽어올 수 없음          |  상태를 아무때나 읽을 수 있음. 하지만 각 랜더는 상태의 변하지 않는 스냅샷을 갖는다.  |

상태로 구현된 카운터입니다.

```jsx
import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return <button onClick={handleClick}>You clicked {count} times</button>;
}
```

카운트 값이 표시되기에, 상태로 만드는 것이 합당합니다. 카운터의 값이 `setCount()`로 설정 되었을 때, 리액트는 리랜더링 되고 화면을 새로운 카운트로 변경하고 보여줍니다.

만약 ref로 구현한다면, 리액트는 컴포넌트를 절대 리랜더하지 않으므로, 카운트의 변경을 볼 수 없습니다.

```jsx
import { useRef } from "react";

export default function Counter() {
  let countRef = useRef(0);

  function handleClick() {
    // This doesn't re-render the component!
    countRef.current = countRef.current + 1;
  }

  return (
    <button onClick={handleClick}>You clicked {countRef.current} times</button>
  );
}
```

이것이 랜더 도중 `ref.current`를 읽게 되면 신뢰할 수 없는 코드를 얻게되는 이유입니다. 만약 원한다면, 상태를 사용하세요.

#### DEEP DIVE - How does useRef work inside?

`useState`와 `useRef` 둘다 리액트에서 제공되지만, 원칙적으로 `useRef`는 `useState`의 상위에서 구현되었습니다.
당신은 리액트 안에서 `useRef`가 다음과 같이 구현되었다고 볼 수 있습니다.

```jsx
// Inside of React
function useRef(initialValue) {
  const [ref, unused] = useState({ current: initialValue });
  return ref;
}
```

첫 랜더시, `useRef`는 `{ current: initialValue }`를 반환합니다.
이 객체는 리액트에 저장되며, 다음 랜더시에도 똑같은 객체가 반환됩니다.
세터가 사용되지 않는 것을 기억하세요.
`useRef`가 항상 똑같은 객체를 반환하기에 필요없습니다.

리액트는 빌트인 버전인 `useRef`를 제공하는데 실제로는 일반적이기 때문입니다.
하지만, 당신은 세터없는 상태 변수라고 생각할 수 있습니다.
당신이 객체 지향 프로그래밍에 익숙하다면, ref는 아마 인스턴스 필드를 상기시킵니다. - `this.something` 대신 `somethingRef.current`인

## When to use refs

일반적으로, 당신은 ref를 컴포넌트의 모습에 영향을 주지 않는 외부 API와 대화할 때 사용할 것 입니다.
다음은 이런 드문 상황들 입니다.

- 타임아웃 아이디 저장
- 돔 요소 조작
- JSX에 불필요한 객체 저장

랜더링 로직에 영향이 없는 정보는 ref에 저장하세요.

## Best practices for refs

다음과 같은 원칙을 따르면, 당신의 컴포넌트는 예측 가능해질 것입니다.

- Treat refs as an escape hatch. ref는 브라우저 API나 외부 시스템을 사용할 때 유용합니다. 당신의 로직이나 데이터 흐름이 ref에 의존한다면 접근 방식을 다시 생각해봐야합니다.
- Don’t read or write `ref.current` during rendering. 만약 어떤 정보가 랜더 중에 필요하다면, 상태를 사용하세요. 리액트는 `ref.current`가 언제 변경될지 모르므로, 이것은 컴포넌트의 작동의 예측을 어렵게 합니다.

ref에는 리액트 상태의 제한이 적용되지 않습니다. 예를 들어, 상태는 매 랜더의 스냅샷이고 동기적으로 변경되지 않습니다. 하지만 ref의 현재 값을 변경하면 즉시 변경됩니다.

```jsx
ref.current = 5;
console.log(ref.current); // 5
```

이것은 ref가 실제로는 일반적인 자바스크립트 객체이기 때문입니다.

또 당신은, ref를 사용하면 뮤테이션을 피할 필요도 없습니다. 객체를 뮤테이트 하는 것은 랜더링에 쓰이지 않습니다. 리액트는 당신이 ref와 뭘 하든 관심없습니다.

## Refs and the DOM

당신은 ref에 어떤 값이든 가리킬 수 있습니다. 하지만, 대부분의 ref 사용 용도는 돔 요소에 접근하기 위해서입니다.
예를 들어, 인풋에 프로그래밍적으로 포커스하는 것이 쉽습니다.
만약 당신이 `ref`를 `<div ref={myRef}`와 같은 JSX의 어트리뷰트로 전달한다면, 리액트는 상응하는 돔 요소를 `myRef.current`에 담습니다.

## Recap

- ref는 탈출구로 랜더링에 불필요한 정보를 보관합니다. 당신은 자주 필요하지 않습니다.
- ref는 `current` 요소를 갖는 자바스크립트 객체로, 읽고 셋할 수 있습니다.
- 당신은 `useRef` 훅을 통해 리액트에게 ref를 요청할 수 있습니다.
- 상태와 같이, ref는 랜더 사이에 정보를 보존합니다.
- 상태와 달리, ref의 `current` 값을 변경하는 것은 리랜더를 일으키지 않습니다.
- 랜더 중 `ref.current`를 읽거나 쓰지 마세요. 이것은 컴포넌트를 예측하기 어렵게 합니다.

---

## What I Learned

ref라는 것을 리액트 공식 문서의 말대로 사용해본적이 많이 없었다. 그러다보니 덜 친숙한 개념이었는데, ref와 state의 차이를 좀 더 명확히 할 수 있게되었다.

리랜더를 트리거 하기위해선 state를 사용하자!

출처

- https://react.dev/learn/referencing-values-with-refs
