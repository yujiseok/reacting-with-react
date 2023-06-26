# useState

`useState`는 컴포넌트에 상태 변수를 추가해주는 리액트 훅이다.

```jsx
const [state, setState] = useState(initialState);
```

## Reference

### `useState(initialState)`

`useState`를 컴포넌트 최상단에 호출해 상태 변수를 선언할 수 있다.

```jsx
import { useState } from 'react';

function MyComponent() {
  const [age, setAge] = useState(28);
  const [name, setName] = useState('Taylor');
  const [todos, setTodos] = useState(() => createTodos());
  // ...
```

상태 변수의 컨벤션은 배열 구조분해를 사용해 `[something, setSomething]`와 같이 작성한다.

**Parameters**

- `initialState`: 상태에 설정하고 싶은 초기값. 어떠한 타입도 가능하며, 함수의 경우 특별한 기능을 한다. 함수 인자는 첫 랜더 후 무시된다.
  - 만약 초기 상태에 함수를 전달하면, 그것은 초기화 함수로 다뤄진다. 그것은 순수해야 하며, 어떤 인자도 받으면 안되고 어떤 타입이라도 반환해야한다. 리액트는 초기화 함수를 컴포넌트가 초기화될 때 호출하고, 리턴 값을 초기 상태로 저장한다.

**Returns**

`useState`는 정확히 두 값을 갖는 배열을 리턴한다.

1. 현재 상태. 처음 랜더 시 초기 상태와 매치된다.
2. 상태를 다른 값으로 변경하고 리랜더를 일으키는 set 함수

**Caveats**

- `useState`는 훅이기에 반드시 컴포넌트의 최상단 혹은 커스텀 훅에서 호출 되어야한다. 반복문 혹은 조건문에서 사용할 수 없다. 만약 필요하다면, 새로운 컴포넌트로 추출하고 상태를 옮겨라.
- 엄격 모드에서 리액트는 초기화 함수를 두 번 호출해 순수하지 않은지 확인한다. 이것은 개발 환경에서만 발생하며 배포시에는 효과가 없다. 만약 초기화 함수가 순수하다면, 동작에 영향을 주지 않고 하나의 호출은 무시된다.

### `set` functions, like `setSomething(nextState)`

useState에서 반환된 set 함수는 상태를 다른 값으로 변경하고 리랜더를 일으킨다.
당신은 값을 직접 추가할 수 있고, 이전 상태에서 계산하는 함수를 전달할 수 있다.

```jsx
const [name, setName] = useState('Edward');

function handleClick() {
  setName('Taylor');
  setAge(a => a + 1);
  // ...
```

**Parameters**

- `nextState`: 상태가 되어야할 값. 어떤 타입도 가능하지만, 함수의 경우 특별한 기능을 한다.
  - 함수를 다음 상태로 전달하면, 이것은 업데이터 함수로 다뤄진다. 이것은 순수해야하며, 오직 보류 상태를 인자로 받고 다음 상태를 반환 해야한다. 리액트는 업데이터 함수를 큐에 등록하고 컴포넌트를 리랜더한다. 다음 랜더 중에, 리액트는 이전 상태에서 업데이터 함수로 다음 상태를 계산한다.

**Returns**
set 함수는 값을 반환하지 않는다.

**Caveats**

- set 함수는 오직 다음 랜더의 상태 값만 변경한다. set 함수를 호출한 후 상태 변수를 읽으면 여전히 호출전에 화면에 있던 이전 값을 얻게 된다.
- 만약 새로운 값이 현재 상태와 같다면, `Object.is` 비교를 통해, 리액트는 컴포넌트와 자식의 리랜더를 무시한다. 이것은 최적화다. 비록 몇 상황에서 자식을 무시하기 전에 호출될 수 있는데, 이것은 당신의 코드에 영향을 주지않는다.
- 리액트는 배치를 통해 상태를 변경한다. 이것은 모든 이벤트 핸들러가 동작하고 set 함수가 호출 되었을 때 화면을 변경한다. 이것은 하나의 이벤트에 대한 여러 리랜더를 막는다. 드문 경우에 강제로 리액트에 화면을 일찍 변경할 경우(돔에 접근) `flushSync`를 사용할 수 있다.
- 랜더링 중에 set 함수를 호출하는 것은 현재 랜더하는 컴포넌트 내에서만 허용된다. 리액튼 출력을 버리고 즉시 새 상태로 리랜더를 시도한다. 이 패턴은 드물게 요구되지만, 이전 랜더에서 정보를 저장할 수 있다.
- 엄격 모드에서, 리액트는 순수한지 검사하기 위해 업데이터 함수를 두 번 호출한다. 이것은 개발 환경에서만 발생하며 배포시에는 효과가 없다. 만약 초기화 함수가 순수하다면, 동작에 영향을 주지 않고 하나의 호출은 무시된다.

## Usage

### Adding state to a component

`useState`를 컴포넌트의 최상단에 호출해 한 개 이상의 상태 변수를 선언할 수 있다.

```jsx
import { useState } from 'react';

function MyComponent() {
  const [age, setAge] = useState(42);
  const [name, setName] = useState('Taylor');
  // ...
```

상태 변수의 컨벤션은 배열 구조분해를 사용해 `[something, setSomething]`와 같이 작성한다.

`useState`는 정확히 두 값을 갖는 배열을 리턴한다.

1. 제공된 초기 상태로 설정된 현재 상태
2. 상호작용을 통해 상태를 다른 값으로 변경하는 set 함수

화면에 보일 것을 변경하기 위해, 다음 상태와 함께 set 함수를 호출한다.

```jsx
function handleClick() {
  setName("Robin");
}
```

리액트는 다음 상태를 저장한 후, 새로운 값으로 화면을 랜더하고 UI를 변경한다.

#### Pitfall

set 함수를 호출 하는 것은 이미 코드가 실행되었어도 현재 상태를 변경하지 않는다.

```jsx
function handleClick() {
  setName("Robin");
  console.log(name); // Still "Taylor"!
}
```

이것은 오직 다음 랜더부터 시작하여 `useState`가 반환하는 상태만 영향을 미친다.

### Updating state based on the previous state

나이가 42세라고 생각해보자. 이 핸들러는 `setAge(age + 1)`를 세 번 호출한다:

```jsx
function handleClick() {
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
}
```

하지만, 클릭 후, 나이는 45가 아닌 43이다. 이것은 set 함수는 이미 실행된 코드의 상태 변수를 변경하지 않기에 그렇다.
즉 각 `setAge(age + 1)`는 `setAge(43)`이 된다.

이 문제를 해결하기 위해, `setAge`에 다음 상태 대신 업데이터 함수를 제공할 수 있다.

```jsx
function handleClick() {
  setAge((a) => a + 1); // setAge(42 => 43)
  setAge((a) => a + 1); // setAge(43 => 44)
  setAge((a) => a + 1); // setAge(44 => 45)
}
```

`a => a + 1`이 업데이터 함수이다. 이것은 보류 상태를 받아 다음 상태로 계산한다.

리액트는 업데이터 함수를 큐에 둔다. 그 후, 다음 랜더시, 같은 순서로 호출한다.

1. `a => a + 1`는 42를 받고 43을 다음 상태로 반환
2. `a => a + 1`는 43을 받고 44를 다음 상태로 반환
3. `a => a + 1`는 44를 받고 45를 다음 상태로 반환

다른 큐에 추가된 변경이 없다면, 리액트는 45를 현재 상태로 저장한다.

컨벤션으로 대기 중인 상태를 상태 변수의 첫 글자를 따서(a / age) 작성한다. 하지만, 더 명시적으로 `prevAge`와 같이 작성할 수 있다.

개발 환경 시 순수한지 판별하기 위해 두 번 호출한다.

#### DEEP DIVE - Is using an updater always preferred?

만약 이전 상태에서 계산할 것이 있다면 `setAge(a => a + 1)`와 같이 코드를 작성하는 것이 항상 추천된다고 들었을 것이다.
그것에 문제는 없지만, 항상 필요한 것은 아니다.

대부분의 상황에서, 두 접근은 차이점이 없다. 리액트는 항상 의도적인 유저의 액션에 대하 상태 변수가 변경되도록 항상 확인한다.
이것은 불필요한 age를 이벤트 핸들러에서 보는 것이 문제가 없다는 뜻이다.

하지만, 만약 같은 이벤트에 여러 변경이 있을 경우, 업데이터는 도움이 된다. 그들은 상태 변수에 접근하는 것이 어려울 때도 도움을 준다.

만약 장황한 문법 보다 일관성을 원한다면, 이전 상태에서 업데이트 하는 업데이터를 사용하는 것이 합리적이다.
만약 다른 상태 변수를 합쳐 계산을 한다면, 리듀서를 사용하는 것을 추천한다.

### Updating objects and arrays in state

객체나 배열을 상태에 둘 수 있다. 리액트에서, 상태는 읽기 전용으로, 뮤테이트 대신 대체해야한다. 예를 들어, form 상태가 있는데, 뮤테이트 하면 안된다:

```jsx
// 🚩 Don't mutate an object in state like this:
form.firstName = "Taylor";
```

대신에, 새로운 객체를 만들어 대체해라.

```jsx
// ✅ Replace state with a new object
setForm({
  ...form,
  firstName: "Taylor",
});
```

### Avoiding recreating the initial state

리액튼 초기 상태를 한 번 설정한 후 다음 랜더부터 무시한다.

```jsx
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos());
  // ...
```

비록 `createInitialTodos`의 결과가 오직 초기 랜더에 사용 되지만, 매 랜더에 이 함수를 호출 하는 것이다.
큰 배열이나 비싼 계산이 필요할 경우 낭비가 될 수 있다.

초기화 함수를 전달해 문제를 해결할 수 있다.

```jsx
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos);
  // ...
```

`createInitialTodos` 함수를 호출 하지 않고 그 자체를 전달하는 것을 알 수 있다.
`useState`에 함수를 전달하면, 리액트는 오직 초기화할 시에 호출한다.

### Resetting state with a key

리스트를 랜더할 때 key 속성을 자주 마주 했을 것이다. 하지만, 이것은 또한 다른 용도가 있다.

컴포넌트에 다른 key를 전달해 컴포넌트의 상태를 초기화할 수 있다. 예에서 리셋 버튼은 version 상태 변수를 변경하는데, 그것은 폼 컴포넌트의 key에 전달된다. key가 변경되면 리액트는 폼 컴포넌트를 다시 만들고 초기화 된다.

```jsx
import { useState } from "react";

export default function App() {
  const [version, setVersion] = useState(0);

  function handleReset() {
    setVersion(version + 1);
  }

  return (
    <>
      <button onClick={handleReset}>Reset</button>
      <Form key={version} />
    </>
  );
}

function Form() {
  const [name, setName] = useState("Taylor");

  return (
    <>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <p>Hello, {name}.</p>
    </>
  );
}
```

### Storing information from previous renders

일반적으로, 상태를 이벤트 핸들러로 변경한다. 하지만, 드문 경우 랜더의 응답으로 상태를 변경하고 싶을 수 있다. - 예를 들어, 프랍이 변경되었을 때 상태를 변경하고 싶을 수 있다.

대부분의 경우 이것은 불필요

- 프랍이나 상태로 계산을 할 수 있다면 중복된 상태를 제거해라. 만약 계산이 너무 자주 된다면 useMemo 훅을 사용해라.
- 컴포넌트 트리의 상태를 초기화하길 원한다면, 다른 key를 컴포넌트에 제공
- 가능하다면, 관련이 있는 모든 상태를 이벤트 핸들러에서 처리

이런 적용 말고 드문 경우, 랜더 도중 set 함수를 호출해 랜더에 기반한 값으로 상태로 변경하는 패턴이 있다.

넘겨 받은 count를 랜더하는 카운트레이블 컴포넌트가 있다.

```jsx
export default function CountLabel({ count }) {
  return <h1>{count}</h1>;
}
```

마지막 변경으로부터 증가와 하락을 보여주길 원한다고 생각하자. count 프랍은 이것을 알려주지 않는다 - 이전 값에서 추적해야한다.
prevCount 상태 변수를 추가해 추적한다. trend라는 상태 변수를 추가해 상승과 하락을 소유하도록 한다.
prevCount와 count가 같을 경우를 제외해 비교하여, prevCount와 trend를 변경한다. 이제 현재 count 프랍과 이전 랜더에서의 변화를 보여줄 수 있다.

```jsx
import { useState } from "react";

export default function CountLabel({ count }) {
  const [prevCount, setPrevCount] = useState(count);
  const [trend, setTrend] = useState(null);
  if (prevCount !== count) {
    setPrevCount(count);
    setTrend(count > prevCount ? "increasing" : "decreasing");
  }
  return (
    <>
      <h1>{count}</h1>
      {trend && <p>The count is {trend}</p>}
    </>
  );
}
```

set 함수를 랜더 중에 호출 하고 싶다면, 이것은 `prevCount !== count`와 같은 조건과 `setPrevCount(count)`와 같이 조건 내부에서 호출 되어야한다. 아닐 경우 컴포넌트가 무한으로 랜더 될 것이다. 또한, 현재 랜더되는 컴포넌트만 변경할 수 있다.
다른 컴포넌트에서 set 함수를 호출하는 것은 에러를 유발한다. 최종적으로, set 호출은 뮤테이션 없이 변경하므로, 순수 함수의 룰을 위반하는 것도 아니다.

이런 패턴은 이해하기 어렵고 기피 된다. 하지만, 이펙트에서 상태를 변경하는 것보다 낫다. set 함수를 랜더 중에 호출하면, 리액트는 컴포넌트가 리턴 문으로 종료된 직후와 자식을 랜더링하기 전에 컴포넌트를 리랜더한다. 이것이 자식이 두 번 랜더되지 않는 이유다.
컴포넌트의 나머지 함수는 실행될 것이다. 만약 조건문이 훅 호출 아래에있다면, return을 추가해 랜더의 재시작을 일찍 시작할 수 있다.

## Troubleshooting

### I’ve updated the state, but logging gives me the old value

set 함수를 호출하는 것은 실행되는 코드의 상태를 변경하지 않는다.

```jsx
function handleClick() {
  console.log(count); // 0

  setCount(count + 1); // Request a re-render with 1
  console.log(count); // Still 0!

  setTimeout(() => {
    console.log(count); // Also 0!
  }, 5000);
}
```

이것은 상태가 스냅샷처럼 동작하기 때문이다. 상태를 변경하는 것은 다른 랜더와 새로운 상태 값을 요청하지만, 이미 실행되는 이벤트 핸들러의 count에 영향을 주지 않느다.

다음 상태를 사용하고 싶다면, set 함수에 전달하기 전에 저장할 수 있다.

```jsx
const nextCount = count + 1;
setCount(nextCount);

console.log(count); // 0
console.log(nextCount); // 1
```

### I’ve updated the state, but the screen doesn’t update

리액트는 `Object.is` 비교로 이전 상태와 다음 상태가 같다면 변경을 무시한다.
이것은 객체나 배열을 직접 비교할 때 일어난다.

```jsx
obj.x = 10; // 🚩 Wrong: mutating existing object
setObj(obj); // 🚩 Doesn't do anything
```

기존에 존재한 obj를 뮤테이트 하고 setObj에 전달하면, 리액트는 변경을 무시한다. 이것을 피하기 위해 반드시 객체나 배열을 뮤테이트 하는 것이 아닌 대체해야한다.

```jsx
// ✅ Correct: creating a new object
setObj({
  ...obj,
  x: 10,
});
```

### I’m getting an error: “Too many re-renders”

`Too many re-renders. React limits the number of renders to prevent an infinite loop.`와 같은 에러를 볼 수 있다.
이것은, 랜더 중에 상태를 설정 했기에 루프를 마주하는 것이다: 랜더 -> 상태 설정 -> 랜더 -> 상태 설정
이것은 이벤트 핸들러를 지정할 때 발생한다.

```jsx
// 🚩 Wrong: calls the handler during render
return <button onClick={handleClick()}>Click me</button>;

// ✅ Correct: passes down the event handler
return <button onClick={handleClick}>Click me</button>;

// ✅ Correct: passes down an inline function
return <button onClick={(e) => handleClick(e)}>Click me</button>;
```

만약 에러의 원인을 찾지 못한다면, 콘솔에서 자바스크립트 스택을 본 후 지정된 set 함수를 찾아라.

### My initializer or updater function runs twice

엄격 모드에서, 리액트는 함수를 두 번 호출한다:

```jsx
function TodoList() {
  // This component function will run twice for every render.

  const [todos, setTodos] = useState(() => {
    // This initializer function will run twice during initialization.
    return createTodos();
  });

  function handleClick() {
    setTodos(prevTodos => {
      // This updater function will run twice for every click.
      return [...prevTodos, createTodo()];
    });
  }
  // ...
```

이것은 예측된 것이고 에러를 일으키지 않는다.

이것은 개발 환경 행동으로 당신의 컴포넌트를 순수하게 해준다. 리액트는 하나의 결과를 사용하고 나머지 호출은 무시한다.
당신의 컴포넌트, 초기화 함수 그리고 업데이터 함수는 순수하므로, 이것은 로직에 영향을 끼치지 않는다. 하지만 순수하지 않을 경우, 이것은 도움을 준다.

예를 들어, 이것은 순수하지 않은 배열을 뮤테이트하는 함수이다.

```jsx
etTodos((prevTodos) => {
  // 🚩 Mistake: mutating state
  prevTodos.push(createTodo());
});
```

리액트가 업데이터 함수를 두 번 호출하기에, 당신은 투두가 두 번 추가된 것을 알 수 있어 문제를 알 수 있다.
배열을 대체해 문제를 해결할 수 있다.

```jsx
setTodos((prevTodos) => {
  // ✅ Correct: replacing with new state
  return [...prevTodos, createTodo()];
});
```

이제 업데이터 함수는 순수하므로, 여러번 실행 되어도 다른 행동을 취하지 않는다. 이것이 리액트가 두 번 호출하는 것이 실수를 찾아주는 이유다.
오직 컴포넌트, 초기화 함수 그리고 업데이터 함수만이 순수해야한다.
이벤트 핸들러는 순수할 필요가 없기에, 리액트는 이벤트 핸들러를 두 번 호출하지 않는다.

### I’m trying to set state to a function, but it gets called instead

당신은 함수를 상태에 다음과 같이 전달할 수 없다.

```jsx
const [fn, setFn] = useState(someFunction);

function handleClick() {
  setFn(someOtherFunction);
}
```

함수를 전달 했기에, 리액트는 `someFunction`을 초기화 함수로, `someOtherFunction`을 업데이터 함수로 추측하므로, 그들을 호출해 결과를 저장하려 한다. 실제로 함수를 저장하기 위해선, `() =>`을 추가해야한다. 그러면 리액트는 함수를 저장한다.

```jsx
const [fn, setFn] = useState(() => someFunction);

function handleClick() {
  setFn(() => someOtherFunction);
}
```

---

## What I Learned

리액트를 처음 사용했을 때, 실행중인 코드에서 상태가 왜 안바뀌는지 의문을 가졌던 경험이 생각이 났다.
상태는 스냅샷이고 다음 랜더의 상태로만 접근이 가능하다는 것을 알았다. 또 배치와 업데이터 함수에 헷갈리는 부분들이 있었는데, 좀 더 명확해졌다.
