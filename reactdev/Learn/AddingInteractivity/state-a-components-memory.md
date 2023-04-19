# State: A Component's Memory

컴포넌트들은 인터렉션의 결과로 화면에 보여지는 것을 변화시킵니다. 입력을 통해 폼의 인풋 필드가 변하고 이미지 캐로셀의 다음을 눌렀을 때 어떤 이미지가 보일지 변화하고, 구매를 눌렀을 때 상품은 장바구니로 가야합니다.
컴포넌트들은 현재 인풋 값, 현재 이미지, 장바구니를 기억해야합니다.
리액트에서 이런 종류의 컴포넌트에 한정된 메모리를 상태라고 합니다.

## When a regular variable isn’t enough

조각물 이미지를 렌더하는 컴포넌트가 있습니다. 다음 버튼을 누르면 다음 조각물이 인덱스에 따라 변화해야합니다. 하지만, 이것은 동작하지 않습니다.

```jsx
import { sculptureList } from "./data.js";

export default function Gallery() {
  let index = 0;

  function handleClick() {
    index = index + 1;
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleClick}>Next</button>
      <h2>
        <i>{sculpture.name} </i>
        by {sculpture.artist}
      </h2>
      <h3>
        ({index + 1} of {sculptureList.length})
      </h3>
      <img src={sculpture.url} alt={sculpture.alt} />
      <p>{sculpture.description}</p>
    </>
  );
}
```

`handleClick` 함수는 지역 변수인 `index`를 업데이트 합니다. 하지만 두 가지가 보여지는 것을 방지하고 있습니다.

1. 지역 변수는 렌더 사이에 유지되지 않습니다. 리액트가 컴포넌트를 두 번째로 렌더링 한다면, 처음 부터 렌더링 됩니다. 즉 어떤 지역 변수의 변화도 고려하지 않습니다.
2. 지역 변수의 변경은 렌더링의 트리거가 되지않습니다. 리액트는 새로운 데이터로 컴포넌트를 리렌더링 해야하는 지 인지하지 못합니다.

컴포넌트를 새로운 데이티로 업데이트 하는 방법에는, 두 가지가 있습니다.

1. 렌더 도중에 데이터 유지하기
2. 새로운 데이터로 렌더링 트리거하기

`useState` 훅은 두 가지를 제공합니다.

1. 렌더 사이에 유지되는 상태 변수
2. 렌더링의 트리거가 되는 세터 함수

## Adding a state variable

상태 변수를 추가하기 위해선 `useState` 훅을 리액트에서 가져옵니다.

```jsx
import { useState } from "react";
```

그리고 아래를 바꿔줍니다.

```jsx
let index = 0;
```

```jsx
const [index, setIndex] = useState(0);
```

`index`는 상태 변수이고 `setIndex`는 세터 함수입니다.

> `[ ]` 문법은 배열의 구조 분해 할당입니다.

`handleClick` 안에서 동작하는 방식입니다.

```jsx
function handleClick() {
  setIndex(index + 1);
}
```

이제 다음 버튼을 누르면 조각물이 바뀝니다.

```jsx
import { useState } from "react";
import { sculptureList } from "./data.js";

export default function Gallery() {
  const [index, setIndex] = useState(0);

  function handleClick() {
    setIndex(index + 1);
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleClick}>Next</button>
      <h2>
        <i>{sculpture.name} </i>
        by {sculpture.artist}
      </h2>
      <h3>
        ({index + 1} of {sculptureList.length})
      </h3>
      <img src={sculpture.url} alt={sculpture.alt} />
      <p>{sculpture.description}</p>
    </>
  );
}
```

## Meet your first Hook

리액트에서 `useState` 처럼 `use`로 시작하는 함수들은 훅이라고 불립니다.

훅은 리액트가 렌더링될 시에만 사용 가능한 특별한 함수들입니다. 이들은 다른 리액트 기능을 사용할 수 있도록 해줍니다.

상태는 이런 기능들 중 하나입니다.

> Pitfall
> 훅은 컴포넌트의 최상단 혹은 커스텀 훅의 최상단에서만 호출되어야 합니다. 조건, 반복문 또는 중첩된 함수 내에서 훅을 호출할 수 없습니다.
> 훅은 함수입니다. 그러나 컴포넌트가 필요한 무조건적인 선언이라고 생각하는 것이 좋습니다.
> 컴포넌트 안에서 훅을 사용하는 것은 파일에서 필요한 모듈을 import 하는 것과 비슷합니다.

## Anatomy of useState

`useState` 훅을 호출 할때, 리액트에게 컴포넌트에서 기억될 무언가를 말하는 것입니다.

```jsx
const [index, setIndex] = useState(0);
```

이 경우에서, 우리는 리액트가 `index`를 기억하길 원합니다.

> Note
> `const [something, setSomething]`와 같은 컨벤션을 사용합니다. 어떤 네이밍도 가능하지만, 컨벤션은 이해를 도와줍니다.

`useState`의 유일한 인자는 초기값입니다. 위 예에서 `index`의 초기값은 `useState(0)`를 통해 0으로 설정 되었습니다.

매 렌더링 시 `useState`는 항상 두 값을 갖는 배열을 반환합니다.

1. 값을 저장하기 위한 상태 변수
2. 상태를 업데이트 하고 리액트의 렌더링을 트리거하는 세터 함수

다음과 같이 동작합니다.

```jsx
const [index, setIndex] = useState(0);
```

1. 컴포넌트가 첫 렌더링을 합니다. `useState`의 초기값을 0으로 지정하여, `[0, setIndex]`를 반환합니다. 리액트는 0을 최신의 값으로 기억합니다.
2. 상태 업데이트를 합니다. 유저가 버튼을 클릭하면, `setIndex(index + 1)`를 호출합니다. index는 0 이기에 `setIndex(1)`가 됩니다. 이것은 리액트에게 index가 1이라는 것을 기억하도록 하고 렌더링을 일으킵니다.
3. 컴포넌트의 두 번째 렌더. 리액트는 여전히 `setIndex(0)`를 사용합니다. 하지만, 리액트는 index가 1이라는 것을 기억하므로 `[1, setIndex]`를 반환합니다.
4. 과정이 계속 됩니다.

## Giving a component multiple state variables

한 컴포넌트 안에서 여러 타입의 여러 상태를 가질 수 있습니다.
이 컴포넌트는 두 개의 상태를 갖고 있습니다. 숫자형인 index와 불리언형인 showMore입니다.

```jsx
import { useState } from "react";
import { sculptureList } from "./data.js";

export default function Gallery() {
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);

  function handleNextClick() {
    setIndex(index + 1);
  }

  function handleMoreClick() {
    setShowMore(!showMore);
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleNextClick}>Next</button>
      <h2>
        <i>{sculpture.name} </i>
        by {sculpture.artist}
      </h2>
      <h3>
        ({index + 1} of {sculptureList.length})
      </h3>
      <button onClick={handleMoreClick}>
        {showMore ? "Hide" : "Show"} details
      </button>
      {showMore && <p>{sculpture.description}</p>}
      <img src={sculpture.url} alt={sculpture.alt} />
    </>
  );
}
```

상태들이 연관이 없을 시 여러 상태를 갖는 것은 좋은 생각입니다. 하지만 두 상태를 동시에 바꾸는 것을 찾게 되면, 하나로 합치는 것이 쉽습니다.
예를 들어 폼의 여러 필드를 관리할 때, 객체로 관리하는 것이 더 좋습니다.

## DEEP DIVE - How does React know which state to return?

아마 `useState` 호출은 어떤 상태 변수인지에대한 어떤 정보도 받지 않는 다는 것을 눈치챘을 것입니다.
`useState`에는 어떤 식별자도 없습니다. 그렇다면 어떻게 상태를 반환하는 걸까요? 어떤 마법에 의지해 함수를 파싱하는 것일까요?
답은 아니오입니다.

대신에, 간결하게 훅은 매 렌더링에 순서에 의존합니다. 훅의 룰에 따르면 잘 동작합니다. 훅은 항상 똑같은 순서로 호출됩니다.

내부적으로 리액트는 모든 구성 요서에 대한 상태 쌍 배열을 보유합니다. 또한 최근 쌍을 유지합니다. `useState`를 호출할 시 리액트는 다음 상태 쌍과 증가된 index를 반환합니다.

## State is isolated and private

상태는 지역적입니다. 다른 말로, 똑같은 컴포넌트를 두 번 호출할 시, 각 복사본은 독립적인 상태를 갖습니다.

```jsx
import Gallery from "./Gallery.js";

export default function Page() {
  return (
    <div className="Page">
      <Gallery />
      <Gallery />
    </div>
  );
}
```

이것이 상태가 모듈의 최상단에 선언한 일반적인 변수와 다른 이유입니다.
상태는 어떤 함수의 호출이나 장소에 구애 받지 않습니다. 하지만 이것은 지역적입니다.

또한 `Page` 컴포넌트는 `Gallery` 컴포넌트의 상태나 무엇을 가졌는지 아무것도 모릅니다. 프랍스와 다르게 상태는 완벽하게 선언된 컴포넌트에서 프라이빗합니다. 부모 컴포넌트는 이것을 변화시킬 수 없습니다. 이것은 상태를 추가하거나 삭제할때, 나머지 컴포넌트에 영향이 없다는 것을 뜻합니다.

만약 갤러리에 상태의 싱크를 맞추고 싶다면? 자식 컴포넌트의 상태를 제거하고 부모 컴포넌트에 추가해 상태를 공유할 수 있습니다.

## Recap

- 상태 변수를 활용해 렌더시에도 정보를 기억하게 할 수 있습니다.
- 상태 변수는 `useState` 훅을 호출해 선언됩니다.
- 훅은 리액트가 렌더링될 시에만 사용 가능한 특별한 함수들입니다. 이들은 다른 리액트 기능을 사용할 수 있도록 해줍니다.
- 훅은 import를 떠올리게 합니다. 훅은 컴포넌트의 최상단 혹은 커스텀 훅의 최상단에서만 호출되어야 합니다. 조건, 반복문 또는 중첩된 함수 내에서 훅을 호출할 수 없습니다.
- `useState` 현재 상태와 업데이트 해주는 함수 쌍을 반환합니다.
- 여러개의 상태를 가질 수 있으며, 내부적으로 리액트가 순서에 따라 매치합니다.
- 상태는 독립적이서, 두 곳에서 렌더할 시 각 복사본은 각자의 상태를 갖습니다.

---

## What I Learned

항상 어떻게 상태를 찾을까 고민을 했는데, 내부적인 순서가 있고 그 순서에 맞는 상태를 반환한다는 것이 놀랍다. 또 항상 똑같은 순서를 보장한다는 점은 처음 알았다.

출처

- https://react.dev/learn/state-a-components-memory
