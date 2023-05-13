# Queueing a Series of State Updates

상태 변수를 설정하는 것은 다른 랜더를 대기 시킵니다.
그러나 가끔 다음 랜더에 추가되기전 여러개의 작업을 수행하고 싶을 수 있습니다.
이걸 하기 위해선, 리액트가 어떻게 배치를 통해 상태를 변경시키는 지 이해가 필요합니다.

## React batches state updates

당신은 +3 버튼을 누르면 `setNumber(number + 1)`를 세번 호출했기 때문에 카운터가 3번 증가할 것을 예상할 것입니다.

```jsx
import { useState } from "react";

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber(number + 1);
          setNumber(number + 1);
          setNumber(number + 1);
        }}
      >
        +3
      </button>
    </>
  );
}
```

하지만 이전 페이지에서 살펴봤듯이, 각 랜더의 상태 값은 고정되었습니다. 그러니 첫 번째 랜더의 이벤트 핸들러 안의 number의 값은 `setNumber(1)` 몇 번 호출했던지 상관없이 항상 0입니다.

```jsx
setNumber(0 + 1);
setNumber(0 + 1);
setNumber(0 + 1);
```

그러나 여기엔 또 다른 사실이 있습니다. 리액트는 상태가 변경되기전 이벤트 핸들러 내부의 코드들이 실행되길 기다립니다. 이것이 모든 `setNumber()`들이 호출되고 리랜더가 일어나는 이유입니다.

이것은 웨이터가 식당에서 주문을 받는 상황을 떠올리게 합니다.
웨이터는 첫 주문을 듣고 주방으로 뛰어가지 않습니다.
대신, 그들은 당신이 주문을 끝마치길 기다리고 주문을 변경하도록 해주며 심지어 다른 테이블의 주문도 받습니다.

![Alt text](https://react.dev/images/docs/illustrations/i_react-batching.png)

이것은 여러 상태 변수를 변경하고 심지어 여러 컴포넌트에서 변경해도 리랜더를 일으키지 않는 이유입니다.
그러나 이것은 또한 UI가 이벤트 핸들러에 어떤 코드가 있든 종료 되지 않으면 변경되지 않는 것을 의미합니다.
이것은 리액트를 더 빠르게 하는 배칭이라고 알려져있습니다. 또한 몇 변수가 바뀐 “half-finished”의 혼란도 피하게 해줍니다.

리액트는 여러개의 의도적인 이벤트들에는 배치를 하지않습니다. 각 클릭들은 독립적입니다.
리액트는 배치를 하기 안전할 때만 배칭을 합니다. 이것은 다음을 보장합니다.
만약 첫 번째 버튼 클릭이 폼을 막을 시, 두 번째 클릭은 제출을 하지 않습니다.

## Updating the same state multiple times before the next render

만약 당신이 상태 변수를 랜더 되기전에 변경하고 싶다면, 다음 상태값을 `setNumber(number + 1)` 와 같이 넘기는 대신, 이전 큐의 상태를 계산하는 `setNumber(n => n + 1)` 함수를 넘겨줄 수 있습니다.
이것은 단순히 상태를 교체하는 것이 아닌, 상태값을 가지고 무언가 하는 것입니다.

버튼을 클릭해 보세요

```jsx
import { useState } from "react";

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber((n) => n + 1);
          setNumber((n) => n + 1);
          setNumber((n) => n + 1);
        }}
      >
        +3
      </button>
    </>
  );
}
```

`n => n + 1`는 업데이터 함수라고 불립니다. 상태의 세터로 넘길 시

1. 리액트는 이 함수를 이벤트 핸들러의 모든 코드가 실행되고 큐에 등록합니다.
2. 다음 랜더 동안, 리액트는 큐를 통과해 마지막으로 변경된 상태를 반환합니다.

```jsx
setNumber((n) => n + 1);
setNumber((n) => n + 1);
setNumber((n) => n + 1);
```

리액트가 이 코드들을 어떻게 처리하는 지 예시입니다.

1. `setNumber(n => n + 1)`: `n => n + 1`는 함수로 리액트는 큐에 등록합니다.

2. `setNumber(n => n + 1)`: `n => n + 1`는 함수로 리액트는 큐에 등록합니다.

3. `setNumber(n => n + 1)`: `n => n + 1`는 함수로 리액트는 큐에 등록합니다.

다음 랜더에서 `useState`를 호출 했을 시, 리액트는 큐를 통과합니다.
이전 number 상태는 0 이므로 리액트는 첫 번째 업데이터 함수의 `n` 인자에 넘겨 줍니다. 그 후 리액트는 이전의 업데이터 함수의 반환값을 다음 업데이터 `n`에 계속해서 넘겨줍니다.

| queued update | `n` |  returns  |
| :-----------: | :-: | :-------: |
|  n => n + 1   |  0  | 0 + 1 = 1 |
|  n => n + 1   |  1  | 1 + 1 = 2 |
|  n => n + 1   |  2  | 2 + 1 = 3 |

리액트는 최종 값인 3을 저장하고 `useState`로 반환 받습니다.

이것이 +3 버튼을 클릭했을 때, 3이 증가한 이유입니다.

### What happens if you update state after replacing it

이 이벤트 핸들러는 어떻게 동작할까요?

```jsx
import { useState } from "react";

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber(number + 5);
          setNumber((n) => n + 1);
        }}
      >
        Increase the number
      </button>
    </>
  );
}
```

이 이벤트 핸들러가 리액트에게 지시하는 일은 다음과 같습니다.

1. `setNumber(number + 5)`: number는 0이므로, `setNumber(0 + 5)`입니다. 리액트는 큐에 5로 대체된 것을 등록합니다.
2. `setNumber(n => n + 1)`: `n => n + 1`은 업데이터 함수입니다. 리액트는 큐에 함수를 추가합니다.

다음 랜더 동안, 리액트는 다음 상태 큐를 진행합니다.

|  queued update   |    `n`     |  returns  |
| :--------------: | :--------: | :-------: |
| “replace with 5” | 0 (unused) |     5     |
|    n => n + 1    |     5      | 5 + 1 = 6 |

리액트는 `useState`의 반환값인 6을 결과로 저장합니다.

> Note
> 아마 `setState(5)`와 `setState(n => 5)`가 동일하게 동작한다는 것을 알아차렸을 것입니다. 대신 n이 사용되지 않은채

### What happens if you replace state after updating it

한 가지 예시를 더 시도해봅시다. number는 다음 랜더에 무엇일까요?

```jsx
import { useState } from "react";

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber(number + 5);
          setNumber((n) => n + 1);
          setNumber(42);
        }}
      >
        Increase the number
      </button>
    </>
  );
}
```

리액트가 이벤트 핸들러를 실행하며 코드를 어떻게 실행했는 지는 다음과 같습니다.

1. `setNumber(number + 5)`: number는 0이므로, `setNumber(0 + 5)`입니다. 리액트는 큐에 5로 대체된 것을 등록합니다.
2. `setNumber(n => n + 1)`: `n => n + 1`은 업데이터 함수입니다.리액트는 큐에 함수를 추가합니다.
3. `setNumber(42)`: 리액트는 상태를 42로 대체해 큐에 등록합니다.

|   queued update   |    `n`     |  returns  |
| :---------------: | :--------: | :-------: |
| “replace with 5”  | 0 (unused) |     5     |
|    n => n + 1     |     5      | 5 + 1 = 6 |
| “replace with 42” | 6 (unused) |    42     |

그 후 리액트는 `useState`의 반환값인 42를 저장합니다.

요약하자면, `setNumber`에 상태 세터를 전달하는 것은 다음과 같습니다.

- 업데이터 함수(e.g. `n => n + 1`)는 큐에 추가
- 모든 값(e.g. 5)는 기존에 큐된 것을 무시하고 5로 대체

이벤트 핸들러가 완료되고, 리액트는 리랜더를 일으킵니다.
리랜더 도중, 리액트는 큐를 실행합니다.
업데이터 함수는 랜더 도중 실행되므로 업데이터 함수는 반드시 순수해야하며 결과를 반환해야합니다. 상태를 그 안에서 설정하는 것과 다른 사이드 이팩트를 실행하는 것을 시도하지 마세요. 엄격 모드에서 이런 업데이터 함수는 실수를 방지하기 위해 2 번 실행됩니다.

### Naming conventions

업데이터 함수의 인자로는 상태 변수의 첫 문자로 하는 것이 일반적입니다.

```jsx
setEnabled((e) => !e);
setLastName((ln) => ln.reverse());
setFriendCount((fc) => fc * 2);
```

만약 좀 더 긴 코드를 선호한다면, `setEnabled(enabled => !enabled)`와 같이 상태 변수의 이름을 완전히 사용하는 것입니다. 또는 `setEnabled(prevEnabled => !prevEnabled)`와 같이 접두사를 붙이는 방법입니다.

## Recap

- 상태를 설정하는 것은 새로운 랜더를 요청할 뿐 기존의 랜더를 바꾸지 않습니다.
- 리액트의 상태 변경은 이벤트 핸들러가 완전히 끝난 뒤 진행됩니다. 이를 배칭이라고 부릅니다.
- 하나의 이벤트에서 여러 번 상태를 변경하고 싶다면, `setNumber(n => n + 1)` 업데이터 함수를 사용할 수 있습니다.

---

## What I Learned

상태라는 게 정말 쉬우면서도 어려운 개념인 것 같다.
좀 더 확실히 이해해야겠다.

상태는 컴포넌트의 메모리 그리고 스냅샷이며 새로운 랜더링을 요청한다.
상태를 업데이트 해주는 과정은 이벤트 핸들러가 종료되는 시점이며 이것을 배칭이라고 부른다.
만약 하나의 이벤트에서 상태를 여러번 업데이트 하고 싶다면? 업데이터 함수를 사용

출처

- https://react.dev/learn/queueing-a-series-of-state-updates
