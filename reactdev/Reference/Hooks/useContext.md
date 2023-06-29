# useContext

`useContext`는 컴포넌트에서 컨텍스트를 구독할 수 있게 해주는 훅이다.

```jsx
const value = useContext(SomeContext);
```

## Reference

### useContext(SomeContext)

`useContext`는 컴포넌트 최상위에 호출해 컨텍스트를 구독할 수 있다.

```jsx
import { useContext } from 'react';

function MyComponent() {
  const theme = useContext(ThemeContext);
  // ...
```

**Parameters**

- SomeContext: createContext를 통해 이전에 생성한 컨텍스트. 컨텍스트 자체는 정보를 갖지 않는다. 오직 정보를 제공하고 컴포넌트에서 읽을 수 있다.

**Returns**

`useContext`는 컨텍스트 값을 호출된 컴포넌트에서 리턴한다. 이것은 가까운 트리의 `SomeContext.Provider`에 전달된 `value`에 의해 결정된다. 이런 프로바이더가 없을 경우 createContext에 전달한 `defaultValue`를 반환한다. 리턴된 값은 항상 최신값이다. 리액트는 컨텍스트가 변경되면 컴포넌트를 자동으로 리랜더 한다.

**Caveats**

- `useContext()`의 호출은 동일한 컴포넌트에서 반환된 프로바이더의 영향을 받지 않는다. 해당 `<Context.Provider>`는 `useContext()`을 호출하는 상위 컴포넌트여야 한다.
- 리액트는 특정한 컨텍스트가 다른 값을 받을 경우 자동으로 자식을 리랜더한다. Object.is로 값을 비교한다. `memo`를 통한 리랜더 무시도 자식이 최신 컨텍스트 값을 받는 것을 막지 못한다.
- 만약 빌드 시스템이 중복된 모듈 아웃풋을 생성할 경우 컨텍스트가 고장날 수 있다. 컨텍스트를 통해 무언가 전달하는 것은 `SomeContext`와 컨텍스트를 읽는데 사용하는 `SomeContext`가 `===` 비교시 동일할 경우만 동작한다.

## Usage

### Passing data deeply into the tree

컴포넌트 최상단에 `useContext`를 호출해 컨텍스트를 구독한다.

```jsx
import { useContext } from 'react';

function Button() {
  const theme = useContext(ThemeContext);
  // ...
```

`useContext`는 컨텍스트에 전달한 컨텍스트 값을 반환한다. 컨텍스트 값을 결정하기 위해, 리액트는 컴포넌트 트리를 찾고 가장 가까운 컨텍스트 프로바이더를 찾는다.

버튼에 컨텍스트를 전달하기 위해, 부모 컴포넌트를 상응하는 컨텍스트 프로바이더로 감싼다.

```jsx
function MyPage() {
  return (
    <ThemeContext.Provider value="dark">
      <Form />
    </ThemeContext.Provider>
  );
}

function Form() {
  // ... renders buttons inside ...
}
```

프로바이더와 버튼 사이 얼마나 많은 계층이 있든 문제되지 않는다. 버튼이 폼 컴포넌트의 어디에 있든 `useContext(ThemeContext)`를 사용하면, `"dark"` 값을 얻을 수 있다.

#### Pitfall

`useContext()`는 항상 가장 가까운 프로바이더를 본다. 위쪽으로 찾으며 어떤 컴포넌트가 `useContext()`를 호출했는지 고려하지 않는다.

```jsx
import { createContext, useContext } from "react";

const ThemeContext = createContext(null);

export default function MyApp() {
  return (
    <ThemeContext.Provider value="dark">
      <Form />
    </ThemeContext.Provider>
  );
}

function Form() {
  return (
    <Panel title="Welcome">
      <Button>Sign up</Button>
      <Button>Log in</Button>
    </Panel>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = "panel-" + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  );
}

function Button({ children }) {
  const theme = useContext(ThemeContext);
  const className = "button-" + theme;
  return <button className={className}>{children}</button>;
}
```

### Updating data passed via context

자주, 당신은 컨텍스트가 변경되길 원할 수 있다. 컨텍스트를 변경하기 위해, 상태와 결합해라. 부모 컴포넌트에 상태를 선언하고 현재 상태를 프로바이더에 컨텍스트 값으로 전달해라.

```jsx
function MyPage() {
  const [theme, setTheme] = useState("dark");
  return (
    <ThemeContext.Provider value={theme}>
      <Form />
      <Button
        onClick={() => {
          setTheme("light");
        }}
      >
        Switch to light theme
      </Button>
    </ThemeContext.Provider>
  );
}
```

이제 프로바이더의 버튼은 현재 테마 값을 얻는다. 만약 setTheme로 프로바이더에 전달한 테마를 변경한다면, 모든 버튼 컴포넌트가 light 값으로 변경되고 리랜더된다.

### Specifying a fallback default value

리액트가 프로바이더의 특정한 컨텍스트를 부모 트리에서 찾지 못한다면, useContext로 반환 되는 컨텍스트 값은 컨텍스트를 만들때 전달한 기본값과 동일하다.

```jsx
const ThemeContext = createContext(null);
```

기본 값은 절대 변경되지 않는다. 컨텍스트를 변경하고 싶다면, 상태를 통해 구독해라.

null을 사용하는 대신, 좀 더 의미있는 값을 기본값으로 설정할 수 있다.

```jsx
const ThemeContext = createContext("light");
```

이것이 상응하는 프로바이더 없이 랜더되어도 문제를 일으키지 않는 이유다.
또 테스트에서 여러 프로바이더를 설정하지 않아도 컴포넌트가 잘 동작하게 도와준다.

아래 예에서 토글 테마 버튼은 컨텍스트 프로바이더의 밖이고 기본값이 라이트이기에 항상 라이트다. 기본 테마를 다크로 설정해보자.

```jsx
import { createContext, useContext, useState } from "react";

const ThemeContext = createContext("light");

export default function MyApp() {
  const [theme, setTheme] = useState("light");
  return (
    <>
      <ThemeContext.Provider value={theme}>
        <Form />
      </ThemeContext.Provider>
      <Button
        onClick={() => {
          setTheme(theme === "dark" ? "light" : "dark");
        }}
      >
        Toggle theme
      </Button>
    </>
  );
}

function Form({ children }) {
  return (
    <Panel title="Welcome">
      <Button>Sign up</Button>
      <Button>Log in</Button>
    </Panel>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = "panel-" + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  );
}

function Button({ children, onClick }) {
  const theme = useContext(ThemeContext);
  const className = "button-" + theme;
  return (
    <button className={className} onClick={onClick}>
      {children}
    </button>
  );
}
```

### Overriding context for a part of the tree

다른 값을 사용하는 프로바이더를 사용해 컨텍스트를 덮어 쓸 수 있다.

```jsx
<ThemeContext.Provider value="dark">
  ...
  <ThemeContext.Provider value="light">
    <Footer />
  </ThemeContext.Provider>
  ...
</ThemeContext.Provider>
```

원하는 만큼 프로바이더를 덮어 쓸 수 있다.

### Optimizing re-renders when passing objects and functions

객체와 함수를 포함한 어떤 값도 컨텍스트로 전달할 수 있다.

```jsx
function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  function login(response) {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }

  return (
    <AuthContext.Provider value={{ currentUser, login }}>
      <Page />
    </AuthContext.Provider>
  );
}
```

컨텍스트 값이 두 프로퍼티이고 하나는 함수를 갖는 자바스크립트 객체이다. 마이앱이 리랜더 될 때, 이것은 다른 객체의 다른 함수를 가리킨다.
그러므로 리액트는 useContext(AuthContext)가 실행되는 깊은 트리의 모든 컴포넌트를 리랜더한다.

작은 앱에서 이것은 문제가 되지 않는다. 하지만, 리랜더가 필요하지 않은 currentUser 변경되지 않은 데이터도 존재한다.
리액트가 그 사실을 활용하기 위해, 로그인 함수는 useCallback으로 객체는 useMemo로 감쌀 수 있다. 이것은 최적화이다.

```jsx
import { useCallback, useMemo } from "react";

function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  const login = useCallback((response) => {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }, []);

  const contextValue = useMemo(
    () => ({
      currentUser,
      login,
    }),
    [currentUser, login]
  );

  return (
    <AuthContext.Provider value={contextValue}>
      <Page />
    </AuthContext.Provider>
  );
}
```

만약 마이앱이 리랜더 되더라도, `useContext(AuthContext)`를 호출 하는 컴포넌트는 currentUser가 변경되지 않으면 리랜더 되지 않는다.

## Troubleshooting

### My component doesn’t see the value from my provider

이것이 일어나는 몇 가지 경우가 있다.

1. <SomeContext.Provider>를 useContext()가 호출되는 동일한 컴포넌트에 랜더. <SomeContext.Provider>를 useContext가 사용되는 상위로 옮겨라.
2. 컴포넌트를 <SomeContext.Provider>로 감싸는 것을 잊은 경우 혹은 다른 트리를 감싼 경우. 리액트 데브툴을 사용해 계층을 확인해라.
3. SomeContext 프로바이더의 제공 컴포넌트와 SomeContext를 읽어들이는 컴포넌트가 다른 객체를 읽을 경우 빌드 문제가 발생한다.

### I am always getting undefined from my context although the default value is different

프로바이더를 값 없이 트리에 전달한 경우

```jsx
// 🚩 Doesn't work: no value prop
<ThemeContext.Provider>
  <Button />
</ThemeContext.Provider>
```

값을 잊은 경우, 이것은 `value={undefined}`와 동일하다.

프랍의 이름을 잘못 작성했을 경우

```jsx
// 🚩 Doesn't work: prop should be called "value"
<ThemeContext.Provider theme={theme}>
  <Button />
</ThemeContext.Provider>
```

위의 오류를 해결하기 위해선 프랍을 value로 작성해라

```jsx
// ✅ Passing the value prop
<ThemeContext.Provider value={theme}>
  <Button />
</ThemeContext.Provider>
```

createContext로 생성된 기본값은 오직 상위에 맞는 프로바이더가 없을 경우 사용된다는 것을 명심해라.
만약 <SomeContext.Provider value={undefined}>가 상위에 있을 경우, useContext(SomeContext)는 undefined를 컨텍스트의 값으로 받는다.

---

## What I Learned

컨텍스트 역시 이전의 리듀서와 마찬가지로 나에게 익숙하지 않은 훅이었는데, 리듀서와 함께 사용해 상태 관리를 해보니 익숙해 졌다. 아마 컨테스트와 리듀서의 번역된 의미가 와닿지 않아서 그런것 같다.

단방향 데이터 흐름의 단점을 해결하는 아주 유용한 훅인것 같다.
