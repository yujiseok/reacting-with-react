- [Preserving and Resetting State](#preserving-and-resetting-state)
  - [The UI tree](#the-ui-tree)
  - [State is tied to a position in the tree](#state-is-tied-to-a-position-in-the-tree)
  - [Same component at the same position preserves state](#same-component-at-the-same-position-preserves-state)
  - [Pitfall](#pitfall)
  - [Different components at the same position reset state](#different-components-at-the-same-position-reset-state)
  - [Pitfall](#pitfall-1)
  - [Resetting state at the same position](#resetting-state-at-the-same-position)
    - [Option 1: Rendering a component in different positions](#option-1-rendering-a-component-in-different-positions)
    - [Option 2: Resetting state with a key](#option-2-resetting-state-with-a-key)
  - [Resetting a form with a key](#resetting-a-form-with-a-key)
  - [DEEP DIVE - Preserving state for removed components](#deep-dive---preserving-state-for-removed-components)
  - [Recap](#recap)
  - [What I Learned](#what-i-learned)

# Preserving and Resetting State

상태는 컴포넌트 간에 독립적입니다. 리액트는 UI 트리에서 상태가 어떤 컴포넌트 기반인지 항상 추적합니다.
당신은 언제 상태를 보존할지 그리고 리렌더 사이에 언제 초기화할지 제어할 수 있습니다.

## The UI tree

브라우저는 모델 UI를 위해 많은 트리 구조들을 사용합니다.
돔은 HTML 요소들을, CSSOM은 CSS 요소들을 나타냅니다.
심지어 접근성 트리도 있습니다.

리액트 역시 UI를 관리하고 모델링하기 위해 트리 구조를 사용합니다.
리액트는 JSX로부터 UI 트리를 생성합니다.
그 후 리액트 돔은 브라우저의 돔 요소를 변경합니다.

[label](https://react.dev/_next/image?url%3D%2Fimages%2Fdocs%2Fdiagrams%2Fpreserving_state_dom_tree.dark.png%26w%3D1920%26q%3D75)

## State is tied to a position in the tree

당신이 컴포넌트에 상태를 추가하면, 당신은 상태는 컴포넌트 내에 존재한다고 생각합니다. 하지만, 상태는 사실 리액트가 들고있습니다.
리액튼 각 상태들을 UI 트리에 존재하는 정확한 컴포넌트와 관계를 맺도록합니다.

여기, 다른 두 공간에서 렌더 되는 하나의 `<Counter />` JSX 태그가 있습니다.

이 예의 트리는 다음과 같습니다.

[label](https://react.dev/_next/image?url%3D%2Fimages%2Fdocs%2Fdiagrams%2Fpreserving_state_tree.dark.png%26w%3D828%26q%3D75)

이것은 각자의 트리 위치에서 렌더된 서로 다른 두 개의 카운터입니다.
리액트를 사용할 때, 이런 위치를 생각할 필요 없지만, 어떻게 동작하는 지 알면 유용합니다.

리액트에서, 스크린에 있는 각 컴포넌트는 완전히 독립적입니다.
예를 들어 만약 두 카운터 컴포넌트를 렌더 시키면, 각 컴포넌트는 독립적인 `score`와 `hover` 상태를 갖습니다.

보다시피, 하나의 카운터가 업데이트 되면, 그 컴포넌트의 상태만 변경됩니다.

[label](https://react.dev/_next/image?url%3D%2Fimages%2Fdocs%2Fdiagrams%2Fpreserving_state_increment.dark.png%26w%3D1080%26q%3D75)

리액트는 컴포넌트를 같은 위치에 렌더하고 상태를 유지합니다.
예를 보기 위해 각 카운터를 증가시키고, 두 번째 컴포넌트를 제거한 후 다시 추가해 증가시켜보세요:

```jsx
import { useState } from "react";

export default function App() {
  const [showB, setShowB] = useState(true);
  return (
    <div>
      <Counter />
      {showB && <Counter />}
      <label>
        <input
          type="checkbox"
          checked={showB}
          onChange={(e) => {
            setShowB(e.target.checked);
          }}
        />
        Render the second counter
      </label>
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = "counter";
  if (hover) {
    className += " hover";
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>Add one</button>
    </div>
  );
}
```

두 번째 컴포넌트가 렌더를 멈추면, 그것의 상태는 완전히 제거됩니다.
이것은 리액트가 컴포넌트를 제거했을 때, 상태를 제거하기 때문입니다.

[label](https://react.dev/_next/image?url%3D%2Fimages%2Fdocs%2Fdiagrams%2Fpreserving_state_remove_component.dark.png%26w%3D1080%26q%3D75)

"Render the second counter"를 눌렀을 때, 두 번째 카운터와 그것의 상태는 초기화 되고 돔에 추가됩니다.

[label](https://react.dev/_next/image?url%3D%2Fimages%2Fdocs%2Fdiagrams%2Fpreserving_state_add_component.dark.png%26w%3D1080%26q%3D75)

리액트는 컴포넌트의 상태를 UI 트리상 위치에 렌더되었을 시 보존합니다.
만약 제거되거나, 다른 컴포넌트가 그 위치에 랜더될 시, 리액트는 상태를 버립니다.

## Same component at the same position preserves state

이 예에서, 두 개의 다른 카운터 컴포넌트가 있습니다:

```jsx
import { useState } from "react";

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? <Counter isFancy={true} /> : <Counter isFancy={false} />}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={(e) => {
            setIsFancy(e.target.checked);
          }}
        />
        Use fancy styling
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = "counter";
  if (hover) {
    className += " hover";
  }
  if (isFancy) {
    className += " fancy";
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>Add one</button>
    </div>
  );
}
```

당신이 체크박스를 클릭할 때, 카운터의 상태는 초기화 되지 않습니다. `isFancy`가 `true`거나 `false`이든지, 당신은 항상 루트 `App`의 첫 번째 컴포넌트로 카운터를 갖습니다.

[label](https://react.dev/_next/image?url%3D%2Fimages%2Fdocs%2Fdiagrams%2Fpreserving_state_same_component.dark.png%26w%3D1200%26q%3D75)

이것은 같은 위치의 같은 컴포넌트입니다. 리액트의 관점에서 이것은 동일한 컴포넌트입니다.

## Pitfall

UI 트리의 위치이지 JSX 마크업이 아니라는 것을 기억하세요.
이 태그는 `if` 안과 밖 두 개의 반환 조항이 있습니다.

당신은 체크박스를 클릭 했을 시 상태가 초기화될 것을 예상하지만, 아닙니다.
이것은 두 카운터 태그가 같은 위치에 렌더되기 때문입니다.
리액트는 함수 안에 조건을 어디에 뒀는지 모릅니다. 오직 트리가 반환한 것을 봅니다.

두 경우에서, `App` 컴포넌트는 카운터가 있는 `<div>`를 첫 번째 자식으로 반환합니다. 리액트에서, 이 두 카운터들은 같은 주소를 갖습니다: 루트의 첫 번째 자식의 첫 번째 자식
이것이 리액트가 이전 랜더와 다음 랜더를 맞추는 방식으로, 로직을 어떻게 구성했는지 개의치않습니다.

## Different components at the same position reset state

이 예에서, 체크박스를 체크하는 것은 카운터를 p로 대체합니다.

```jsx
import { useState } from "react";

export default function App() {
  const [isPaused, setIsPaused] = useState(false);
  return (
    <div>
      {isPaused ? <p>See you later!</p> : <Counter />}
      <label>
        <input
          type="checkbox"
          checked={isPaused}
          onChange={(e) => {
            setIsPaused(e.target.checked);
          }}
        />
        Take a break
      </label>
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = "counter";
  if (hover) {
    className += " hover";
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>Add one</button>
    </div>
  );
}
```

당신이 똑같은 위치에 다른 컴포넌트로 교체했을 때.
처음으로, div의 첫 번째 자식은 카운터입니다.
하지만, p로 변경했을 시, 리액트는 카운터를 UI 트리에서 제거한 후 상태를 제거합니다.

[label](https://react.dev/_next/image?url%3D%2Fimages%2Fdocs%2Fdiagrams%2Fpreserving_state_diff_pt1.dark.png%26w%3D1920%26q%3D75)

[label](https://react.dev/_next/image?url%3D%2Fimages%2Fdocs%2Fdiagrams%2Fpreserving_state_diff_pt2.dark.png%26w%3D1920%26q%3D75)

또한, 똑같은 위치에 다른 컴포넌트를 랜더시킬 시, 하위 트리까지 초기화합니다.

```jsx
import { useState } from "react";

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <div>
          <Counter isFancy={true} />
        </div>
      ) : (
        <section>
          <Counter isFancy={false} />
        </section>
      )}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={(e) => {
            setIsFancy(e.target.checked);
          }}
        />
        Use fancy styling
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = "counter";
  if (hover) {
    className += " hover";
  }
  if (isFancy) {
    className += " fancy";
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>Add one</button>
    </div>
  );
}
```

카운터 상태는 체크박스를 클릭 시 초기화 됩니다.
비록 카운터를 랜더하지만, div의 첫 번째 자식이 div -> section 태그로 바뀝니다.
자식 div가 돔에서 사라지면, 모든 하위 트리(카운터와 상태를 포함한)도 제거됩니다.

[label](https://react.dev/_next/image?url%3D%2Fimages%2Fdocs%2Fdiagrams%2Fpreserving_state_diff_same_pt1.dark.png%26w%3D1920%26q%3D75)

[label](https://react.dev/_next/image?url%3D%2Fimages%2Fdocs%2Fdiagrams%2Fpreserving_state_diff_same_pt2.dark.png%26w%3D1920%26q%3D75)

만약 리랜더시 상태를 보존하고 싶다면, 트리의 구조는 하나의 랜더와 다른 랜더가 매치업될 필요가 있습니다.
만약 구조가 다르다면, 상태는 파괴됩니다. 이유는 리액트가 컴포넌트가 트리에서 제거되면 상태도 제거하기 때문입니다.

## Pitfall

이것이 당신이 중첩된 함수 컴포넌트 정의를 하면 안되는 이유입니다.

`MyComponent` 내부에 정의된 `MyTextField`가 있습니다.

```jsx
import { useState } from "react";

export default function MyComponent() {
  const [counter, setCounter] = useState(0);

  function MyTextField() {
    const [text, setText] = useState("");

    return <input value={text} onChange={(e) => setText(e.target.value)} />;
  }

  return (
    <>
      <MyTextField />
      <button
        onClick={() => {
          setCounter(counter + 1);
        }}
      >
        Clicked {counter} times
      </button>
    </>
  );
}
```

버튼을 클릭 시, 인풋의 상태는 사라집니다. `MyComponent`가 매 랜더시 다른 `MyTextField`가 생성되기 때문입니다.
당신은 다른 컴포넌트를 같은 위치에 랜더합니다. 리액트는 모든 상태를 초기화합니다.
이것은 버그와 퍼포먼스 문제를 유발합니다.
이것을 피하기 위해선, 항상 최상위에 함수형 컴포넌트를 정의하고 중첩하지 않는 것입니다.

## Resetting state at the same position

기본적으로, 리액트는 같은 위치의 상태를 보존합니다. 보통, 이것은 당신이 원하는 것이므로 말이 됩니다. 하지만, 가끔씩, 컴포넌트의 상태를 초기화하길 원합니다.
두 선수의 점수를 기록하는 앱을 상상해보세요.

현재는, 선수를 바꿔도 점수가 보존됩니다.
두 카운터는 같은 위치에 보여져, 리액트는 `person` 프랍이 변경된 같은 컴포넌트로 봅니다.

하지만, 이 앱에서 그들은 서로 다른 두 개의 카운터 여야합니다.
그들은 같은 UI 위치에 나타나지만, 하나는 테일러, 나머지는 사라의 카운터 여야합니다.

상태를 초기화하는 두 가지 방법이 있습니다.

1. 다른 위치에 컴포넌트 랜더시키기
2. 각 컴포넌트에 키를 통해 명백히하기

### Option 1: Rendering a component in different positions

만약 카운터를 독립시키고 싶다면, 다른 두 위치에 랜더시킬 수 있습니다.

```jsx
import { useState } from "react";

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA && <Counter person="Taylor" />}
      {!isPlayerA && <Counter person="Sarah" />}
      <button
        onClick={() => {
          setIsPlayerA(!isPlayerA);
        }}
      >
        Next player!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = "counter";
  if (hover) {
    className += " hover";
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>
        {person}'s score: {score}
      </h1>
      <button onClick={() => setScore(score + 1)}>Add one</button>
    </div>
  );
}
```

- 처음에, `isPlayerA`는 `true`입니다. 즉 첫 위치는 카운터 상태를 가지며 두 번째는 비어있습니다.
- 당신이 다음 선수를 클릭했을 시 첫 위치는 비게되고 두 번째가 카운터를 갖습니다.

[label](https://react.dev/_next/image?url%3D%2Fimages%2Fdocs%2Fdiagrams%2Fpreserving_state_diff_position_p1.dark.png%26w%3D1080%26q%3D75)

[label](https://react.dev/_next/image?url%3D%2Fimages%2Fdocs%2Fdiagrams%2Fpreserving_state_diff_position_p2.dark.png%26w%3D1080%26q%3D75)

[label](https://react.dev/_next/image?url%3D%2Fimages%2Fdocs%2Fdiagrams%2Fpreserving_state_diff_position_p3.dark.png%26w%3D1080%26q%3D75)

각 카운터의 상태는 돔에서 사라질 때, 파괴됩니다. 이것이 버튼을 클릭시 초기화되는 이유입니다.

이 해결법은 같은 장소에 독립적인 컴포넌트가 적을 때 편리합니다.
이 예에서 오직 두 개만 갖기에 서로 다른 JSX를 랜더하는 것이 혼란스럽지 않습니다.

### Option 2: Resetting state with a key

더 제너릭하게, 컴포넌트의 상태를 초기화하는 법이 있습니다.

당신은 리스트를 랜더하는 것에서 키를 봤을 것입니다. 키는 리스트만을 위한 것이 아닙니다.
당신은 키를 통해 컴포넌트를 구별할 수 있습니다.
기본적으로, 리액트는 부모의 순서로 컴포넌트를 구별합니다.
하지만 키는 첫 번째 카운터, 두 번째 카운터가 아닌, 리액트에 특정한 카운터를 말해줍니다.
이 방식으로, 리액트는 테일러의 카운터가 트리 어디에 존재할지 알게합니다.

이 예에서, 두 카운터는 같은 장소에 있어도 상태를 공유하지 않습니다.

```jsx
import { useState } from "react";

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter key="Taylor" person="Taylor" />
      ) : (
        <Counter key="Sarah" person="Sarah" />
      )}
      <button
        onClick={() => {
          setIsPlayerA(!isPlayerA);
        }}
      >
        Next player!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = "counter";
  if (hover) {
    className += " hover";
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>
        {person}'s score: {score}
      </h1>
      <button onClick={() => setScore(score + 1)}>Add one</button>
    </div>
  );
}
```

테일러와 사라를 교체하는 것은 상태를 보존하지 않습니다. 이것은 그들에 다른 키를 부여해서입니다.

```jsx
{
  isPlayerA ? (
    <Counter key="Taylor" person="Taylor" />
  ) : (
    <Counter key="Sarah" person="Sarah" />
  );
}
```

키를 특정하는 것은 리액트에게 키를 순서 대신 위치로 사용하라고 알려줍니.
이 방식으로, JSX의 같은 장소에 랜더시켜도 리액트는 서로 다른 카운터로 보며 상태를 보존하지 않습니다.
카운터가 스크린에 보일 때마다, 그것의 상태가 만들어집니다.
제거될 때마다, 상태가 제거됩니다.
그들을 토글하는 것은 그들의 상태를 계속해서 초기화합니다.

> Note
> 키는 전역적으로 유니크한 것이 아닙니다. 그들은 부모의 위치에서 특정합니다.

## Resetting a form with a key

키를 통해 상태를 초기화하는 것은 폼을 다룰 때 유용합니다.

이 채팅 앱에서 챗 컴포넌트는 인풋 상태를 갖습니다.

```jsx
import { useState } from "react";
import Chat from "./Chat.js";
import ContactList from "./ContactList.js";

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={(contact) => setTo(contact)}
      />
      <Chat contact={to} />
    </div>
  );
}

const contacts = [
  { id: 0, name: "Taylor", email: "taylor@mail.com" },
  { id: 1, name: "Alice", email: "alice@mail.com" },
  { id: 2, name: "Bob", email: "bob@mail.com" },
];
```

인풋에 무언가 작성하고 앨리스나 밥을 수신자로 변경해보세요.
당신은 챗이 트리의 같은 위치에 랜더되어 상태가 보존되는 것을 알 수 있습니다.

만은 앱에서 이것은 유효한 동작이지만, 챗 앱은 아닙니다!
당신은 잘못된 수신자에게 적은 내용이 전송되길 원치않습니다.
이것을 고치기 위해 키를 추가합니다.

```jsx
<Chat key={to.id} contact={to} />
```

이것은 다른 수신자를 클릭 시, 컴포넌트가 상태를 포함해 처음부터 재생성되는 것을 보장합니다.
리액트는 또한 돔을 재사용하지 않고 재생성합니다.

## DEEP DIVE - Preserving state for removed components

진짜 챗 앱에서, 다른 수신자를 눌렀을 시 유지되길 원할 것입니다.
컴포넌트가 보이지 않아도 몇 가지 상태를 유지할 수 있는 법이 있습니다.

- 현재 하나만 랜더하는 것이 아닌 모든 챗을 랜더하고, 나머지를 CSS로 숨기는 것. 챗들은 트리에서 사라지지 않습니다. 즉 그들의 지역 상태는 보존됩니다. 이 해결법은 간단한 UI에 적합합니다. 하지만 트리가 커지고 돔 노드가 많아지면 매우 느려집니다.

- 각 수신자의 메세지를 부모 컴포넌트에서 관리할 수 있도록 상태를 리프트할 수 있습니다. 이 방법은, 부모가 중요한 정보를 갖고 있기에 자식 컴포넌트가 제거되어도 문제 없습니다. 이것이 가장 흔한 해결법입니다.

- 다른 상태를 추가해 사용할 수 있습니다. 예를 들어, 페이지를 닫아도 메세지를 보존하고 싶을 수 있습니다. 이것을 구현하기 위해, 챗 컴포넌트의 상태를 로컬 스토리지에서 읽어오도록 할 수 있으며, 저장할 수 있습니다.

어떤 방법을 당신이 고르든, 앨리스와 챗은 밥과의 챗과 구분되어야 하며, 키를 제공함으로써 가능하게 합니다.

## Recap

- 리액트는 같은 위치에 랜더되면 상태를 보존
- 상태는 JSX 태그에 유지되는 것이 아니며, 트리 위치에 따라
- 다른 키를 제공함으로써 상태를 초기화할 수 있다.
- 함수 컴포넌트 정의를 중첩하지 말아라, 원치 않게 상태 초기화

---

## What I Learned

상태라는 것이 어떻게 보존되는 지 사실 생각을 해본적이 없는 데, 이번 기회에 알게 되었다.
또 키가 단순히 리스트에만 사용되는 줄 알았는데, 키의 다른 용법을 알게 되었다.

출처

- https://react.dev/learn/preserving-and-resetting-state
