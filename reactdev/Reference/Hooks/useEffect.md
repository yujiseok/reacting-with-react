# useEffect

`useEffect`는 외부 시스템에 컴포넌트의 싱크를 맞추도록 해주는 리액트 훅이다.

```jsx
useEffect(setup, dependencies?)
```

## Reference

### `useEffect(setup, dependencies?)`

컴포넌트 최상단에 `useEffect`를 호출해 컴포넌트에 이펙트를 선언한다.

```jsx
import { useEffect } from "react";
import { createConnection } from "./chat.js";

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState("https://localhost:1234");

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);
  // ...
}
```

**Parameters**

- `setup`: 이펙트의 로직이 있는 함수. 셋업 함수는 옵서녈하게 클린업 함수를 리턴한다. 컴포넌트가 돔에 추가될 때, 리액트는 셋업 함수를 실행한다. 의존성이 바뀌는 매 리랜더시, 리액트는 먼저 클린업 함수를 이전 값과 실행한 후, 새로운 값과 셋업 함수를 실행한다. 컴포넌트가 돔에서 제거되었을 때, 리액트는 클린업 함수를 실행한다.
- optional `dependencies`: 셋업 코드에서 참조되는 반응 값. 프랍, 상태 그리고 컴포넌트 몸체에 선언된 모든 변수와 함수들은 반응 값이다. 만약 린터가 리액트를 위해 설정 되었다면, 모든 반응 값을 알맞게 구분할 것이다. 의존성 배열의 항목 수가 일정해야 하며 `[dep1, dep2, dep3]`와 같이 인라인으로 작성되어야한다. 리액트는 각 의존성을 이전 값과 `Object.is` 비교한다. 인자를 빠트릴 시, 이펙트는 매 리랜더 후 재실행된다.

**Returns**
`useEffect`는 `undefined`를 반환한다.

**Caveats**

- `useEffect`는 훅이므로, 오직 컴포넌트의 최상단에서 호출되어야한다. 반복문 또는 조건문에서 호출할 수 없다. 만약 필요하다면 컴포넌트로 추출한 후, 상태를 옮겨라.
- 외부 시스템과 싱크를 맞출 필요 없다면, 이펙트가 필요하지 않을 것이다.
- 엄격 모드시, 리액트는 셋업과 클린업 함수를 진짜 셋업이 호출되기 전에 호출한다. 이것은 클린업 로직이 셋업 로직을 미러링하고 셋업이 하는 일을 멈추는 것을 보장하는 스트레스 테스트다. 만약 문제가 발생하면, 클린업 함수를 발전 시켜라.
- 만약 의존성이 컴포넌트 내부에 정의된함수나 함수일 경우, 그들은 의도하지 않은 이펙트의 재실행을 유발한다. 이것을 고치기 위해, 불필요한 객체나

함수 의존성을 제거한다. 또한 상태 업데이트와 반응하지 않는 로직을 분리할 수 있다.

- 만약 상호작용에 의해 이펙트가 발생된 것이 아니라면, 리액트는 이펙트가 실행되기 전 브라우저의 화면을 페인트한다. 만약 이펙트가 비주얼을 처리하고, 지연이 눈에 띄면 `useEffect`를 `useLayoutEffect`로 대체해라.
- 만약 당신의 이펙트가 상호작용에 의해 실행 되어, 이펙트 안에서 상태 변경이 처리되기 전에 화면이 리페인트 되는 경우. 일반적으로 당신이 원하는 경우. 하지만, 화면을 리페인트 하는 것을 막기를 원한다면, `useEffect`를 `useLayoutEffect`로 대체해라.
- 이펙트는 오직 클라이언트에서 실행된다. 그들은 서버 랜더링에서 실행되지 않는다.

## Usage

### Connecting to an external system

어떤 컴포넌트들은 네트워크, 브라우저 API, 또는 써드파티 라이브러리와 페이지에 보여졌을 때 연결되어야 한다. 이런 시스템은 리액트의 제어를 벗어나 외부라 불린다.

외부 시스템과 컴포넌트를 연결하기 위해 `useEffect`를 컴포넌트 최상단에 호출한다.

```jsx
import { useEffect } from "react";
import { createConnection } from "./chat.js";

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState("https://localhost:1234");

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);
  // ...
}
```

당신은 `useEffect`에 두 인자를 넘겨야한다.

1. 셋업 코드가 있는 시스템을 연결하기 위한 셋업 함수
   - 시스템과 연결을 끊기 위한 클린업 코드가 있는 클린업 함수를 반환 해야한다.
2. 컴포넌트에서 사용되는 모든 값을 포함해 함수 내부에서 사용되는 의존성 배열

리액트는 셋업과 클린업 함수를 필요에 따라 호출한다.

1. 당신의 셋업 코드는 컴포넌트가 페이지에 추가되었을 시 실행된다.(마운트)
2. 컴포넌트의 매 리랜더 시 의존성이 변경되었을 경우
   - 우선, 클린업 코드가 오래된 프랍과 상태와 실행된다.
   - 후에, 셋업 코드가 새로운 프랍과 상태와 실행된다.
3. 당신의 클린업 코드는 최후에 컴포넌트가 페이지에서 사라질 시 실행된다.(언마운트)

위의 시퀀스를 예로 설명해보자.

챗룸 컴포넌트가 페이지에 추가되었을 시, 초기 `serverUrl`과 `roomId`와 챗 룸을 연결한다.
`serverUrl`과 `roomId`가 리랜더의 결과로 변경되면, 이펙트는 이전 룸의 연결을 끊고, 다음 룸과 연결한다.
챗룸이 페이지에서 제거되면, 당신의 이펙트는 마지막에 연결을 끊는다.

버그를 찾기 위해, 개발 환경의 리액트는 셋업과 클린업을 셋업 전에 호출한다. 이것은 이펙트가 제대로 구현되었는지 테스트이다.
만약 이슈가 있다면, 클린업 함수에 로직이 빠진 것이다. 클린업 함수는 셋업 함수가 하는 동작이 무엇이든 멈춰고 그만두게해야 한다.
일반적으로 사용자는 한 번 호출되는 셋업과 셋업 -> 클린업 -> 셋업 단계를 구분할 수 없어야 한다.

모든 이펙트를 독립적인 절차로 작성하고 하나의 셋업/클린업 사이클을 생각해라. 이것은 컴포넌트가 마운트, 업데이트 그리고 언마운트 될 때 문제가 되지 않는다. 클린업 로직이 셋업 로직을 정확히 미러링하면, 당신의 이펙트는 원하는 만큼 탄력적으로 셋업과 클린업을 실행한다.

#### note

이펙트는 당신의 컴포넌트가 외부 시스템과 싱크를 맞출 수 있도록 해준다.
외부 시스템은 리액트에 의해 제어되지 않는 것을 뜻 한다. 다음과 같은:

- setInterval과 clearInterval 같은 타이머
- window.addEventListener()와 window.removeEventListener() 같은 이벤트 구독
- animation.start()와 animation.reset() 같은 써드 파티 애니메이션 라이브러리

### Wrapping Effects in custom Hooks

이펙트는 탈출구로 리액트 밖이나 빌트인 해결책이 없을 경우 사용해야한다.
만약 이펙트가 필요하다면, 커스텀 훅으로 추출해야한다.

예를 들어 `useChatRoom` 커스텀 훅은 이펙트의 로직을 숨기고 더 선언적인 API가 된다.

```jsx
function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId,
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

```jsx
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });
  // ...
```

### Controlling a non-React widget

가끔, 당신은 외부 시스템에 프랍이나 상태의 싱크를 맞추고 싶어한다.

예를 들어, 당신이 서드파티 맵 위젯이나 리액트로 쓰여지지 않은 비디오 플레이어 컴포넌트가 있을 경우, 이펙트를 사용해 메서드를 호출할 수 있으며 현재 상태를 그것의 상태와 매치할 수 있다.
이 이펙트는 `MapWidget` 인스턴스를 `map-widget.js`에서 생성한다. 만약 맵 컴포넌트의 `zoomLevel` 프랍을 변경한다면, 이펙트는 `setZoom()` 메서드를 싱크를 맞추기 위해 호출할 것이다.

이 예에서, 클린업 함수는 필요 없는데 MapWidget 클래스가 오직 돔 노드를 관리하기 때문이다. 맵 컴포넌트가 트리에서 사라지면, 돔 노드와 MapWidget 클래스 인스턴스 역시 자바스크립트 엔진에 의해 가비지 콜렉트가 된다.

### Fetching data with Effects

이펙트를 사용해, 컴포넌트의 데이터를 페치할 수 있다. 만약 프레임워크를 사용한다면, 프레임워크의 데이터 페칭 메카니즘을 사용하는 것이 이펙트를 작성하는 것 보다 더 효율적이다.

```jsx
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);

  useEffect(() => {
    let ignore = false;
    setBio(null);
    fetchBio(person).then(result => {
      if (!ignore) {
        setBio(result);
      }
    });
    return () => {
      ignore = true;
    };
  }, [person]);

  // ...
```

`ignore` 변수는 false로 초기화 되고, 클린업 함수에서 true로 변경된다. 이것은 레이스 컨디션에서 해방을 보장한다.
네트워크 응답은 다른 순서로 올 수 있기에.

`async / await` 문법을 사용할 수 있다.

```jsx
import { useState, useEffect } from "react";
import { fetchBio } from "./api.js";

export default function Page() {
  const [person, setPerson] = useState("Alice");
  const [bio, setBio] = useState(null);
  useEffect(() => {
    async function startFetching() {
      setBio(null);
      const result = await fetchBio(person);
      if (!ignore) {
        setBio(result);
      }
    }

    let ignore = false;
    startFetching();
    return () => {
      ignore = true;
    };
  }, [person]);

  return (
    <>
      <select
        value={person}
        onChange={(e) => {
          setPerson(e.target.value);
        }}
      >
        <option value="Alice">Alice</option>
        <option value="Bob">Bob</option>
        <option value="Taylor">Taylor</option>
      </select>
      <hr />
      <p>
        <i>{bio ?? "Loading..."}</i>
      </p>
    </>
  );
}
```

이펙트로 데이터 페칭을 작성하는 것은 반복적이고 캐싱이나 서버 랜더등 최적화하기 어렵다. 커스텀 훅으로 뽑아내면 좀 더 쉽다.

#### DEEP DIVE - What are good alternatives to data fetching in Effects?

이펙트에 페치를 호출하는 것은 완전한 클라이언트 앱에서 데이터를 페치하는 유명한 방법이다.
하지만, 이것은 매우 수동적이며 단점이 존재한다.

- 이펙트는 서버에서 실행되지 않는다. 초기 서버에서 랜더된 html은 오직 로딩 상태만을 갖고 데이터는 없다. 클라이언트 컴퓨터는 자바스크립트를 다운 받고 데이터가 로드 되야하는 것을 필요로 할때 랜더된다. 이것은 효율적이 못하다.
- 이펙트 안에서 페칭은 네트워크 워터폴을 만든다. 데이터를 페치하는 부모 컴포넌트를 랜더하고, 자식 컴포넌트를 랜더한 후 그들은 데이터를 페칭한다. 만약 네트워크가 빠르지 않다면, 모든 데이터를 병렬로 페치하는 것보다 느릴것이다.
- 이펙트 안에서 데이터 페칭은 프리로드와 캐시를 하지않은 것을 뜻한다. 예를 들어, 만약 컴포넌트가 언마운트되고 마운트 되는 것을 반복할 시, 데이터 페치를 다시 할것이다.
- 공학적이지 않다. 레이스 컨디션으로 벗어나기 위해 작성해야하는 코드가 많다.

### Specifying reactive dependencies

이펙트의 의존성을 고를 수 없다는 것을 명심해라. 이펙트 코드 내부에서 사용되는 반응 값은 반드시 의존성에 추가되어야 한다.
이펙트의 의존성은 둘러 싸인 코드에 결정된다.

```jsx
function ChatRoom({ roomId }) {
  // This is a reactive value
  const [serverUrl, setServerUrl] = useState("https://localhost:1234"); // This is a reactive value too

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // This Effect reads these reactive values
    connection.connect();
    return () => connection.disconnect();
  }, [serverUrl, roomId]); // ✅ So you must specify them as dependencies of your Effect
  // ...
}
```

만약 serverUrl과 roomId가 변경되면, 이펙트는 챗을 새로운 조건으로 재연결할 것이다.

반응 값은 컴포넌트 내부에서 직접 선언된 프랍과 모든 변수 그리고 함수를 포함한다.
serverUrl과 roomId가 반응 값이기에, 의존성에서 제거할 수 없다.
만약 그들을 제거하려 하면 린터가 에러를 발생하고, 당신이 고쳐야할 문제이다.

```jsx
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState("https://localhost:1234");

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // 🔴 React Hook useEffect has missing dependencies: 'roomId' and 'serverUrl'
  // ...
}
```

의존성을 제거하기 위해, 의존성이 될 필요가 없다는 것을 증명해야한다. 예를 들어, serverUrl을 컴포넌트 외부에 생성해 리랜더를 일으키지 않는다고 증명할 수 있다.

```jsx
const serverUrl = "https://localhost:1234"; // Not a reactive value anymore

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
}
```

이제 serverUrl은 반응 값이 아니기에, 의존성일 필요가 없다.
만약 이펙트의 코드가 반응 값을 사용하지 않는다면, 의존성 배열은 비었을 것이다.

```jsx
const serverUrl = "https://localhost:1234"; // Not a reactive value anymore
const roomId = "music"; // Not a reactive value anymore

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ All dependencies declared
  // ...
}
```

빈 의존성은 프랍이나 상태가 변경되도 재실행되지 않는다.

### Updating state based on previous state from an Effect

이펙트를 통해 이전 상태에 따라 상태를 변경하는 것은, 문제를 일으킨다.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      setCount(count + 1); // You want to increment the counter every second...
    }, 1000);
    return () => clearInterval(intervalId);
  }, [count]); // 🚩 ... but specifying `count` as a dependency always resets the interval.
  // ...
}
```

카운트가 반응 값이기에, 의존성에 명시되어야한다. 하지만, 이것은 이펙트가 클린업과 셋업을 카운트가 변경될때마다 실행하도록 한다. 이것은 이상적이지 않다.

이것을 고치기 위해 `c => c + 1` 업데이터를 `setCount`에 전달할 수 있다.

```jsx
import { useState, useEffect } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      setCount((c) => c + 1); // ✅ Pass a state updater
    }, 1000);
    return () => clearInterval(intervalId);
  }, []); // ✅ Now count is not a dependency

  return <h1>{count}</h1>;
}
```

`count + 1` 대신 `c => c + 1`을 전달했기에, 이펙트는 더 이상 카운트에 의존하지 않는다.
카운트가 변경되어도 이펙트가 재실행되지 않는다.

### Removing unnecessary object dependencies

만약 이펙트가 랜더 도중 객체나 함수에 의존한다면, 이것은 더 자주 실행될 것이다. 예를 들어, 이 이펙트는 `options` 객체가 매 랜더시 다르기에 매 리랜더시 재연결을 할 것이다.

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const options = { // 🚩 This object is created from scratch on every re-render
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options); // It's used inside the Effect
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // 🚩 As a result, these dependencies are always different on a re-render
  // ...
```

의존성으로 랜더 중에 객체를 생성하는 것을 피하고, 이펙트 내에서 생성해라.

```jsx
import { useState, useEffect } from "react";
import { createConnection } from "./chat.js";

const serverUrl = "https://localhost:1234";

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState("");

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId,
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return (
    <>
      <h1>Welcome to the {roomId} room!</h1>
      <input value={message} onChange={(e) => setMessage(e.target.value)} />
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

이제 옵션 객체가 이펙트 내부에서 생성되어, 이펙트는 오직 `roomId` 문자열에 의존한다.

이 해결로 인풋에 타이핑을 해도 챗에 재연결 되지 않는다. 문자열 `roomId`는 재 생성되는 객체와 달리 값이 변화하지 않으면 변경되지 않는다.

### Removing unnecessary function dependencies

만약 이펙트가 랜더 도중 객체나 함수에 의존한다면, 이것은 더 자주 실행될 것이다. 예를 들어, 이 이펙트는 `createOptions` 함수가 매 랜더시 다르기에 매 리랜더시 재연결을 할 것이다.

```jsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  function createOptions() { // 🚩 This function is created from scratch on every re-render
    return {
      serverUrl: serverUrl,
      roomId: roomId
    };
  }

  useEffect(() => {
    const options = createOptions(); // It's used inside the Effect
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]); // 🚩 As a result, these dependencies are always different on a re-render
  // ...
```

매 리랜더시 함수를 생성하는 것은 문제가 되지 않느다. 또 최적화할 필요가 없다. 하지만, 이펙트의 의존성으로 사용했을 경우, 이것은 리랜더시 이펙트의 재실행을 유발한다.

의존성으로 랜더 중에 함수를 생성하는 것을 피하고, 이펙트 내에서 생성해라.

```jsx
import { useState, useEffect } from "react";
import { createConnection } from "./chat.js";

const serverUrl = "https://localhost:1234";

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState("");

  useEffect(() => {
    function createOptions() {
      return {
        serverUrl: serverUrl,
        roomId: roomId,
      };
    }

    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return (
    <>
      <h1>Welcome to the {roomId} room!</h1>
      <input value={message} onChange={(e) => setMessage(e.target.value)} />
    </>
  );
}
```

`createOptions` 함수를 이펙트 내부에 선언하여, 이펙트는 오직 `roomId`에만 의존한다.

### Reading the latest props and state from an Effect

> Under Construction
> 실험적인 API로 스테이블 버전에서는 사용할 수 없다.

기본적으로, 이펙트 내부에서 반응 값을 읽기 위해선, 의존성에 추가해야한다. 이것은 이펙트가 모든 변화에 반응 하는 것을 보장한다.
대부분의 의존성의 경우 원하는 동작을 할 것이다.

하지만, 가끔 이펙트에 반응하지 않고 최신 프랍과 상태를 읽고 싶은 경우가 있을 것이다.
예를 들어, 모든 페이지에서 장바구니의 수를 로그로 찍고 싶다고 생각해보자.

```jsx
function Page({ url, shoppingCart }) {
  useEffect(() => {
    logVisit(url, shoppingCart.length);
  }, [url, shoppingCart]); // ✅ All dependencies declared
  // ...
}
```

만약 url이 변경 되었을 경우만 로그를 찍고 싶을 경우는? `shoppingCart`를 반응 값 규칙을 어기지 않고 의존성에서 제거할 수 없다.
하지만, 이펙트 내부에서 호출 하더라도 반응하지 않고 싶은 경우가 있을 수 있다.
이펙트 이벤트를 `useEffectEvent` 훅을 사용해 선언하고, `shoppingCart`를 그 내부에서 읽어라.

```jsx
function Page({ url, shoppingCart }) {
  const onVisit = useEffectEvent((visitedUrl) => {
    logVisit(visitedUrl, shoppingCart.length);
  });

  useEffect(() => {
    onVisit(url);
  }, [url]); // ✅ All dependencies declared
  // ...
}
```

이펙트 이벤트는 반응성이 없으므로 반드시 의존성에서 제거되어야한다. 이것은 반응성 없는 코드를 둘 수 있게 해준다.
`onVisit`에서 `shoppingCart`를 읽는 것은 이펙트의 재실행을 유발하지 않는다.

### Displaying different content on the server and the client

만약 당신이 앱이 서버 랜더링을 사용한다면, 당신의 컴포넌트는 다른 두가지 환경에서 랜더한다.
서버에서, 초기 html을 랜더한다. 클라이언트에서, 리액트는 코드를 재 실행해 이벤트 핸들러를 부착한다.
이것은 하이드레이션으로, 당신의 서버와 클라이언트의 초기 랜더 결과가 같아야한다.

드문 경우, 클라이언트에 다른 컨텐트를 보여줄 경우가 있다. 예를 들어 서버에서 불가능한 로컬스토리지에서 데이터를 읽는 것이다.

```jsx
function MyComponent() {
  const [didMount, setDidMount] = useState(false);

  useEffect(() => {
    setDidMount(true);
  }, []);

  if (didMount) {
    // ... return client-only JSX ...
  } else {
    // ... return initial JSX ...
  }
}
```

앱이 로딩 될 때, 유저는 초기 랜더 결과를 볼것이다. 그 후, 로드와 하이드레이트가 되면 이펙트는 didMount를 true로 변경하고 리랜더를 일으킨다.
이것은 클라이언트 랜더 결과로 스위치한다. 이펙트는 서버에서 호출되지 않는다, 그러므로 didMount가 초기 서버 랜더시 false인 이유다.

이런 패턴은 아껴서 사용해라. 느린 연결 시간을 갖는 유저는 초기 컨텐트를 꽤 오랜 시간 동안 보게 되므로, 컴포넌트의 모양을 크게 변경하지 않는 게 좋다. 많은 경우, 다른 css를 보여줘서 피할 수 있다.

## Troubleshooting

### My Effect runs twice when the component mounts

개발 환경과 엄격 모드시, 리액트는 실제 셋업 전에 셋업과 클린업을 추가로 실행한다.

### My Effect runs after every re-render

우선, 의존성 배열을 깜빡했는지 확인해라:

```jsx
useEffect(() => {
  // ...
}); // 🚩 No dependency array: re-runs after every render!
```

만약, 의존성에 명시했지만 여전히 반복되면, 의존성이 매 랜더에 다르기 때문이다.

수동으로 로그를 디버그할 수 있다

```jsx
useEffect(() => {
  // ..
}, [serverUrl, roomId]);

console.log([serverUrl, roomId]);
```

다른 랜더의 배열 결과를 전역 변수로 담아라. 첫 번째를 temp1로 두 번째를 temp2로 저장하고, 두 배열이 같은지 체크할 수 있다:

```jsx
Object.is(temp1[0], temp2[0]); // Is the first dependency the same between the arrays?
Object.is(temp1[1], temp2[1]); // Is the second dependency the same between the arrays?
Object.is(temp1[2], temp2[2]); // ... and so on for every dependency ...
```

매 리랜더시 의존성이 다르다면 다음으로 고칠 수 있다.

- 이펙트에서 이전 상태로 상태 변경하기
- 객체 의존성 제거하기
- 함수 의존성 제거하기
- 최신 프랍과 상태 이펙트에서 읽기

최후의 수단으로, useMemo와 useCallback을 사용해 감싸라.

### My Effect keeps re-running in an infinite cycle

만약 이펙트가 무한으로 실행되면, 두 가지는 반드시 참이다:

- 이펙트가 상태를 변경
- 상태가 리랜더를 일으키고, 이펙트의 의존성을 변경

이 문제를 해결하기 전에, 이펙트가 외부 시스템과 연결이 되어있는지 물어봐라. 왜 이펙트가 상태를 설정해야하는가?
외부 시스템과 싱크를 맞추는가? 혹은 애플리케이션의 데이터 흐름을 관리하기 위해서인가?

만약 외부 시스템이 없다면, 이펙트를 전부 지워서 로직을 단순화해라.

외부 시스템과 싱크를 맞춘다면, 어떤 조건이 이펙트가 상태를 변경하게 하는지 생각하라. 어떤 변경이 컴포넌트의 모습에 영향을 끼치는가?
만약 랜더에 사용되지 않는 데이터를 추적한다면, ref를 사용하는 것이 적합하다. 이펙트가 의도치 않게 상태를 업데이트하는 것을 검증해라.

마지막으로, 알맞게 이펙트가 상태를 변경하지만, 루프가 있다면 그것은 상태 변경이 이펙트의 의존성을 변경해서 그렇다.

### My cleanup logic runs even though my component didn’t unmount

클린업 함수는 언마운트에만 일어나는 것이 아니고, 의존성이 변경 되었을 경우도 실행된다.
게다가, 개발 환경에서 리액트는 셋업과 클린업을 한 번더 호출한다.

만약 셋업 코드 없는 클린업은 다음과 같을 것이다:

```jsx
useEffect(() => {
  // 🔴 Avoid: Cleanup logic without corresponding setup logic
  return () => {
    doSomething();
  };
}, []);
```

클린업 로직은 셋업 로직과 대칭이어야 하고 셋업의 동작을 멈춰야한다.

```jsx
useEffect(() => {
  const connection = createConnection(serverUrl, roomId);
  connection.connect();
  return () => {
    connection.disconnect();
  };
}, [serverUrl, roomId]);
```

### My Effect does something visual, and I see a flicker before it runs

만약 이펙트가 브라우저가 스크린을 페인팅하는 것을 막는 다면, `useEffect`를 `useLayoutEffect`로 대체해라.
대부분의 이펙트에는 필요하지 않는다. 브라우저가 페인트하기 전에 중대한 경우에만 사용해라: 유저가 보기전 툴팁의 위치 계산

---

## What I Learned

이펙트를 이스케이프 해치부터 제법 읽고 공부하고 있는데, 확실히 이펙트를 좀 더 이해하게 됐다.
엄격 모드의 필요성(클린업 함수가 잘 구현되었는지 확인 해줌)도 느끼게 되었다.
