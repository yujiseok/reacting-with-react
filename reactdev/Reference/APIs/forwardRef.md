# forwardRef

forwardRef는 부모 컴포넌트에 돔 노드를 레프와 함께 드러내게 해준다.

```jsx
const SomeComponent = forwardRef(render);
```

## Reference

### forwardRef(render)

forwardRef()를 호출해 자식 컴포넌트에 레프와 포워드를 받도록 한다.

```jsx
import { forwardRef } from "react";

const MyInput = forwardRef(function MyInput(props, ref) {
  // ...
});
```

**Parameters**

- render: 컴포넌트의 랜더 함수. 리액트는 부모로 전달받은 프랍과 레프와 함께 함수를 호출한다. jsx가 리턴하는 값이 컴포넌트의 아웃풋이다.

**Returns**

forwardRef는 jsx로 랜더 되는 리액트 컴포넌트를 반환한다. 일반적인 함수로 정의된 리액트 컴포넌트와 다르게 forwardRef로 리턴되는 컴포넌트는 레프 프랍을 받는 것이 가능하다.

### render function

forwardRef는 랜더 함수를 인자로 받는다. 리액트는 이것을 프랍과 레프와 함께 호출한다.

```jsx
const MyInput = forwardRef(function MyInput(props, ref) {
  return (
    <label>
      {props.label}
      <input ref={ref} />
    </label>
  );
});
```

**Parameters**

- props: 부모 컴포넌트에서 전달 받은 프랍.
- ref: 부모 컴포너트에서 전달 받은 레프 속성. 부모 컴포넌트가 레프를 전달하지 않을 경우 null이다. 다른 컴포넌트에서 받는 레프를 전달해야하며 useImperativeHandle에 전달해야한다.

**Returns**

forwardRef는 jsx로 랜더 되는 리액트 컴포넌트를 반환한다. 일반적인 함수로 정의된 리액트 컴포넌트와 다르게 forwardRef로 리턴되는 컴포넌트는 레프 프랍을 받는 것이 가능하다.

## Usage

### Exposing a DOM node to the parent component

기본적으로, 각 컴포넌트의 돔 노드는 프라이빗하다.
하지만, 가끔 부모 컴포넌트에 노출하는 것은 유용하다 - 포커싱.
이것을 하기 위해, 컴포넌트를 forwardRef()로 감싼다:

```jsx
import { forwardRef } from "react";

const MyInput = forwardRef(function MyInput(props, ref) {
  const { label, ...otherProps } = props;
  return (
    <label>
      {label}
      <input {...otherProps} />
    </label>
  );
});
```

프랍 다음으로 레프를 받을 것이다. 이것을 노출하고 싶은 돔 노드에 전달한다.

```jsx
import { forwardRef } from "react";

const MyInput = forwardRef(function MyInput(props, ref) {
  const { label, ...otherProps } = props;
  return (
    <label>
      {label}
      <input {...otherProps} ref={ref} />
    </label>
  );
});
```

이것은 부모 폼 컴포넌트가 마이인풋이 노출한 인풋의 돔 노드에 접근할 수 있다는 것을 뜻한다.

```jsx
function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
  }

  return (
    <form>
      <MyInput label="Enter your name:" ref={ref} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

폼 컴포넌트가 레프를 마이인풋에 넘긴다. 마이인풋은 인풋 태그에 레프를 포워드한다. 그 결과 폼 컴포넌트는 인풋 돔 노드에 접근할 수 있고 focus()를 호출할 수 있다.

돔 노드를 노출하는 것은 컴포넌트 내부를 변경하는 것을 어렵게한다. 일반적으로 재사용 가능한 로우 레벨 컴포넌트의 돔을 노출해라 버튼과 인풋과 같은, 하지만 애플리케이션 레벨 컴포넌트(아바타, 댓글)는 피해라.

### Forwarding a ref through multiple components

돔 노드에 레프를 포워딩하는 것 대신, 마이인풋과 같은 컴포넌트에 포워드할 수 있다.

```jsx
const FormField = forwardRef(function FormField(props, ref) {
  // ...
  return (
    <>
      <MyInput ref={ref} />
      ...
    </>
  );
});
```

마이인풋 컴포넌트가 레프를 인풋에 포워드 했다면, 폼필드의 인풋 레프를 준다.

```jsx
function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
  }

  return (
    <form>
      <FormField label="Enter your name:" ref={ref} isRequired={true} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

폼 컴포넌트가 레프를 정의하고 폼필드에 전달한다.
폼필드가 레프를 마이인풋에 포워드하고, 마이인풋은 인풋 돔 노드를 폼이 접근할 수 있도록 한다.

### Exposing an imperative handle instead of a DOM node

전체 돔 노드를 노출하는 것 대신, 커스텀 객체를 노출할 수 있다. imperative handle를 호출해, 메서드를 더 제약할 수 있다.
이것을 위해, 돔 노드를 들고 있는 개별적인 레프를 정의해야한다.

```jsx
const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  // ...

  return <input {...props} ref={inputRef} />;
});
```

레프를 useImperativeHandle에 전달하고 노출하고 싶은 레프의 값을 정한다.

```jsx
import { forwardRef, useRef, useImperativeHandle } from "react";

const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(
    ref,
    () => {
      return {
        focus() {
          inputRef.current.focus();
        },
        scrollIntoView() {
          inputRef.current.scrollIntoView();
        },
      };
    },
    []
  );

  return <input {...props} ref={inputRef} />;
});
```

만약 어떤 컴포넌트가 마이인풋에서 레프를 얻는 다면, 전체 돔 노드가 아닌 `{ focus, scrollIntoView }`를 받을 것이다.
이것은 한정된 최소의 돔 노드 정보를 노출하도록 해준다.

```jsx
import { useRef } from "react";
import MyInput from "./MyInput.js";

export default function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
    // This won't work because the DOM node isn't exposed:
    // ref.current.style.opacity = 0.5;
  }

  return (
    <form>
      <MyInput label="Enter your name:" ref={ref} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

## Troubleshooting

### My component is wrapped in forwardRef, but the ref to it is always null

이것은 종종 전달 받은 레프를 실제로 사용하는 것을 잊었기에 발생한다.

예를 들어, 이 컴포넌트는 레프를 가지고 어떤 것도 하지 않는다.

```jsx
const MyInput = forwardRef(function MyInput({ label }, ref) {
  return (
    <label>
      {label}
      <input />
    </label>
  );
});
```

이것을 고치기 위해, 레프를 돔 노드 또는 레프를 받는 컴포넌트에 전달한다.

```jsx
const MyInput = forwardRef(function MyInput({ label }, ref) {
  return (
    <label>
      {label}
      <input ref={ref} />
    </label>
  );
});
```

마이인풋의 레프는 조건문일 경우 null일 수 있다.

```jsx
const MyInput = forwardRef(function MyInput({ label, showInput }, ref) {
  return (
    <label>
      {label}
      {showInput && <input ref={ref} />}
    </label>
  );
});
```

showInput가 false일 경우 레프는 어떤 노드도 포워드 하지 않고 마이인풋의 레프는 비어있을 것이다. 이것은 패널과 같은 예에서 쉽게 놓친다.

```jsx
const MyInput = forwardRef(function MyInput({ label, showInput }, ref) {
  return (
    <label>
      {label}
      <Panel isExpanded={showInput}>
        <input ref={ref} />
      </Panel>
    </label>
  );
});
```

---

## What I Learned

forwardRef는 부모 컴포넌트에서 자식 컴포넌트의 돔 노드에 접근하는 방법이다.
기존의 컴포넌트는 돔 노드를 노출하지 않기에 사용한다.
