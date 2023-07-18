# createContext

createContext는 컴포넌트에 제공하고 읽을 수 있는 컨텍스트를 만들 수 있게 해준다.

```js
const SomeContext = createContext(defaultValue);
```

## Reference

### createContext(defaultValue)

컴포넌트 외부에서 createContext를 호출해 컨텍스트를 생성한다.

```jsx
import { createContext } from "react";

const ThemeContext = createContext("light");
```

**Parameters**

- defaultValue: 컴포넌트 트리 상위에 매치되는 컨텍스트 프로바이더가 없을 경우 읽어들이는 값이다. 만약 의미있는 값이 없다면, null로 지정해라. 기본 값은 최후의 수단을 의미한다. 이것은 정적이며 변경되지 않는다.

**Returns**
createContext는 컨텍스트 객체를 반환한다.

컨텍스트 객체 자체로는 어떤 정보도 갖지 않는다.
이것은 다른 어떤 컴포넌트에서 읽거나 제공되는 것을 나타낸다.
일반적으로, SomeContext.Provider를 컴포넌트의 상위에 사용해, 컨텍스트 값을 정의하고 useContext(SomeContext)를 호출해 컴포넌트 내부에서 읽는다.
컨텍스트 객체는 몇 개의 속성을 갖는다:

- SomeContext.Provider는 컴포넌트에 컨텍스트 값을 제공한다.
- SomeContext.Consumer는 대체 및 드물게 컨텍스트 값을 읽기 위해 사용된다.

### SomeContext.Provider

컴포넌트를 컨텍스트 프로바이더로 감싸 그 내부 컴포넌트에서 사용되는 값을 지정한다.

```jsx
function App() {
  const [theme, setTheme] = useState("light");
  // ...
  return (
    <ThemeContext.Provider value={theme}>
      <Page />
    </ThemeContext.Provider>
  );
}
```

**Props**

- value: 얼마나 깊든지 상관 없이 컴포넌트가 읽게 하고 싶은 값. 컨텍스트 값은 어떤 타입도 가능하다. useContext(SomeContext)를 호출한 컴포넌트는 그 위에 있는 컨텍스트 프로바이더의 값을 받는다.
-

### SomeContext.Consumer

useContext가 존재하기 전 컨텍스트를 읽기 위한 예전 방식이다.

```jsx
function Button() {
  // 🟡 Legacy way (not recommended)
  return (
    <ThemeContext.Consumer>
      {(theme) => <button className={theme} />}
    </ThemeContext.Consumer>
  );
}
```

비록 오래 되었지만, 여전히 동작한다. 하지만, 컨텍스트를 새롭게 작성 했다면, useContext()를 사용해라.

```jsx
function Button() {
  // ✅ Recommended way
  const theme = useContext(ThemeContext);
  return <button className={theme} />;
}
```

**Props**

- children: 함수. useContext()과 동일한 알고리즘을 통해 컨텍스트 값이 결정된 결과를 반환해 랜더하는 함수이다. 리액트는 부모의 컴포넌트의 컨텍스트가 변경되면 함수를 재 실행한다.

## Usage

### Creating context

컨텍스트는 컴포넌트에 정보를 명시적인 프랍 전달 없이 깊이 까지 전달할 수 있게 해준다.

하나 혹은 여러개의 컨텍스트를 생성하기 위해 createContext를 컴포넌트 외부에서 호출한다.

```jsx
import { createContext } from "react";

const ThemeContext = createContext("light");
const AuthContext = createContext(null);
```

createContext는 컨텍스트 객체를 반환한다. 컴포넌트는 useContext()를 통해 컨텍스트를 읽을 수 있다.

```jsx
function Button() {
  const theme = useContext(ThemeContext);
  // ...
}

function Profile() {
  const currentUser = useContext(AuthContext);
  // ...
}
```

기본적으로, 그들은 컨텍스트를 생성했을 시 설정한 기본값을 받는다.
하지만, 기본값은 변하지 않기에 유용하지 않다.

컨텍스트는 다른 다양한 값을 컴포넌트에 제공할 수 있어 유용하다.

```jsx
function App() {
  const [theme, setTheme] = useState("dark");
  const [currentUser, setCurrentUser] = useState({ name: "Taylor" });

  // ...

  return (
    <ThemeContext.Provider value={theme}>
      <AuthContext.Provider value={currentUser}>
        <Page />
      </AuthContext.Provider>
    </ThemeContext.Provider>
  );
}
```

이제 페이지 컴포넌트와 그 내부의 컴포넌트들은, 얼마나 깊든지 상관없이 컨텍스트 값을 볼 수 있다.
만약 컨텍스트 값이 변경되면, 리액트는 컨텍스트의 값을 읽기 위해 리랜더한다.

### Importing and exporting context from a file

종종, 다른 파일에 있는 컴포넌트들이 동일한 컨텍스트에 접근해야할 때가 있다.
이것이 컨텍스트를 다른 파일로 분리하는 것이 일반적인 이유다.
export를 통해 컨텍스트를 다른 파일에서 사용할 수 있도록 한다.

```jsx
// Contexts.js
import { createContext } from "react";

export const ThemeContext = createContext("light");
export const AuthContext = createContext(null);
```

다른 파일에 선언된 컴포넌트는 import를 통해 컨텍스트를 읽거나 제공한다.

```jsx
// Button.js
import { ThemeContext } from "./Contexts.js";

function Button() {
  const theme = useContext(ThemeContext);
  // ...
}
```

```jsx
// App.js
import { ThemeContext, AuthContext } from "./Contexts.js";

function App() {
  // ...
  return (
    <ThemeContext.Provider value={theme}>
      <AuthContext.Provider value={currentUser}>
        <Page />
      </AuthContext.Provider>
    </ThemeContext.Provider>
  );
}
```

## Troubleshooting

### I can’t find a way to change the context value

기본 컨텍스트 값을 설정한 코드가 있다.

```jsx
const ThemeContext = createContext("light");
```

이 값은 절대 변하지 않는다. 리액트는 매치되는 프로바이더가 없을 경우 이 값을 사용한다.

컨텍스트를 변하게 하려면, 상태로 만들고 컴포넌트를 컨텍스트 프로바이더로 감싸라.

---

## What I Learned

컨텍스트를 변하게 하기 위해선, 상태로 만들어야 한다.
기본값으로 설정된 컨텍스트는 변경이 되지 않는다.
