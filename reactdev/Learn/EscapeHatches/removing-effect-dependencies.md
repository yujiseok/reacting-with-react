# Removing Effect Dependencies

이펙트를 작성할 때, 린터는 이펙트가 읽는 반응 값을 의존성에 포함하도록 검사합니다.
이것은 당신의 이펙트가 최신의 프랍과 상태와 싱크를 맞추는 것을 보장한다.
불필요한 의존성은 이펙트를 너무 많이 실행시키고 무한 루프를 유발한다.
이 가이드에 따라 불필요 의존성을 이펙트에서 제거해라.

## Dependencies should match the code

이펙트를 작성할 때, 이펙트가 당신이 원하는 대로 어떻게 시작하고 중지할지 명시해야한다.

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  	// ...
}
```

이펙트의 의존성을 비우면, 린터는 맞는 의존성을 추천한다.

```jsx
import { useState, useEffect } from "react";
import { createConnection } from "./chat.js";

const serverUrl = "https://localhost:1234";

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // <-- Fix the mistake here!
  return <h1>Welcome to the {roomId} room!</h1>;
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

> 11:6 - React Hook useEffect has a missing dependency: 'roomId'. Either include it or remove the dependency array.

린터의 말 대로 추가한다:

```jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
}
```

이펙트는 반응 값에 반응한다. roomId가 반응 값이기에, 린터는 의존성에 추가하도록 검사한다.
만약 roomId에 다른 값이 온다면, 리액트는 이펙트의 싱크를 다시 맞춘다.
이것은 챗이 선택된 룸과 연결을 유지하고 드롭다운과 반응하는 것을 보장한다.

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

### To remove a dependency, prove that it’s not a dependency

당신은 이펙트의 의존성을 선택할 수 없다. 이펙트 안에서 사용되는 모든 반응 값은 의존성 배열에 명시되어야한다.
의존성 배열은 주변 코드에 의해 결정된다.

```jsx
const serverUrl = "https://localhost:1234";

function ChatRoom({ roomId }) {
  // This is a reactive value
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // This Effect reads that reactive value
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ So you must specify that reactive value as a dependency of your Effect
  // ...
}
```

반응 값은 프랍과 컴포넌트 내부에 선언된 변수와 함수를 모드 포함한다.
roomId가 반응 값이기에, 의존성에서 제거할 수 없다.

```jsx
const serverUrl = "https://localhost:1234";

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // 🔴 React Hook useEffect has a missing dependency: 'roomId'
  // ...
}
```

roomId가 변경 되면, 이것은 버그를 유발한다.

의존성을 제거하기 위해선, 린터에 의존성에 필요 없다는 것을 증명해야한다.
예를 들어, 당신은 roomId를 컴포넌트 외부로 옮겨서 반응하지 않고 리랜더를 유발하지 않는 것을 증명해야한다.

```jsx
const serverUrl = "https://localhost:1234";
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

이제 roomId는 반응 값이 아니기에, 의존성에 필요 없다.

이것이 빈 의존성 배열을 설명한다. 당신의 이펙트는 이제 어떤 반응 값에도 의존하지 않으며, 컴포넌트의 프랍이나 상태가 변경되어도 재실행되지 않는다.

### To change the dependencies, change the code

워크플로우에 특정한 패턴을 발견했을 것이다.

1. 이펙트의 코드를 수정하거나 반응 값이 선언된 것을 수정
2. 린터를 따라 수정된 코드에 맞는 의존성을 적용
3. 만약 의존성 배열에 문제가 있다면, 첫 단계로 돌아가라

마지막 파트가 중요하다. 만약 당신이 의존성을 교체하고 싶다면, 주변 코드를 먼저 수정해라.
당신은 의존성 배열을 이펙트 코드에서 사용되는 모든 반응 값의 리스트로 생각할 수 있다.
당신은 무엇을 넣을지 선택하면 안된다. 리스트가 코드를 설명한다. 의존성 배열을 변경하려면, 코드를 변경해라.

이것은 방정식을 푸는 느낌을 준다. 목표(의존성 제거)를 갖고, 그 목표를 매치하는.
모든 사람이 방정식을 푸는 것이 재밌다고 생각하지 않으며, 이펙트 작성도 마찬가지다.

## Removing unnecessary dependencies

코드 반영을 위해, 의존성을 수정할때, 의존성 배열을 봐라.
이펙트가 재실행되기에 맞는 의존성 변화가 있는지? 가끔 답은 아니오다.

- 당신은 다른 조건에 따라 이펙트를 재실행 시키고 싶을 수 있다.
- 가끔 당신은 최신 값의 변경에 반응하는 것이 아닌 읽기만을 원할 수 있다.
- 의존성이 객체나 함수여서 의도하지 않은 변화를 일으킨다.

올바른 해결법을 위해, 당신은 당신의 이펙트에 질문을 할 필요가 있다.

### Should this code move to an event handler?

첫째로 생각할 것은 코드가 이펙트 안에 존재해야 하는 가이다.

제출시, submitted 상태를 true로 바꾸는 폼을 생각해보자.
당신은 post 요청과 알림을 보여줘야한다.
당신은 submitted가 true일 때 반응하는 로직을 이펙트에 작성했다.

```jsx
function Form() {
  const [submitted, setSubmitted] = useState(false);

  useEffect(() => {
    if (submitted) {
      // 🔴 Avoid: Event-specific logic inside an Effect
      post("/api/register");
      showNotification("Successfully registered!");
    }
  }, [submitted]);

  function handleSubmit() {
    setSubmitted(true);
  }

  // ...
}
```

후에, 테마에 맞는 알림 스타일을 추가하기 위해 당신은 현재 테마를 읽는다. theme가 컴포넌트 몸체에 선언되었기에, 이것은 반응 값이며, 의존성에 추가한다.

```jsx
function Form() {
  const [submitted, setSubmitted] = useState(false);
  const theme = useContext(ThemeContext);

  useEffect(() => {
    if (submitted) {
      // 🔴 Avoid: Event-specific logic inside an Effect
      post("/api/register");
      showNotification("Successfully registered!", theme);
    }
  }, [submitted, theme]); // ✅ All dependencies declared

  function handleSubmit() {
    setSubmitted(true);
  }

  // ...
}
```

이렇게 하면 당신은 버그를 유발한다. 폼을 제출하고 다크/라이트 테마를 전환하는 것을 상상해보아라.
theme은 변경될 것이고, 이펙트는 재실행되고, 동일한 알림을 보낼 것이다.

여기서 문제는 애초에 이것은 이펙트안에 있으면 안된다는 것이다.
당신은 특정한 상호 작용인 폼이 제출 될때 post 요청을 보내고 싶을 것이다.
특정 상호 작용에 맞는 코드를 실행하고 싶다면, 로직을 직접적으로 상응하는 이벤트 작성해라.

```jsx
function Form() {
  const theme = useContext(ThemeContext);

  function handleSubmit() {
    // ✅ Good: Event-specific logic is called from event handlers
    post("/api/register");
    showNotification("Successfully registered!", theme);
  }

  // ...
}
```

이제 코드는 이벤트 핸들러 내부에 있으며, 반응성이 없으며 유저가 폼을 제출했을 시에만 동작한다.

### Is your Effect doing several unrelated things?

다음으로 이펙트가 관련이 없는 것들을 처리하지 않는지 물어봐야한다.

도시와 지역을 선택하는 택배 폼이 있다고 생각해라.
선택된 나라에 따라 도시 리스트를 페치해 드롭다운에 보여준다.

```jsx
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  const [city, setCity] = useState(null);

  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setCities(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [country]); // ✅ All dependencies declared

  // ...
```

이것은 데이터를 이펙트에서 페치하는 좋은 예이다.
당신은 도시의 상태를 네트워크와 나라 프랍으로 싱크를 맞춘다.
ShippingForm이 보이자마자 그리고 나라의 변경에 따라 페치되어야 하므로 이벤트 핸들러는 이것을 처리할 수 없습니다.

이제 당신은 현재 도시에 따라 지역을 가져오는 셀렉트 박스를 추가합니다.
당신은 아마 같은 이펙트안에 두번째 페치 호출을 추가할 것입니다.

```jsx
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);

  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setCities(json);
        }
      });
    // 🔴 Avoid: A single Effect synchronizes two independent processes
    if (city) {
      fetch(`/api/areas?city=${city}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setAreas(json);
          }
        });
    }
    return () => {
      ignore = true;
    };
  }, [country, city]); // ✅ All dependencies declared

  // ...
```

하지만, 이펙트가 도시 상태 변수를 사용하기에, 당신은 도시를 의존성에 추가해야한다.
이것은 문제를 유발하는데: 다른 도시를 선택하면, 이펙트는 재실행되고, fetchCities(country)를 호출한다.
이 결과로, 불필요한 도시의 목록을 리페치하게 된다.

문제는 연관이 없는 것의 싱크를 맞추려고 해서 일어난다.

1. 당신은 cities의 상태를 country 프랍에 기반한 네트워크 상태에 싱크를 맞추고 싶다.
2. 당신은 areas의 상태를 city 상태에 기반한 네트워크 상태에 싱크를 맞추고 싶다.

두 이펙트로 로직을 분리한다.

```jsx
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setCities(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [country]); // ✅ All dependencies declared

  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);
  useEffect(() => {
    if (city) {
      let ignore = false;
      fetch(`/api/areas?city=${city}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setAreas(json);
          }
        });
      return () => {
        ignore = true;
      };
    }
  }, [city]); // ✅ All dependencies declared

  // ...

```

이제 첫 이펙트는 country가 변경될 때 재실행되고, 두번재 이펙트는 city가 변할때 재실행된다.
당신은 목적에 따라 나눌수 있다: 두 개의 다른 것은 두 개의 다른 이펙트에 싱크를 맞게 한다.

최종 코드는 이전 코드 보다 길지만, 이펙트를 분리하는 것이 맞다.
각 이펙트는 독립적인 싱크 작업을 나타내야한다.
이 예에서, 하나의 이펙트를 지워도 다른 이펙트가 정상 작동한다.
이것은 그들이 서로 다른 것의 싱크를 맞추며, 분리하는 것이 좋다는 것을 뜻한다.
만약 복제가 두렵다면, 커스텀 훅으로 생성할 수 있다.

### Are you reading some state to calculate the next state?

이 이펙트는 새로운 메세지가 도착하면 새로운 배열을 생성에 messages를 업데이트 한다.

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages([...messages, receivedMessage]);
    });
    // ...
```

messages는 기존에 존재하는 메세지와 새메세지를 마지막에 추가해 새로운 배열을 만들어 내고있다.
하지만, messages는 반응 값이기에 의존성에 추가되어야 한다.

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages([...messages, receivedMessage]);
    });
    return () => connection.disconnect();
  }, [roomId, messages]); // ✅ All dependencies declared
  // ...
```

messages를 의존성에 추가하는 것은 문제를 유발한다.

메세지를 받을 경우, setMessages()가 컴포넌트의 리랜더와 수신한 메세지를 포함한 새로운 messages 배열을 생성한다.
하지만, 이펙트가 messages에 의존하기에, 이것은 이펙트의 싱크를 다시 맞춘다. 즉 새로운 메세지를 받을 때마다, 챗은 재연결된다.
유저는 이것을 싫어할 것이다.

이것을 고치기 위해, messages를 이펙트안에서 읽지 않도록 해라. 대신에, 업데이터 함수를 전달해라.

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
```

이제 당신의 이펙트가 messages 변수를 읽지 않는 것을 보아라. 당신은 오직 msgs => [...msgs, receivedMessage]와 같은 업데이터 함수를 전달하면 된다. 리액트는 업데이터 함수를 큐에 등록하고 다음 랜더에 msgs 인자를 제공한다. 이것이 이펙트에 messages가 필요하지 않게된 이유이다. 이것의 결과로 수신된 메세지는 챗 재연결을 일으키지 않는다.

### Do you want to read a value without “reacting” to its changes?

> Under Construction
> 이 섹션은 정식 출시 되지 않은 실험적인 API를 다루고 있다.

당신은 isMuted가 true가 아닐 시 유저가 메세지를 받으면 소리를 재생시키고 싶어한다.

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [isMuted, setIsMuted] = useState(false);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]);
      if (!isMuted) {
        playSound();
      }
    });
    // ...
```

이펙트의 코드가 isMuted를 사용하기에, 의존성에 추가해야한다.

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [isMuted, setIsMuted] = useState(false);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]);
      if (!isMuted) {
        playSound();
      }
    });
    return () => connection.disconnect();
  }, [roomId, isMuted]); // ✅ All dependencies declared
  // ...

```

isMuted가 변할때 마다 문제가 발생하는데, 이펙트는 싱크를 다시 맞추고 챗과 재연결한다.
이것은 원하는 유저 경험이 아니다.

이 문제를 해결하기 위해, 당신은 이펙트 내부에서 반응이 없어야하는 로직을 분리할 수 있다.
당신은 이펙트가 isMuted의 변화에 반응하기를 원치 않는다. 이런 반응성이 없는 로직을 이펙트 이벤트로 옮겨라.

```jsx
import { useState, useEffect, useEffectEvent } from 'react';

function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [isMuted, setIsMuted] = useState(false);

  const onMessage = useEffectEvent(receivedMessage => {
    setMessages(msgs => [...msgs, receivedMessage]);
    if (!isMuted) {
      playSound();
    }
  });

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
```

이펙트 이벤트는 당신의 이펙트를 반응 파트와 비반응 파트로 분리해준다.
이제 당신은 isMuted를 이펙트 이벤트에서 읽고 있으므로, 이펙트의 의존성으로 필요가 없다.
결과로, muted 속성이 토글 되어도 재연결이 일어나지 않는다.

#### Wrapping an event handler from the props

이벤트 핸들러를 프랍으로 전달 받으면 동일한 문제를 직면할 수 있다.

```jsx
function ChatRoom({ roomId, onReceiveMessage }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onReceiveMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId, onReceiveMessage]); // ✅ All dependencies declared
  // ...
```

부모 컴포넌트가 매 랜더시 다른 onReceiveMessage를 전달한다고 생각해보자.

```jsx
<ChatRoom
  roomId={roomId}
  onReceiveMessage={(receivedMessage) => {
    // ...
  }}
/>
```

onReceiveMessage가 의존성이므로, 이것은 부모가 리랜더될 시 이펙트의 싱크를 다시 맞추게 된다.
이것은 챗의 재연결을 일으킨다. 이것을 해결하기 위해, 이펙트 이벤트로 감싸준다.

```jsx
function ChatRoom({ roomId, onReceiveMessage }) {
  const [messages, setMessages] = useState([]);

  const onMessage = useEffectEvent(receivedMessage => {
    onReceiveMessage(receivedMessage);
  });

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
```

이펙트 이벤트는 비반응이기 때문에, 그들을 의존성으로 명시할 필요 없다. 그 결과 챗은 부모 컴포넌트가 매 랜더시 다른 함수를 전달해도 더 이상 재연결 되지 않는다.

#### Separating reactive and non-reactive code

예에서, roomId가 변했을 경우 당신은 로그를 찍고 싶다. 모든 로그에 notificationCount를 포함하고 싶지만, notificationCount의 변경이 로그 이벤트를 일으키게 하고 싶지는 않다.

해결법은 비반응 코드를 이펙트 이벤트에 추가하는 것이다.

```jsx
function Chat({ roomId, notificationCount }) {
  const onVisit = useEffectEvent((visitedRoomId) => {
    logVisit(visitedRoomId, notificationCount);
  });

  useEffect(() => {
    onVisit(roomId);
  }, [roomId]); // ✅ All dependencies declared
  // ...
}
```

roomId와는 반응을 하고 싶기에, 이펙트 내부에 작성한다.
하지만 notificationCount 변경이 로그를 찍는 것을 원치 않기에, 이펙트 이벤트에 추가한다.

### Does some reactive value change unintentionally?

때로 이펙트가 특정 값에 반응하기를 원하지만 해당 값이 원하는 것보다 더 자주 변경되어 사용자 관점에서 변경을 반영하지 않을 수 있다.
예를 들어, 당신은 컴포넌트 내부에 options 객체를 만들었고 이펙트 내부에서 읽는다.

```jsx
function ChatRoom({ roomId }) {
  // ...
  const options = {
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    // ...
```

이 객체는 컴포넌트 내부에 선언 되었기에, 반응 값이다. 반응 값을 이펙트에 읽기 위해선 의존성에 추가해야한다.
이것은 이펙트가 그것의 변화에 반응하는 것을 보장한다.

```jsx
// ...
useEffect(() => {
  const connection = createConnection(options);
  connection.connect();
  return () => connection.disconnect();
}, [options]); // ✅ All dependencies declared
// ...
```

하지만 이것은 문제를 일으킨다.

```jsx
import { useState, useEffect } from "react";
import { createConnection } from "./chat.js";

const serverUrl = "https://localhost:1234";

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState("");

  // Temporarily disable the linter to demonstrate the problem
  // eslint-disable-next-line react-hooks/exhaustive-deps
  const options = {
    serverUrl: serverUrl,
    roomId: roomId,
  };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]);

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

인풋이 message 상태를 변경시킨다. 유저의 관점에서, 이것은 챗 연결에 영향을 끼치면 안된다. 하지만, message가 변경될 때마다 당신의 컴포넌트는 리랜더 된다. 컴포넌트가 리랜더 되면, 그 안의 코드는 재실행 된다.

새로운 options 객체는 챗룸 컴포넌트의 리랜더 시 새로 생성된다.
리액트는 기존의 options 객체가 이전 랜더의 options과 다르다고 본다.
이것이 이펙트가 싱크를 다시 맞추는 이유이고, 타입할 때 재연결되는 이유다.

이 문제는 오직 객체와 함수에서 일어난다.
자바스크립트에서, 새로 만들어진 객체와 함수는 다른 것이라고 보기 때문이다. 내부에 같은 값을 소용해도 상관없다.

```js
// During the first render
const options1 = { serverUrl: "https://localhost:1234", roomId: "music" };

// During the next render
const options2 = { serverUrl: "https://localhost:1234", roomId: "music" };

// These are two different objects!
console.log(Object.is(options1, options2)); // false
```

객체와 함수 의존성은 당신이 원하는 것보다 많이 이펙트의 싱크를 다시 맞추게 한다.
이것이 당신이 이펙트의 의존성에 객체나 함수를 포함하는 것을 피해야하는 이유다.
대신에 컴포넌트 외부나, 이펙트 내부 혹은 원시값을 추출해라.

#### Move static objects and functions outside your component

만약 객체가 프랍이나 상태에 의존한 것이 아니라면, 컴포넌트 밖으로 옮겨라.

```jsx
const options = {
  serverUrl: 'https://localhost:1234',
  roomId: 'music'
};

function ChatRoom() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ All dependencies declared
  // ...
```

이 방법은 린터에 반응하지 않는 다는 것을 증명한다. 이것은 리랜더의 결과로 변경될 수 없고, 의존성이 될 필요가 없다.
이제 챗룸을 리랜더 하는 것은 이펙트가 재싱크를 맞추는 것을 일으키지 않는다.

이것은 함수도 마찬가지다.

```jsx
function createOptions() {
  return {
    serverUrl: 'https://localhost:1234',
    roomId: 'music'
  };
}

function ChatRoom() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ All dependencies declared
  // ...
```

createOptions이 컴포넌트 외부에 선언되었기에, 이것은 반응 값이 아니다. 이것이 의존성이 될 필요가 없는 이유와, 재싱크를 유발하지 않는 이유다.

#### Move dynamic objects and functions inside your Effect

만약 roomId 프랍과 같이 컴포넌트 외부로 뺄 수 없는 반응 값이 있을 수 있다.
당신은 이펙트 코드 내부에서 작성할 수 있다.

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
```

이제 options은 이펙트 내부에 선언되었으므로, 이것은 더이상 의존성이 필요 없다.
대신, 이펙트 안의 반응 값은 오직 roomId이다. roomId가 객체 또는 함수가 아니기에 이것이 의도와 다르게 동작하지 않는 다는 것을 보장한다. 자바스크립트에서, 숫자와 문자는 값으로 비교된다.

```js
// During the first render
const roomId1 = "music";

// During the next render
const roomId2 = "music";

// These two strings are the same!
console.log(Object.is(roomId1, roomId2)); // true
```

감사합니다. 더 이상 인풋 인풋 타입시 재연결은 없다.

이것은 roomId가 변경되어도 다시 연결이 된다.

함수도 마찬가지다.

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    function createOptions() {
      return {
        serverUrl: serverUrl,
        roomId: roomId
      };
    }

    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
```

당신은 이펙트 안에서 당신의 로직을 함수로 작성할 수 있다. 이펙트 안의 선언된 것은 역시 반응 값이 아니므로, 의존성에 추가될 필요가 없다.

#### Read primitive values from objects

때로 객체를 프랍으로 받을 경우가 있다.

```jsx
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // ✅ All dependencies declared
  // ...
```

위험은 부모 컴포넌트가 객체를 랜대중에 생성한다는 것이다.

```jsx
<ChatRoom
  roomId={roomId}
  options={{
    serverUrl: serverUrl,
    roomId: roomId,
  }}
/>
```

이것은 이펙트가 부모 컴포넌트가 리랜더 될 시 재싱크를 맞추는 것을 유발한다.
이것을 고치기 위해, 이펙트 밖에서 정보를 읽고, 객체나 함수를 의존성에 두는 것을 피해라

```jsx
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = options;
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ All dependencies declared
  // ...

```

로직이 반복적이게 되었지만, 이것은 이펙트가 어떤 정보에 의존해야하는지 명시적으로 알려준다.
만약 객체가 의도되지 않고 부모에 의해 재생성 되어도, 챗은 재연결되지 않는다. 하지만, options.roomId나 options.serverUrl는 다르기에, 챗은 재연결된다.

#### Calculate primitive values from functions

함수도 같은 접근이 가능하다. 부모 컴포넌트가 함수를 전달한다고 생각하자.

```jsx
<ChatRoom
  roomId={roomId}
  getOptions={() => {
    return {
      serverUrl: serverUrl,
      roomId: roomId,
    };
  }}
/>
```

의존성으로 만드는 것을 피하기 위해 이펙트 밖에서 호출한다.
이것은 객체가 아닌 roomId와 serverUrl를 반환하고, 이펙트 내부에서 읽을 수 있게 된다.

```jsx
function ChatRoom({ getOptions }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = getOptions();
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ All dependencies declared
  // ...
```

이것은 랜더 도중 호출하기에 순수 함수여야한다.
만약 당신의 함수가 이벤트 핸들러이며, 이펙트의 싱크를 맞추고 싶다면 이펙트 이벤트로 감싸라.

## Recap

- 의존성은 항상 코드와 일치해야한다.
- 의존성에 문제가 있다면, 코드를 수정해야한다.
- 의존성을 제거하기 위해선, 린터에 그것이 불필요하다는 것을 증명해야한다.
- 만약 코드가 어떤 상호 작용에 의해 응답할 경우 이벤트 핸들러로 옮겨라
- 만약 다른 이유로 이펙트가 호출 되면, 이펙트를 분리해라
- 만약 이전 상태 기반으로 상태를 변경하고 싶다면, 업데이터 함수를 사용해라
- 반응 없이 최신 값을 읽고 싶다면 이펙트 이벤트로 분리해라
- 자바스크립트에서, 다른 시간에 만들어진 객체와 함수는 다르다고 여겨진다.
- 객체와 함수 의존성을 피하기 위해, 컴포넌트 외부로 옮기거나 이펙트 내부로 옮겨라.

---

## What I Learned

이펙트 관련 문서를 읽으며 이펙트에 대해 이전보다 확실히 이해도가 올라갔다.
이벤트 핸들러와 이펙트의 기준을 명확히 해야하고, 만약 이펙트 내부에 반응 하지 않는 로직을 추가하고 싶으면 이펙트 이벤트로 분리해야한다.

또 의존성에 객체와 함수는 피하자 -> 항상 새로운 것이기에 이펙트의 싱크를 다시 맞추게함
