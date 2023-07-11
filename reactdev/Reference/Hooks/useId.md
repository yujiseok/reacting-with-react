# useId

useId는 접근성 속성에 전달할 수 있는 고유한 아이디를 생성하는 훅이다.

```jsx
const id = useId();
```

## Reference

### useId()

useId를 컴포넌트 최상단에 호출해 고유한 아이디를 생성한다.

```jsx
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  // ...
```

**Parameters**
useId는 파라미터를 갖지 않는다.

**Returns**
useId는 호출된 컴포넌트에서 고유한 아이디를 리턴한다.

**Caveats**

- useId는 목록의 키로 사용되면 안된다. 키는 데이터에서 생성되어야 한다.

## Usage

### Generating unique IDs for accessibility attributes

useId를 컴포넌트 최상단에 호출해 고유한 아이디를 생성한다.

```jsx
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  // ...
```

generated ID를 다른 속성에 전달할 수 있다.

```jsx
<>
  <input type="password" aria-describedby={passwordHintId} />
  <p id={passwordHintId}>
</>
```

aria-describedby와 같은 html의 접근성 속성은 두 태그가 서로 연관이 있다는 것을 알려준다.
예를 들어, 요소를 다른 요소로 설명할 수 있다.

일반 html에서는 다음과 같이 사용된다.

```jsx
<label>
  Password:
  <input
    type="password"
    aria-describedby="password-hint"
  />
</label>
<p id="password-hint">
  The password should contain at least 18 characters
</p>
```

하지만, 하드코딩된 아이디는 리액트에 좋지 않다. 컴포넌트는 페이지에서 여러번 랜더 되는데, 아이디는 반드시 고유해야한다.
아이디를 하드코딩하는 대신, useId로 생성한다.

```jsx
import { useId } from "react";

function PasswordField() {
  const passwordHintId = useId();
  return (
    <>
      <label>
        Password:
        <input type="password" aria-describedby={passwordHintId} />
      </label>
      <p id={passwordHintId}>
        The password should contain at least 18 characters
      </p>
    </>
  );
}
```

이제 PasswordField가 화면에 여러번 등장하여도, 생성된 아이디로 충돌이 일어나지 않는다.

#### Pitfall

서버 랜더링에서, useId는 서버와 클라이언트 트리에 동일한 컴포넌트를 요한다. 만약 일치하지 않는다면, 아이디가 매치하지 않을 것이다.

#### DEEP DIVE - Why is useId better than an incrementing counter?

useId가 nextId++ 보다 나은 이유를 궁금할 것이다.

useId의 이점은 리액트가 서버 랜더시 동작을 보장한다는 것이다. 서버 랜더 시, 컴포넌트는 html을 생성한다. 후에, 클라이언트에서 생성된 html에 이벤트를 부착하는 하이드레이션을 진행한다. 하이드레이션이 동작하려면, 클라이언트 결과물이 서버 html과 맞아야한다.

이것은 카운터를 증가하는 것으로 클라이언트와 서버를 매치하는 것이 어렵기 때문이다. useId를 호출해, 하이드레이션을 보장하며, 클라이언트와 서버의 결과물이 매치될 것이다.

리액트 내부에서, useId는 컴포넌트 호출의 부모 경로에서 생성된다. 이것은, 만약 클라이언트와 서버 트리가 같다면, 부모 경로가 랜더 순서와 상관없이 매치하기 때문이다.

### Generating IDs for several related elements

만약 연관된 여러 요소에 아이디를 주고 싶을 경우, useId를 호출해 접두사로 사용할 수 있다.

```jsx
import { useId } from "react";

export default function Form() {
  const id = useId();
  return (
    <form>
      <label htmlFor={id + "-firstName"}>First Name:</label>
      <input id={id + "-firstName"} type="text" />
      <hr />
      <label htmlFor={id + "-lastName"}>Last Name:</label>
      <input id={id + "-lastName"} type="text" />
    </form>
  );
}
```

이것은 유일한 아이디가 필요한 모든 컴포넌트에서 useId를 호출하는 것을 방지한다.

### Specifying a shared prefix for all generated IDs

만약 하나의 페이지에 여러 독립적인 리액트 애플리케이션을 랜더 한다면, identifierPrefix 옵션을 createRoot또는 hydrateRoot 호출의 옵션으로 추가할 수 있다. 이것은 useId로 생성한 아이디가 절대 충돌하지 않는 것을 보장한다.

```jsx
import { createRoot } from "react-dom/client";
import App from "./App.js";
import "./styles.css";

const root1 = createRoot(document.getElementById("root1"), {
  identifierPrefix: "my-first-app-",
});
root1.render(<App />);

const root2 = createRoot(document.getElementById("root2"), {
  identifierPrefix: "my-second-app-",
});
root2.render(<App />);
```

---

## What I Learned

useId는 고유한 아이디 값을 생성해주는 훅이다.
배열의 키로 사용은 지양해야한다.
