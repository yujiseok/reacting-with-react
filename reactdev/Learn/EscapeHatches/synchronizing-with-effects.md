# Synchronizing with Effects

몇 컴포넌트는 외부 시스템과 동기화할 필요가 있습니다.
예를 들어, 당신은 상태 기반의 리액트 컴포넌트가 아닌 서버와 연결 또는 컴포넌트가 화면에 등장했을 때 분석 로그를 보내는 것을 제어하고 싶을 수 있습니다.
이펙트는 어떤 코드를 랜더 후에 실행해 당신이 리액트 외부 시스템과 동기화할 수 있도록 해줍니다.

## What are Effects and how are they different from events?

이펙트를 알기 전, 당신은 리액트 컴포넌트의 두 타입의 로직을 이해해야합니다.

- 랜더링 코드는 컴포넌트 최상단에 존재합니다. 이것은 프랍과 상태를 받고 변형하고 JSX를 화면에 반환합니다. 랜더링 코드는 반드시 순수해야합니다. 수학 공식 같이, 결과를 계산해야하고 다른 것은 하지 말아야합니다.

- 이벤트 핸들러는 단지 계산하는 것이 아닌, 컴포넌트에 중첩된 함수입니다. 이벤트 핸들러는 인풋 필드를 업데이트, 상품 구매를 위한 HTTP POST, 또는 유저를 다른 화면으로 이동과 같은 작동을 합니다. 이벤트 핸들러는 유저의 행동에 따른 사이드 이펙트를 포함합니다.

가끔 이것은 충분치 않습니다. 화면에 보이면 서버와 연결되는 챗룸 컴포넌트를 생각해보세요. 서버와의 연결은 순수한 계산이 아니므로 랜더 중 일어날 수 없습니다. 하지만, 챗룸을 보여주기 위한 특정한 이벤트인 클릭과 같은 이벤트도 없습니다.

이펙트는 특정 이벤트가 아닌 랜더 그 자체의 사이드 이펙트를 설명합니다.
메세지를 보내는 것은 유저가 특정 버튼을 클릭 했기에 이벤트입니다.
하지만, 어떤 상호작용이 있든지에 상관없이 컴포넌트가 보일 때 동작하므로 서버와 연결은 이펙트입니다. 이펙트는 스크린이 업데이트 되는 커밋 마지막에 일어납니다.
이 때가 리액트 컴포넌트가 외부 시스템과 동기화하기 좋은 때입니다.

## You might not need an Effect

컴포넌트에 이펙트를 추가하는 것을 서두르지 마세요. 이펙트는 리액트 외부의 시스템과 동기화하는 것을 명심하세요.
이것은 브라우저 api, 써드파티 위젯, 네트워크 등이 있습니다.
만약 이펙트가 단지 어떤 상태에 기반해 상태를 변경한다면, 당신은 이펙트가 필요하지 않습니다.

## How to write an Effect

이펙트를 작성하기 위한 세 단계:

1. 이펙트 선언. 기본적으로, 이펙트는 매 랜더 후에 동작
2. 이펙트의 의존성 명시. 대부분의 이펙트는 매 랜더에 동작하지 않고 필요시에만 동작합니다. 예를 들어, 페이드인 애니메이션은 오직 컴포넌트가 등장했을 때 일어납니다. 접속과 비접속은 컴포넌트가 등장하고 사라질 때나 챗룸이 변경되었을 때만 동작합니다. 당신은 의존성 명시를 통해 이것을 배울 것입니다.
3. 정리 함수 추가. 몇 이펙트는 당신의 동작을 종료, 취소, 정리등을 명시할 필요가 있습니다. 예를 들어, 접속은 비접속을 필요하고, 구독은 구독해제를 필요로, 페치 역시 취소 또는 무시를 필요로합니다. 당신은 어떻게 정리 함수를 반환하는 지 배울 것입니다.

### Step 1: Declare an Effect

이펙트를 선언하기 위해 `useEffect` 훅을 리액트에서 가져옵니다.

```jsx
import { useEffect } from "react";
```

그 후, 컴포넌트 최상단에서 이펙트 안에 코드를 작성합니다.

```jsx
function MyComponent() {
  useEffect(() => {
    // Code here will run after *every* render
  });
  return <div />;
}
```

매 컴포넌트가 랜더 시, 리액트는 화면을 업데이트하고 `useEffect`안의 코드를 실행합니다. 다른 말로, `useEffect`는 해당 랜더가 화면에 반영될 때까지 코드를 실행에서 미룹니다.

어떻게 이펙트가 외부 시스템과 동기화하는지 봅시다.
비디오플레이어 컴포넌트가 있다고 합시다.
이것은 재생과 정지를 `isPlaying` 프랍으로 제어됩니다.

```jsx
<VideoPlayer isPlaying={isPlaying} />
```

당신은 브라우저의 비디오 태크로 비디오플레이어를 만들 수 있습니다.

```jsx
function VideoPlayer({ src, isPlaying }) {
  // TODO: do something with isPlaying
  return <video src={src} />;
}
```

하지만 비디오 태그에 `isPlaying` 프랍이 존재하지 않습니다.
이것을 제어하는 방법은 수동적으로 `play()`와 `pause()` 메서드를 호출하는 것입니다.
당신은 현재 비디오가 재생되는지 `isPlaying`의 값을 동기화 시켜야합니다.

우리는 우선 비디오 노드의 ref가 필요합니다.

당신은 `play()`와 `pause()`를 다음과 같이 시도하지만, 이것은 틀렸습니다.

```jsx
import { useState, useRef, useEffect } from "react";

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  if (isPlaying) {
    ref.current.play(); // Calling these while rendering isn't allowed.
  } else {
    ref.current.pause(); // Also, this crashes.
  }

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  return (
    <>
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? "Pause" : "Play"}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

이 코드가 동작하지 않는 이유는 돔 노드가 랜더링 중일때 무언가 시도했기 때문입니다. 리액트에서, 랜더링은 JSX의 순수한 계산으로 돔 수정과 같은 사이드 이펙트를 포함하면 안됩니다.

게다가, 비디오플레이어가 처음 호출 되었을 때는 돔은 아직 존재하지 않습니다.
재생하고 멈출 돔 노드가 아직 존재하지 않으며, 리액트는 JSX를 반환할 때까지 어떤 돔을 생성할지 모릅니다.

이 해결책은 사이드 이펙트를 `useEffect`로 감싸 랜더 계산에서 옮겼습니다.

```jsx
import { useEffect, useRef } from "react";

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}
```

돔 업데이트를 이펙트로 감싸므로, 당신은 리액트가 화면을 먼저 업데이트할 수 있도록 합니다. 그 후 이펙트가 실행

비디오플레이어 컴포넌트가 실행되면, 몇 가지 일이 발생합니다.
첫 째, 리액트는 올바른 프랍과 비디오 태그를 보장하고, 화면을 업데이트 합니다.
그 후 리액트는 이펙트를 실행합니다. 마지막으로, 이펙트가 재생과 멈춤을 `isPlaying` 값에 따라 실행합니다.

이 예에서, 당신이 리액트의 상태와 동기화를 맞춘 외부 시스템은 브라우저 미디어 API입니다.
당신은 비슷한 방식으로 레가시한 리액트가 아닌 코드를 감싸 리액트 컴포넌트에 선언적으로 접근할 수 있습니다.

#### Pitfall

기본적으로, 이펙트는 매 랜더 시 동작합니다. 이것은 무한 루프를 생성합니다.

```jsx
const [count, setCount] = useState(0);
useEffect(() => {
  setCount(count + 1);
});
```

이펙트는 랜더의 결과로 동작합니다. 상태를 설정하는 것은 랜더링을 유발합니다.
이펙트에서 상태를 즉시 설정하는 것은 전원 콘센트 자체에 꽂는 것과 같습니다.
이펙트가 실행되면, 상태가 설정되고, 리랜더를 일으키고 이펙트를 실행하고, 상태를 또 설정하고 이것은 계속해서 리랜더를 유발합니다.

이펙트는 외부 시스템과 동기화를 해야합니다. 외부 시스템은 없고 당신이 오직 다른 상태 기반의 상태를 변경하면 당신은 이펙트가 필요 없습니다.

### Step 2: Specify the Effect dependencies

기본적으로 이펙트는 매 랜더에 일어납니다. 때때로, 이것을 원치 않을 수 있습니다.

- 때때로, 이것은 느립니다. 외부 시스템과 동기화하는 것은 항상 즉각적이지 않으므로, 당신은 필요할 때까지, 미루고 싶습니다. 예를 들어, 당신은 키를 입력할 때마다 챗 서버와 연결을 원치 않습니다.
- 때로, 이것은 틀립니다. 예를 들어, 당신은 키를 입력할 때마다 컴포넌트 페이드인 애니메이션을 원치 않습니다. 애니메이션은 오직 컴포넌트가 처음 등장했을 때 실행되어야 합니다.

이 이슈를 증명하기 위해, 이전 예제에 부모 컴포넌트의 상태를 변경하는 인풋과 콘솔로그를 추가했습니다.

```jsx
import { useState, useRef, useEffect } from "react";

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log("Calling video.play()");
      ref.current.play();
    } else {
      console.log("Calling video.pause()");
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}
```

당신은 리액트에게 `useEffect`의 두 번째 인자로 배열을 전달해 불필요한 이펙트 재실행을 넘기라고 할 수 있습니다. 빈 배열을 추가해보세요.

```jsx
useEffect(() => {
  // ...
}, []);
```

당신은 `React Hook useEffect has a missing dependency: 'isPlaying'` 에러를 볼 수 있습니다.

이것은 이펙트가 `isPlaying` 프랍에 의존해 무엇을 할지 정하는데, 의존성이 명시되지 않아서입니다. 이것을 고치기 위해, `isPlaying`를 배열에 추가합니다.

```jsx
useEffect(() => {
  if (isPlaying) {
    // It's used here...
    // ...
  } else {
    // ...
  }
}, [isPlaying]); // ...so it must be declared here!
```

이제 모든 의존성이 선언되었기에, 에러가 없습니다.
`[isPlaying]`를 명시해 의존성 배열은 리액트에게 `isPlaying`이 이전 랜더와 동일하다면 이펙트 재실행을 무시하도록 알려줍니다.
인풋을 타입하는 것은 이펙트 재실행을 일으키지 않지만, 재생/멈춤은 일으킵니다.

```jsx
import { useState, useRef, useEffect } from "react";

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log("Calling video.play()");
      ref.current.play();
    } else {
      console.log("Calling video.pause()");
      ref.current.pause();
    }
  }, [isPlaying]);

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState("");
  return (
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? "Pause" : "Play"}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

의존성 배열은 여러 의존성을 포함할 수 있습니다. 리액트는 모든 의존성이 이전과 같다면 이펙트 재실행을 무시합니다.
리액트는 의존성의 값을 `object.is`로 비교합니다.

당신은 당신의 의존성을 고를 수 없습니다. 만약 리액트가 예상한 의존성과 다르다면 린트 에러를 보게될 것읍니다.
이것은 당신의 코드 에러를 잡는 데 도움을 줍니다. 코드가 재실행 되기 원치 않는 다면, 의존성이 필요 없도록 이펙트 코드를 변경합니다.

#### Pitfall

의존성 배열이 없는 것, 비어있는 것은 다르게 동작합니다.

```jsx
useEffect(() => {
  // This runs after every render
});

useEffect(() => {
  // This runs only on mount (when the component appears)
}, []);

useEffect(() => {
  // This runs on mount *and also* if either a or b have changed since the last render
}, [a, b]);
```

#### DEEP DIVE - Why was the ref omitted from the dependency array?

이 이펙트는 `ref`와 `isPlaying`를 동시에 사용하지만, `isPlaying`만 의존성으로 선언되었습니다.

이것은 `ref`는 매 랜더 시 항상 같은 객체를 갖기에 리액트가 보장하는 안정된 정체성을 갖기 때문입니다. 이것은 절대 변하지 않으므로, 이펙트를 재실행하지 않습니다. 그러므로 추가하든 안하든 문제 없습니다.

`useState`의 반환인 set 함수는 역시 안정된 정체성을 갖으므로, 의존성에서 생략되는 것을 자주 봅니다. 만약 린트가 의존성을 에러 없이 생략한다면, 이것은 안전합니다.

린트가 안정된 객체를 볼 경우만, 안정된 의존성을 생략 가능합니다.
예를 들어 `ref`가 부모 컴포넌트에서 왔다면, 당신은 의존성에 명시해야합니다.
부모 컴포넌트가 항상 같은 ref를 전달할지 모르기에 이것은 좋습니다.
그러니 당신의 이펙트는 어떤 ref가 전달되는 지에 달려있습니다.

### Step 3: Add cleanup if needed

다른 예를 생각해봅시다. 챗룸이 등장했을 때, 챗 서버와 연결하는 컴포넌트를 작성 중입니다. 당신은 `createConnection()`로 부터 `connect()`와 `disconnect()` 메서드를 반환 받습니다. 어떻게 유저 화면에 보일 동안 접속할 수 있을까요?

이펙트를 작성하며 시작

```jsx
useEffect(() => {
  const connection = createConnection();
  connection.connect();
});
```

매 리랜더 시 접속 하는 것은 느리므로 의존성 배열을 추가합니다.

```jsx
useEffect(() => {
  const connection = createConnection();
  connection.connect();
}, []);
```

이 이펙트 코드는 어떤 프랍이나 상태를 필요로 하지 않기에, 의존성 배열은 비었습니다. 이것은 리액트에게 컴포넌트가 오직 마운트 되었을 때 실행하라고 알려줍니다.

이 이펙트는 마운트 시에만 실행되어, `"✅ Connecting..."`가 한번 콘솔에 찍힐 것을 예상합니다.
하지만, 콘솔을 체크하면, `"✅ Connecting..."`이 두 번 찍힙니다.

챗룸이 큰 앱의 한 부분이라고 생각해보세요. 챗룸 페이지를 위한 여정을 할 것입니다. 컴포넌트가 마운트 되고 `connection.connect()`를 호출합니다.
그리고 유저가 다른 화면으로 이동한다고 생각해보세요. 챗룸 컴포넌트는 언마운트 됩니다. 최종적으로, 유저가 돌아오면 챗룸은 마운트 됩니다.
이것은 두 번째 접속을 설정합니다. - 하지만 첫 번째 접속은 끊어지지 않았습니다.
유저가 앱을 이동하면, 연결은 쌓이게 됩니다.

이런 버그는 테스트 없이 놓치기 쉽습니다. 신속하게 발견할 수 있도록 리액트는 개발 단계에서 모든 컴포넌트를 처음 마운트한 후 즉시 다시 마운트합니다.

`"✅ Connecting..."` 로그를 두 번 보는 것은 실제 이슈를 알 수 있게 도와줍니다: 당신의 코드는 컴포넌트가 언마운트 될 때 접속을 끊지 않았다.

이것을 고치기 위해, 정리 함수를 반환합니다.

```jsx
useEffect(() => {
  const connection = createConnection();
  connection.connect();
  return () => {
    connection.disconnect();
  };
}, []);
```

리액트는 이펙트가 다시 실행 되기 전에 정리 함수를 호출하고, 컴포넌트가 언마운트 될 때 호출됩니다.

당신은 개발환경에서 이제 3 개의 콘솔로그를 보게 됩니다.

1. "✅ Connecting..."
2. "❌ Disconnected."
3. "✅ Connecting..."

이것은 개발 환경에서 올바른 작동입니다. 리액트는 이동을 검증하고 당신의 코드를 멈추지 않습니다.
접속 해제와 다시 접속하는 것은 정확히 일어나야 하는 것입니다.
당신이 정리를 잘 구현한다면, 유저 화면에는 이펙트 한번, 여러번 실행, 정리의 차이가 없습니다.
리액트가 개발 환경에서 버그를 철저히 조사하기 위해 connect/disconnect 쌍이 존재합니다. 이것은 일반적입니다.

배포 환경에서, 당신은 "✅ Connecting..."를 한 번 보게 될 것입니다.
컴포넌트를 재마운트하는 것은 정리가 필요한 이펙트를 찾기 위해 오직 개발 환경에서만 실행됩니다. 당신은 개발 환경 행동을 끄기 위해 엄격 모드를 꺼도 됩니다. 하지만 우리는 켜두길 추천합니다. 이것은 버그를 찾아 줍니다.

## How to handle the Effect firing twice in development?

리액트는 개발 환경에서 의도적으로 리마운트되어 위의 예처럼 버그를 잡습니다.
올바른 질문은 어떻게 이펙트를 한번만 실행시키냐가 아닌 어떻게 리마운트 이후에 실행하느냐입니다.

일반적으로, 해답은 정리 함수를 구현하는 것입니다.
정리 함수는 이펙트가 하는 것을 멈춥니다. 룰은 유저는 이런 이펙트가 동작하는 것을 구분하지 못한다는 것입니다.

대부분의 이펙트는 아래의 패턴과 일치합니다.

### Controlling non-React widgets

때로 당신은 리액트로 작성되지 않은 UI 위젯을 추가해야합니다. 예를 들어, `setZoomLevel()` 메서드를 갖는 맵 컴포넌트를 추가할 수 있습니다.
그리고 줌 레벨을 `zoomLevel` 리액트의 상태 변수와 동기화하고 싶습니다.
당신의 이펙트는 다음과 같을 것입니다.

```jsx
useEffect(() => {
  const map = mapRef.current;
  map.setZoomLevel(zoomLevel);
}, [zoomLevel]);
```

이번에는 정리가 필요없다는 것을 기억하세요. 개발 환경에서 리액트는 이펙트를 두 번 호출할 것입니다. 하지만 똑같은 값의 `setZoomLevel`을 호출하는 것은 문제가 되지 않습니다.
이것은 살짝 느릴뿐, 이것은 배포시 재마운트 되지 않기에 문제가 되지 않습니다.

어떤 API는 한 줄에 두 번 호출하는 것을 허락하지 않습니다.
예를 들어 `<dialog>`의 빌트인 메서드인 `showModal`은 두 번 호출됩니다.
정리 함수를 만들어 닫도록 합니다.

```jsx
useEffect(() => {
  const dialog = dialogRef.current;
  dialog.showModal();
  return () => dialog.close();
}, []);
```

개발 환경에서, 이펙트는 `showModal`을 호출하고, 즉시 `close`한 후, 다시 `showModal`합니다. 이것은 당신이 배포 환경에서 유저들이 경험하는 것과 같습니다.

### Subscribing to events

만약 이펙트가 무언가 구독하면, 정리 함수가 구독을 해지합니다.

```jsx
useEffect(() => {
  function handleScroll(e) {
    console.log(window.scrollX, window.scrollY);
  }
  window.addEventListener("scroll", handleScroll);
  return () => window.removeEventListener("scroll", handleScroll);
}, []);
```

개발 환경에서, `addEventListener()`를 호출 -> `removeEventListener()` 호출 -> 다시 `addEventListener()`를 호출합니다.
오직 한번의 활성화된 구독이 있습니다. 배포 환경과 동일합니다.

### Triggering animations

이펙트가 어떤 애니메이션 동작을 취하면, 정리 함수는 그것을 기본 값으로 초기화합니다.

```jsx
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1; // Trigger the animation
  return () => {
    node.style.opacity = 0; // Reset to the initial value
  };
}, []);
```

개발 환경에서 투명도는 1 -> 0 -> 1로 설정됩니다. 배포 환경과 동일합니다.
만약 써드파티 애니메이션 라이브러리를 사용하면, 그것의 타임 라인을 초기 상태로 설정합니다.

### Fetching data

당신의 이펙트가 무언가 페치할 때, 정리 함수는 abort 하거나 무시합니다.

```jsx
useEffect(() => {
  let ignore = false;

  async function startFetching() {
    const json = await fetchTodos(userId);
    if (!ignore) {
      setTodos(json);
    }
  }

  startFetching();

  return () => {
    ignore = true;
  };
}, [userId]);
```

정리 함수가 그 페치가 더이상 앱에 관련이 없고 가질 필요가 없다고 확신하면, 당신은 이미 요청한 네트워크를 취소할 수 있습니다.
만약 유저 아이디가 앨리스에서 밥으로 바꼈다면, 정리는 밥 이후에 앨리스의 응답이 와도 무시합니다.

개발 환경에서, 당신은 네트웨크 탭에서 두 번의 페치들을 봅니다. 그것은 문제가 아닙니다.

배포 환경에서, 오직 하나의 요청만 있습니다. 만약 개발 환경에서 두 번째 요청이 당신을 귀찮게 하면, 요청을 복제하거나 캐싱하는 것입니다.

```jsx
function TodoList() {
  const todos = useSomeDataLibrary(`/api/user/${userId}/todos`);
  // ...
```

이것은 개발 경험을 올리는 것 뿐 아니라 앱을 빠르게 합니다. 예를 들어
유저가 뒤로가기를 눌렀을 때, 데이터를 기다릴 필요없이 캐시에서 가져옵니다.
당신의 고유한 캐시를 만들 수 있고 이펙트 안에서 수동적으로 처리할 수 있습니다.

#### DEEP DIVE - What are good alternatives to data fetching in Effects?

`fetch`를 이펙트 안에서 호출하는 것은 특히 클라이언트 사이드에서 인기있는 방법이다. 하지만, 이것은 중대한 단점들이 있습니다.

- 이펙트는 서버에서 동작하지 않음. 이것은 서버에서 렌더된 HTML은 데이터 없이 로딩 상태만 갖는 것을 말합니다. 클라이언트 컴퓨터는 모든 자바스크립트를 다운로드하고 앱을 랜더링해야 데이터를 읽어올 수 있습니다. 이것은 그다지 효율적이지 않습니다.
- 이펙트 안에서 데이터 페칭은 네트워크 워터폴을 유발. 부모 컴포넌트가 랜더 되고 데이터를 페치합니다. 그리고 자식이 랜더 되고 데이터를 페치합니다. 만약 네트워크가 느리다면 이것은 데이터를 병렬로 가져오는 것 보다 매우 느릴 것입니다.
- 이펙트 안에서 페치하는 것은 프리로드나 캐시하지 않는다는 것. 예를 들어, 컴포넌트가 언마운트 되고 다시 마운트 되면 데이터를 다시 페치합니다.
- 인체 공학적이지 않음. 페치를 위한 제법 많은 race conditions으로 부터 고통받지 않을 보일러플레이트 코드가 필요합니다.

이런 단점 목록은 리액트만 해당되지 않습니다. 모든 라이브러리를 사용하여 마운트에서 데이터를 가져오는 경우 적용됩니다. 데이터를 잘 가져오기 위해 다음과 같은 방법을 권장합니다.

- 프레임워크를 사용한다면, 빌트인 데이터 페치 메커니즘을 사용하세요. 모던 리액트 프레임워크들은 통합된 데이터 페칭을 메커니즘이 있으며 이것은 효율적이며 위와 같은 단점에 고통받지 않습니다.
- 클라이언트 사이드 캐시를 고려하세요. 유명한 오픈 소스는 리액트 쿼리, swr 그리고 리액트 라우터 6.4+ 등이 있습니다. 당신은 당신의 고유한 해결책을 만들 수 있습니다. 이펙트 안에서 하지만 복제된 요청 로직을 추가하고, 응답을 캐시하고 네트워크 워터폴을 피하는.

당신은 위 같은 접근이 맞지 않는다면 이펙트를 써도 됩니다.

### Sending analytics

페이지를 방문하면 분석을 보내는 이벤트가 있다고 생각하세요.

```jsx
useEffect(() => {
  logVisit(url); // Sends a POST request
}, [url]);
```

개발 환경에서 두 번 호출되는데, 우리는 그냥 놔두길 권장합니다.

배포 환경에서, 복제된 방문 로그는 없습니다.
분석을 디버그 하기 위해 스테이지 환경에 배포하거나 임시로 엄격 모드를 제거할 수 있습니다. 당신은 라우트 변경 이벤트에 분석을 추가할 수 있습니다. 좀 더 정확한 분석을 위해 intersection observers로 남은 뷰포트를 분석할 수 있습니다.

### Not an Effect: Initializing the application

앱 실행시 오직 한번만 실행되는 로직들이 있습니다. 당신은 컴포넌트 밖에 둘 수 있습니다.

```jsx
if (typeof window !== "undefined") {
  // Check if we're running in the browser.
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

이것은 브라우저가 페이지를 로드한 후 오직 한번 실행되는 것을 보장합니다.

### Not an Effect: Buying a product

때로, 정리 함수를 작성해도 이펙트가 두 번 발생하는 것을 막을 수 없을 때가 있습니다. 예를 들어, 이펙트가 POST 요청을 보냈을 때:

```jsx
useEffect(() => {
  // 🔴 Wrong: This Effect fires twice in development, exposing a problem in the code.
  fetch("/api/buy", { method: "POST" });
}, []);
```

당신은 물건을 두 번 구매하고 싶지 않습니다. 하지만, 이것은 당신이 이펙트에 이런 로직을 추가하면 안되는 이유이긷 합니다.
만약 유저가 뒤로 갔다가 다시 돌아온다면? 당신의 이펙트는 다시 실행됩니다.
당신은 유저가 페이지에 방문하면 물건을 사길 원치 않습니다. 버튼 클릭을 통해 구매하길 원합니다.

구매는 랜더링에 의한 것이 아닙니다. 이것은 상호작용에 의해 일어납니다.
이것은 유저가 버튼을 눌렀을 때 일어나야합니다. 이펙트에서 이것을 지운 후 버튼의 이벤트 핸들러에 추가합니다.

이것은 리마운팅이 당신 앱의 로직을 고장내면, 이것은 존재된 버그를 밝힙니다.
사용자 입장에서 페이지 방문이 링크를 클릭후 뒤로 누르는 것과 다를 수 없습니다.
리액트는 개발 중인 컴포넌트를 리마운트하여 컴포넌트가 원칙을 준수했는지 확인합니다.

## Putting it all together

이펙트가 어떻게 동작하는지 느껴볼 수 있습니다.

이 예에서는 `setTimeout`를 활용해 이펙트가 실행된 후 3초 인풋 텍스트가 나오도록 스케줄합니다. 정리 함수는 대기중인 타임아웃을 취소합니다.
Mount the component를 클릭해보세요.

```jsx
import { useState, useEffect } from "react";

function Playground() {
  const [text, setText] = useState("a");

  useEffect(() => {
    function onTimeout() {
      console.log("⏰ " + text);
    }

    console.log('🔵 Schedule "' + text + '" log');
    const timeoutId = setTimeout(onTimeout, 3000);

    return () => {
      console.log('🟡 Cancel "' + text + '" log');
      clearTimeout(timeoutId);
    };
  }, [text]);

  return (
    <>
      <label>
        What to log:{" "}
        <input value={text} onChange={(e) => setText(e.target.value)} />
      </label>
      <h1>{text}</h1>
    </>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? "Unmount" : "Mount"} the component
      </button>
      {show && <hr />}
      {show && <Playground />}
    </>
  );
}
```

먼저 Schedule "a" log -> Cancel "a" log, -> Schedule "a" log를 다시 볼 것입니다.
3초 후 a라고 말하는 로그를 볼 것입니다. 앞서 배웠듯, schedule/cancel 쌍은 리액트가 개발 환경시 정리 함수가 잘 구현되었는지 검증하기 위해서입니다.

인풋을 `abc`로 변경합니다. 빠르게 작성하면 "ab" log -> Cancel "ab" log, -> Schedule "abc" log를 볼 것입니다. 리액트는 항상 이전의 이펙트를 다음 랜더의 이펙트 전에 정리합니다. 따라서 빠르게 입력하더라도 한 번에 최대 하나의 시간 초과가 예약됩니다. 인풋을 몇 초에 걸쳐 수정하고 이펙트가 정리하는 것을 콘솔로 보세요.

인풋에 무언가 타입하고 즉시 Unmount the component를 클릭하세요.
어떻게 언마운트가 마지막 랜더의 이펙트를 정리하는지 보세요.
마지막 타임아웃을 호출 되기 전에 제거합니다.

마지막으로, 정리 함수를 주석처리 하고 타임아웃이 취소되지 않도록 하세요.
abcde를 빠르게 타입해보세요. 3초 뒤에 무슨일이 벌어질까요?
최신의 `text`인 다섯개의 abcde를 콘솔로 프린트 할까요?

3초 뒤, 당신은 다섯개의 abcde 대신 순서대로 로그를 볼 것입니다. 각 이펙트는 상응하는 `text` 값을 갖습니다. 이것은 `text`가 변화하든 안하든 상관없습니다. `text = 'ab'`이면 항상 `'ab'`를 봅니다.
각 랜더의 이펙트들은 서로 독립적입니다. 이것이 어떻게 일어나는지 궁금하면, 클로저를 읽어보세요.

#### DEEP DIVE - Each render has its own Effects

당신은 `useEffect`가 랜더 결과물에 동작을 붙인다고 생각할 것입니다.

```jsx
export default function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>Welcome to {roomId}!</h1>;
}
```

유저가 앱을 이동할 때 정확히 무엇이 일어나는지 봅시다.

##### Initial render

유저가 `<ChatRoom roomId="general" />`를 방문합니다.
`roomID`를 general이라고 대체합니다.

```jsx
// JSX for the first render (roomId = "general")
return <h1>Welcome to general!</h1>;
```

이펙트는 랜더의 결과물입니다. 첫 랜더의 이펙트는 다음과 같습니다:

```jsx
// Effect for the first render (roomId = "general")
() => {
  const connection = createConnection("general");
  connection.connect();
  return () => connection.disconnect();
},
  // Dependencies for the first render (roomId = "general")
  ["general"];
```

리액트는 이펙트를 실행해, general 챗룸과 연결합니다.

#### Re-render with same dependencies

`<ChatRoom roomId="general" />`가 리랜더 된다고 생각합시다.
JSX 결과는 같습니다.

```jsx
// JSX for the second render (roomId = "general")
return <h1>Welcome to general!</h1>;
```

리액트는 랜더 결과물이 변경되지 않은 것을 보고, 돔을 업데이트 하지 않습니다.

두 번째 랜더의 이펙트는 다음과 같습니다.

```jsx
// Effect for the second render (roomId = "general")
() => {
  const connection = createConnection("general");
  connection.connect();
  return () => connection.disconnect();
},
  // Dependencies for the second render (roomId = "general")
  ["general"];
```

리액트는 첫 랜더의 `["general"]`와 두 번째 `["general"]`를 비교합니다.
모든 의존성이 같기에, 리액트는 두 번째 랜더에 이펙트 실행을 무시합니다. 이것은 절대 호출되지 않습니다.

#### Re-render with different dependencies

유저가 `<ChatRoom roomId="travel" />`에 방문했을 경우.
이번에는 다른 JSX를 반환합니다.

```jsx
// JSX for the third render (roomId = "travel")
return <h1>Welcome to travel!</h1>;
```

리액트는 돔을 `"Welcome to general"`에서 `"Welcome to travel"`로 업데이트합니다.

세 번째 랜더의 이펙트는 다음과 같습니다.

```jsx
// Effect for the third render (roomId = "travel")
() => {
  const connection = createConnection("travel");
  connection.connect();
  return () => connection.disconnect();
},
  // Dependencies for the third render (roomId = "travel")
  ["travel"];
```

리액트는 세 번째 랜더의 `['travel']`와 두 번째 랜더의 `['general']`를 비교합니다. 의존성이 다르기에 `Object.is('travel', 'general')`는 false입니다. 이펙트는 무시될 수 없습니다.

세 번째 랜더의 이펙트를 적용하기 전에, 마지막 이펙트를 정리할 필요가 있으며 실행됩니다. 두 번째 이펙트는 무시되므로, 리액트는 첫 번째 랜더의 이펙트를 정리합니다. 정리 함수가 `disconnect()`를 호출한다는 것을 알 것입니다.
이것은 `'general'` 챗 룸의 연결을 해제합니다.

그 후, 리액트는 세 번째 랜더의 이펙트를 실행합니다. 이것은 `'travel'`을 연결합니다.

#### Unmount

마지막으로, 유저가 다른 곳으로 이동하고, 챗룸 컴포넌트가 언마운트 됐습니다.
리액트는 마지막 이펙트의 정리 함수를 실행합니다. 세 번째 랜더에서 온 마지막 이펙트입니다. 세 번째 랜더의 정리는 `createConnection('travel')` 접속을 끊습니다. 앱은 `'travel'` 룸과 접속이 끊겼습니다.

#### Development-only behaviors

엄격 모드시 리액트는 모든 컴포넌트를 마운트 후 리마운트 합니다.
이것은 정리가 필요한 이펙트를 찾도록 도와주고 버그를 일찍 노출시킵니다.
게다가, 리액트는 파일을 저장할 때마다 리마운트합니다. 이 행동들은 오직 개발 환경에서만 일어납니다.

## Recap

- 이벤트와 다르게, 이펙트는 상호작용이 아닌 랜더 그자체에서 일어남
- 이펙트는 컴포넌트가 다른 외부 시스템과 동기화할 수 있도록 해줌
- 기본적으로, 이펙트는 매 랜더시 발생
- 리액트는 마지막 랜더와 의존성이 같으면 이펙트를 무시
- 당신은 의존성을 선택할 수 있으며, 그것은 이펙트 안 코드에서 결정
- 빈 의존성 배열은 컴포넌트 마운팅과 상응
- 엄격 모드시, 리액트는 컴포넌트를 두 번 마운트해 이펙트를 테스트
- 만약 리마운트로 앱이 고장나면, 정리 함수를 구현해야함
- 리액트는 정리 함수를 이펙트가 다음에 일어나기 전과 언마운트 시 호출

---

## What I Learned

리액트의 이펙트는 외부와 싱크를 맞추기 위한 것이다.
이벤트와 다르게 랜더 자체에서 일어나며, 기본적으론 매 랜더시 발생한다.
의존성 배열을 추가해 원하는 의존성에 때라 이펙트를 발생시킬 수 있다.
클린업 함수를 통해 로직을 정리할 수 있다. 이때 클린업은 다음 랜더가 일어나기 전과 언마운트 시 발생한다.

출처

- https://react.dev/learn/synchronizing-with-effects
