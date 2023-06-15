# Lifecycle of Reactive Effects

이펙트는 컴포넌트의 서로 다른 생명주기를 갖는다. 컴포넌트는 아마 마운트, 업데이트 그리고 언마운트 된다.
이펙트는 오직 두 가지를 할 수 있다: 무언가와 싱크를 맞추는 것, 그리고 싱크를 멈추는 것.
이런 주기는 이펙트가 프랍과 상태의 변화에 기반하면 여러번 일어날 수 있다.
리액트는 린터룰을 추가해 올바른 의존성을 추가했는지 검사해준다.
이것은 이펙트가 최소의 프랍과 상태에 싱크를 맞추도록 해준다.

## The lifecycle of an Effect

모든 리액트 컴포넌트는 동일한 생명주기를 갖는다.

- 컴포넌트가 화면에 추가되었을 때 마운트된다.
- 상호작용의 결과로 새로운 프랍이나 상태를 받아 컴포넌트가 업데이트된다.
- 화면에서 사라질 때 컴포넌트가 언마운트된다.

이것은 이펙트가 아니라, 컴포넌트 관점에서 생각하는 것이 좋다. 대신 각 이펙트가 컴포넌트 생명주기와 독립적이라고 생각해라.
이펙트는 현재 프랍과 상태를 외부 시스템과 어떻게 싱크를 맞출지 설명한다.
당신의 코드가 변경되면 싱크는 더 많이 혹은 적게 동작해야한다.

이것을 설명하기 위해, 챗 서버에 컴포넌트를 연결하는 이펙트를 생각해보라.

```jsx
const serverUrl = "https://localhost:1234";

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

당신의 이펙트 바디는 어떻게 싱크를 맞출지 설명한다.

이펙트에서 리턴되는 정리 함수는 싱크를 어떻게 멈출지 설명한다.

직관적으로, 당신은 리액트가 마운트 되면 싱크를 맞추고 언마운트 되면 싱크를 멈춘다는 것이라고 생각할 것이다.
하지만, 이것으로 이야기는 끝나지 않는다. 때로, 컴포넌트가 마운트된 상태여도 싱크를 맞추고 멈추는 것이 필요하다.

왜 필요한지, 언제 일어나는지, 그리고 이런 동작을 어떻게 제어할지 살펴보자

#### Note

어떤 이펙트는 정리 함수를 반환하지 않는다. 그렇지 않은 경우 보다 많게, 당신은 하나는 반환할 것 이다. - 당신이 하지 않는다면, 리액트는 빈 정리 함수를 반환할 것이다.

### Why synchronization may need to happen more than once

챗룸 컴포넌트가 유저가 드롭다운에서 고른 roomId 프랍을 받는다고 상상해보자. 유저가 처음에는 general을 roomId로 골랐다고 하자.
당신의 앱은 general 챗룸을 보여줄 것이다.

```jsx
const serverUrl = "https://localhost:1234";

function ChatRoom({ roomId /* "general" */ }) {
  // ...
  return <h1>Welcome to the {roomId} room!</h1>;
}
```

UI가 보여지고, 리액트는 이펙트를 실행해 싱크를 맞춘다. 이것은 general 룸과 연결한다.

```jsx
function ChatRoom({ roomId /* "general" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Connects to the "general" room
    connection.connect();
    return () => {
      connection.disconnect(); // Disconnects from the "general" room
    };
  }, [roomId]);
  // ...
```

후에, 유저가 드롭다운에서 다른 룸을 골랐을 시 e.g travel. 먼저, 리액트는 UI를 업데이트한다.

```jsx
function ChatRoom({ roomId /* "travel" */ }) {
  // ...
  return <h1>Welcome to the {roomId} room!</h1>;
}
```

다음에 어떤일이 일어날지 생각해봐라. 유저는 선택된 travel 챗 룸 UI를 볼 것이다.
하지만, 이펙트는 이전에 일어났기에, 여전히 general 룸에 연결되어있다. roomId 프랍이 변경되었어도, 이펙트가 이전에 했던(general 룸과 연결)것은 UI와 더이상 맞지 않는다.

이 시점에서, 당신은 리액트가 두 가지를 수행하길 원한다.

1. 예전 roomId와 싱크를 끊기
2. 새로운 roomId와 싱크 맞추기

다행히도, 리액트가 이것을 어떻게 처리하는지 배웠다. 당신의 이펙트 바디는 싱크를 맞추고 정리 함수는 싱크를 멈춘다. 리액트가 이것을 위해 필요한 것은 오직 올바른 호출 순서와 올바른 프랍과 상태다.

### How React re-synchronizes your Effect

챗룸 컴포넌트를 재호출해 roomId 프랍의 새로운 값을 받는다. general이었던 것은 이제 travel이다.
리액트는 이펙트의 싱크를 다시 맞추고 다른 챗룸을 연결해야한다.

싱크를 멈추기 위해, 리액트는 general 룸과 연결을 맞추고 반환된 정리 함수를 호출한다.
roomId가 general이기에, 정리 함수는 general의 연결을 끊는다.

```jsx
function ChatRoom({ roomId /* "general" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Connects to the "general" room
    connection.connect();
    return () => {
      connection.disconnect(); // Disconnects from the "general" room
    };
    // ...

```

그 후 리액트는 이번 랜더에서 제공된 이펙트를 실행한다. 이제, roomId는 travel로 travel 챗룸과 싱크를 맞춘다. (정리 함수가 호출되기 전까지)

```jsx
function ChatRoom({ roomId /* "travel" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Connects to the "travel" room
    connection.connect();
    // ...
```

이것에 감사해라, 당신은 이제 유저가 고른 UI와 똑같은 룸에 연결됐다. 재앙을 피했다.

당신의 컴포넌트는 다른 roomId와 매번 리랜더를 일으키고, 당신의 이펙트는 싱크를 다시 맞춘다.
예를 들어, 유저가 roomId를 travel -> music으로 바꿨다고 생각하자.
리액트는 또 다시 싱크를 멈추고 travel과 연결을 끊는다. 그리고 새로운 roomId(music)과 싱크를 맞춘다.

최종적으로, 유저가 다른 화면으로 이동해 챗룸이 언마운트 되면, 이제 연결될 필요가 전혀없다.
리액트는 최종적으로 싱크를 멈추고 music 챗룸과 연결을 끊는다.

### Thinking from the Effect’s perspective

챗룸 컴포넌트의 관점에서 생각해보자.

1. 챗룸이 general로 마운트
2. 챗룸이 travel로 업데이트
3. 챗룸이 music으로 업데이트
4. 챗룸 언마운트

각 컴포넌트의 생명주기 동안, 이펙트는 다른 동작을 취한다.

1. 이펙트는 general 룸과 연결
2. general과 연결을 끊고 travel과 연결
3. travel과 연결을 끊고 music과 연결
4. music 룸의 연결을 끊음

이제 이펙트 관점에서 무슨일이 일어나는지 보자.

```jsx
useEffect(() => {
  // Your Effect connected to the room specified with roomId...
  const connection = createConnection(serverUrl, roomId);
  connection.connect();
  return () => {
    // ...until it disconnected
    connection.disconnect();
  };
}, [roomId]);
```

이 코드 구조는 겹치지 안흔 시간 순서에 따라 어떻게 동작했는지 알려준다.

1. 이펙트가 general 룸을 연결(연결 해제될 때까지)
2. 이펙트가 travel 룸을 연결(연결 해제될 때까지)
3. 이펙트가 music 룸을 연결(연결 해제될 때까지)

이전에, 당신은 컴포넌트 관점에서 봤다. 컴포넌트 관점에서, 이펙트를 랜더 후 혹은 언마운트 전 등 특정 시간에 호출되는 콜백 혹은 생명주기 이벤트로 생각하기 쉽다. 이런 생각은 복잡해지기 쉽고, 피해야한다.

대신에, 항상 한 번에 하나의 시작/정지 사이클에 집중해라. 컴포넌트가 마운트, 업데이트 또는 언마운트 되는 것은 중요하지 않다.
당신이 명심해야하는 것은 어떻게 싱크를 맞추는 것을 시작할지와 어떻게 멈출지이다.
이것을 잘하면, 당신의 이펙트는 그것이 원하는 만큼 탄력적으로 시작하고 멈출것이다.

이것은 당신이 JSX를 생성하기 위한 랜더링 로직을 작성하며, 컴포넌트가 어떻게 마운트되고 업데이트되는지 고려를 안해도 되는 것을 떠올리게한다. 당신은 무엇을 화면에 보일지 결정하고, 리액트는 나머지를 수행한다.

### How React verifies that your Effect can re-synchronize

오픈 챗 버튼을 눌러 챗룸 컴포넌트를 마운트 시키는 예가 있다.

```jsx
import { useState, useEffect } from "react";
import { createConnection } from "./chat.js";

const serverUrl = "https://localhost:1234";

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
  return <h1>Welcome to the {roomId} room!</h1>;
}

export default function App() {
  const [roomId, setRoomId] = useState("general");
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        Choose the chat room:{" "}
        <select value={roomId} onChange={(e) => setRoomId(e.target.value)}>
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? "Close chat" : "Open chat"}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

컴포넌트가 처음 마운트 되면, 당신은 세 가지 로그를 보게될 것이다.

1. ✅ Connecting to "general" room at https://localhost:1234... (development-only)
2. ❌ Disconnected from "general" room at https://localhost:1234. (development-only)
3. ✅ Connecting to "general" room at https://localhost:1234...

앞의 두 로그는 오직 개발 환경에서만 작동한다. 개발 환경에서 리액트는 항상 컴포넌트를 한 번씩 리마운트한다.

리액트는 개발시 즉시 이펙트가 싱크를 다시 맞출수 있도록 확인한다. 이것은 당신이 문이 잠겼나 수시로 문을 확인하는 것을 떠올리게한다.
리액트는 이펙트를 시작하고 멈추며 당신이 정리 함수를 잘 작성했는지 확인한다.

예에서 이펙트가 싱크를 다시 맞추는 주 이유는 사용되는 데이터가 변했기 때문이다. 샌드박스에서 선택된 챗룸을 바꾸는 것.
roomId가 변하면 어떻게 이펙트가 싱크를 다시 맞추는지 확인해라.

하지만, 싱크를 다시 맞춰야하는 일반적이지 않은 상황도 있다. 예를 들어, 예에서 챗이 열려있는데 serverUrl을 변경했을 경우이다.
이펙트가 코드 변경의 응답으로 어떻게 싱크를 다시 맞추는지 확인해라. 후에, 리액트는 싱크를 다시 맞추는 것에 더 많은 기능을 추가할 것이다.

### How React knows that it needs to re-synchronize the Effect

당신은 리액트가 roomId가 변한 뒤 재싱크를 맞추는 것을 아는지 궁금할 것이다.
그것은 당신이 리액트에게 roomId에 달려있다고 의존성 배열에 추가했기 때문이다.

```jsx
function ChatRoom({ roomId }) { // The roomId prop may change over time
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // This Effect reads roomId
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]); // So you tell React that this Effect "depends on" roomId
  // ...
```

작동 방식

1. 당신은 roomId가 수시로 바뀌는 프랍이라는 것을 안다.
2. 당신은 이펙트가 roomId를 읽는 것을 안다.
3. 이것이 특정 의존성으로 명시한 이유이다.

매번 컴포넌트가 리랜더될 시, 리액트는 당신이 전달한 의존성 배열을 살펴본다.
만약 배열의 동일한 장소 의존성이 이전 랜더와 다르다면, 리액트는 이펙트의 싱크를 다시 맞춘다.

예를 들어, 초기 랜더시 ["general"]를 전달하고, 후에 당신이 ["travel"]를 다음 랜더에 전달했다면, 리액트는 general과
travel을 비교한다. 이것들은 다른 값이기에(Object.is 비교), 리액트는 이펙의 싱크를 다시 맞춘다.
만약, roomId가 리랜더 후에도 변경되지 않았다면, 이펙트는 동일한 챗룸을 연결된 상태로 둔다.

### Each Effect represents a separate synchronization process

이전에 작성된 로직과 관련이 없는 로직을 추가하지마라. 챗룸에 접속했을 시 분석 이벤트를 보내는 예가 있다.
당신은 이미 roomId에 의존하는 이펙트를 작성했고, 당신은 분석의 호출을 추가하고 싶을 것이다.

```jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    logVisit(roomId);
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

하지만 이후에 연결을 재시도하는 다른 의존성을 추가한다고 생각해보자. 만약, 이펙트가 싱크를 다시 맞추게 되면, 당신이 의도하지 않은 logVisit(roomId) 역시 호출될 것이다. 방문 로그를 찍는 것은 연결과 분리된 작업이다.
두 개의 다른 이펙트에 작성한다.

```jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    logVisit(roomId);
  }, [roomId]);

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    // ...
  }, [roomId]);
  // ...
}
```

각 이펙트안에는 분리되고 독립적인 싱크 작업 코드를 작성해야한다.

위의 예에서, 하나의 이펙트를 제거한다고 다른 이펙트 로직을 망가트리지 않는다. 이것은 그들이 다른 것에 싱크를 맞춘다는 좋은 지침이며, 분리하는 것이 말이 된다.
이렇게 코드를 분리하는 것은 가독성이 좋지만, 유지 보수가 어려울수 있다.
이것이 당신이 가독성이 아닌, 코드의 작업이 같을지 다를지를 생각해야하는 이유다.

## Effects “react” to reactive values

당신의 이펙트는 serverUrl와 roomId 두 가지 변수를 읽지만, roomId만을 의존성으로 명시했다.

```jsx
const serverUrl = "https://localhost:1234";

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

왜 serverUrl은 의존성이 아닐까?

serverUrl은 리랜더 중에 절대 변하지 않기 때문이다. 그리고 이것은 컴포넌트가 리랜더를 아무리 많이해도 똑같다.
serverUrl이 절대 변하지 않기에 의존성에 추가하지 않는 것이 말이 된다.
의존성은 그들이 변했을 시에만 동작한다.

roomId는 리랜더시 달라질 수 있다. 프랍, 상태 그리고 컴포넌트 안에 선언된 값들은 랜더 중 계산되고, 리액트 데이터 플로우에 속해있다.

만약 serverUrl이 상태 변수라면, 이것은 반응성이 있다. 반응성 값들은 반드시 의존성에 포함되어야한다.

```jsx
function ChatRoom({ roomId }) {
  // Props change over time
  const [serverUrl, setServerUrl] = useState("https://localhost:1234"); // State may change over time

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Your Effect reads props and state
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]); // So you tell React that this Effect "depends on" on props and state
  // ...
}
```

serverUrl를 의존성에 추가해, 당신은 그것이 변할 시 이펙트가 싱크를 다시 맞추는 것을 보장한다.

roomId나 serverUrl과 같은 반응성 값들이 변화하면, 이펙트는 챗 서버에 다시 연결한다.

### What an Effect with empty dependencies means

만약 roomId와 serverUrl이 컴포넌트 외부에 있다면 어떨까?

```jsx
const serverUrl = "https://localhost:1234";
const roomId = "general";

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ All dependencies declared
  // ...
}
```

이제 당신의 이펙트 코드는 반응성 값이 없으므로, 의존성은 비게된다([]).

컴포넌트 관점에서, 빈 의존성 배열은 이펙트는 마운트될 시 챗룸을 연결하고 언마운트될 시 연결을 끊는다.

```jsx
import { useState, useEffect } from "react";
import { createConnection } from "./chat.js";

const serverUrl = "https://localhost:1234";
const roomId = "general";

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []);
  return <h1>Welcome to the {roomId} room!</h1>;
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? "Close chat" : "Open chat"}
      </button>
      {show && <hr />}
      {show && <ChatRoom />}
    </>
  );
}
```

하지만, 이펙트의 관점에서 당시는 마운트와 언마운트를 생각할 필요 없다.
중요한 것은 당신이 이펙트가 무엇에 싱크를 맞추는 것을 시작하고 중지하는지 명시하는 것이다.
현재, 이것은 반응성있는 의존성이 없다. 하지만, 유저가 roomId나 serverUrl을 변경하게 하고 싶다면, 당신의 이펙트는 변하지 않을 것이다. 당신은 그들을 의존성에 추가해야한다.

### All variables declared in the component body are reactive

프랍과 상태만이 반응 값인것은 아니다. 그들로 계산한 값들도 역시 반응성이있다.
만약 프랍과 상태가 변화하면, 당신의 컴포넌트가 리랜더되고, 그들로 계산되는 값들 역시 변경된다.
이것이 컴포넌트 몸체의 모든 값들이 이펙트의 의존성에 추가되어야하는 이유다.

드롭다운으로 챗 서버를 선택할 수 있고, 기본 서버으로 설정된 서버를 사용할수 있는 예가 있다.
당신이 이미 컨텍스트에 기본 설정을 했다고 생각하자.
이제 당신은 serverUrl을 선택된 서버 프랍과 기본 서버에서 계산할 수 있다.

```jsx
function ChatRoom({ roomId, selectedServerUrl }) {
  // roomId is reactive
  const settings = useContext(SettingsContext); // settings is reactive
  const serverUrl = selectedServerUrl ?? settings.defaultServerUrl; // serverUrl is reactive
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Your Effect reads roomId and serverUrl
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]); // So it needs to re-synchronize when either of them changes!
  // ...
}
```

이 예에서 serverUrl은 프랍이나 상태 변수가 아니다. 이것은 랜더중에 생성되는 일반적인 변수다.
그러나 이것은 랜더중에 계산되므로, 이것은 리랜더중 변경될 수 있다. 이것이 반응성이 있는 이유다.

컴포넌트 안의 모든 값(프랍, 상태, 컴포넌트 몸체의 변수들)은 반응성이있다.
이런 반응성 값은 리랜더 시 변경될 수 있으므로, 이펙트의 의존성에 추가해야한다.

다른 말로, 이펙트는 컴포넌트의 모든 값으로 부터 반응한다.

#### DEEP DIVE - Can global or mutable values be dependencies?

뮤터블 값(전역 변수 포함)은 반응성이 아니다.

location.pathname과 같은 뮤터블 값은 의존성이 될 수 없다. 이것은 뮤터블하므로, 리액트의 랜더링 데이터 플로우 밖에서도 변경될 수 있다. 이것의 변경이 컴포넌트 리랜더링을 일으키지 않는다. 그러므로, 의존성에 명시했어도 리액트는 이펙트를 언제 싱크를 다시 맞춰야하는지 알 수 없다. 이것은 또한 리액트의 룰을 어기는 것으로, 랜더링중 뮤터블한 데이터를 읽는 것은 랜더의 순수성을 해친다.
대신에, useSyncExternalStore를 통해 외부의 뮤터블한 값을 구독할 수 있다.

ref.current와 같은 뮤터블 값 역시 의존성이 될 수 없다. useRef로 반환되는 ref 오브젝트는 의존성이 될 수 있지만, current 프로퍼티는 뮤터블하다. 이것은 리랜더를 일으키지 않고 값을 추적할 수 있게 한다. 이것의 변경이 리랜더를 일으키지 않기에 이것은 반응 값이 아니며, 리액트는 언제 이펙트를 실행해야하는지 모른다.

린터가 이것을 자동으로 체크해준다.

### React verifies that you specified every reactive value as a dependency

만약 린터가 설정이 되어있다면, 이것은 의존성에 선언되어 이펙트 코드에 사용되는 반응 값을 체크한다.
예를 들어, roomId와 serverUrl는 반응 값이기에 린터가 에러를 발생한다.

```jsx
import { useState, useEffect } from "react";
import { createConnection } from "./chat.js";

function ChatRoom({ roomId }) {
  // roomId is reactive
  const [serverUrl, setServerUrl] = useState("https://localhost:1234"); // serverUrl is reactive

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // <-- Something's wrong here!

  return (
    <>
      <label>
        Server URL:{" "}
        <input
          value={serverUrl}
          onChange={(e) => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState("general");
  return (
    <>
      <label>
        Choose the chat room:{" "}
        <select value={roomId} onChange={(e) => setRoomId(e.target.value)}>
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

이것은 리액트 에러 같지만, 이것은 사실 리액트가 버그가 있는 코드를 알려주는 것이다. roomId와 serverUrl은 매번 변경 되는데, 당신은 그들이 변경 됐을 때 싱크를 다시 맞추는 것을 깜빡했다. 유저가 UI에서 다른 값을 선택해도, 연결 초기의 roomId와 serverUrl로 남아있을 것이다.

이 버그를 고치기 위해, 린터의 제안을 따라 roomId와 serverUrl을 이펙트의 의존성에 추가한다.

```jsx
function ChatRoom({ roomId }) {
  // roomId is reactive
  const [serverUrl, setServerUrl] = useState("https://localhost:1234"); // serverUrl is reactive
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]); // ✅ All dependencies declared
  // ...
}
```

린터의 에러는 사라지고, 챗은 필요에 따라 재연결된다.

#### Note

가끔, 리액트는 컴포넌트 내부에 선언된 값이 절대 변하지 않는 것을 안다.
예를 들어, useState의 반환 값인 set 함수와 useRef에서 반환된 ref 오브젝트는 안전하다 - 그들은 리랜더시에도 변경되지 않는 것이 보장된다. 이런 안정된 값은 반응이 아니므로, 의존성에서 생략할 수 있다. 포함하는 것도 허용된다. - 그들은 변하지 않으므로, 문제가 되지 않는다.

### What to do when you don’t want to re-synchronize

이전 예에서 roomId와 roomId를 의존성에 추가해 린트 에러를 해결했다.

하지만, 린터에 반응성 값이 아니라는 것을 증명하기 위해(변경이 리랜더를 일으키지 않는 것), 컴포넌트 외부로 옮길 수 있다.

```jsx
const serverUrl = "https://localhost:1234"; // serverUrl is not reactive
const roomId = "general"; // roomId is not reactive

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ All dependencies declared
  // ...
}
```

또 당신은, 이펙트 안으로 옮길 수 있다. 그들은 랜더중 계산 되지 않으므로, 그들은 반응성이 아니다.

```jsx
function ChatRoom() {
  useEffect(() => {
    const serverUrl = "https://localhost:1234"; // serverUrl is not reactive
    const roomId = "general"; // roomId is not reactive
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ All dependencies declared
  // ...
}
```

이펙트는 반응하는 코드의 블락이다. 그들은 내부의 값이 바뀌면 싱크를 다시 맞춘다. 상호 작용에 실행되는 이벤트 핸들러와 다르게, 이펙트는 싱크가 필요한 경우 실행된다.

당신은 의존성을 선택할 수 없다. 당신은 반드시 이펙트에서 읽는 모든 반응 값을 의존성에 포함시켜야한다.
린터가 이것을 강제한다. 가끔 이것은 무한 루프와 너무 자주 싱크를 맞추는 문제를 일으킨다.
린터를 억제해 문제를 해결하려 하지 마라.

대신 시도할 방법

- 당신의 이펙트가 독립적인 싱크 작업을 하는지 확인. 만약 당신의 이펙트가 어떤것도 싱크를 맞추지 않는 다면 불필요하다. 만약 여러 싱크를 맞춘다면 분리해라.
- 만약 최신 상태와 프랍을 반응하는 것을 이펙트의 싱크를 다시 맞추는 것으로 읽기 싫다면, 당신은 이펙트를 반응하는 부분과 반응하지 않는 부분으로 나눌 수 있다.
- 객체와 함수를 의존성에 넣는 것을 피해라. 만약 당신이 랜더중 객체와 함수를 생성하고 이펙트가 읽도록 하면, 그들은 매번 다를 것이다. 이것은 이펙트가 매번 싱크를 맞추도록 한다.

#### Pitfall

린터는 당신의 친구지만, 한정적이다. 린터는 의존성이 틀렸을 경우만 안다.
이것은 각 상황을 해결하는 최고의 방법을 알지 못한다. 만약 린터가 의존성에 추가하라는 제안을 하고, 추가가 루프를 유발한다고 해서 린터가 무시되어야 하는 것은 아니다. 당신의 이펙트 내부와 외부 코드의 반응하지 않는 값과 불필요 의존성을 변경해야 한다.

기존의 코드가 존재하면 이펙트를 다음과 같이 린트를 제한할 수 있다.

```jsx
useEffect(() => {
  // ...
  // 🔴 Avoid suppressing the linter like this:
  // eslint-ignore-next-line react-hooks/exhaustive-deps
}, []);
```

다음 페이지에서 룰을 깨지않고 해결하는 법을 다룬다.

## Recap

- 컴포넌트는 마운트, 업데이트, 언마운트 한다.
- 각 이펙트는 주위 컴포넌트에 따라 다른 라이프사이클을 갖는다.
- 각 이펙트는 싱크를 시작/중지하는 각 작업을 설명한다.
- 이펙트를 작성할 때, 이펙트의 관점에서 작성해라.
- 컴포넌트 몸체에 선언된 값은 반응성이 있다.
- 반응 값은 변경되므로 이펙트가 싱크를 다시 맞추도록 한다.
- 린터는 이펙트 안에 사용되는 모든 반응 값을 의존성에 명시하도록 검사한다.
- 모든 린터 에러는 정당하다. 룰을 깨지 않고 고치는 법이 있다.

---

## What I Learned

이펙트를 생각할 때 항상 컴포넌트 관점에서 생각을 했는데, 문서를 읽으며 이펙트의 관점에서 생각해야 하는 것을 알게되었다.
즉 마운트, 업데이트, 언마운트가 아닌 언제 싱크를 맞추고 중지할지의 관점에서 봐야한다.

또 가끔 상태를 set해주는 함수를 의존성 배열에 추가했었는데, 리액트가 이미 변경되지 않는 것을 알고 있는 값이기에 추가하지 않아도 되는 것을 알게되었다. -> 예전에 어디서 추가안해도 된다는 글을 읽었는데, 좀 더 명확해졌다.
