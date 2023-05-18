- [Choosing the State Structure](#choosing-the-state-structure)
  - [Principles for structuring state](#principles-for-structuring-state)
  - [Group related state](#group-related-state)
  - [Pitfall](#pitfall)
  - [Avoid contradictions in state](#avoid-contradictions-in-state)
  - [Avoid redundant state](#avoid-redundant-state)
  - [DEEP DIVE - Don’t mirror props in state](#deep-dive---dont-mirror-props-in-state)
  - [Avoid duplication in state](#avoid-duplication-in-state)
  - [Avoid deeply nested state](#avoid-deeply-nested-state)
  - [Recap](#recap)
  - [What I Learned](#what-i-learned)

# Choosing the State Structure

상태를 잘 구조화 하는 것은 수정 및 디버그 하기 좋은 컴포넌트와 지속적인 버그의 원인이 되는 구성 요소 간에 차이가 있을 수 있습니다.
상태를 구조화할 때 팁들이 있습니다.

## Principles for structuring state

컴포넌트에 상태를 주기위해선, 얼마나 많은 상태 변수가 필요한지와 데이터는 어떤 모양일지 선택해야합니다.
차선의 상태 구조와 올바른 프로그램을 작성하기 위한 몇 개의 좋은 선택지가 있습니다.

1. Group related state. 만약 당신이 두 개 이상의 상태를 동시에 변경한다면, 하나의 상태로 병합하는 것을 고려해보세요.
2. Avoid contradictions in state. 상태가 구조화 되면서 몇 개의 모순적인 상태들이 서로 불일치 됩니다. 이런 경우를 피하세요.
3. Avoid redundant state. 만약 컴포넌트의 프랍이나 상태로 랜더 중 계산할 수 있다면, 컴포넌트의 상태로 두지 마세요.
4. Avoid duplication in state. 만약 같은 데이터가 여러 상태에서 중복되거나, 객체에 중첩 된다면, 이것은 싱크를 맞추기 어렵게 합니다. 이런 중복을 피하세요.
5. Avoid deeply nested state. 깊은 계층 구조를 가진 상태는 변경시키기 불편합니다. 가능하다면, 상태를 플랫하게 두세요.

이런 원칙은 실수 없이 상태를 변경하게 해줍니다.
장황하고 중복된 상태를 없애는 것은 싱크를 맞추도록 보장합니다. 이것은 데이터베이스 개발자들이 데이터 베이스를 표준화해 버그를 줄이고 싶어하는 것과 닮아있습니다.
아인슈타인의 말을 바꿔 말하면 "상태를 단순하게 만들 순 있지만, 단순하지 않습니다."

이제 이 원칙들이 어떻게 적용되는지 봅시다.

## Group related state

당신은 하나 혹은 여러 상태 변수를 사용하는 것에 있어서 불확실할 것입니다.

이렇게?

```jsx
const [x, setX] = useState(0);
const [y, setY] = useState(0);
```

혹은 이렇게?

```jsx
const [position, setPosition] = useState({ x: 0, y: 0 });
```

기술적으로, 둘 다 사용할 수 있습니다. 하지만 두 상태가 항상 같이 변한다면, 하나의 상태 변수로 합치는 것이 좋습니다.
그렇게 되면, 당신은 싱크를 맞출일을 까먹을 수 없게됩니다.

얼마나 많은 상태가 필요할지 모를때, 객체나 배열로 그룹화 할 수 있습니다. 예를 들어, 유저가 커스텀 필드를 추가할 수 있는 폼에 도움이 됩니다.

## Pitfall

만약 당신의 상태 변수가 객체일 시, 나머지 필드를 복사하지 않고 하나의 필드만 업데이트할 수 없다는 것을 기억하세요.
예를 들어, 위의 예제에서 `setPosition({ x: 100 })`는 `y` 프로퍼티가 있끼에 불가합니다.
대신 `x`만 변경하고 싶다면, `setPosition({ ...position, x: 100 })`로 변경 하거나 두 상태로 나눠 `setX(100)`로 변경합니다.

## Avoid contradictions in state

`isSending`와 `isSent`를 갖는 호텔 피드백 폼이 있습니다.

```jsx
import { useState } from "react";

export default function FeedbackForm() {
  const [text, setText] = useState("");
  const [isSending, setIsSending] = useState(false);
  const [isSent, setIsSent] = useState(false);

  async function handleSubmit(e) {
    e.preventDefault();
    setIsSending(true);
    await sendMessage(text);
    setIsSending(false);
    setIsSent(true);
  }

  if (isSent) {
    return <h1>Thanks for feedback!</h1>;
  }

  return (
    <form onSubmit={handleSubmit}>
      <p>How was your stay at The Prancing Pony?</p>
      <textarea
        disabled={isSending}
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <br />
      <button disabled={isSending} type="submit">
        Send
      </button>
      {isSending && <p>Sending...</p>}
    </form>
  );
}

// Pretend to send a message.
function sendMessage(text) {
  return new Promise((resolve) => {
    setTimeout(resolve, 2000);
  });
}
```

코드가 실행되면서, 불가능한 상태의 가능성을 열어둡니다. 예를 들어, 만약 `setIsSent`와 `setIsSending`를 동시에 호출하는 것을 까먹었을 경우, `isSending`과 `isSent`는 동시에 `true`가 됩니다. 만약 컴포넌트가 복잡해지면, 이것은 무슨 일이 일어나는지 파악하기 어렵게 합니다.

`isSending`과 `isSent`가 동시에 절대 `true`가 되면 안되기에, 'typing' (initial), 'sending', 그리고 'sent'를 갖는 `status` 상태 변수로 대체하는 것이 좋습니다.

```jsx
export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [status, setStatus] = useState('typing');

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('sending');
    await sendMessage(text);
    setStatus('sent');
  }

  const isSending = status === 'sending';
  const isSent = status === 'sent';

  // ...
```

가독성이 있는 변수로 선언할 수 있습니다.

```jsx
const isSending = status === "sending";
const isSent = status === "sent";
```

그들은 상태 변수가 아니므로, 각각의 싱크를 맞추는 것을 고민할 필요가 없습니다.

## Avoid redundant state

컴포넌트의 프랍이나 상태로 랜더 중 계산할 수 있는 변수의 경우 컴포넌트의 상태로 두면 안됩니다.

예를 들어, 아래의 폼을 보세요. 이것은 동작하지만, 불필요한 상태를 찾으셨나요?

```jsx
import { useState } from "react";

export default function Form() {
  const [firstName, setFirstName] = useState("");
  const [lastName, setLastName] = useState("");
  const [fullName, setFullName] = useState("");

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
    setFullName(e.target.value + " " + lastName);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
    setFullName(firstName + " " + e.target.value);
  }

  return (
    <>
      <h2>Let’s check you in</h2>
      <label>
        First name: <input value={firstName} onChange={handleFirstNameChange} />
      </label>
      <label>
        Last name: <input value={lastName} onChange={handleLastNameChange} />
      </label>
      <p>
        Your ticket will be issued to: <b>{fullName}</b>
      </p>
    </>
  );
}
```

이 폼은 세 가지 상태 변수를 갖습니다. 하지만 `fullName`은 불필요합니다.
`firstName`과 `lastName` 으로 랜더 중 계산할 수 있습니다. 그러니 상태에서 제거하세요.

```jsx
const fullName = firstName + " " + lastName;
```

그 결과로 체인지 핸들러는 변경할 필요가 없어졌습니다. `setFirstName`과 `setLastName`을 호출할 때, 리랜더가 일어나고, `fullName`은 새로운 데이터로 계산됩니다.

## DEEP DIVE - Don’t mirror props in state

불필요한 상태의 예로는 다음과 같습니다.

```jsx
function Message({ messageColor }) {
  const [color, setColor] = useState(messageColor);
```

`color` 상태 변수는 `messageColor` 프랍으로 초기화 됩니다.
문제는, 부모 컴포넌트가 `messageColor`에 다른 값을 넘겨준다면, `color` 상태 변수는 변경되지 않을 것입니다. 상태는 첫 번째 랜더에서 초기화 됩니다!

이것은 상태 안의 몇 프랍들을 미러링하는 것이 혼란을 유발하는 이유입니다. 대신에, `messageColor` 프랍을 직접적으로 코드에 적용하세요. 만약 짧은 이름을 원한다면 상수를 사용하세요.

```jsx
function Message({ messageColor }) {
  const color = messageColor;
```

이런 방식으로 부모에게 전달 받은 프랍이 싱크를 맞추지 못하는 일은 없습니다.

프랍을 미러링하는 것은 특정 프랍에 모든 변경을 원치 않을 때 사용됩니다.
컨벤션으로 프랍의 이름을 `initial` 또는 `default`로 명확히 해 새로운 값은 무시되도록 합니다.

```jsx
function Message({ initialColor }) {
  // The `color` state variable holds the *first* value of `initialColor`.
  // Further changes to the `initialColor` prop are ignored.
  const [color, setColor] = useState(initialColor);
```

## Avoid duplication in state

이 메뉴 컴포넌트는 여행 간식을 선택하도록 해줍니다.

```jsx
import { useState } from "react";

const initialItems = [
  { title: "pretzels", id: 0 },
  { title: "crispy seaweed", id: 1 },
  { title: "granola bar", id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState(items[0]);

  return (
    <>
      <h2>What's your travel snack?</h2>
      <ul>
        {items.map((item) => (
          <li key={item.id}>
            {item.title}{" "}
            <button
              onClick={() => {
                setSelectedItem(item);
              }}
            >
              Choose
            </button>
          </li>
        ))}
      </ul>
      <p>You picked {selectedItem.title}.</p>
    </>
  );
}
```

이것은 선택된 아이템을 `selectedItem` 상태 변수에 저장합니다. 하지만, 이것은 좋지 않습니다:
`selectedItem`의 컨텐츠는 `items` 리스트 중 하나의 아이템과 동일한 객체입니다.
이것은 두 장소에서 복사될 수 있음을 뜻합니다.

이게 왜 문제일까요? 각각을 수정 가능하도록 해봅시다.

```jsx
import { useState } from "react";

const initialItems = [
  { title: "pretzels", id: 0 },
  { title: "crispy seaweed", id: 1 },
  { title: "granola bar", id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState(items[0]);

  function handleItemChange(id, e) {
    setItems(
      items.map((item) => {
        if (item.id === id) {
          return {
            ...item,
            title: e.target.value,
          };
        } else {
          return item;
        }
      })
    );
  }

  return (
    <>
      <h2>What's your travel snack?</h2>
      <ul>
        {items.map((item, index) => (
          <li key={item.id}>
            <input
              value={item.title}
              onChange={(e) => {
                handleItemChange(item.id, e);
              }}
            />{" "}
            <button
              onClick={() => {
                setSelectedItem(item);
              }}
            >
              Choose
            </button>
          </li>
        ))}
      </ul>
      <p>You picked {selectedItem.title}.</p>
    </>
  );
}
```

먼저 선택 버튼을 클릭 후 아이템을 수정하면, 인풋은 변경된 반면 아래의 레이블은 변경되지 않았습니다.
이것은 당신이 상태를 복제한 후, `selectedItem`을 변경하지 않았기 때문입니다.

비록 `selectedItem`를 변경할 수 있지만, 가장 쉬운 방법은 제거하는 것입니다. 예에서, `selectedItem`객체 대신 당신은 `selectedId`를 둘 수 있습니다. 그 후 `items`를 순회하며 ID가 같은 아이템을 찾습니다.

```jsx
import { useState } from "react";

const initialItems = [
  { title: "pretzels", id: 0 },
  { title: "crispy seaweed", id: 1 },
  { title: "granola bar", id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedId, setSelectedId] = useState(0);

  const selectedItem = items.find((item) => item.id === selectedId);

  function handleItemChange(id, e) {
    setItems(
      items.map((item) => {
        if (item.id === id) {
          return {
            ...item,
            title: e.target.value,
          };
        } else {
          return item;
        }
      })
    );
  }

  return (
    <>
      <h2>What's your travel snack?</h2>
      <ul>
        {items.map((item, index) => (
          <li key={item.id}>
            <input
              value={item.title}
              onChange={(e) => {
                handleItemChange(item.id, e);
              }}
            />{" "}
            <button
              onClick={() => {
                setSelectedId(item.id);
              }}
            >
              Choose
            </button>
          </li>
        ))}
      </ul>
      <p>You picked {selectedItem.title}.</p>
    </>
  );
}
```

이전 상태는 다음과 같이 복사 되었습니다.

- items = [{ id: 0, title: 'pretzels'}, ...]
- selectedItem = {id: 0, title: 'pretzels'}

그러나 변경 후는 다음과 같습니다.

- items = [{ id: 0, title: 'pretzels'}, ...]
- selectedId = 0

복제는 사라지고, 필요한 상태만 가질 수 있습니다.

만약 선택된 아이템을 변경할 시, 아래의 메세지가 즉시 변경될 것입니다. `setItems`이 리랜더를 일으키고, `items.find`로 변경된 제목의 아이템을 찾습니다. 당신은 선택된 아이디가 필요한 상태기에, 선택된 아이템을 상태로 가질 필요가 없어졌습니다. 나머지는 랜더 중 계산됩니다.

## Avoid deeply nested state

만약 행성, 대륙 그리고 나라들의 여행 계획을 상상해보세요. 당신은 아마 중첩된 객체나 배열로 상태를 구조화할 것입니다.

이제 방문한 곳을 삭제하는 버튼을 추가하고 싶습니다. 어떻게 할건가요? 중첩된 상태를 변경하기 위해선, 변경점부터 모든 객체를 복사해야합니다. 깊이 중첩된 장소를 삭제하는 것은 모든 부모의 장소를 복사해야합니다.
이런 코드는 제법 길어집니다.

만약 상태가 너무 깊어 변경하기 쉽지 않다면, 플랫하게 만드는 것을 고려하세요. 재구조화 하는 방법이 있습니다.
각 장소가 자식 배열을 갖는 트리 구조 대신, 자식의 장소 id를 갖도록 할 수 있습니다. 그 후 각 id에 맞는 장소를 매핑합니다.

이제 상태는 플랫해져, 상태를 변경하기 쉬워졌습니다.

장소를 삭제하기 위해, 두 단계만이 필요합니다.

- 변경된 버전의 부모 장소는 `childIds` 배열에서 삭제된 id를 배제해야합니다.
- 변경된 루트 테이블 객체는 변경된 부모 버전을 포함해야합니다.

당신은 마음껏 중첩된 상태를 만들 수 있지만, 플랫하게 만드는 것이 여러 문제를 해결해줍니다. 이것은 상태 변경을 쉽게 해주며, 다른 부분을 복사할 필요 없게 해줍니다.

자식 컴포넌트로 중첩 상태를 넘겨 이런 상태를 줄일 수 있다. 생명 주기가 짧은 UI에 적합합니다.

## Recap

- 만약 두 상태가 항상 동시에 변경된다면, 합치는 것을 고려
- 불가능한 상태를 만들지 않도록 주의
- 실수로 변경하는 것을 줄이도록 구조화
- 불필요하고 복제된 상태를 제거해 싱크를 맞추지 않아도 되도록
- 상태를 변경하는 것을 막는 경우가 아니라면 프랍을 상태로 지정하지 않기
- 선택같은 UI 패턴에서 객체 자체보단, 아이디를 상태로 갖도록
- 만약 중첩된 객체를 변경해야 한다면, 플랫하게 하도록 노력

---

## What I Learned

상태의 구조를 만들때, 위와 같은 원칙을 고려해서 만들어야겠다.
중첩된 객체를 다루는 것에 좀 더 익숙해질 필요가 있을 것 같다.

출처

- https://react.dev/learn/choosing-the-state-structure
