# State as a Snapshot

상태 변수들은 읽고 쓸수 있다는 점에서 보통 자바스크립트 변수와 비슷하게 보입니다.
하지만, 상태는 스냅샷과 더 비슷합니다.
set하는 것은 상태 변수를 변경할 수 없지만, 리랜더를 일으킵니다.

## Setting state triggers renders

당신은 당신의 유저 인터페이스가 클릭과 동시에 변경될 것이라고 생각합니다.
리액트에서는 살짝 다른 멘탈 모델로 작용합니다. 이벤트에 반응하기 위해선, 상태를 변경해야 합니다.

아래의 예에서, send 버튼을 누르면 `setIsSent(true)`는 리액트에게 UI를 리랜더하라고 말해줍니다.

```jsx
import { useState } from "react";

export default function Form() {
  const [isSent, setIsSent] = useState(false);
  const [message, setMessage] = useState("Hi!");
  if (isSent) {
    return <h1>Your message is on its way!</h1>;
  }
  return (
    <form
      onSubmit={(e) => {
        e.preventDefault();
        setIsSent(true);
        sendMessage(message);
      }}
    >
      <textarea
        placeholder="Message"
        value={message}
        onChange={(e) => setMessage(e.target.value)}
      />
      <button type="submit">Send</button>
    </form>
  );
}

function sendMessage(message) {
  // ...
}
```

버튼 클릭 시 일어나는 일은 다음과 같습니다.

1. `onSubmit` 이벤트 핸들러가 실행
2. `setIsSent(true)`로 `isSent`를 true로 변경해 새로운 랜더를 큐에 추가
3. 새로운 `isSent` 값에 따라 리액트가 리랜더

상태와 랜더링에 연관성을 좀 더 살펴봅니다.

## Rendering takes a snapshot in time

랜더링은 리액트가 함수인 컴포넌트를 호출하는 것을 뜻합니다. 반환되는 JSX는 그 시점의 UI의 스냅샷입니다. 프랍, 이벤트 핸들러 그리고 지역 변수로 랜더시에 상태를 통해 계산됩니다.

사진과 영화와 다르게 UI는 상호작용의 스냅샷입니다.
그것은 이벤트 핸들러와 같이 특정 입력에 따른 반응을 하는 로직을 포함합니다.
리액트는 스냅샷과 이벤트 핸들러를 연결하기위해 화면을 업데이트합니다.
그 결과로, 버튼을 클릭하는 것은 JSX에서 클릭 핸들러를 유발합니다.

리액트가 컴포넌트를 리랜더링 할 때

1. 리액트는 당신의 함수를 다시 호출
2. 함수가 새로운 JSX 스냅샷을 반환
3. 반환된 스냅샷을 화면에 업데이트

컴포넌트의 메모리로, 상태는 함수가 반환되고 사라지는 기존의 변수와 다릅니다. 상태는 리액트 내부에 존재합니다.
리액트가 당신의 컴포넌트를 호출할 때, 그것은 특정 랜더를 위한 상태의 스냅샷을 제공합니다.
컴포넌트는 랜더로부터 모든 계산된 상태 값을 새로운 프랍과 이벤트 핸들러 UI를 스냅샷으로 반환합니다.

이것이 어떻게 동작하는 지 간단한 실험이 있습니다. 이 예에서, "+3" 버튼을 클릭하면 아마 3번을 증가시킬 것이라고 예상할 것입니다. 왜냐하면 `setNumber(number + 1)`를 세번 호출했기 때문입니다.

"+3" 버튼을 클릭하면 어떻게 되는지

`number`가 오직 클릭당 한 번씩만 증가하는 것을 볼 수 있습니다.

상태를 변경하는 것은 오직 그것의 다음 랜더만 변경합니다. 첫 번째 랜더에서, `number`는 `0`입니다. 이것이 `setNumber(number + 1)`이 호출 되어도 `number`가 `0`인 이유입니다.

```jsx
<button
  onClick={() => {
    setNumber(number + 1);
    setNumber(number + 1);
    setNumber(number + 1);
  }}
>
  +3
</button>
```

버튼의 클릭 핸들러가 리액트에게 시키는 일은 다음과 같습니다.

1. `setNumber(number + 1)`: number는 0이므로 `setNumber(0 + 1)`
   - 리액트는 다음 랜더에 number를 1로 변경하기위해 준히합니다.
2. `setNumber(number + 1)`: number는 0이므로 `setNumber(0 + 1)`
   - 리액트는 다음 랜더에 number를 1로 변경하기위해 준히합니다.
3. `setNumber(number + 1)`: number는 0이므로 `setNumber(0 + 1)`
   - 리액트는 다음 랜더에 number를 1로 변경하기위해 준히합니다.

`setNumber(number + 1)`를 3번 호출했음에도, 이 랜더에서 이벤트 핸들러 number는 항상 0입니다.당신은 상태를 1로 세 번 설정한 것입니다.
이것이, 이벤트 핸들러가 끝나고, number가 3이 아닌 1로 리랜더 되는 이유입니다.

당신은 코드에서 상태 변수를 대입해 멘탈적으로 시각화하할 수 있습니다. 이번 랜더에서 number 상태 변수는 0이기에 핸들러는 다음과 같습니다.

```jsx
<button
  onClick={() => {
    setNumber(0 + 1);
    setNumber(0 + 1);
    setNumber(0 + 1);
  }}
>
  +3
</button>
```

다음 랜더에서 number는 1이기에 클릭 핸들러는 다음과 같습니다.

```jsx
<button
  onClick={() => {
    setNumber(1 + 1);
    setNumber(1 + 1);
    setNumber(1 + 1);
  }}
>
  +3
</button>
```

버튼을 다시 눌렀을 때, 카운터가 2가되고 3이 되는 이유입니다.

## State over time

버튼을 눌렀을 때 무엇을 알럿할 지 맞혀보세요.

```jsx
export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber(number + 5);
          alert(number);
        }}
      >
        +5
      </button>
    </>
  );
}
```

만약 이전처럼 대입 방법을 사용했다면, 알럿이 5을 보여줄 것을 예상했을 것입니다.

```jsx
setNumber(0 + 5);
alert(0);
```

그런데, 만약 타이머가 알럿을 보여주도록 하면 어떨까요? 이것은 컴포넌트가 리랜더된 후에 나타날까요? 0일까요 5일까요?

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
          setTimeout(() => {
            alert(number);
          }, 3000);
        }}
      >
        +5
      </button>
    </>
  );
}
```

놀라셨나요? 대입 방법을 사용했다면, 알럿에 전달된 상태의 스냅샷을 보셨을 것입니다.

```jsx
setNumber(0 + 5);
setTimeout(() => {
  alert(0);
}, 3000);
```

리액트에 저장된 상태는 알럿이 실행되고 바뀔 것입니다. 그러나 이것은 유저가 상호작용 했을 때의 상태를 스냅샷으로 갖습니다!

상태 변수는 랜더 도중에는 절대 바뀌지 않습니다. 이벤트 핸들러의 코드가 비동기적이라고 해도 말이죠. `onClick`의 랜더 안에서, number 값은 `setNumber(number + 5)`가 호출 됐더라도 여전히 0입니다. 컴포넌트를 호출 했을 때 그 값은 리액트가 스냅샷을 찍기에 고정 됩니다.

타이밍 실수를 하지않도록 하는 예가 있습니다.
아래의 폼은 5초의 지연 후 전송됩니다. 시나리오를 상상해보세요.

1. 전송 버튼을 눌러 앨리스에게 안녕을 보낸다.
2. 5초의 지연시간이 끝나기 전, 수신자를 밥으로 변경한다.

알럿이 무엇을 표시할까요?

```jsx
import { useState } from "react";

export default function Form() {
  const [to, setTo] = useState("Alice");
  const [message, setMessage] = useState("Hello");

  function handleSubmit(e) {
    e.preventDefault();
    setTimeout(() => {
      alert(`You said ${message} to ${to}`);
    }, 5000);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        To:{" "}
        <select value={to} onChange={(e) => setTo(e.target.value)}>
          <option value="Alice">Alice</option>
          <option value="Bob">Bob</option>
        </select>
      </label>
      <textarea
        placeholder="Message"
        value={message}
        onChange={(e) => setMessage(e.target.value)}
      />
      <button type="submit">Send</button>
    </form>
  );
}
```

리액트는 랜더 도중의 상태를 고정값으로 같습니다.
코드가 실행될 때, 상태가 변화하는 것을 걱정할 필요 없습니다.

만약 최신의 상태를 리랜더전에 읽고싶다면?
상태 업데이트 함수를 사용하고 싶을 것입니다.

## Recap

- 새로운 랜더를 위해 상태를 설정
- 리액트는 상태를 선반에 있는 것 처럼 컴포넌트 밖에 저장
- `useState`를 호출 했을 시, 그 랜더를 위한 리액트는 상태의 스냅샷을 제공
- 상태와 이벤트 핸들러는 리랜더시 살아남지 못합니다. 매 랜더에는 그에 맞는 이벤트 핸들러가 존재
- 매 랜더시(또는 함수 안) 항상 리액트가 제공해준 스냅샷을 볼 수 있습니다.
- 이벤트 핸들러에 대입을 통해, 리액트가 어떻게 JSX를 랜더하는지 비슷하게 알 수 있습니다.
- 과거에 만들어진 이벤트 핸들러는 그 때 생성된 상태 값을 갖습니다.

---

## What I Learned

상태라는 것을 많이 안다고 생각했는데, 막상 상태가 뭐냐는 질문을 받았을 때 제법 식은땀이 난 경험을 했었다. 상태는 컴포넌트의 메모리이자 스냅샷이라는 것을 항상 기억해야겠다.

출처

- https://react.dev/learn/state-as-a-snapshot
