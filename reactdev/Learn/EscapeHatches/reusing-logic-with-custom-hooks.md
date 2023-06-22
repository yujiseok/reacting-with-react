# Reusing Logic with Custom Hooks

리액트는 useState, useContext 그리고 useEffect와 같은 빌트인 훅이 존재한다.
가끔, 당신은 좀 더 특정한 훅이 있으면 좋겠다고 생각한다. 예를 들어, 데이터를 페치하거나, 유저가 온라인인지 추적하거나, 챗 룸과 연결하는 것과 같은.
당신은 리액트에서 이런 훅을 찾을 수 없을 것이지만, 당신의 애플리케이션에서 필요한 당신의 훅을 만들 수 있다.

## Custom Hooks: Sharing logic between components

네트워크에 크게 의존하는 앱을 만든다고 생각해보자. 당신은 유저에게 네트워크가 끊기면 경고를 주고싶다.
이것을 어떻게 처리 해야할까? 이것을 구현하기 위해 컴포넌트에 두 가지가 필요하다.

1. 네트워크가 온라인인지 추적하기 위한 상태
2. 온라인 오프라인을 구독해 상태를 업데이트할 이펙트

이것을 당신이 네트워크 상태와 싱크를 맞추도록 해준다. 이런식으로 시작할 수 있다.

```jsx
import { useState, useEffect } from "react";

export default function StatusBar() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener("online", handleOnline);
    window.addEventListener("offline", handleOffline);
    return () => {
      window.removeEventListener("online", handleOnline);
      window.removeEventListener("offline", handleOffline);
    };
  }, []);

  return <h1>{isOnline ? "✅ Online" : "❌ Disconnected"}</h1>;
}
```

네트워크를 키고 끄면, 상태바가 이벤트에 따라 어떻게 업데이트 되는지 볼 수 있다.

똑같은 로직을 다른 컴포넌트에 사용한다고 생각해보자. 당신은 네트워크가 꺼지면 저장 대신 재연결을 보여주는 저장 버튼을 구현하고 싶다.

이것을 시작하기 우해 당신은 isOnline 상태와 이펙트를 저장 버튼에 가져올 수 있다.

```jsx
import { useState, useEffect } from "react";

export default function SaveButton() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener("online", handleOnline);
    window.addEventListener("offline", handleOffline);
    return () => {
      window.removeEventListener("online", handleOnline);
      window.removeEventListener("offline", handleOffline);
    };
  }, []);

  function handleSaveClick() {
    console.log("✅ Progress saved");
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? "Save progress" : "Reconnecting..."}
    </button>
  );
}
```

이것을 검증하기 위해, 네트워크를 끄면, 버튼의 모습이 변경될 것이다.

두 컴포넌트는 정상 동작하지만, 중복된 로직이 존재한다.
다른 화면을 갖지만, 당신은 로직의 재사용을 원한다.

### Extracting your own custom Hook from a component

useState와 useEffect 같은 빌트인 훅인 useOnlineStatus가 있다고 생각해보자.
그러면 두 컴포넌트는 심플해지고 중복되는 코드를 줄일 수 있다.

```jsx
function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? "✅ Online" : "❌ Disconnected"}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    console.log("✅ Progress saved");
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? "Save progress" : "Reconnecting..."}
    </button>
  );
}
```

비록 그런 빌트인 훅은 없지만, 직접 작성할 수 있다. useOnlineStatus라는 함수를 선언하고 컴포넌트에 작성한 중복 코드를 옮긴다.

```jsx
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener("online", handleOnline);
    window.addEventListener("offline", handleOffline);
    return () => {
      window.removeEventListener("online", handleOnline);
      window.removeEventListener("offline", handleOffline);
    };
  }, []);
  return isOnline;
}
```

함수의 마지막에서, isOnline을 반환한다. 이것은 당신의 컴포넌트가 값을 읽을 수 있게 해준다.

```jsx
import { useOnlineStatus } from "./useOnlineStatus.js";

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? "✅ Online" : "❌ Disconnected"}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    console.log("✅ Progress saved");
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? "Save progress" : "Reconnecting..."}
    </button>
  );
}

export default function App() {
  return (
    <>
      <SaveButton />
      <StatusBar />
    </>
  );
}
```

두 컴포넌트를 검증하기 위해 네트워크를 껐다 켜봐라.

이제 당신의 컴포넌트는 중복되는 코드가 없다. 더 중요한 것은 코드가 설명하는 것이 어떻게 하는가가 아닌, 무엇을 하기를 원하는지이다.

커스텀 훅으로 로직을 뽑아낼 때, 당신은 외부 시스템이나 브라우저를 어떻게 다룰지를 숨길 수 있다.
컴포넌트의 코드는 코드의 구현이 아니라 의도를 나타낸다.

### Hook names always start with use

리액트 애플리케이션은 컴포넌트로 구성된다. 컴포넌트는 빌트인 훅 또는 커스텀 훅으로 구성된다.
당신은 다른 사람이 작성한 커스텀 훅 또는 당신이 직접 작성한 커스텀 훅을 사용할 것이다.

당신은 아래의 네이밍 컨벤션을 반드시 따라야한다.

1. 리액트 컴포넌트의 이름은 StatusBar나 SaveButton와 같이 반드시 대문자로 시작해야한다. 리액트의 컴포넌트는 리액트가 화면에 보여줄 무언가(jsx 같은)를 반환해야한다.
2. 훅의 이름은 반드시 use로 시작하고 그 후에는 useState와 같이 대문자로 시작해야한다. 훅은 임의의 값을 반환한다.

이런 컨벤션은 상태와 이펙트가 어디에 있는지 또는 다른 리액트 기능이 숨겨져있는지를 보장한다.
예를 들어, 컴포넌트 내부에 getColor라는 함수가 호출 된다면, 이것은 use로 시작하지 않았기에 리액트의 상태를 소유할 수 없다는 것을 알 수 있다. 하지만 useOnlineStatus 호출은 그 내부에 다른 훅이 있다는 것을 알 수 잇다.

#### Note

만약 린터가 있다면, 이것은 네이밍 컨벤션을 강제한다. 오직 훅이나 다른 컴포넌트만이 다른 훅을 호출할 수 있다.

#### DEEP DIVE - Should all functions called during rendering start with the use prefix?

아니다. 훅이 아닌 훅은 호출되지 않는다.

만약 어떤 훅도 호출하지 않는 다면 use 프리픽스를 사용하는 것을 피해라. 대힌에, use 프리픽스를 제거하고 일반 함수로 작성해라.
예를 들어, useSorted는 훅을 호출 하지 않으므로 getSorted로 호출해라.

```jsx
// 🔴 Avoid: A Hook that doesn't use Hooks
function useSorted(items) {
  return items.slice().sort();
}

// ✅ Good: A regular function that doesn't use Hooks
function getSorted(items) {
  return items.slice().sort();
}
```

이것은 일반 함수이기에 조건문 안에서도 호출이 가능하다.

```jsx
function List({ items, shouldSort }) {
  let displayedItems = items;
  if (shouldSort) {
    // ✅ It's ok to call getSorted() conditionally because it's not a Hook
    displayedItems = getSorted(items);
  }
  // ...
}
```

만약 적어도 하나이상의 훅을 사용한다면 use 프리픽스를 사용해라.

```jsx
// ✅ Good: A Hook that uses other Hooks
function useAuth() {
  return useContext(Auth);
}
```

기술적으로, 이것은 리액트에 강제되지 않는다. 원칙적으로, 훅을 호출하지 않는 훅을 만들 수 있다.
이것은 가끔 혼란스럽고 제한적이므로 피해야하는 패턴이다. 하지만, 희귀한 케이스에서 도움을 줄 수 있다.
예를 들어, 현재에는 훅이 없지만, 후에 훅 호출을 추가할 계획일 때이다.
그러면, use 프리픽스를 다는 것이 말이된다.

```jsx
// ✅ Good: A Hook that will likely use some other Hooks later
function useAuth() {
  // TODO: Replace with this line when authentication is implemented:
  // return useContext(Auth);
  return TEST_USER;
}
```

그러면, 조건부로 호출할 수 없게 된다. 이것은 훅을 추가하고 호출할 때 중요하다. 내부에 훅을 추가할 예정이 없다면, 훅으로 만들지 마라.

### Custom Hooks let you share stateful logic, not state itself

위의 예에서, 네트워크를 끄고 켰을 때, 두 컴포넌트는 동시에 업데이트 된다.
하지만, 각 isOnline 상태가 공유되고 있다는 것은 잘못된 생각이다.

```jsx
function StatusBar() {
  const isOnline = useOnlineStatus();
  // ...
}

function SaveButton() {
  const isOnline = useOnlineStatus();
  // ...
}
```

이것은 중복을 뽑아내기 전과 동일하게 동작한다.

```jsx
function StatusBar() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    // ...
  }, []);
  // ...
}

function SaveButton() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    // ...
  }, []);
  // ...
}
```

두 개의 완전히 독립적인 상태 변수와 이펙트다. 그들은 외부의 동일한 값에 싱크를 맞추어서 동시에 일어나는 것이다.

명확히 하기 위해, 다른 예가 필요하다. 폼 컴포넌트를 생각해보아라.

```jsx
import { useState } from "react";

export default function Form() {
  const [firstName, setFirstName] = useState("Mary");
  const [lastName, setLastName] = useState("Poppins");

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
  }

  return (
    <>
      <label>
        First name:
        <input value={firstName} onChange={handleFirstNameChange} />
      </label>
      <label>
        Last name:
        <input value={lastName} onChange={handleLastNameChange} />
      </label>
      <p>
        <b>
          Good morning, {firstName} {lastName}.
        </b>
      </p>
    </>
  );
}
```

각 필드 별 중복되는 코드가 존대한다.

1. 상태
2. 체인지 핸들러
3. value와 onChange

useFormInput 훅으로 반복되는 로직을 추출할 수 있다.

```jsx
import { useState } from "react";

export function useFormInput(initialValue) {
  const [value, setValue] = useState(initialValue);

  function handleChange(e) {
    setValue(e.target.value);
  }

  const inputProps = {
    value: value,
    onChange: handleChange,
  };

  return inputProps;
}
```

value 라는 하나의 상태만 선언된 것을 알 수 있다.

하지만, 폼 컴포넌트는 useFormInput을 두 번 호출 하고 있다.

이것이 두 상태 변수 선언으로 동작하는 이유이다.

커스텀 훅은 상태 로직을 공유하지 상태 그 자체를 공유하지 않는다. 각각의 훅 호출은 똑같은 훅이어도 완전히 독립적이다.
이것이 두 코드 샌드 박스 예제가 같은 이유이다.

만약 여러 컴포넌트에서 상태를 공유하고 싶다면 끌어올려서 아래로 전달해라.

## Passing reactive values between Hooks

당신의 커스텀 훅 안에 있는 코드는, 당신의 컴포넌트가 리랜더 될때마다 재실행될 것이다.
이것이 컴포넌트와 같이 커스텀 훅이 순수해야하는 이유이다.
커스텀 훅을 컴포넌트의 몸체라고 생각해라.

커스텀 훅이 컴포넌트의 리랜더와 함께 하기에, 그들은 항상 최신의 상태와 프랍을 받는다.
serverUrl과 roomId를 변경했을 시, 이펙트는 변경에 반응하고 다시 싱크를 맞춘다.
이펙트의 의존성을 변경시 챗이 항상 재연결된다.

이제 이펙트의 코드를 커스텀 훅으로 옮겨보자.

```jsx
export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId,
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on("message", (msg) => {
      showNotification("New message: " + msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

이것은 챗룸 컴포넌트에 커스텀 훅이 내부적으로 어떻게 동작하는지에 관계없이 호출할 수 있게된다.

```jsx
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState("https://localhost:1234");

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl,
  });

  return (
    <>
      <label>
        Server URL:
        <input
          value={serverUrl}
          onChange={(e) => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
    </>
  );
}
```

이것은 더 간결해졌다.

서버 url과 선택된 룸을 변경해 이 로직이 프랍과 상태에 반응하는 것을 확인해라.

```jsx
import { useState } from "react";
import { useChatRoom } from "./useChatRoom.js";

export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState("https://localhost:1234");

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl,
  });

  return (
    <>
      <label>
        Server URL:
        <input
          value={serverUrl}
          onChange={(e) => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
    </>
  );
}
```

하나의 훅에서 리턴 값을 얻는 다는 것을 보아라. 그리고 다른 훅에 전달한다.

챗룸 컴포넌트가 리랜더될 시, 최신의 roomId와 serverUrl을 훅에 넘겨준다.
이것이 리랜더 후에 값이 달라도 다시 챗과 연결되는 이유다.

### Passing event handlers to custom Hooks

> Under Construction
> 이 섹션은 정식 출시 되지 않은 실험적인 API를 다루고 있다.

useChatRoom을 많은 컴포넌트에서 사용시, 당신은 컴포넌트의 커스텀 동작을 추가하고 싶을 것이다.
예를 들어, 현재 메세지를 받을 시 무엇을 할지 하드코딩되어 있다.

```jsx
export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId,
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on("message", (msg) => {
      showNotification("New message: " + msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

다시 이 로직을 컴포넌트로 옮기고 싶다고 생각하자.

```jsx
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl,
    onReceiveMessage(msg) {
      showNotification('New message: ' + msg);
    }
  });
  // ...
```

이 동작을 작동 시키기 위해, 당신의 커스텀 훅에 onReceiveMessage를 옵션으로 받도록 한다.

```jsx
export function useChatRoom({ serverUrl, roomId, onReceiveMessage }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId,
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on("message", (msg) => {
      onReceiveMessage(msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl, onReceiveMessage]); // ✅ All dependencies declared
}
```

이것은 동작 하지만, 훅이 이벤트 핸들러를 받을 시 한가지 개선이 필요하다.

의존성에 onReceiveMessage를 추가하는 것은 이상적이지 않은데, 컴포넌트가 매 리랜더시 챗이 재연결 되기 때문이다.
이벤트 핸들러를 이펙트 이벤트로 감싸 의존성에서 제거해라.

```jsx
import { useEffect, useEffectEvent } from "react";
// ...

export function useChatRoom({ serverUrl, roomId, onReceiveMessage }) {
  const onMessage = useEffectEvent(onReceiveMessage);

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId,
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on("message", (msg) => {
      onMessage(msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ All dependencies declared
}
```

이제 챗룸 컴포넌트가 리랜더될 시 재연결 되지 않는다.
useChatRoom이 어떻게 동작하는지 더 이상 알필요가 없어졌다. 이제 다른 컴포넌트에서 사용할 수 있으며, 다른 옵션을 추가해도 똑같은 방식으로 동작한다. 이것이 커스텀 훅의 장점이다.

## When to use custom Hooks

몇 개의 중복된 코드가 있다고 커스텀 훅으로 뽑아낼 필요는 없다.
작은 중복은 괜찮다. 예를 들어, 이전의 useState 훅 하나를 감싸기 위한 useFormInput 사실 불필요하다.

하지만, 이펙트를 작성한다면, 커스텀 훅으로 감싸는 것이 좀 더 명확하다. 당신은 아마 이펙트가 필요 없을 때가 많을 것이지만, 당신이 작성한다면, 그것은 리액트에서 벗어나 리액트가 빌트인으로 갖지 않은 외부 시스템과 싱크를 맞추는 것을 뜻한다.
커스텀 훅으로 감싸는 것은 정확하게 당신의 의도와 어떻게 데이터가 흐르는지 알려준다.

예를 들어, ShippingForm 컴포넌트가 두개의 드롭다운을 보여준다고 생각하자:
하나는 도시들을, 하나는 선택된 도시의 지역을 보여준다.
아래의 코드와 같이 시작할 것이다.

```jsx
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  // This Effect fetches cities for a country
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
  }, [country]);

  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);
  // This Effect fetches areas for the selected city
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
  }, [city]);

  // ...
```

이것은 제법 반복적이지만, 각 이펙트를 나누는 것이 맞다.
그들은 서로 다른 두가지의 싱크를 맞추므로, 그들을 하나의 이펙트로 합치면 안된다.
대신, 공통 로직을 useData 훅으로 뽑아낼 수 있다.

```jsx
function useData(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
    if (url) {
      let ignore = false;
      fetch(url)
        .then((response) => response.json())
        .then((json) => {
          if (!ignore) {
            setData(json);
          }
        });
      return () => {
        ignore = true;
      };
    }
  }, [url]);
  return data;
}
```

이제 당신은 ShippingForm 컴포넌트의 각 이펙트를 useData로 대체할 수 있다.

```jsx
function ShippingForm({ country }) {
  const cities = useData(`/api/cities?country=${country}`);
  const [city, setCity] = useState(null);
  const areas = useData(city ? `/api/areas?city=${city}` : null);
  // ...
```

커스텀 훅으로 만드는 것은 데이터를 명시적으로 한다. url을 전달하면 당신은 data를 갖느다.
이펙트가 useData에서 동작하므로 당신은 ShippingForm에서 불필요한 의존성 배열을 추가하는 것을 막을 수 있다.
시간이 지남에 따라, 당신 앱의 대부분의 이펙트는 커스텀 훅이 될 것이다.

#### DEEP DIVE - Keep your custom Hooks focused on concrete high-level use cases

커스텀 훅의 이름을 선택하는 것에서 시작해라. 만약 당신이 명확한 이름을 짓기에 고심한다면, 이것은 당신의 이펙트가 당신의 컴포넌트 로직과 너무 결합되어 아직 추출될 준비가 안된 것이다.

이상적으로, 커스텀 훅의 이름을 명확히 지어야 코드를 자주 작성하지 않는 사람들도 커스텀 훅이 무엇을 하는지, 무엇을 해야 하는지, 무엇을 반환하는지 추측할 수 있어야 한다.

- ✅ useData(url)
- ✅ useImpressionLog(eventName, extraData)
- ✅ useChatRoom(options)

만약 외부 시스템에 싱크를 맞춘다면, 당신의 커스텀 훅 이름은 시스템에 맞는 특수 용어와 기술적으로 지어야한다.
시스템과 친숙하기 위에 이름이 긴것은 좋다.

- ✅ useMediaQuery(query)
- ✅ useSocket(url)
- ✅ useIntersectionObserver(ref, options)

커스텀 훅을 견고한 하이-레벨 사용 사례에 초점을 맞추는 것을 유지해라.
커스텀 라이프사이클 훅을 만드는 것을 피해라.

- 🔴 useMount(fn)
- 🔴 useEffectOnce(fn)
- 🔴 useUpdateEffect(fn)

예를 들어, 이 useMount 훅은 오직 마운트 될 때 실행을 보장한다.

```jsx
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState("https://localhost:1234");

  // 🔴 Avoid: using custom "lifecycle" Hooks
  useMount(() => {
    const connection = createConnection({ roomId, serverUrl });
    connection.connect();

    post("/analytics/event", { eventName: "visit_chat" });
  });
  // ...
}

// 🔴 Avoid: creating custom "lifecycle" Hooks
function useMount(fn) {
  useEffect(() => {
    fn();
  }, []); // 🔴 React Hook useEffect has a missing dependency: 'fn'
}
```

useMount 같은 커스텀 훅은 리액트의 패러다임과 맞지 않는다.
예를 들어, 이 코드 예는 오류가 있는데, 직접 이펙트가 호출 되는 것이 아니어서 린터가 경고를 하지 못한다.
그것은 당신의 훅을 알지 못한다.

당신이 이펙트를 작성한다면, 리액트 api를 직접 사용해라

```jsx
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  // ✅ Good: two raw Effects separated by purpose

  useEffect(() => {
    const connection = createConnection({ serverUrl, roomId });
    connection.connect();
    return () => connection.disconnect();
  }, [serverUrl, roomId]);

  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_chat', roomId });
  }, [roomId]);

  // ...
```

그 후, 좀 더 하이-레벨 사용 사례로 뽑아 내라.

```jsx
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState("https://localhost:1234");

  // ✅ Great: custom Hooks named after their purpose
  useChatRoom({ serverUrl, roomId });
  useImpressionLog("visit_chat", { roomId });
  // ...
}
```

좋은 커스텀 훅은 코드의 호출을 제한 하며 선언적으로 해준다.
예를 들어, useChatRoom은 오직 챗 룸을 연결하고 useImpressionLog은 인상 로그를 애널리틱스에 보낸다.
만약 당신의 커스텀 훅이 제한적이지 않고 추상적이면, 추후에 더 많은 문제를 유발한다.

### Custom Hooks help you migrate to better patterns

이펙트는 탈출구로: 리액트에서 벗어나고 싶을 경우에 사용해야 하며 더 이상 빌트인 해결책이 없을 시 사용해야한다.
시간이 지남에 따라, 리액트 팀의 목표는 이펙트의 수를 줄이고 더 구체적인 제공을 통해 구체적인 문제를 해결하려 한다.
이펙트를 커스텀 훅으로 감싸는 것은 그때까지 당신의 코드를 업그레이드 해준다.

이전의 예로 돌아오자.

```jsx
import { useState, useEffect } from "react";

export function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener("online", handleOnline);
    window.addEventListener("offline", handleOffline);
    return () => {
      window.removeEventListener("online", handleOnline);
      window.removeEventListener("offline", handleOffline);
    };
  }, []);
  return isOnline;
}
```

위의 예에서 useOnlineStatus은 useState와 useEffect로 구현되었다.
하지만, 이것은 베스트 해결책이 아니다. 몇 개의 생각하지 못한 예외 사례가 있다.
예를 들어, 이미 네트워크가 꺼진 경우 컴포넌트 마운트시 isOnline이 true이기에 맞지 않는다.
당신은 navigator.onLine api를 사용해 할 수 있지만, 이것은 초기 html을 만드는 서버에서 동작하지 않는다.
결과적으로 이 코드는 개선되어야 한다.

운이 좋게, 리액트 18은 useSyncExternalStore라는 api를 제공해 이런 문제를 해결한다.
useOnlineStatus가 새로운 api를 사용해 다시 작성된 모습니다.

```jsx
import { useSyncExternalStore } from "react";

function subscribe(callback) {
  window.addEventListener("online", callback);
  window.addEventListener("offline", callback);
  return () => {
    window.removeEventListener("online", callback);
    window.removeEventListener("offline", callback);
  };
}

export function useOnlineStatus() {
  return useSyncExternalStore(
    subscribe,
    () => navigator.onLine, // How to get the value on the client
    () => true // How to get the value on the server
  );
}
```

당신은 컴포넌트에서 마이그레이션 할 필요 없다.

이것이 이펙트로 커스텀 훅을 감싼 이점이다:

1. 당신은 데이터의 흐름을 이펙트로 부터 또는 이펙트로 명시적으로 만들었다.
2. 당신은 컴포넌트에 이펙트의 구현 방식이 아닌 의도에 초점을 맞춘다.
3. 리액트가 새 기능을 추가하면, 당신은 컴포넌트의 변경없이 이펙트를 제거할 수 있다.

디자인 시스템과 비슷하게, 앱의 컴포넌트에서 커스텀 훅으로 관용구를 추출하는 것이 도움이 될 것이다.
이것은 당신의 컴포넌트 코드를 의도에 초점을 맞추게 할 수 있고 로우한 이펙트 작성을 피하게 해준다.
많은 좋은 커스텀 훅이 리액트 커뮤니티에서 유지된다.

#### DEEP DIVE - Will React provide any built-in solution for data fetching?

우리는 세부 사항을 작업 중이며, 미래에 데이터 페칭을 다음과 같이 할 수 있을 것이다.

```jsx
import { use } from 'react'; // Not available yet!

function ShippingForm({ country }) {
  const cities = use(fetch(`/api/cities?country=${country}`));
  const [city, setCity] = useState(null);
  const areas = city ? use(fetch(`/api/areas?city=${city}`)) : null;
  // ...
```

useData와 같이 커스텀 훅으로 작성한 경우 로우하게 이펙트를 수동으로 작성한 것 보다 마이그레이션할 변경 사항이 적다.
하지만, 로우하게 이펙트를 작성하는 것은 잘 동작하므로 그렇게 하고싶으면 작성해도 된다.

### There is more than one way to do it

페이드인 애니메이션을 requestAnimationFrame API를 사용해 처음부터 구현한다고 치자.
애니메이션 루프를 위한 이펙트를 작성할 것이다.
애니메이션의 각 프레임에서, 돔 노드의 투명값을 1일 될때까지 유지해야한다.

```jsx
import { useState, useEffect, useRef } from "react";

function Welcome() {
  const ref = useRef(null);

  useEffect(() => {
    const duration = 1000;
    const node = ref.current;

    let startTime = performance.now();
    let frameId = null;

    function onFrame(now) {
      const timePassed = now - startTime;
      const progress = Math.min(timePassed / duration, 1);
      onProgress(progress);
      if (progress < 1) {
        // We still have more frames to paint
        frameId = requestAnimationFrame(onFrame);
      }
    }

    function onProgress(progress) {
      node.style.opacity = progress;
    }

    function start() {
      onProgress(0);
      startTime = performance.now();
      frameId = requestAnimationFrame(onFrame);
    }

    function stop() {
      cancelAnimationFrame(frameId);
      startTime = null;
      frameId = null;
    }

    start();
    return () => stop();
  }, []);

  return (
    <h1 className="welcome" ref={ref}>
      Welcome
    </h1>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>{show ? "Remove" : "Show"}</button>
      <hr />
      {show && <Welcome />}
    </>
  );
}
```

컴포넌트의 가독성을 위해, 당신은 useFadeIn 커스텀 훅으로 추출할 수 있다.

```jsx
import { useState, useEffect, useRef } from "react";
import { useFadeIn } from "./useFadeIn.js";

function Welcome() {
  const ref = useRef(null);

  useFadeIn(ref, 1000);

  return (
    <h1 className="welcome" ref={ref}>
      Welcome
    </h1>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>{show ? "Remove" : "Show"}</button>
      <hr />
      {show && <Welcome />}
    </>
  );
}
```

useFadeIn을 그대로 유지할 수 있지만, 좀 더 리팩토링을 해야한다. 예를 들어, 애니메이션 루프를 useAnimationLoop 훅으로 추출할 수 있다.

```jsx
import { useState, useEffect } from "react";
import { experimental_useEffectEvent as useEffectEvent } from "react";

export function useFadeIn(ref, duration) {
  const [isRunning, setIsRunning] = useState(true);

  useAnimationLoop(isRunning, (timePassed) => {
    const progress = Math.min(timePassed / duration, 1);
    ref.current.style.opacity = progress;
    if (progress === 1) {
      setIsRunning(false);
    }
  });
}

function useAnimationLoop(isRunning, drawFrame) {
  const onFrame = useEffectEvent(drawFrame);

  useEffect(() => {
    if (!isRunning) {
      return;
    }

    const startTime = performance.now();
    let frameId = null;

    function tick(now) {
      const timePassed = now - startTime;
      onFrame(timePassed);
      frameId = requestAnimationFrame(tick);
    }

    tick();
    return () => cancelAnimationFrame(frameId);
  }, [isRunning]);
}
```

하지만, 당신은 이렇게 할 필요 없다. 일반 함수와 마찬가지로 코드의 다른 부분 사이에 경계를 그릴 위치를 결정한다.
당신은 아주 다른 접근을 할 수 있다. 이펙트에 로직을 두는 대신, 자바스크립트 클래스에 둘 수 있다.

이펙트는 리액트의 외부 시스템과 연결하게 해준다. 이펙트의 조정이 더 많이 필요할수록 위의 예처럼 효과와 이펙트에서 해당 로직을 완전히 추출하는 것이 더 합리적이다.
그러면 추출한 코드가 외부 시스템이 된다.
이것은 이펙트가 더 간결하게 해주며 오직 리액트의 외부 시스템에 메세지를 보내도록 한다.

위의 예는 페이드인 로직을 자바스크립트로 작성한 것이다. 하지만, css 애니메이션으로 작성하는 것이 더 효율적이다.

```jsx
import { useState, useEffect } from "react";
import "./welcome.css";

function Welcome() {
  return <h1 className="welcome">Welcome</h1>;
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>{show ? "Remove" : "Show"}</button>
      <hr />
      {show && <Welcome />}
    </>
  );
}
```

때로는, 당신은 훅도 필요 없다.

## Recap

- 커스텀 훅은 컴포넌트 간 로직을 공유하게 해준다.
- 커스텀 훅은 반드시 use로 시작한 후 대문자로 시작해야한다.
- 커스텀 훅은 상태 로직만을 공유하고, 상태는 아니다.
- 당신은 반응 값을 다른 훅에 전달해 상태를 업데이트할 수 있다.
- 모든 훅은 컴포넌트가 리랜더 되면 재실행된다.
- 컴포넌트 코드와 같이 커스텀 훅은 순수해야한다.
- 커스텀 훅에서 받은 이벤트 핸들러는 이펙트 이벤트로 감싼다.
- useMount와 같은 훅을 생성하지 마라. 그들의 목적을 구체적으로 유지해라.
- 코드의 경계를 선택하는 방법과 위치는 사용자에 달렸다.

---

## What I Learned

커스텀 훅을 작성하는 것을 조금 어려워 했었어서 커스텀 훅을 작성하는 것에 사실 최근 익숙해졌다.
확실히 커스텀 훅을 사용하면, 어떻게 구현했는지 보다 어떤 의도인지 더 명확히 보여줄 수 있는 것 같다.
또 다양한 컴포넌트에서 로직을 재사용할 수 있는 것이 효율적인 것 같다.

커스텀 훅을 사용한다고 해서 상태를 공유하는 것이 아니고, 그 상태 로직을 공유하는 것이다.
