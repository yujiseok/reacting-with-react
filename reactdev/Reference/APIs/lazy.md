# lazy

lazy는 컴포넌트가 처음 랜더될 때, 코드의 로딩을 연기해준다.

```jsx
const SomeComponent = lazy(load);
```

## Reference

### lazy(load)

lazy를 컴포넌트 외부에 호출해 lazy-loaded되는 리액트 컴포넌트를 선언한다.

```jsx
import { lazy } from "react";

const MarkdownPreview = lazy(() => import("./MarkdownPreview.js"));
```

**Parameters**

- load: 프로미스나 thenable한 객체를 반환한다. 리액트는 첫 랜더 시도가 있을 때까지 load를 호출하지 않는다. 리액트가 첫 load를 호출하고, 해결되길 기다리고, 해결된 값을 리액트 컴포넌트로 반환한다. 반환된 프로미스와 해결된 값은 둘다 캐시되므로, 리액트는 한번이상 호출하지 않는다. 만약 프로미스가 거절되면, 리액트는 거절 이유를 throw해서 가까운 에러 바운더리가 다루도록 한다.

**Returns**

lazy는 트리에 랜더할 수 있는 리액트 컴포넌트를 반환한다.
lazy 컴포넌트의 코드가 여전히 로딩 중이라면, 랜더링을 시도하다 중단된다.
서스펜스를 사용해 로딩시 로딩 인디케이터를 보여줄 수 있다.

### load function

**Parameters**

load은 파라미터를 받지 않느다.

**Returns**

프로미스나 thenable한 객체를 반환한다. 최종적으로 유효한 리액트 컴포넌트(메모, forwardRef 가능한)로 리졸브 되어야한다.

## Usage

### Lazy-loading components with Suspense

보통, 정적인 import와 함께 컴포넌트를 불러온다.

```jsx
import MarkdownPreview from "./MarkdownPreview.js";
```

첫 번째 랜더가 진행될 때까지 컴포넌트의 코드를 지연 로딩하기 위해서, import를 대체해라.

```jsx
import { lazy } from "react";

const MarkdownPreview = lazy(() => import("./MarkdownPreview.js"));
```

이 코드는 동적 import에 의존해, 번들러 또는 프레임워크의 지원이 필요할 수 있다.

이제 컴포넌트의 코드가 요청에 의해 랜더된다. 또한, 로딩 시 무엇을 보여줄지 역시 정할 수 있다. 부모를 서스펜스 바운더리로 감싸 처리할 수 있다.

```jsx
<Suspense fallback={<Loading />}>
  <h2>Preview</h2>
  <MarkdownPreview />
</Suspense>
```

마크다운프리뷰는 랜더를 시도하기 전까지는 로드되지 않는다.
만약 마크다운프리뷰가 아직 로드되지 않았다면, 로딩이 보여질것이다.

```jsx
import { useState, Suspense, lazy } from "react";
import Loading from "./Loading.js";

const MarkdownPreview = lazy(() =>
  delayForDemo(import("./MarkdownPreview.js"))
);

export default function MarkdownEditor() {
  const [showPreview, setShowPreview] = useState(false);
  const [markdown, setMarkdown] = useState("Hello, **world**!");
  return (
    <>
      <textarea
        value={markdown}
        onChange={(e) => setMarkdown(e.target.value)}
      />
      <label>
        <input
          type="checkbox"
          checked={showPreview}
          onChange={(e) => setShowPreview(e.target.checked)}
        />
        Show preview
      </label>
      <hr />
      {showPreview && (
        <Suspense fallback={<Loading />}>
          <h2>Preview</h2>
          <MarkdownPreview markdown={markdown} />
        </Suspense>
      )}
    </>
  );
}

// Add a fixed delay so you can see the loading state
function delayForDemo(promise) {
  return new Promise((resolve) => {
    setTimeout(resolve, 2000);
  }).then(() => promise);
}
```

이 예시는 인공적인 딜레이와 로드된다. 체크박스의 토글했을 시, 프리뷰는 캐시되고, 더 이상 로딩 상태가 없다. 다시 보기 위해선, 샌드박스의 리셋 버튼을 클릭해라.

## Troubleshooting

### My lazy component’s state gets reset unexpectedly

다른 컴포넌트 내부에서 lazy를 선언하지 말아라.

```jsx
import { lazy } from "react";

function Editor() {
  // 🔴 Bad: This will cause all state to be reset on re-renders
  const MarkdownPreview = lazy(() => import("./MarkdownPreview.js"));
  // ...
}
```

대신 항상 모듈의 최상위에서 선언해라.

```jsx
import { lazy } from "react";

// ✅ Good: Declare lazy components outside of your components
const MarkdownPreview = lazy(() => import("./MarkdownPreview.js"));

function Editor() {
  // ...
}
```

---

## What I Learned

lazy는 컴포넌트가 첫 랜더를 시도하기 전까지 지연된 로딩을 제공해준다.
서스펜스와 같이 활용하면 로딩 상태를 다룰 수 있다.
