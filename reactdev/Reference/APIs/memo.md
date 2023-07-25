# memo

memo는 컴포넌트의 프랍이 변경되지 않았을 경우 랜더를 무시하게 해준다.

```jsx
const MemoizedComponent = memo(SomeComponent, arePropsEqual?)
```

## Reference

### memo(Component, arePropsEqual?)

컴포넌트를 memo로 감싸 메모이즈드된 버전의 컴포넌트를 얻는다.
이 메모된 컴포넌트는 프랍이 변경되지 않으면, 리랜더되지 않는다.
하지만, 리액트는 여전히 래랜더할 것이다. 메모이제이션은 성능최적화로 보장되지 않는다.

```jsx
import { memo } from "react";

const SomeComponent = memo(function SomeComponent(props) {
  // ...
});
```

**Parameters**

- Component: 메모이제이션 하고 싶은 컴포넌트. 메모가 컴포넌트를 변경하지 않는다. 하지만 새롭게 메모된 컴포넌트를 반환한다. 유효한 리액트의 컴포넌트.
- optional arePropsEqual: 이전 프랍과 새로운 프랍 두 가지 인수를 받은 함수. 이전 프랍과 새로운 프랍이 같을 경우 true를 반환한다. 즉 같은 결과와 같은 방식으로 랜더 된다. 아닐경우 false를 반환한다. 보통 이 함수를 명시할 필요 없다. 기본적으로, 리액트는 각 프랍을 object.is로 비교한다.

**Returns**

메모는 새로운 리액트 컴포넌트를 반환한다. 프랍이 변경되지 않는한 리랜더가 되지 않는 다는 점을 제외하면, 메모에 제공된 컴포넌트와 동일하게 동작한다.

## Usage

### Skipping re-rendering when props are unchanged

리액트는 부모가 리랜더될 경우 컴포넌트가 리랜더 된다.
메모와 함께, 컴포넌트의 프랍이 같다면 리랜더를 하지 않을 수 있다.
이런 컴포넌트를 메모이제이션이된 컴포넌트라 한다.

컴포넌트를 메모하기 위해, 실제 컴포넌트를 메모로 감싼다.

```jsx
const Greeting = memo(function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
});

export default Greeting;
```

리액트 컴포넌트는 항상 순수한 랜더 로직을 가져야한다.
이것은 프랍, 상태, 컨텍스트가 변경되지 않았을 경우 항상 똑같은 결과를 반환해야하는 것을 뜻한다.
메모를 사용하면, 리액트에게 프랍이 변경되지 않았다면 리랜더를 하지 말도록 알려준다. 메모를 사용하더라도, 그 컴포넌트의 상태 혹은 컨텍스트가 변경되면 랜더된다.

예에서, 그리팅 컴포넌트가 네임이 변경될 때, 리랜더되는 것을 인지하고, 주소가 변경되었을 때는 아니라는 것을 인지해라.

```jsx
import { memo, useState } from "react";

export default function MyApp() {
  const [name, setName] = useState("");
  const [address, setAddress] = useState("");
  return (
    <>
      <label>
        Name{": "}
        <input value={name} onChange={(e) => setName(e.target.value)} />
      </label>
      <label>
        Address{": "}
        <input value={address} onChange={(e) => setAddress(e.target.value)} />
      </label>
      <Greeting name={name} />
    </>
  );
}

const Greeting = memo(function Greeting({ name }) {
  console.log("Greeting was rendered at", new Date().toLocaleTimeString());
  return (
    <h3>
      Hello{name && ", "}
      {name}!
    </h3>
  );
});
```

> Note
> 메모를 오직 성능 최적화로 사용해야한다. 만약 메모 없이 동작되지 않는 다면, 문제를 찾고 해결하는 것이 먼저다. 그 후 메모로 감싸 성능을 최적화해라.

### Updating a memoized component using state

컴포넌트가 메모 되어도, 자신의 상태가 변경되면 리랜더된다. 메모이제이션은 부모 컴포넌트로 프랍을 전달받은 경우만 해당된다.

```jsx
import { memo, useState } from "react";

export default function MyApp() {
  const [name, setName] = useState("");
  const [address, setAddress] = useState("");
  return (
    <>
      <label>
        Name{": "}
        <input value={name} onChange={(e) => setName(e.target.value)} />
      </label>
      <label>
        Address{": "}
        <input value={address} onChange={(e) => setAddress(e.target.value)} />
      </label>
      <Greeting name={name} />
    </>
  );
}

const Greeting = memo(function Greeting({ name }) {
  console.log("Greeting was rendered at", new Date().toLocaleTimeString());
  const [greeting, setGreeting] = useState("Hello");
  return (
    <>
      <h3>
        {greeting}
        {name && ", "}
        {name}!
      </h3>
      <GreetingSelector value={greeting} onChange={setGreeting} />
    </>
  );
});

function GreetingSelector({ value, onChange }) {
  return (
    <>
      <label>
        <input
          type="radio"
          checked={value === "Hello"}
          onChange={(e) => onChange("Hello")}
        />
        Regular greeting
      </label>
      <label>
        <input
          type="radio"
          checked={value === "Hello and welcome"}
          onChange={(e) => onChange("Hello and welcome")}
        />
        Enthusiastic greeting
      </label>
    </>
  );
}
```

만약 상태가 현재 값으로 설정되면, 리액트는 메모가 되어있지 않아도 리랜더를 무시할 것이다. 컴포넌트 함수가 여전히 호출되는 것을 볼 수 있으며, 결과는 버려진다.

### Updating a memoized component using a context

컴포넌트가 메모되어도, 컨텍스트가 변경되면 여전히 리랜더된다. 메모이제이션은 오직 부모에서 전달된 프랍에 해당된다.

```jsx
export default function MyApp() {
  const [theme, setTheme] = useState("dark");

  function handleClick() {
    setTheme(theme === "dark" ? "light" : "dark");
  }

  return (
    <ThemeContext.Provider value={theme}>
      <button onClick={handleClick}>Switch theme</button>
      <Greeting name="Taylor" />
    </ThemeContext.Provider>
  );
}

const Greeting = memo(function Greeting({ name }) {
  console.log("Greeting was rendered at", new Date().toLocaleTimeString());
  const theme = useContext(ThemeContext);
  return <h3 className={theme}>Hello, {name}!</h3>;
});
```

컨텍스트가 변경되었을 때만, 리랜더를 하고 싶다면, 컴포넌트를 두개로 나눠라. 외부 컴포넌트에서 컨텍스트 값을 메모해서 프랍으로 전달해라.

### Minimizing props changes

메모를 사용할 때, 어떤 프랍이든 얕은 비교로 동일하지 않을 경우 리랜더된다.
이것은 리액트가 모든 프랍을 object.is로 비교하는 것을 뜻한다. Object.is(3, 3)은 참이고, Object.is({}, {})는 거짓이다.

메모를 최대화하기 위해, 프랍의 변경을 최소화해라.
예를 들어, 만약 프랍이 객체일경우, useMemo를 사용해 객체를 매번 재생성하는 것을 피해라.

```jsx
function Page() {
  const [name, setName] = useState("Taylor");
  const [age, setAge] = useState(42);

  const person = useMemo(() => ({ name, age }), [name, age]);

  return <Profile person={person} />;
}

const Profile = memo(function Profile({ person }) {
  // ...
});
```

프랍의 변경을 최소화하는 더 좋은 방법은 필요한 정보만 받아들이는 것이다.
예를 들어, 전체 객체가 아닌 각각의 값으로 받아들여진다.

```jsx
function Page() {
  const [name, setName] = useState("Taylor");
  const [age, setAge] = useState(42);
  return <Profile name={name} age={age} />;
}

const Profile = memo(function Profile({ name, age }) {
  // ...
});
```

때로는 개별 값도 자주 변경되지 않는 값으로 예상될 수 있다.
예를 들어, 값 자체가 아니라 값을 통한 불리언 값을 받는 컴포넌트가 있다.

```jsx
function GroupsLanding({ person }) {
  const hasGroups = person.groups !== null;
  return <CallToAction hasGroups={hasGroups} />;
}

const CallToAction = memo(function CallToAction({ hasGroups }) {
  // ...
});
```

메모된 함수를 컴포넌트로 전달할 경우 컴포넌트 외부에 선언하거나, useCallback을 사용해 리랜더 중에 캐시해라.

### Specifying a custom comparison function

아주 드문 경우, 프랍 변경을 최소화 하거나 컴포넌트를 메모하는 것이 불가할 수 있다. 이런 경우, 기존에 리액트가 얕은 비교로 값을 판단하는 것 대신 커스텀 비교 함수를 전달할 수 있다. 이것은 메모의 두 번째 인자로 받아진다. 오직 새로운 프랍이 예전 프랍과 같을 경우에 참을 반환한다. 반대의 경우 거짓을 반환한다.

```jsx
const Chart = memo(function Chart({ dataPoints }) {
  // ...
}, arePropsEqual);

function arePropsEqual(oldProps, newProps) {
  return (
    oldProps.dataPoints.length === newProps.dataPoints.length &&
    oldProps.dataPoints.every((oldPoint, index) => {
      const newPoint = newProps.dataPoints[index];
      return oldPoint.x === newPoint.x && oldPoint.y === newPoint.y;
    })
  );
}
```

이것을 사용하면, 비교 함수가 컴포넌트 리랜더보다 빠르다는 것을 알 수 있다.

성능을 측정하기 위해선, 프로덕션 환경에서 해라.

#### Pitfall

만약 커스텀 arePropsEqual을 구현할 경우, 함수를 포함한 모든 프랍을 비교해야한다. 함수는 종종 부모의 프랍과 상태의 클로저다. 만약 oldProps.onClick !== newProps.onClick이 참일 경우, 컴포넌트가 여전히 온클릭 핸들러의 이전 프롭과 상태를 지켜본다는 것을 뜻하고 이것은 혼란스러운 버그를 유발한다.

arePropsEqual 내부에서는 100% 확신해도 깊은 비교를 사용하지 마라. 깊은 비교는 엄청나게 느리고 데이터 구조가 변경될 시 앱을 멈출 수 있다.

## Troubleshooting

### My component re-renders when a prop is an object, array, or function

리액트는 얕은 비교로 이전 프랍과 새로운 프랍을 비교한다. 즉, 각 새로운 프랍이 이전 프랍의 참조가 같다는 것이다.
만약 새로운 객체나 배열을 리랜더시 부모에서 생성한다면, 각 값이 같더라도 리액트는 변경된 것으로 간주한다. 동일하게, 새로운 함수를 부모 컴포넌트가 랜더시 생성한다면, 리액트는 동일한 정의라도 변경된 것으로 간주한다.
이것을 피하기 위해, 프랍을 단순화하고 부모에서 프랍을 메모해라.

---

## What I Learned

모든 컴포넌트를 메모할 필요는 없으며, 메모되어 있어도 프랍이 변경되는 경우 혹은 고유한 상태를 지녔을 경우에 리랜더 된다.
