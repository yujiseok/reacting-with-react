# useReducer

`useReducer`는 컴포넌트에 리듀서를 추가해주는 훅이다.

```js
const [state, dispatch] = useReducer(reducer, initialArg, init?)
```

## Reference

### useReducer(reducer, initialArg, init?)

`useReducer`를 컴포넌트 최상위에 호출에 리듀서로 상태를 관리한다.

```jsx
import { useReducer } from 'react';

function reducer(state, action) {
  // ...
}

function MyComponent() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });
  // ...
```

**Parameters**

- reducer: 상태가 어떻게 변경될지 지정하는 함수. 반드시 순수해야 하며, 상태와 액션을 인자로 받아, 다음 상태를 리턴해야한다. 상태와 액션은 어떤 타입도 가능하다.
- initialArg: 초기 상태가 계산되는 값. 어떤 타입도 가능하다. init 인자에 따라 어떻게 계산될지 정해진다.
- optional init: 초기화 함수로 초기 상태를 리턴한다. 만약 특정되지 않으면, initialArg가 초기 상태로 설정된다. 즉 초기 상태는 init의 호출로 결정된다.

**Returns**

`useReducer`는 두 값의 배열을 반환한다.

1. 현재 상태. 첫 랜더시 init또는 initialArg로 설정된다.
2. 디스패치 함수는 상태를 변경하고 리랜더를 일으킨다.

**Caveats**

- `useReducer`는 훅이므로, 오직 컴포넌트의 최상단에서 호출되어야한다. 반복문 또는 조건문에서 호출할 수 없다. 만약 필요하다면 컴포넌트로 추출한 후, 상태를 옮겨라.
- 엄격 모드시, 리액트는 리듀서와 초기화를 두 번 호출해 순수한지 판별한다. 이것은 개발 환경 동작으로 프로덕션에는 영향을 끼치지않는다. 만약 리듀서와 초기화가 순수하면 로직에 영향을 끼치지 않는다. 하나의 호출은 무시된다.

### dispatch function

`useReducer`에서 반환되는 디스패치 함수는 상태를 다른 값으로 변경하고 리랜더를 일으킨다.
디스패치 함수에 액션을 전달해줘야한다.

```jsx
const [state, dispatch] = useReducer(reducer, { age: 42 });

function handleClick() {
  dispatch({ type: 'incremented_age' });
  // ...
```

리액트는 다음 상태를 현재 상태와 디스패치에 전달한 액션을 통해 리듀서 함수 호출의 결과로 설정할 것이다.

**Parameters**

- action: 유저에 의해서 동작하는 액션. 어떤 타입도 될 수 있다. 컨벤션으로 액션은 다른 프로퍼티와 구별할 수 있는 `type` 프로퍼티가 있는 객체다.

**Returns**

디스패치는 리턴 값이 없다.

**Caveats**

- 디스패치는 오직 다음 랜더의 상태 변수를 변경한다. 만약 디스패치 함수 호출 후 상태 변수를 읽으려 하면, 여전히 예전 값을 얻을 것이다.
- 만약 새로운 값이 현재 값과 동일하다면, Object.is 비교로 리액트는 컴포넌트와 자식의 리랜더를 무시한다. 이것은 최적화이다. 리액트는 결과를 무시하기 전에 컴포넌트를 호출하지만, 이것은 코드에 영향을 주지 않는다.
- 리액트는 상태를 배치 변경한다. 이것은 모든 이벤트 핸들러가 동작하고 set 함수를 호출 했을 시 화면이 변경되게한다. 이것은 하나의 이벤트 동안 여러 리랜더를 막는다. 드문 경우 화면에 빨리 보여주고 싶은 경우 flushSync를 사용해라.

## Usage

### Adding a reducer to a component

`useReducer`를 컴포넌트 최상위에 호출에 리듀서로 상태를 관리한다.

```jsx
import { useReducer } from 'react';

function reducer(state, action) {
  // ...
}

function MyComponent() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });
  // ...
```

`useReducer`는 두 아이템을 갖는 배열을 반환한다.

1. 상태 변수의 현재 상태, 초기 상태로 설정된
2. 상호작용의 결과를 변경하는 디스패치 함수

화면을 변경하기 위해, 유저가 무엇을 하는지 나타내는 액션과 디스패치를 호출해라.

```jsx
function handleClick() {
  dispatch({ type: "incremented_age" });
}
```

리액트는 현재 상태와 액션을 랜더 함수에 전달한다. 리듀서는 계산을 하고 다음 상태를 반환한다.
리액트는 다음 상태를 저장하고, 컴포넌트와 함께 랜더하고, UI를 변경한다.

```jsx
import { useReducer } from "react";

function reducer(state, action) {
  if (action.type === "incremented_age") {
    return {
      age: state.age + 1,
    };
  }
  throw Error("Unknown action.");
}

export default function Counter() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });

  return (
    <>
      <button
        onClick={() => {
          dispatch({ type: "incremented_age" });
        }}
      >
        Increment age
      </button>
      <p>Hello! You are {state.age}.</p>
    </>
  );
}
```

`useReducer`는 `useState`와 매우 비슷하지만, 이벤트 핸들러의 로직을 컴포넌트 밖으로 분리해준다.

### Writing the reducer function

리듀서 함수는 다음과 같이 선언된다.

```jsx
function reducer(state, action) {
  // ...
}
```

그 후 다음 상태를 리턴하는 코드를 채운다. 컨벤션으로 스위치문을 사용하는 것이 일반적이다.
스위치의 각 케이스는, 계산되고 다음 상태를 리턴한다.

```jsx
function reducer(state, action) {
  switch (action.type) {
    case "incremented_age": {
      return {
        name: state.name,
        age: state.age + 1,
      };
    }
    case "changed_name": {
      return {
        name: action.nextName,
        age: state.age,
      };
    }
  }
  throw Error("Unknown action: " + action.type);
}
```

액션은 어떤 모습도 가능하다. 컨벤션으로, 액션을 분별하기 위한 `type` 프로퍼티가 있는 객체를 전달하는 것이 일반적이다.
이것은 리듀서가 다음 상태를 계산하기 위한 최소한의 정보가 포함 될 수 있다.

```jsx
function Form() {
  const [state, dispatch] = useReducer(reducer, { name: 'Taylor', age: 42 });

  function handleButtonClick() {
    dispatch({ type: 'incremented_age' });
  }

  function handleInputChange(e) {
    dispatch({
      type: 'changed_name',
      nextName: e.target.value
    });
  }
  // ...
```

액션 타입의 이름은 컴포넌트에서 지역적이다. 각 액션은 데이터의 여러 변경을 유발해도, 하나의 상호작용을 설명해야 한다. 상태의 모습은 임의이지만, 보통은 객체 또는 배열이다.

#### Pitfall

상태는 읽기 전용이다. 객체나 배열을 수정하지 말아라.

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // 🚩 Don't mutate an object in state like this:
      state.age = state.age + 1;
      return state;
    }
```

대신, 리듀서에서 항상 새로운 객체를 반환 해라

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // ✅ Instead, return a new object
      return {
        ...state,
        age: state.age + 1
      };
    }
```

### Avoiding recreating the initial state

리액트는 초기 상태를 처음 저장하고 다음 랜더시에는 무시한다.

```jsx
function createInitialState(username) {
  // ...
}

function TodoList({ username }) {
  const [state, dispatch] = useReducer(reducer, createInitialState(username));
  // ...
```

`createInitialState(username)` 함수의 결과가 초기 랜더에만 사용되지만, 매 랜더시 함수를 호출한다.
이것은 큰 배열을 생성할때나 비싼 연산을 할 시 낭비적이다.

이것을 해결하기 위해 초기화 함수를 세번째 인자로 `useReducer`에 전달할 수 있다.

```jsx
function createInitialState(username) {
  // ...
}

function TodoList({ username }) {
  const [state, dispatch] = useReducer(reducer, username, createInitialState);
  // ...
```

createInitialState를 호출 하는 것이 아닌, 그 자체를 전달하는 것을 알 수 있다. 이 방법으로 초기 상태는 초기화 이후에 재 생성되지 않는다.

createInitialState은 username인자를 필요로 한다. 만약 초기화가 초기 상태를 계산하는 데 어떤 정보도 필요하지 않다면 두번째 인자로 `null`을 전달해라.

## Troubleshooting

### I’ve dispatched an action, but logging gives me the old state value

디스패치 함수 호출이 실행 중인 코드의 상태를 변경하지 않는다.

```jsx
function handleClick() {
  console.log(state.age); // 42

  dispatch({ type: "incremented_age" }); // Request a re-render with 43
  console.log(state.age); // Still 42!

  setTimeout(() => {
    console.log(state.age); // Also 42!
  }, 5000);
}
```

이것은 상태가 스냅샷처럼 동작하기 때문이. 상태를 변경해 새로운 값으로 랜더가 일어나는 것은, 이미 이벤트 핸들러에서 실행되는 상태 자바스크립트 변수에 영향을 끼치지 않는다.

만약 당신이 다음 상태를 얻고 싶다면, 리듀서를 직접 호출해 계산할 수 있다.

```jsx
const action = { type: "incremented_age" };
dispatch(action);

const nextState = reducer(state, action);
console.log(state); // { age: 42 }
console.log(nextState); // { age: 43 }
```

### I’ve dispatched an action, but the screen doesn’t update

리액트는 다음 상태가 이전 상태가 같다면, Object.is 비교를 통해 무시한다.
이것은 객체나 배열을 직접 변경할 때 나타난다.

```jsx
function reducer(state, action) {
  switch (action.type) {
    case "incremented_age": {
      // 🚩 Wrong: mutating existing object
      state.age++;
      return state;
    }
    case "changed_name": {
      // 🚩 Wrong: mutating existing object
      state.name = action.nextName;
      return state;
    }
    // ...
  }
}
```

이미 존재하는 뮤테이트된 객체 상태를 반환해, 리액트는 변경을 무시한다. 이것을 고치기 위해 뮤테이트 하지않고 변경해야 한다.

```jsx
function reducer(state, action) {
  switch (action.type) {
    case "incremented_age": {
      // ✅ Correct: creating a new object
      return {
        ...state,
        age: state.age + 1,
      };
    }
    case "changed_name": {
      // ✅ Correct: creating a new object
      return {
        ...state,
        name: action.nextName,
      };
    }
    // ...
  }
}
```

### A part of my reducer state becomes undefined after dispatching

새로운 상태를 반환할 시, 모든 케이스 브랜치가 이전에 존재했던 필드를 복사하도록 해라

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      return {
        ...state, // Don't forget this!
        age: state.age + 1
      };
    }
    // ...
```

`...state`가 없다면, 반환되는 다음 상태는 오직 `age` 필드만 포함한다.

### My entire reducer state becomes undefined after dispatching

만약 상태가 undefined가 될 시, 케이스에서 상태를 리턴하는 것을 잊었을 수 있다. 또는 액션 타입이 케이스 상태에 부합하지 않을 수 있다.
왜 그런지 알기 위해, 스위치에서 에러를 던지도록 해라

```jsx
function reducer(state, action) {
  switch (action.type) {
    case "incremented_age": {
      // ...
    }
    case "edited_name": {
      // ...
    }
  }
  throw Error("Unknown action: " + action.type);
}
```

정적 타입 체커인 타입스크립트를 사용해 이런 문제를 잡을 수 있다.

### I’m getting an error: “Too many re-renders”

`Too many re-renders. React limits the number of renders to prevent an infinite loop.`와 같은 에러를 얻을 수 있다.
일반적으로, 이것은 무조건적으로 액션 디스패칭을 랜더 중에 하는 것을 뜻한다: 랜더 디스패치 계속
매우 자주, 이것은 이벤트 핸들러를 잘못 작성해 발생한다.

```jsx
// 🚩 Wrong: calls the handler during render
return <button onClick={handleClick()}>Click me</button>;

// ✅ Correct: passes down the event handler
return <button onClick={handleClick}>Click me</button>;

// ✅ Correct: passes down an inline function
return <button onClick={(e) => handleClick(e)}>Click me</button>;
```

만약 에러를 찾지 못하면 콘솔을 확인해라.

### My reducer or initializer function runs twice

엄격 모드시, 리액트는 리듀서와 초기화 함수를 두 번 호출한다. 이것은 코드를 망가트리지 않는다.

이 개발 환경 동작은 컴포넌트가 순수함을 유지하도록 도와준다. 리액트는 결과의 하나를 사용하고, 나머지는 무시한다.
리듀서와 초기화 함수가 순수하면 로직에 영향을 끼치지 않는다. 하지만, 만약 순수하지 않다면, 문제를 인지하도록 도와준다.

```jsx
function reducer(state, action) {
  switch (action.type) {
    case "added_todo": {
      // 🚩 Mistake: mutating state
      state.todos.push({ id: nextId++, text: action.text });
      return state;
    }
    // ...
  }
}
```

리액트가 리듀서 함수를 두 번 호출하기에, 투두가 두 번 추가되는 것을 볼 수 있으며, 문제라는 것을 알 수 있다.
뮤테이트 대신 대체해 문제를 해결할 수 있다.

```jsx
function reducer(state, action) {
  switch (action.type) {
    case "added_todo": {
      // ✅ Correct: replacing with new state
      return {
        ...state,
        todos: [...state.todos, { id: nextId++, text: action.text }],
      };
    }
    // ...
  }
}
```

이제 리듀서 함수는 순수하므로, 여러번 호출이 다른 결과를 초래하지 않는다. 이것이 리액트가 두 번 호출해 문제를 도와주는 이유다.
오직 컴포넌트, 초기화 함수, 리듀서 함수가 순수해야한다. 이벤트 핸들러는 순수할 필요가 없으므로, 두 번 호출되지 않는다.

---

## What I Learned

useReducer를 처음 공부할 당시에는 정말 너무 어렵고 이해가 안되었는데, 프로젝트에 몇 번 사용하고 문서를 읽어보니 제법 이해를 하게 된 것 같다.

특히 로직을 이벤트 핸들러에서 분리해 컴포넌트 외부에 작성할 수 있는 것이 관심사 분리의 관점에서 매우 좋은 것 같다.
또 하나의 리듀서에서 특정 상태에 관련된 업데이트 동작을 처리할 수 있는 것이 장점인 것 같다.
