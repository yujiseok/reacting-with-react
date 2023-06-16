# Separating Events from Effects

이벤트 핸들러는 같은 상호작용이 있을 경우만 재실행된다. 이벤트 핸들러와 다르게 이펙트는 이전 랜더와 프랍이나 상태 변수가 다를 경우 싱크를 다시 맞춘다. 가끔, 당신은 이 두 가지를 섞고 싶을 수 있다: 어떤 값에만 응답해 이펙트가 재실행되도록.

## Choosing between event handlers and Effects

우선 이벤트 핸들러와 이펙트의 차이를 복습해보자.

당신이 챗 룸 컴포넌트를 구현해야 한다고 생각하자. 당신의 요구 사항은 다음과 같다.

1. 선택된 챗 룸에 자동으로 연결되어야함
2. 전송 버튼을 눌렀을 시 메세지를 보내야함

이미 코드를 구현했지만, 그것을 어디에 둘지 모른다고 가정하자.
이벤트 핸들러를 사용해야하는 지 이펙트를 사용해야하는지
이 질문에 답이 필요한 경우, 코드가 왜 실행되어야 하는지를 고려해라.

### Event handlers run in response to specific interactions

유저의 관점에서, 메세지를 보내는 것은 특정한 전송 버튼을 눌렀기 때문이다.
만약 메세지가 다른 상황이나 시간에 보내지면 유저들은 화날 것이다.
이것이 메세지를 전송하는 것이 이벤트 핸들러야하는 이유다. 이벤트 핸들러는 특정한 상호 작용을 다룰 수 있게 해준다.

```jsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState("");
  // ...
  function handleSendClick() {
    sendMessage(message);
  }
  // ...
  return (
    <>
      <input value={message} onChange={(e) => setMessage(e.target.value)} />
      <button onClick={handleSendClick}>Send</button>;
    </>
  );
}
```

이벤트 핸들러와는 sendMessage(message)가 유저가 버튼을 눌렀을 경우에만 실행된다.

### Effects run whenever synchronization is needed

당신은 또한 컴포넌트가 챗 룸에 연결을 유지하길 원한다. 이런 코드는 어디에 있을까?

이런 코드를 실행하는 것은 어떤 특정한 상호 작용이 아니다. 유저가 왜 또는 어떻게 챗 룸 화면에 접속했는지는 중요하지 않다.
그들은 그것을 보고 상호작용 한다. 컴포넌트는 선택된 챗 서버와 연결을 유지해야한다.
심지어 챗 룸 컴포넌트가 당신의 앱 초기화면이고, 상호 작용이 없어도 당신은 연결이 필요하다.
이것이 이펙트가 필요한 이유다:

```jsx
function ChatRoom({ roomId }) {
  // ...
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

이 코드와, 당신은 유저의 상호 작용과 관계 없이 현재 선택된 챗 서버와 항상 연결되는 것을 보장해야한다.
유저가 앱을 열고, 다른 룸을 선택하고, 다른 화면으로 이동해도 당신의 이펙트는 현재 선택된 룸과 연결 싱크를 맞추는 것을 보장해야하며 필요에 따라 재연결 해야한다.

## Reactive values and reactive logic

직관적으로, 당신은 이벤트 핸들러는 버튼을 클릭하는 것과 같이 항상 수동적으로 실행된다 말할 수 있다.
그에 반해 이펙트는, 자동적이다: 그들은 싱크를 맞추는 것이 필요하면 실행/재실행 된다.

좀 더 정확하게 생각하는 법이 있다.

프랍, 상태 그리고 컴포넌트 몸체의 변수들은 반응 값이라 불린다. 이 예에서 serverUrl을 제외한 roomId와 message가 반응 값이다. 그들은 랜더링 데이터 플로우에 참여한다.

```jsx
const serverUrl = "https://localhost:1234";

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState("");

  // ...
}
```

이 같은 반응 값들은 리랜더중 변경될 수 있다. 유저는 message를 변경할 수 있고 드롭다운으로 roomId를 선택할 수 있다.
이벤트 핸들러와 이펙트는 변화에 다르게 반응한다.

- 이벤트 핸들러의 로직은 반응성이 없다. 사용자가 동일한 상호 작용을 하지 않는 이상 재실행되지 않는다. 이벤트 핸들러는 반응 값을 반응 없이 읽을 수 있다.
- 이펙트의 로직은 반응성이 있다. 당신의 이펙트가 반응 값을 읽는 다면, 의존성에 추가해야한다. 그러면, 그 값의 변경에 따라 리랜더가 발생하면, 리액트는 이펙트의 로직을 새 값과 재실행한다.

### Logic inside event handlers is not reactive

유저의 관점에서, message를 변경하는 것이 그것을 전송하는 것을 뜻하지 않는다. 오직 유저가 입력했을 때이다. 다른 말로, 메세지를 전송하는 것은 반응성이 있으면 안된다. 반응 값이 변했을 때 실행되면 안된다. 이것이 이벤트 핸들러에 작성되는 이유다.

이벤트 핸들러는 반응성이 없으므로, sendMessage(message)는 오직 유저가 전송 버튼을 눌렀을 경우 실행되어야 한다.

### Logic inside Effects is reactive

유저의 관점에서, roomId를 변경하는 것은 다른 룸과 연결하기를 원하는 것이다.
다른 말로, 다른 룸과 연결하는 것은 반응성이 있어야한다. 당신은 반응 값과 이런 코드를 유지와 값이 달라지면 재실행을 원한다.
이것이 이펙트 내부에 작성되는 이유다.

이펙트는 반응하므로, createConnection(serverUrl, roomId)와 connection.connect()는 roomId가 다를 경우 실행된다.
당신의 이펙트는 현재 선택된 룸과 연결을 유지해야한다.

## Extracting non-reactive logic out of Effects

반응성 로직과 그렇지 않은 로직을 같이 두면 복잡해진다.

예를 들어, 유저가 연결되었을 때, 알림을 보여주고 싶다. 당신은 현재 테마를 읽어 알림을 테마에 맞게 보여준다.

```jsx
function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('Connected!', theme);
    });
    connection.connect();
    // ...
```

하지만, theme은 반응 값이고 모든 반응 값은 의존성에 추가되어야 한다. 이제 당신은 theme을 의존성에 명시한다.

```jsx
unction ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('Connected!', theme);
    });
    connection.connect();
    return () => {
      connection.disconnect()
    };
  }, [roomId, theme]); // ✅ All dependencies declared
  // ...
```

roomId가 변경 됐을 경우, 재연결은 당신이 예상한 것이다. 하지만, theme 역시 의존성이기에, 다크와 라이트 테마를 변경할 때도 다시 연결된다. 이것은 좋지 않다.

즉, 이펙트에 있더라도 반응 하지 않는 것을 원할 것이다.

```jsx
showNotification("Connected!", theme);
```

당신은 반응성이 없는 로직을 이펙트에서 분리해야한다.

### Declaring an Effect Event

> Under Construction
> 이 섹션은 정식 출시 되지 않은 실험적인 API를 다루고 있다.

useEffectEvent 훅을 통해 이런 반응성 없는 로직을 분리할 수 있다.

```jsx
import { useEffect, useEffectEvent } from 'react';

function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme);
  });
  // ...
```

onConnected는 이펙트 이벤트로 불린다. 이것은 이펙트 로직의 한부분이지만, 이것은 이벤트 핸들러와 같이 동작한다.
이 안의 로직은 반응성이 없지만, 항상 프랍과 상태의 최신 값을 지켜본다.

이제 onConnected 이펙트 이벤트를 이펙트안에서 호출할 수 있다.

```jsx
function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      onConnected();
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...

```

이것은 문제를 해결한다. onConnected를 의존성에서 제거해야 하는 것을 기억해라.
이펙트 이벤트는 반응성이 없기에 의존성에서 제거되어야 한다.

당신은 이펙트 이벤트가 이벤트 핸들러와 매우 비슷하다고 생각할 것이다. 가장 큰 차이점은 이벤트 핸들러는 유저의 상호 작용에 동작하고, 이펙트 이벤트는 이펙트에 의해 실행된다는 것이다. 이펙트 이벤트는 반응하는 이펙트와 반응성이 없는 코드의 관계를 끊어버린다.

#### Reading latest props and state with Effect Events

> Under Construction
> 이 섹션은 정식 출시 되지 않은 실험적인 API를 다루고 있다.

이펙트 이벤트는 의존성 린터를 억제하고 싶은 많은 패턴을 고칠 수 있다.

예를 들어, 페이지 접속 로그를 찍는 이펙트가 있다.

```jsx
function Page() {
  useEffect(() => {
    logVisit();
  }, []);
  // ...
}
```

후에, 당신은 여러 라우트를 사이트에 추가했다. 이제 페이지 컴포넌트는 url 프랍으로 현재 경로를 받는다.
이제 당신은 logVisit을 호출할 때, url을 넘긴다. 하지만, 의존성 린터가 에러를 보인다.

```jsx
function Page({ url }) {
  useEffect(() => {
    logVisit(url);
  }, []); // 🔴 React Hook useEffect has a missing dependency: 'url'
  // ...
}
```

코드가 무엇을 하길 원하는지 생각해라. 당신은 다른 페이지의 다른 url에 따른 로그를 원한다. 즉, logVisit은 url에 따라 호출되어야 한다. 이 이유로 url을 의존성에 추가해 의존성 린터를 따르는 것이 맞다.

```jsx
function Page({ url }) {
  useEffect(() => {
    logVisit(url);
  }, [url]); // ✅ All dependencies declared
  // ...
}
```

이제 쇼핑 카트에 담긴 아이템의 개수를 같이 포함하고 싶다.

```jsx
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  useEffect(() => {
    logVisit(url, numberOfItems);
  }, [url]); // 🔴 React Hook useEffect has a missing dependency: 'numberOfItems'
  // ...
}
```

numberOfItems를 이펙트 안에 사용하므로, 린터는 의존성에 추가하라 요청합니다.
하지만, 당신은 numberOfItems이 변했을 시 logVisit을 호출하고 싶지않습니다.
만약 유저가 카트에 무언가 추가하면, numberOfItems이 변경되고 이것은 유저가 페이지를 재방문한 것을 뜻하지 않는다.
다시 말해, 페이지 방문은 어떤 의미에서 이벤트이다. 그것은 정확한 때에 일어난다.

코드를 두 부분으로 나눈다:

```jsx
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  const onVisit = useEffectEvent((visitedUrl) => {
    logVisit(visitedUrl, numberOfItems);
  });

  useEffect(() => {
    onVisit(url);
  }, [url]); // ✅ All dependencies declared
  // ...
}
```

onVisit은 이펙트 이벤트이다. 이 안의 코드는 반응성이 없다. 이것이 당신이 numberOfItems을 변화에 재실행되는 것을 상관 하지않아도 되는 이유다.

즉 이펙트 자체는 반응하도록 남아있다. 이펙트는 url 프랍을 사용하므로, url이 변경으로 리랜더가 되면 이펙트도 재실행된다.
이 이유로 onVisit가 이펙트 이벤트이다.

이 결과로 url이 변경했을 시 onVisit가 호출되고, 항상 최신의 numberOfItems 값을 읽는다.
하지만, numberOfItems가 변해도 코드 재실행이 일어나지 않는다.

#### Note

인자 없이 onVisit()을 호출하고 url을 읽는 다면 어떻게 될지 궁금할 것이다.

```jsx
const onVisit = useEffectEvent(() => {
  logVisit(url, numberOfItems);
});

useEffect(() => {
  onVisit();
}, [url]);
```

이것은 동작 하지만, 명시적으로 이펙트 이벤트에 url을 추가하는 것이 좋다.
url을 이펙트 이벤트의 인자로 전달하는 것은 다른 url을 가지고 페이지에 방문하는 것을 유저 관점에서 다른 이벤트로 나눌 수 있다.
visitedUrl은 이벤트 동작의 한 부분이다.

```jsx
const onVisit = useEffectEvent((visitedUrl) => {
  logVisit(visitedUrl, numberOfItems);
});

useEffect(() => {
  onVisit(url);
}, [url]);
```

이펙트 이벤트가 visitedUrl을 요구하기에, 당신은 url을 의존성에서 제거할 수 없다. 만약 당신이 url 의존성을 제거하면, 린터는 경고를 할 것이다. onVisit이 url과 반응하기 위해, 내부에서 url을 읽기 보다, 이펙트에서 전달해라.

이것은 특히 비동기 로직이 이펙트에 있을 경우 중요하다.

```jsx
const onVisit = useEffectEvent((visitedUrl) => {
  logVisit(visitedUrl, numberOfItems);
});

useEffect(() => {
  setTimeout(() => {
    onVisit(url);
  }, 5000); // Delay logging visits
}, [url]);
```

onVisit 내부의 url은 최신 url에 상응하지만, visitedUrl은 이펙트를 실행하게한 url에 해당한다.

#### DEEP DIVE - Is it okay to suppress the dependency linter instead?

기존 코드에서 당신은 아마 다음과 같이 린트 룰을 꺼둔 것을 볼 수 있다.

```jsx
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  useEffect(() => {
    logVisit(url, numberOfItems);
    // 🔴 Avoid suppressing the linter like this:
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [url]);
  // ...
}
```

useEffectEvent가 리액트의 안정적이 부분이 된다면, 우리는 린터를 절대 끄지 않는 것을 추천한다.

린트 룰을 끄는 것의 첫 번째 단점은 리액트가 더 이상 이펙트가 언제 새로운 반응 값을 코드에 주입해야하는지 모르는 것이다.
이전 예에서 리액트가 알려주어 당신은 url을 의존성에 추가했다. 린트를 끄면 더 이상 이런 알림을 받지 못한다. 이것은 버그를 유발한다.

린트를 억제해 발생하는 예가 있다. handleMove 함수는 현재 canMove 상태를 읽어 닷이 커서를 따라다녀야 하는지 보여준다.
하지만 canMove는 항상 handleMove 안에서 항상 true이다.

```jsx
import { useState, useEffect } from "react";

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [canMove, setCanMove] = useState(true);

  function handleMove(e) {
    if (canMove) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
  }

  useEffect(() => {
    window.addEventListener("pointermove", handleMove);
    return () => window.removeEventListener("pointermove", handleMove);
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={canMove}
          onChange={(e) => setCanMove(e.target.checked)}
        />
        The dot is allowed to move
      </label>
      <hr />
      <div
        style={{
          position: "absolute",
          backgroundColor: "pink",
          borderRadius: "50%",
          opacity: 0.6,
          transform: `translate(${position.x}px, ${position.y}px)`,
          pointerEvents: "none",
          left: -20,
          top: -20,
          width: 40,
          height: 40,
        }}
      />
    </>
  );
}
```

이 코드의 문제는 의존성 린터를 억제했기 때문이다. 이 억제를 제거하면, 이펙트가 handleMove 함수에 의존하는 것을 알 수 있다.
이것은 말이 된다: handleMove는 컴포넌트 내부에 선언 되어 반응 값이다. 모든 반응 값은 의존성으로 명시되어야 하며, 그렇지 않을 경우 stale 상태가 될 수 있다.

코드의 원작자는 리액트에 어떤 의존성도 필요없다고 거짓말을 하고 있다.
이것이 리액트의 이펙트가 canMove가 변경되어도 싱크를 다시 맞추지 않는 이유다.
리액트의 이펙트가 싱크를 다시 맞추지 않기에, handleMove가 부착된 리스너는 초기 랜더에 생성된 것이다.
초기 랜더시 canMove는 true이기에, 초기 랜더의 handleMove는 영원히 값을 값을 본다.

린터를 억제하지 않으면, stale 값에서 오는 문제가 없다.

useEffectEvent를 사용하면, 린터에 거짓말을 할 필요가 없으며 코드도 당신의 의도대로 동작한다.

```jsx
import { experimental_useEffectEvent as useEffectEvent } from "react";

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [canMove, setCanMove] = useState(true);

  const onMove = useEffectEvent((e) => {
    if (canMove) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
  });

  useEffect(() => {
    window.addEventListener("pointermove", onMove);
    return () => window.removeEventListener("pointermove", onMove);
  }, []);

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={canMove}
          onChange={(e) => setCanMove(e.target.checked)}
        />
        The dot is allowed to move
      </label>
      <hr />
      <div
        style={{
          position: "absolute",
          backgroundColor: "pink",
          borderRadius: "50%",
          opacity: 0.6,
          transform: `translate(${position.x}px, ${position.y}px)`,
          pointerEvents: "none",
          left: -20,
          top: -20,
          width: 40,
          height: 40,
        }}
      />
    </>
  );
}
```

useEffectEvent가 항상 맞는 해결책은 아니다. 당신은 반응하고 싶지 않은 코드에만 적용해야한다.
위 예에서, canMove와 관련하여 이펙트의 코드가 반응하는 것을 원하지 않았다.
이것이 이펙트 이벤트로 추출하는 것이 말이되는 이유다.

### Limitations of Effect Events

> Under Construction
> 이 섹션은 정식 출시 되지 않은 실험적인 API를 다루고 있다.

이펙트 이벤트는 매우 한정적인 사용 용도를 갖는다.

- 오직 이펙트 안에서 호출
- 다른 컴포넌트나 훅으로 전달 하지 않기

예를 들어, 이펙트 이벤트를 다음과 같이 선언/전달하지 말아야한다.

```jsx
function Timer() {
  const [count, setCount] = useState(0);

  const onTick = useEffectEvent(() => {
    setCount(count + 1);
  });

  useTimer(onTick, 1000); // 🔴 Avoid: Passing Effect Events

  return <h1>{count}</h1>;
}

function useTimer(callback, delay) {
  useEffect(() => {
    const id = setInterval(() => {
      callback();
    }, delay);
    return () => {
      clearInterval(id);
    };
  }, [delay, callback]); // Need to specify "callback" in dependencies
}
```

대신에, 이펙트 이벤트를 사용하는 이펙트 옆에 선언해라.

```jsx
function Timer() {
  const [count, setCount] = useState(0);
  useTimer(() => {
    setCount(count + 1);
  }, 1000);
  return <h1>{count}</h1>;
}

function useTimer(callback, delay) {
  const onTick = useEffectEvent(() => {
    callback();
  });

  useEffect(() => {
    const id = setInterval(() => {
      onTick(); // ✅ Good: Only called locally inside an Effect
    }, delay);
    return () => {
      clearInterval(id);
    };
  }, [delay]); // No need to specify "onTick" (an Effect Event) as a dependency
}
```

이펙트 이벤트는 이펙트 코드의 반응성이 없는 부분이다. 그들은 사용되는 이펙트 옆에 있어야 한다.

## Recap

- 이벤트 핸들러는 특정 상호 작용에 실행된다.
- 이펙트는 싱크를 맞출 필요가 있다면 실행된다.
- 이벤트 핸들러 내부의 로직은 반응성이 없다.
- 이펙트 내부의 로직은 반응성이 있다.
- 반응성이 없는 이펙트의 로직을 이펙트 이벤트로 옮길 수 있다.
- 이펙트 이벤트는 오직 이펙트 안에서 호출
- 이펙트 이벤트를 다른 컴포넌트나 훅에 전달 x

---

## What I Learned

이펙트와 이벤트 핸들러의 차이점을 좀 더 깊게 알 수 있게 되었다.
상호 작용이 필요하다 -> 이벤트 핸들러
상포 작용 필요 없이 싱크를 맞춰야한다 -> 이펙트

useEffectEvent는 처음 보는 훅인데, 여러 문제를 해결할 수 있는 훅인 것 같다.
아직 이해가 안가 좀 더 공부를 해야겠다.
