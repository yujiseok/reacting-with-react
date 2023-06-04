# Manipulating the DOM with Refs

리액트는 랜더의 결과를 자동적으로 돔에 업데이트하므로, 당신의 컴포넌트는 조작할 필요가 자주 없습니다.
하지만, 가끔 당신은 리액트과 관리하는 돔 요소를 접근하고 싶을 수 있습니다. 예를 들어, 노드에 포커스, 스크롤, 크기와 위치 측정 등.
리액트에서는 위와 같은 것을 처리할 수 있는 빌트인 수단이 없으므로, 당신은 돔 노드에 ref가 필요합니다.

## Getting a ref to the node

리액트가 관리하는 돔 노드에 접근하기 위해선, 우선 `useRef` 훅을 불러옵니다.

```jsx
import { useRef } from "react";
```

그 후, 컴포넌트 내부에서 ref를 선언합니다.

```jsx
const myRef = useRef(null);
```

마지막으로, 원하는 돔 노드에 ref를 JSX 태그의 `ref` 어트리뷰트로 전달합니다.

```jsx
<div ref={myRef}>
```

`useRef` 훅은 `current` 오직 속성을 갖는 객체를 반환합니다.
초기에 `myRef.current`는 `null`일 것입니다.
리액트가 `<div>` 돔 노드를 만들때, 리액트는 이 노드의 레퍼런스를 `myRef.current`에 주입합니다. 당신은 이제 당신의 이벤트 핸들로와 브라우저 API에서 돔 노드를 접근할 수 있습니다.

```jsx
// You can use any browser APIs, for example:
myRef.current.scrollIntoView();
```

### Example: Focusing a text input

이 예에서, 버튼을 클릭하면 인풋에 포커스됩니다.

```jsx
import { useRef } from "react";

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
```

이것을 구현하기 위해:

1. `inputRef`를 `useRef`와 선언
2. `<input ref={inputRef}>`와 같이 전달. 이것은 리액트에게 인풋의 돔 노드를 `inputRef.current`에 담도록 알려줌
3. `handleClick` 함수 내에서, 인풋 돔을 `inputRef.current`로 읽어와 `focus()`를 `inputRef.current.focus()`로 호출
4. `handleClick` 이벤트 핸들러를 버튼의 `onClick`에 넘김

비록 돔 조작이 ref의 일반적인 사용이지만, `useRef` 훅은 타이머 아이디와 같은 리액트 외부의 것을 저장할 수 있습니다.
상태와 비슷하게 랜더 중에 유지됩니다. ref는 리랜더를 일으키지 않습니다.

### Example: Scrolling to an element

당신은 당신은 컴포넌트안에 여러개의 ref를 가질 수 있습니다.
이 예에서, 3개의 이미지를 갖는 캐로셀이 있습니다.
각 버튼은 브라우저의 `scrollIntoView()` 메서드를 호출해 이미지를 가운데 정렬합니다.

```jsx
import { useRef } from "react";

export default function CatFriends() {
  const firstCatRef = useRef(null);
  const secondCatRef = useRef(null);
  const thirdCatRef = useRef(null);

  function handleScrollToFirstCat() {
    firstCatRef.current.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }

  function handleScrollToSecondCat() {
    secondCatRef.current.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }

  function handleScrollToThirdCat() {
    thirdCatRef.current.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }

  return (
    <>
      <nav>
        <button onClick={handleScrollToFirstCat}>Tom</button>
        <button onClick={handleScrollToSecondCat}>Maru</button>
        <button onClick={handleScrollToThirdCat}>Jellylorum</button>
      </nav>
      <div>
        <ul>
          <li>
            <img
              src="https://placekitten.com/g/200/200"
              alt="Tom"
              ref={firstCatRef}
            />
          </li>
          <li>
            <img
              src="https://placekitten.com/g/300/200"
              alt="Maru"
              ref={secondCatRef}
            />
          </li>
          <li>
            <img
              src="https://placekitten.com/g/250/200"
              alt="Jellylorum"
              ref={thirdCatRef}
            />
          </li>
        </ul>
      </div>
    </>
  );
}
```

#### DEEP DIVE - How to manage a list of refs using a ref callback

위의 예에서, 미리 정의된 ref들이 있습니다. 하지만, 때로 당신은 리스트의 각 아이템에 ref를 원할 수 있지만, 얼마나 가져야할지 모릅니다. 이런 방식은 동작하지 않습니다.

```jsx
<ul>
  {items.map((item) => {
    // Doesn't work!
    const ref = useRef(null);
    return <li ref={ref} />;
  })}
</ul>
```

이것은 훅은 반드시 컴포넌트의 최상단에서 호출되어야 하기 때문입니다.
당신은 `useRef`를 반복문, 조건문 또는 `map()`에서 호출할 수 없습니다.

하나의 가능한 방식은 그들의 부모 요소에 ref를 갖게 한 후,
`querySelectorAll`과 같은 돔 조작 메서드를 사용해 개별 자식 노드를 찾아내는 것입니다.
하지만, 이것은 깨지기 쉬우며 돔의 구조가 변경되면 깨집니다.

다른 해결책은 `ref` 어트리뷰트에 함수를 전달하는 것입니다.
이것은 `ref` 콜백이라 불립니다. 리액트는 ref가 설정될 때, 콜백 함수를 실행하고 `null`일 때 초기화합니다.
이것은 배열이나 맵을 유지하며, 그들의 아이디나 인덱스로 접근하도록 합니다.

이 예는 당신이 긴 리스트의 임의의 노드로 스크롤하는 것을 보여줍니다.

```jsx
mport { useRef } from 'react';

export default function CatFriends() {
  const itemsRef = useRef(null);

  function scrollToId(itemId) {
    const map = getMap();
    const node = map.get(itemId);
    node.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  function getMap() {
    if (!itemsRef.current) {
      // Initialize the Map on first usage.
      itemsRef.current = new Map();
    }
    return itemsRef.current;
  }

  return (
    <>
      <nav>
        <button onClick={() => scrollToId(0)}>
          Tom
        </button>
        <button onClick={() => scrollToId(5)}>
          Maru
        </button>
        <button onClick={() => scrollToId(9)}>
          Jellylorum
        </button>
      </nav>
      <div>
        <ul>
          {catList.map(cat => (
            <li
              key={cat.id}
              ref={(node) => {
                const map = getMap();
                if (node) {
                  map.set(cat.id, node);
                } else {
                  map.delete(cat.id);
                }
              }}
            >
              <img
                src={cat.imageUrl}
                alt={'Cat #' + cat.id}
              />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}

const catList = [];
for (let i = 0; i < 10; i++) {
  catList.push({
    id: i,
    imageUrl: 'https://placekitten.com/250/200?image=' + i
  });
}
```

이 예에서, `itemsRef`는 하나의 돔 노드만을 갖지 않습니다. 이것은 돔 노드의 맵을 보관합니다. 모든 리스트의 `ref` 콜백은 맵 업데이트에 주의를 기울입니다:

```jsx
<li
  key={cat.id}
  ref={node => {
    const map = getMap();
    if (node) {
      // Add to the Map
      map.set(cat.id, node);
    } else {
      // Remove from the Map
      map.delete(cat.id);
    }
  }}
>
```

이것은 맵에서 각 돔 노드를 읽을 수 있게 해줍니다.

## Accessing another component’s DOM nodes

`<input/>`과 같은 빌트인 컴포넌트에 ref를 설정하면, 리액트는 ref의 `current` 속성에 상응하는 돔 노드를 설정합니다.

하지만, 당신이 `<MyInput/>` 같은 고유한 컴포넌트에 초기값이 null인 ref를 설정하려합니다. 이 예이입니다. 버튼을 클릭하는 것이 인풋을 포커스하지 않는 것을 보세요:

```jsx
import { useRef } from "react";

function MyInput(props) {
  return <input {...props} />;
}

export default function MyForm() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
```

이 이슈를 도와주기 위해, 리액트는 콘솔에 에러를 보여줍니다.

> Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?

이것은 리액트가 다른 컴포넌트의 돔 노드에 접근하는 것을 기본적으로 막기 때문입니다. 심지어 그들의 자식에게도! 이것은 의도적입니다.
ref는 탈출구로 드물게 사용해야합니다.
수동으로 다른 컴포넌트의 노드를 조작하는 것은 당신의 코드를 더 취약하게 합니다.

대신에, 돔 노드를 노출하고 싶은 컴포넌트는 동작을 선택해야합니다. 컴포넌트는 하위 항목 중 하나에 대한 ref를 전달하도록 지정할 수 있습니다.
`MyInput`이 `forwardRef` API를 사용하는 법입니다.

```jsx
const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});
```

1. `<MyInput ref={inputRef} />`는 리액트에게 상응하는 돔 노드를 `inputRef.current`에 담도록 합니다. 하지만, 선택하는 것은 마이인풋 컴포넌트에 달려있습니다. 기본적으로는 그렇게 되지 않습니다.
2. 마이인풋 컴포넌트는 `forwardRef`를 사용해 선언. 이것은 `inputRef`를 프랍 다음에 선언된 `ref` 인자로 받아옵니다.
3. 마이인풋은 ref를 받아 인풋에 전달합니다.

```jsx
import { forwardRef, useRef } from "react";

const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
```

디자인 시스템에서 버튼, 인풋과 같은 로우 레벨 컴포넌트는 그들의 ref를 돔 노드에 전달합니다. 다른 쪽으로, 폼, 리스트 그리고 페이지 섹션과 같은 하이 레벨 컴포넌트들 보통 그들의 돔 노드를 노출하지 않고 돔 구조에 우발적인 의존성을 방지합니다.

#### DEEP DIVE - Exposing a subset of the API with an imperative handle

위의 예에서, 마이인풋은 돔 인풋 요소를 노출합니다. 이것은 부모 컴포넌트가 `focus()`를 호출할 수 있도록 해줍니다. 하지만, 이것은 부모 컴포넌트가 다른 동작을 취할 수 있다는 것을 말합니다.
이런 일반적이지 않은 상황에, 당신은 노출을 제한하고 싶을 것 입니다.
당신은 `useImperativeHandle`와 그것을 할 수 있습니다.

```jsx
import { forwardRef, useRef, useImperativeHandle } from "react";

const MyInput = forwardRef((props, ref) => {
  const realInputRef = useRef(null);
  useImperativeHandle(ref, () => ({
    // Only expose focus and nothing else
    focus() {
      realInputRef.current.focus();
    },
  }));
  return <input {...props} ref={realInputRef} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
```

마이인풋 안의 `realInputRef`가 실제 인풋의 돔 노드를 갖습니다.
하지만, `useImperativeHandle`는 리액트에 부모 컴포넌트에 특별한 객체를 제공하라고 지시합니다.
폼 컴포넌트의 `inputRef.current`는 오직 `focus` 메서드만 갖습니다.
이 경우에, ref는 돔 노드를 다루는 것이 아닌, `useImperativeHandle` 내부에서 생성된 객체를 다룹니다.

## When React attaches the refs

리액트에서 모든 업데이트는 두 단계로 나뉩니다.

- 랜더중, 리액트는 당신의 컴포넌트를 호출해 무엇이 화면에 보일지 알아냅니다.
- 커밋중, 리액트는 돔에 변경을 적용합니다.

보통, 당신은 랜더중에 ref에 접근하고 싶지 않습니다. 이것은 돔 노드를 소유하는 ref에도 해당됩니다.
첫 랜더중, 돔 노드는 아직 생성되지 않으며, `ref.current`는 `null`입니다. 그리고 변경 랜더중에도, 돔 노드는 업데이트 되지 않습니다.
이것은 그들을 읽기에 너무 이릅니다.

리액트는 `ref.current`를 커밋 도중에 설정합니다. 돔을 업데이트하기 전, 리액트는 `ref.current`의 값을 `null`로 설정합니다. 돔 업데이트 후, 리액트는 즉시 상응하는 돔 노드를 설정합니다.

보통, 당신은 ref를 이벤트 핸들러에서 접근할 것입니다. 만약 ref를 다루고 싶은데 마땅한 이벤트가 없다면, 당신은 이펙트가 필요합니다.

#### DEEP DIVE - Flushing state updates synchronously with flushSync

새로운 투두를 추가하면 마지막 자식으로 스크롤하는 다음과 같은 코드가 있습니다.
이상하게도, 이것은 항상 마지막 이전으로 스크롤합니다.

```jsx
import { useState, useRef } from "react";

export default function TodoList() {
  const listRef = useRef(null);
  const [text, setText] = useState("");
  const [todos, setTodos] = useState(initialTodos);

  function handleAdd() {
    const newTodo = { id: nextId++, text: text };
    setText("");
    setTodos([...todos, newTodo]);
    listRef.current.lastChild.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
    });
  }

  return (
    <>
      <button onClick={handleAdd}>Add</button>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <ul ref={listRef}>
        {todos.map((todo) => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}

let nextId = 0;
let initialTodos = [];
for (let i = 0; i < 20; i++) {
  initialTodos.push({
    id: nextId++,
    text: "Todo #" + (i + 1),
  });
}
```

이 이슈는 다음 두 줄 때문입니다.

```jsx
setTodos([...todos, newTodo]);
listRef.current.lastChild.scrollIntoView();
```

리액트에서, 상태 업데이트는 큐됩니다. 보통 이것은 우리가 원하는 것입니다.
하지만, `setTodos`가 돔을 즉시 업데이트하지 않기에 문제가 발생합니다.
마지막 요소로 스크롤 될 때, 투두는 아직 추가되지 않았습니다.
한 아이템 이전으로 스크롤 되는 이유입니다.

이것을 해결하기 위해, 당신은 리액트에게 돔을 동기적으로 업데이트할 수 있도록 강제할 수 있습니다. 이것을 하기 위해 `flushSync`를 `react-dom`에서 가져온 후 상태 업데이트를 감싸줍니다.

```jsx
flushSync(() => {
  setTodos([...todos, newTodo]);
});
listRef.current.lastChild.scrollIntoView();
```

이것은 리액트에게 `flushSync`로 감싸진 코드가 실행되면 돔을 동기적으로 업데이트하도록 지시합니다.
이것의 결과로 마지막 투두는 이미 돔에 추가되어 있습니다.

```jsx
import { useState, useRef } from "react";
import { flushSync } from "react-dom";

export default function TodoList() {
  const listRef = useRef(null);
  const [text, setText] = useState("");
  const [todos, setTodos] = useState(initialTodos);

  function handleAdd() {
    const newTodo = { id: nextId++, text: text };
    flushSync(() => {
      setText("");
      setTodos([...todos, newTodo]);
    });
    listRef.current.lastChild.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
    });
  }

  return (
    <>
      <button onClick={handleAdd}>Add</button>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <ul ref={listRef}>
        {todos.map((todo) => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}

let nextId = 0;
let initialTodos = [];
for (let i = 0; i < 20; i++) {
  initialTodos.push({
    id: nextId++,
    text: "Todo #" + (i + 1),
  });
}
```

## Best practices for DOM manipulation with refs

ref는 탈출구입니다. 리액트에서 벗어나고 싶을 경우에만 그것을 사용해야합니다.
가장 보편적인 예는, 포커스 관리, 스크롤 위치 또는 리액트가 노출하지 않은 브라우저 API 사용 시 입니다.

당신이 포커싱과 스크롤과 같은 비파괴적인 것을 고수한다면, 문제와 마주하지 말아야합니다. 하지만, 당신이 돔을 수동으로 조작하려하면, 당신은 리액트가 만든 변화에 충돌할 수 있습니다.

이 문제의 예는, 두 버튼과 환영 메세지를 포함합니다.
첫 번째 버튼은 일반적인 리액트가 하는 조건부 랜더링과 상태를 사용합니다.
두 번째 버튼은 `remove()` 돔 API를 사용해 리액트의 제어 밖에서 돔을 삭제 합니다.

수동적으로 돔 요소를 삭제한 후, `setState`를 사용하면 앱은 충돌을 유발합니다. 이것은 당신이 돔을 변경 하였는데, 리액트는 그것을 어떻게 관리할지 모르기 때문입니다.

리액트가 관리하는 돔을 변경하는 것을 피하세요.
리액트로 관리되는 요소를 수정, 자식에 추가 또는 자식 삭제하는 것은 일관되지 않은 화면 결과와 충돌을 일으킵니다.

하지만, 이것은 당신이 아예 하지말라는 것은 아닙니다.
이것은 주의를 기울여야합니다. 당신은 리액트가 변경할 필요 없는 돔의 일부를 안전하게 변경할 수 있습니다. 예를 들어, 어떤 `div`가 항상 비어야 한다면, 리액트는 자식 리스트를 다룰 필요 없습니다. 그러므로 이것은 수동적으로 추가하고 삭제하는 것이 안전합니다.

## Recap

- ref는 제네릭한 컨셉. 하지만 대부분 돔 요소를 다루기 위해 사용
- 리액트에 `myRef.current`에 돔 노드를 담기 위해 `<div ref={myRef}>`를 전달하도록 지시
- 보통, ref는 포커싱, 스크롤 그리고 돔 요소 측정과 같은 비파괴 액션에 사용
- 컴포넌트는 돔 노드를 기본적으로 노출하지 않음. `forwardRef`를 통해 선택적으로 노출할 수 있으며, 두 번째 인자 `ref`로 특정 노드에 전달
- 리액트가 관리하는 돔 노드 변경을 피해야함
- 리액트에서 관리하는 돔 노드를 수정하는 경우 리액트에서 업데이트할 이유가 없는 컴포넌트 수정

---

## What I Learned

이 부분은 확실히 기존에 내가 알려고 하지 않은 부분의 개념들이 많이 나온다.
특히 forwardRef, flushSync, useImperativeHandle 같은 것들은 사실 한번도 사용해보지 않고 정말 생소한 것들이다. (이런 것들을 사용하는 경우가 얼마나 될지는 모르겠지만) 리액트를 이해하는데 많이 도움이 되는 것 같다.

출처

- https://react.dev/learn/manipulating-the-dom-with-refs
