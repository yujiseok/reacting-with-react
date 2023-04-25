- [Responding to Events](#responding-to-events)
  - [Adding event handlers](#adding-event-handlers)
  - [Pitfall](#pitfall)
  - [Reading props in event handlers](#reading-props-in-event-handlers)
  - [Passing event handlers as props](#passing-event-handlers-as-props)
  - [Naming event handler props](#naming-event-handler-props)
  - [Event propagation](#event-propagation)
  - [Stopping propagation](#stopping-propagation)
  - [DEEP DIVE Capture phase events](#deep-dive-capture-phase-events)
  - [Passing handlers as alternative to propagation](#passing-handlers-as-alternative-to-propagation)
  - [Preventing default behavior](#preventing-default-behavior)
  - [Can event handlers have side effects?](#can-event-handlers-have-side-effects)
  - [Recap](#recap)
  - [What I Learned](#what-i-learned)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

# Responding to Events

리액트는 JSX에 이벤트 핸들러를 추가할 수 있게 해줍니다.
이벤트 핸들러는 인터렉션의 반응으로 트리거 되는 함수입니다.

## Adding event handlers

이벤트 핸들러를 추가하려면, 정의된 함수를 적합한 JSX에 프랍으로 넘겨줘야합니다.
예를 들어, 아무것도 안하는 버튼이 있습니다.

```jsx
export default function Button() {
  return <button>I don't do anything</button>;
}
```

클릭시 메시지가 보이도록 할 수 있습니다.

1. `handleClick`이라는 함수를 `Button` 컴포넌트에 선언합니다.
2. 함수 안에서 로직을 구현합니다.
3. `onClick={handleClick}`를 `<button>`에 추가합니다.

```jsx
export default function Button() {
  function handleClick() {
    alert("You clicked me!");
  }

  return <button onClick={handleClick}>Click me</button>;
}
```

`handleClick` 함수를 만들어 `<button>`에 프랍으로 넘겨주었습니다. `handleClick`는 이벤트 핸들러입니다.
이벤트 핸들러

- 컴포넌트 안에서 정의됩니다.
- `handle`로 시작하고, 이벤트 이름을 따릅니다.

컨벤션적으로 `handle`로 시작해서 이벤트 이름을 따릅니다. 당신은 종종 `onClick={handleClick}`, `onMouseEnter={handleMouseEnter}`와 같은 것을 볼 것입니다.

인라인으로 이벤트 핸들러를 정의할 수 있습니다.

```jsx
<button onClick={function handleClick() {
  alert('You clicked me!');
}}>

<button onClick={() => {
  alert('You clicked me!');
}}>
```

## Pitfall

이벤트 핸들러로 넘겨지는 함수는 호출이 아닌, 넘겨져야한다. 예를 들어

|   passing a function (correct)   |   calling a function (incorrect)   |
| :------------------------------: | :--------------------------------: |
| `<button onClick={handleClick}>` | `<button onClick={handleClick()}>` |

두 차이는 미묘합니다. 첫 번째 예의, `handleClick`은 이벤트 핸들러로 넘겨집니다.
이것은 리액트가 유저가 버튼을 클릭했을 시에만 함수를 호출하도록 합니다.

두 번째 예에서, `()`는 렌더링시 함수를 호출 해버립니다. 이것은 `{}`안의 자바스크립트는 즉시 호출되기 때문입니다.

인라인으로 코드를 작성할 때 역시 같은 함정이 보입니다.

|      passing a function (correct)       |  calling a function (incorrect)   |
| :-------------------------------------: | :-------------------------------: |
| `<button onClick={() => alert('...')}>` | `<button onClick={alert('...')}>` |

아래와 같은 인라인 코드는 클릭시 호출되는 것이 아닌 매 렌더시 호출됩니다.

```jsx
// This alert fires when the component renders, not when clicked!
<button onClick={alert('You clicked me!')}>
```

만약 인라인으로 이벤트 핸들러를 정의하고 싶다면, 익명 함수로 감싸야합니다.

```jsx
<button onClick={() => alert('You clicked me!')}>
```

매 렌더시 호출하는 것이 아닌, 나중에 호출되는 함수를 만듭니다.

## Reading props in event handlers

이벤트 핸들러들은 컴포넌트 안에서 선언되므로, 컴포넌트의 프랍에 접근할 수 있습니다.
프랍으로 받은 `message`를 알럿으로 보여주는 컴포넌트가 있습니다.

```jsx
function AlertButton({ message, children }) {
  return <button onClick={() => alert(message)}>{children}</button>;
}

export default function Toolbar() {
  return (
    <div>
      <AlertButton message="Playing!">Play Movie</AlertButton>
      <AlertButton message="Uploading!">Upload Image</AlertButton>
    </div>
  );
}
```

이것은 두 버튼이 서로 다른 메시지를 보여주게 해줍니다.

## Passing event handlers as props

자주 당신은 부모 컴포넌트에서 자식 컴포넌트의 이벤트 핸들러를 특정하고 싶을 수 있습니다. 버튼을 예로, 버튼 컴포넌트를 어디서 사용하는 지에따라 다른 함수를 호출하고 싶을 수 있습니다.

이걸 하기 위해, 이벤트 핸들러를 프랍으로 넘겨줍니다.

```jsx
function Button({ onClick, children }) {
  return <button onClick={onClick}>{children}</button>;
}

function PlayButton({ movieName }) {
  function handlePlayClick() {
    alert(`Playing ${movieName}!`);
  }

  return <Button onClick={handlePlayClick}>Play "{movieName}"</Button>;
}

function UploadButton() {
  return <Button onClick={() => alert("Uploading!")}>Upload Image</Button>;
}

export default function Toolbar() {
  return (
    <div>
      <PlayButton movieName="Kiki's Delivery Service" />
      <UploadButton />
    </div>
  );
}
```

툴바 컴포넌트는 플레이버튼과 업로드 버튼을 렌더합니다.

- 플레이버튼은 handlePlayClick를 onClick 프랍으로 버튼 컴포넌트에 넘겨줍니다.
- 업로드버튼은 `() => alert('Uploading!')`를 onClick 프랍으로 버튼 컴포넌트에 넘겨줍니다.

최종적으로, 당신의 버튼 컴포넌트는 onClick 프랍을 받습니다. 이 컴포넌트는 즉시 빌트인 버튼에 프랍을 `onClick={onClick}` 형태로 넘겨줍니다.
이것은 리액트에게 넘겨 받은 함수를 클릭시 호출하도록 합니다.

만약 디자인 시스템을 사용하면, 버튼 컴포넌트는 스타일은 포함하지만 특정 행동은 포함하지 않습니다. 대신 플레이버튼, 업로드버튼은 이벤트 핸들러를 넘겨 받습니다.

## Naming event handler props

빌트인 컴포넌트 버튼이나 div는 오직 브라우저 이벤트만 지원합니다.
하지만 고유의 컴포넌트를 만들시, 이벤트 핸들러를 원하는 이름으로 지을 수 있습니다.

컨벤션으로 이벤트 핸들러 프랍은 on으로 시작합니다.
예를들어, 버튼 컴포넌트의 `onClick` 프랍은 `onSmash`라고 호출될 수 있습니다.

```jsx
function Button({ onSmash, children }) {
  return <button onClick={onSmash}>{children}</button>;
}

export default function App() {
  return (
    <div>
      <Button onSmash={() => alert("Playing!")}>Play Movie</Button>
      <Button onSmash={() => alert("Uploading!")}>Upload Image</Button>
    </div>
  );
}
```

위의 예에서 `<button onClick={onSmash}>` 브라우저의 버튼은 onClick 프랍이 필요합니다. 하지만, 커스텀 버튼 컴포넌트의 프랍 이름은 당신에게 달렸습니다.

여러 인터렉션을 지원해야한다면, 앱별 상호작용에 따른 이벤트 핸들러의 이름을 지을 수 있습니다. 예를 들어, 툴바 컴포넌트는 onPlayMovie와 onUploadImage 이벤트 핸들러를 받습니다.

```jsx
export default function App() {
  return (
    <Toolbar
      onPlayMovie={() => alert("Playing!")}
      onUploadImage={() => alert("Uploading!")}
    />
  );
}

function Toolbar({ onPlayMovie, onUploadImage }) {
  return (
    <div>
      <Button onClick={onPlayMovie}>Play Movie</Button>
      <Button onClick={onUploadImage}>Upload Image</Button>
    </div>
  );
}

function Button({ onClick, children }) {
  return <button onClick={onClick}>{children}</button>;
}
```

앱 컴포넌트는 툴바 컴포넌트의 onPlayMovie와 onUploadImage가 무슨 역을 하는 지 몰라도 됩니다.
이것은 툴바의 구현 세부 정보입니다. 툴바는 onClick 핸들러를 버튼 컴포넌트에 내려줍니다. 하지만 이것은 추후에 키보드 단축키로 호출 될 수 있습니다.
프랍의 이름을 앱별 상호작용에 맞게 짓는 것은 사용에 유연함을 줍니다.

## Event propagation

이벤트 핸들러는 모든 자식 컴포넌트에서 잡힙니다. 우리는 이것을 이벤트 bubbles 혹은 propagates라고 합니다. 이벤트가 시작되는 곳에서 부터 트리 위로 올라옵니다.

div는 두 개의 버튼을 포합합니다. 각 div는 각각의 onClick 핸들러를 갖습니다.

```jsx
export default function Toolbar() {
  return (
    <div
      className="Toolbar"
      onClick={() => {
        alert("You clicked on the toolbar!");
      }}
    >
      <button onClick={() => alert("Playing!")}>Play Movie</button>
      <button onClick={() => alert("Uploading!")}>Upload Image</button>
    </div>
  );
}
```

두 버튼을 클릭하면 버튼의 onClick가 먼저 실행되고 부모 div의 onClick가 실행됩니다.
즉, 두 메시지가 보입니다. 툴바를 클릭하면 div의 onClick만 실행됩니다.

> Pitfall
> onScroll 이벤트(JSX 태그에 부착한)를 제외한 리액트의 모든 이벤트는 propagate 됩니다.

## Stopping propagation

이벤트 핸들러들은 오직 인자로 이벤트 객체를 받습니다. 컨벤션으로 보통 e라고 불리며 이벤트를 뜻합니다.
이벤트의 정보를 읽기 위해 이벤트 객체를 사용할 수 있습니다.

이벤트 객체는 이벤트 전파를 막을 수 있게 해줍니다. 부모 컴포넌트까지 전파를 막기 위해선, `e.stopPropagation()`를 추가합니다.

```jsx
function Button({ onClick, children }) {
  return (
    <button
      onClick={(e) => {
        e.stopPropagation();
        onClick();
      }}
    >
      {children}
    </button>
  );
}

export default function Toolbar() {
  return (
    <div
      className="Toolbar"
      onClick={() => {
        alert("You clicked on the toolbar!");
      }}
    >
      <Button onClick={() => alert("Playing!")}>Play Movie</Button>
      <Button onClick={() => alert("Uploading!")}>Upload Image</Button>
    </div>
  );
}
```

버튼을 클릭 했을 시 :

1. 리액트는 `onClick` 핸들러를 호출하고 <button>에 넘겨줍니다.
2. Button 컴포넌트에서 정의된 핸들러는 다음을 따릅니다.
   - `e.stopPropagation()`를 호출, 이벤트의 버블링을 멈춥니다.
   - Toolbar 컴포넌트에서 넘겨받은 프랍인 `onClick` 함수를 호출합니다.
3. Toolbar 컴포넌트내에서 정의된 함수는 버튼의 alert를 표시합니다.
4. 전파가 멈추면 부모 div의 `onClick` 핸들러는 실행되지 않습니다.

`e.stopPropagation()`의 결과로, 버튼을 클릭하면 하나의 alert만 표시됩니다.
버튼을 클릭하는 것은 툴바를 클릭하는 것과 다르므로, 전파를 막는 것이 UI적으로 맞습니다.

## DEEP DIVE Capture phase events

아주 드문 경우에, 자식 요소의 모든 이벤트를 잡아야하는 경우가 있습니다. 예를 들어 전파 로직에 상관없이 모든 클릭에대한 로그를 원할 경우 `Capture`를 이벤트 이름에 추가해 사용할 수 있습니다.

```jsx
<div
  onClickCapture={() => {
    /* this runs first */
  }}
>
  <button onClick={(e) => e.stopPropagation()} />
  <button onClick={(e) => e.stopPropagation()} />
</div>
```

전파는 3 단계에 거쳐 일어납니다.

1. 아래로 전파되며, 모든 `onClickCapture` 핸들러를 호출합니다.
2. 클릭된 요소의 `onClick` 핸들러를 실행합니다.
3. 위로 올라가며, 모든 `onClick` 핸들러를 호출합니다.

캡쳐 이벤트는 라우터나 애널리틱스에 유용하나, 앱 코드에서는 사용하지말아야합니다.

## Passing handlers as alternative to propagation

클릭 핸들러가 한줄의 코드를 실행하고 프랍으로 받은 `onClick`을 실행하는 것을 생각해 보세요.

당신은 부모 컴포넌트가 `onClick` 핸들러를 호출하기 전에 더 많은 코드를 추가할 수 있습니다.
이런 패턴은 대체 전파를 제공합니다. 이것은 자식 컴포넌트가 이벤트를 다루는 것을 허용하며, 동시에 부모 컴포넌트에서 특정한 행동을 할 수 있게 해줍니다.
전파와 다르게, 이것은 자동적으로 일어나지 않습니다. 하지만 이런 패턴의 장점은 코드의 추적을 명확히 할 수 있다는 점입니다.

만약 어떤 핸들러에서 전파가 일어났는지 추적하기 어렵다면, 이런 접근 방식을 시도해보세요.

## Preventing default behavior

몇 개의 브라우저 이벤트들은 기본적인 동작을 갖고 있습니다.
예를 들어 form의 서브밋 이벤트의 경우, 내부의 버튼이 클릭되면 브라우저를 새로고침합니다.

`e.preventDefault()`를 호출해 이벤트 객체에 이런 동작을 멈추게할 수 있습니다.

```jsx
export default function Signup() {
  return (
    <form
      onSubmit={(e) => {
        e.preventDefault();
        alert("Submitting!");
      }}
    >
      <input />
      <button>Send</button>
    </form>
  );
}
```

`e.stopPropagation()`와 `e.preventDefault()`를 혼동하지 마세요.
그들은 둘 다 유용하지만, 관련이 없습니다.

- e.stopPropagation() 호출되는 위치 상위의 이벤트를 멈춥니다.
- e.preventDefault() 브라우저 이벤트의 기본 동작을 막습니다.

## Can event handlers have side effects?

이벤트 핸들러들은 사이드 이펙트의 최적의 장소입니다.

렌더링 함수들과 다르게, 이벤트 핸들러는 순수할 필요가 없습니다.
그래서, 무언가 변경 되는데 최적의 장소입니다.
예를 들어, 타이핑에 따른 인풋 값 변경, 버튼 클릭에 따른 리스트 변경
하지만, 정보를 바꾸기 위해선, 먼저 저장할 필요가 있습니다. 리액트에서 이것을 상태라고 부릅니다.

## Recap

- 함수를 프랍으로 넘겨서 이벤트를 다룰 수 있습니다.
- 이벤트 핸들러는 넘겨져야하며, 호출되면 안됩니다!
- 이벤트 핸들러 함수의 경우 인라인 또는 따로 정의할 수 있습니다.
- 컴포넌트 안에서 정의된 이벤트 핸들러의 경우, 프랍에 접근할 수 있습니다.
- 고유의 이벤트 핸들러를 만들 수 있습니다.
- 이벤트는 위로 전파됩니다. `e.stopPropagation()`를 호출하여 방지할 수 있습니다.
- 이벤트들은 원하지 않는 기본 브라우저 행동을 갖습니다. `e.preventDefault()`를 호출하여 방지할 수 있습니다.
- 명시된 이벤트 핸들러를 자식 핸들러에서 호출하는 것은 전파의 좋은 대체가 됩니다.

---

## What I Learned

이벤트를 다루면서 이벤트 자체에 대한 공부는 생각보다 하지 않았다는 것을 문서를 읽으며 알게 되었다.
특히 캡처링과 전파에 대해서는 정확히 공부하도록 해야겠다.

출처

- https://react.dev/learn/responding-to-events
