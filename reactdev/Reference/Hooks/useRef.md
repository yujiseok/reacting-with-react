# useRef

`useRef`는 랜더에 필요하지 않는 값을 레퍼런스 하게 해주는 훅이다.

```jsx
const ref = useRef(initialValue);
```

## Reference

### useRef(initialValue)

`useRef`를 컴포넌트 최상단에 호출해 레프를 선언한다.

```jsx
import { useRef } from 'react';

function MyComponent() {
  const intervalRef = useRef(0);
  const inputRef = useRef(null);
  // ...
```

**Parameters**

- initialValue: 레프 객체의 초기 `current` 값. 어떤 타입도 될 수 있다. 이 인자는 초기 랜더 후 무시된다.

**Returns**

`useRef`는 하나의 프로퍼티를 갖는 객체를 리턴한다.

- current: 이것은 초기에는 초기값으로 설정된다. 후에 다른 값으로 설정할 수 있다. 만약 리액트의 jsx 노드에 레프값을 추가하면, 리액트는 current 프로퍼티로 설정한다.

다음 랜더시, 리액트는 같은 객체를 리턴한다.

**Caveats**

- ref.current 프로퍼티를 조작할 수 있다. 상태와 다르게, 레프는 뮤터블하다. 하지만, 랜더에 사용되는 객체를 갖는다면, 객체를 조작해선 안된다.
- ref.current를 변경해도, 리액트는 컴포넌트를 리랜더 하지 않는다. 리액트는 레프가 기본적인 객체이기에 변경을 감지하지 못한다.
- 초기 랜더를 제외한 랜더 중 ref.current를 읽거나 쓰지 말아야한다. 이것은 컴포넌트를 예측불가하게 한다.

## Usage

### Referencing a value with a ref

useRef를 컴포넌트 최상단에 호출해 레프를 선언해라.

```jsx
import { useRef } from 'react';

function Stopwatch() {
  const intervalRef = useRef(0);
  // ...
```

useRef는 초기값으로 설정된 current 프로퍼티를 갖는 레프 객체를 리턴한다.

다음 랜더시, useRef는 동일한 객체를 리턴한다. current 프로퍼티는 정보를 저장하고 나중에 읽을 수 있다. 이것은 상태를 떠올리게하지만, 중요한 차이가 있다.

레프를 변경하는 것은 리랜더를 일으키지 않는다. 즉, 레프는 컴포넌트의 시각적인 부분에 영향이 없는 것을 저장하는데 알맞다. 예를 들어, 인터벌 id를 저장해 접근할 경우 레프에 저장할 수 있다. 레프 내부의 값을 변경하기 위해선, current 프로퍼티를 수정할 수 있다.

```jsx
function handleStartClick() {
  const intervalId = setInterval(() => {
    // ...
  }, 1000);
  intervalRef.current = intervalId;
}
```

후에, 인터벌 id를 읽어서 인터벌을 정리할 수 있다.

```jsx
function handleStopClick() {
  const intervalId = intervalRef.current;
  clearInterval(intervalId);
}
```

레프를 사용하면, 다음을 보장한다.

- 리랜더 사이에 정보를 저장할 수 있다. (매 랜더시 리셋되는 정규 변수와 다르게)
- 변경이 리랜더를 일으키지 않는다. (리랜더를 유발하는 상태와는 다르게)
- 컴포넌트 내부에 지역 정보이다. (공유되는 변수와 다르게)

레프는 리랜더를 일으키지 않기에, 화면에 보여질 정보로는 맞지 않는다.

#### Pitfall

ref.current를 랜더 중에 읽거나 수정하지 말아라.

리액트는 컴포넌트 몸체가 순수함수로 동작하길 원한다.

- 만약 인풋이 같다면, 동일한 jsx를 리턴해야한다.
- 다른 순서나 다른 인자로 호출하는 것은 다른 호출에 영향을 주지않는다.

랜더 중에 레프를 읽고 수정하는 것은 위의 룰을 깬다.

```jsx
function MyComponent() {
  // ...
  // 🚩 Don't write a ref during rendering
  myRef.current = 123;
  // ...
  // 🚩 Don't read a ref during rendering
  return <h1>{myOtherRef.current}</h1>;
}
```

이벤트 핸들러나 이펙트에서 레프를 읽거나 수정할 수 있다.

```jsx
function MyComponent() {
  // ...
  useEffect(() => {
    // ✅ You can read or write refs in effects
    myRef.current = 123;
  });
  // ...
  function handleClick() {
    // ✅ You can read or write refs in event handlers
    doSomething(myOtherRef.current);
  }
  // ...
}
```

만약 랜더 중에 읽거나 수정을 원한다면 상태를 사용해라.

이런 룰을 어겨도, 컴포넌트는 여전히 잘 작동한다. 하지만, 리액트가 추가할 새로운 기능은 룰에 달려있다.

### Manipulating the DOM with a ref

돔을 조작하는 데 레프를 사용하는 것은 일반적이다.
리액트는 빌트인 지원이 존재한다.

먼저, 레프 객체를 null로 초기화해 선언한다.

```jsx
import { useRef } from 'react';

function MyComponent() {
  const inputRef = useRef(null);
  // ...
```

그리고, 조작하고 싶은 돔 노드에 레프 속성에 전달한다.

```jsx
// ...
return <input ref={inputRef} />;
```

리액트가 돔 노드를 생성하고 화면에 추가하였을 때, 리액트는 current 프로퍼티를 돔 노드로 설정한다. 이제 인풋 돔 노드에 접근해 focus() 같은 메서드에 접근할 수 있다.

```jsx
function handleClick() {
  inputRef.current.focus();
}
```

리액트는 노드가 화면에서 사라지면, current를 null로 설정한다.

### Avoiding recreating the ref contents

리액트는 초기 레프 값을 저장하고 다음 랜더 시 무시한다.

```jsx
function Video() {
  const playerRef = useRef(new VideoPlayer());
  // ...
```

비록 new VideoPlayer()의 결과가 초기 랜더에만 사용되어도, 매 랜더시 함수를 호출하는 것이다. 만약 비싼 객체를 생성하면 이것은 낭비이다.

문제를 풀기 위해, 레프를 다음과 같이 초기화할 수 있다.

```jsx
function Video() {
  const playerRef = useRef(null);
  if (playerRef.current === null) {
    playerRef.current = new VideoPlayer();
  }
  // ...
```

일반적으로 ref.current를 랜더 중에 읽고 수정하는 것은 허용되지 않는다.
하지만, 항상 같을 경우 문제가 되지 않는다. 또한, 조건문은 항상 초기화시에 이루어지기 때문에 예측 가능하다.

#### DEEP DIVE - How to avoid null checks when initializing useRef later

항상 null을 체크하기 싫다면, 다음 패턴을 사용할 수 있다.

```jsx
function Video() {
  const playerRef = useRef(null);

  function getPlayer() {
    if (playerRef.current !== null) {
      return playerRef.current;
    }
    const player = new VideoPlayer();
    playerRef.current = player;
    return player;
  }

  // ...
```

playerRef 자체는 널러블이다. 하지만, getPlayer()가 리턴하는 값이 null일 수 없다고 확신할 수 있다. getPlayer()를 이벤트 핸들러에서 사용해라.

## Troubleshooting

### I can’t get a ref to a custom component

레프를 당신의 컴포넌트에 전달하려고 할 때:

```jsx
const inputRef = useRef(null);

return <MyInput ref={inputRef} />;
```

다음과 같은 에러를 마주할 것이다.

> Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?

기본적으로, 당신의 컴포넌트는 내부의 돔 노드의 레프를 노출하지 않는다.

이것을 해결하기 위해, 레프를 얻을 컴포넌트를 찾는다.

```jsx
export default function MyInput({ value, onChange }) {
  return <input value={value} onChange={onChange} />;
}
```

그 후 forwardRef로 감싼다.

```jsx
import { forwardRef } from "react";

const MyInput = forwardRef(({ value, onChange }, ref) => {
  return <input value={value} onChange={onChange} ref={ref} />;
});

export default MyInput;
```

그러면 부모 컴포넌트에서 레프를 전달할 수 있다.

---

## What I Learned

레프와 상태의 차이를 명확히 알게 되었다.

리랜더가 일어나 화면에 반영되어야하는 경우 -> state
값이 변경되어도 리랜더가 일어나지 않고 값이 동일한 경우 -> ref
