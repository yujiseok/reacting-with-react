# useImperativeHandle

useImperativeHandle은 노출된 레프를 다룰 수 있는 리액트의 훅이다.

```jsx
useImperativeHandle(ref, createHandle, dependencies?)
```

## Reference

### useImperativeHandle(ref, createHandle, dependencies?)

컴포넌트 최상단에 useImperativeHandle를 호출해 노출된 레프를 다룬다.

```jsx
import { forwardRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  useImperativeHandle(ref, () => {
    return {
      // ... your methods ...
    };
  }, []);
  // ...
```

**Parameters**

- ref: forwardRef로 랜더되는 함수의 인자인 레프를 받는다.
- createHandle: 인자를 받지 않고, 노출된 레프를 다룰 핸들러를 반환한다. 레프 핸들은 어떤 타입도 가능하다. 보통 이것은 메서드가 있는 객체를 반환한다.
- optional dependencies: createHandle 코드 내에서 참조되는 반응 값.

**Returns**

useImperativeHandle은 undefined를 반환한다.

## Usage

### Exposing a custom ref handle to the parent component

기본적으로, 컴포넌트는 부모 컴포넌트에게 돔 노드를 노출하지 않는다. 예를 들어, 만약 MyInput의 부모 컴포넌트에서 input 돔 노드에 접근 하고 싶다면, forwardRef로 접근해야한다.

```jsx
import { forwardRef } from "react";

const MyInput = forwardRef(function MyInput(props, ref) {
  return <input {...props} ref={ref} />;
});
```

위 코드에서 MyInput은 input 돔 노드의 레프를 받을 것이다. 하지만, 임의의 값을 노출할 수 있다.
useImperativeHandle를 컴포넌트 최상단에 호출해, 임의의 노출된 핸들을 할 수 있다.

```jsx
import { forwardRef, useImperativeHandle } from "react";

const MyInput = forwardRef(function MyInput(props, ref) {
  useImperativeHandle(
    ref,
    () => {
      return {
        // ... your methods ...
      };
    },
    []
  );

  return <input {...props} />;
});
```

레프는 더이상 input에 포워딩 되지 않는다.

예를 들어, 인풋 노드의 전체를 노출하는 것이 아닌, focus와 scrollIntoView 두 메서드만을 노출 시키고 싶을 수 있다.
이 경우, 실제 돔 노드를 레프로 부터 분리한다. 그 후 useImperativeHandle를 사용해 부모 컴포넌트에게 오직 메서드만 노출할 수 있다.

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

이제, 부모 컴포넌트가 MyInput에서 레프를 가져올 시, focus와 scrollIntoView 메서드를 호출 할 수 있다. 하지만, 인풋 돔 노드에 완전한 접근은 불가능하다.

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

### Exposing your own imperative methods

명령형 핸들을 통해 노출하는 메서드는 돔 노드의 메서드와 정확히 일치할 필요 없다.
예를 들어, 포스트 컴포넌트는 scrollAndFocusAddComment 메서드를 명령형 핸들로 노출한다.
이것은 부모 페이지가 버튼을 클릭 시, 인풋 필드에 포커싱되고 댓글 목록으로 스크롤하게 한다.

```jsx
import { forwardRef, useRef, useImperativeHandle } from "react";
import CommentList from "./CommentList.js";
import AddComment from "./AddComment.js";

const Post = forwardRef((props, ref) => {
  const commentsRef = useRef(null);
  const addCommentRef = useRef(null);

  useImperativeHandle(
    ref,
    () => {
      return {
        scrollAndFocusAddComment() {
          commentsRef.current.scrollToBottom();
          addCommentRef.current.focus();
        },
      };
    },
    []
  );

  return (
    <>
      <article>
        <p>Welcome to my blog!</p>
      </article>
      <CommentList ref={commentsRef} />
      <AddComment ref={addCommentRef} />
    </>
  );
});

export default Post;
```

#### Pitfall

레프를 남용하지 마라. 프랍으로 표현이 되지 않을 경우에 명령형 레프를 사용해야한다. 예를 들어, 노드로 스크롤, 노드에 포커스, 애니메이션 트리거, 텍스트 선택 등

만약 프랍으로 표현을 할 수 있다면, 레프를 사용하지 마라. 예를 들어, 명령형 핸들로 모달 컴포넌트를 { open, close } 하는 것 보다, isOpen과 같이 <Modal isOpen={isOpen} />의 프랍으로 전달하는 것이 좋다. 이펙트가 프랍을 명령적으로 노출하는 것을 도와줄 것이다.

---

## What I Learned

useImperativeHandle은 노출된 레프에 특정 메서드를 사용할 수 있도록 하는 훅이다.
useImperativeHandle을 사용하면 레프의 전체 노드를 접근하지 않고 메서드만을 호출 할 수 있다.
