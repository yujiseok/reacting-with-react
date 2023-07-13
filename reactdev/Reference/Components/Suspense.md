# \<Suspense>

서스펜스는 자식의 로딩이 종료 될 때까지 대체를 보여주게 한다.

```jsx
<Suspense fallback={<Loading />}>
  <SomeComponent />
</Suspense>
```

## Reference

### <Suspense>

**Props**

- children: 실제로 랜더를 의도한 UI. 만약 자식이 랜더 중 지연되면, 서스펜스 바운더리가 대체 UI로 교체한다.
- fallback: 실제 UI가 로딩이 완료될 때까지 대체되는 UI. 리액트의 어떤 노드도 받아지고, 대체는 가벼운 플레이스 홀더 예를 들어, 로딩 스피너 혹은 스켈레톤이다. 서스펜스는 자동으로 자식이 지연될 경우 대체로 교체하고, 데이터가 준비가 되면 자식으로 되돌린다. 만약 대체가 랜더중에 지연되면, 부모 서스펜스 바운더리를 작동시킨다.

**Caveats**

- 리액트는 랜더링이 처음으로 마운트되기 전에 일시 중단된 랜더링에 대한 상태를 보존하지 않는다. 컴포넌트가 로드 되었을 때, 리액트는 지연된 트리를 처음부터 다시 랜더를 시도한다.
- 만약 서스펜스가 트리를 위해 컨텐츠를 보여주지만, 지연이 계속될 경우, startTransition나 useDeferredValue로 변경이 일어나지 않는한 fallback이 계속 보여질 것이다.
- 다시 지연이 되는 것에서 이미 보여지는 컨텐츠를 숨기기 위해선, 컨텐츠 트리의 레이아웃 이펙트에서 정리 될 수 있다. 컨텐츠가 다시 보여지는 것이 준비될 경우, 리액트는 레이아웃 이펙트를 재실행한다. 컨텐츠가 보이지 않을 경우 사용하지 마라. 이것은 돔의 레이아웃을 측정하는 것을 보장한다.
- 리액트는 스트리밍 서버 랜더와 셀렉티브 하이드레이션과 서스펜스가 통합된 내부적으로 최적화를 포함한다.

## Usage

### Displaying a fallback while content is loading

애플리케이션의 어떤 부분이든 서스펜스 바운더리로 감쌀 수 있다.

```jsx
<Suspense fallback={<Loading />}>
  <Albums />
</Suspense>
```

리액트는 자식의 모든 코드와 데이터가 로드될 때까지 로딩을 보여준다.

아래 예에서, 앨범 컴포넌트는 앨범이 페치될 때까지 지연된다.
랜더 준비가 될때까지, 리액트는 상위의 가까운 서스펜스 바운더리의 fallback 컴포넌트를 보여준다.
데이터가 로드가 되고, 리액트는 로딩 fallback을 숨긴 후, 앨범 컴포넌트를 데이터와 랜더한다.

```jsx
import { Suspense } from "react";
import Albums from "./Albums.js";

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Suspense fallback={<Loading />}>
        <Albums artistId={artist.id} />
      </Suspense>
    </>
  );
}

function Loading() {
  return <h2>🌀 Loading...</h2>;
}
```

#### Note

서스펜스가 가능한 데이터 소스는 오직 서스펜스 컴포넌트에서 동작한다.

- 데이터 페칭이 서스펜스와 가능한 프레임워크 relay와 next.js
- lazy 코드로 작성된 lazy-loading 컴포넌트

서스펜스는 이펙트나 이벤트 핸들러 내부의 데이터 페치는 발견하지 못한다.

위의 앨범 컴포넌트의 데이터를 올바르게 로드하는 것은 프레임워크에 달렸다.
만약 서스펜스가 가능한 프레임워크를 쓸 경우, 데이터 페칭 문서에서 디테일을 알 수 있다.

프레임워크 없이 데이터 페칭을 하는 것은 아직 지원되지 않는다. 서스펜스 데이터 소스를 구현하기 위한 요구사항이 불안정하고 문서화되지 않았습니다.
서스펜스와 함께 데이터 소스를 가져오는 통합 공식 API는 리액트의 미래 버전에 배포될 것이다.

### Revealing content together at once

기본적으로, 서스펜스 내부의 트리는 하나의 유닛으로 여겨진다. 예를 들어, 오직 하나의 컴포넌트만 데이터를 기다리더라도, 모든 컴포넌트가 로딩 인디케이터로 대체된다.

모든 것이 보여질 준비가 되면, 그것들은 동시에 나타난다.

예에서 바이오그래피와 앨범은 데이터를 페치한다. 하지만, 그들은 서스펜스 내부에 있기에, 항상 동시에 나타난다.

```jsx
import { Suspense } from "react";
import Albums from "./Albums.js";
import Biography from "./Biography.js";
import Panel from "./Panel.js";

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Suspense fallback={<Loading />}>
        <Biography artistId={artist.id} />
        <Panel>
          <Albums artistId={artist.id} />
        </Panel>
      </Suspense>
    </>
  );
}

function Loading() {
  return <h2>🌀 Loading...</h2>;
}
```

데이터를 로드하는 컴포넌트가 서스펜스 바운더리의 직접적인 자식일 필요는 없다. 예를 들어, 바이오그래피와 앨범을 새로운 디테일 컴포넌트로 옮길 수 있다. 이것은 동작을 변경하지 않는다. 바이오그래피와 앨범은 동일한 상위의 서스펜스 바운더리를 공유하고, 함께 보여진다.

```jsx
<Suspense fallback={<Loading />}>
  <Details artistId={artist.id} />
</Suspense>;

function Details({ artistId }) {
  return (
    <>
      <Biography artistId={artistId} />
      <Panel>
        <Albums artistId={artistId} />
      </Panel>
    </>
  );
}
```

### Revealing nested content as it loads

컴포넌트가 지연될 시, 가까운 부모 서스펜스 컴포넌트의 폴백을 보여준다. 이것은 중첩된 여러 서스펜스 컴포넌트가 로딩 시퀀스를 만든다는 뜻이다.
각 서스펜스 바운더리의 폴백은 상위 레벨의 컨텐츠가 가능해질 경우 채워진다.

```jsx
<Suspense fallback={<BigSpinner />}>
  <Biography />
  <Suspense fallback={<AlbumsGlimmer />}>
    <Panel>
      <Albums />
    </Panel>
  </Suspense>
</Suspense>
```

이 변경을 통해 바이오그래피는 앨범이 로드 되는 것을 기다릴 필요가 없다.

시퀀스는 다음과 같다.

1. 만약 바이오그래피가 로드 되지 않았다면, 빅스피너가 전체 컨텐츠를 대체한다.
2. 바이오그래피가 로딩이 끝나면, 빅스피너는 컨텐츠로 대체된다.
3. 만약 앨범이 로드 되지 않았다면, 앨범글리머가 앨범과 부모인 패널에 보여진다.
4. 최종적으로, 앨범의 로딩이 끝나면, 앨범글리머를 대체한다.

```jsx
import { Suspense } from "react";
import Albums from "./Albums.js";
import Biography from "./Biography.js";
import Panel from "./Panel.js";

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Suspense fallback={<BigSpinner />}>
        <Biography artistId={artist.id} />
        <Suspense fallback={<AlbumsGlimmer />}>
          <Panel>
            <Albums artistId={artist.id} />
          </Panel>
        </Suspense>
      </Suspense>
    </>
  );
}

function BigSpinner() {
  return <h2>🌀 Loading...</h2>;
}

function AlbumsGlimmer() {
  return (
    <div className="glimmer-panel">
      <div className="glimmer-line" />
      <div className="glimmer-line" />
      <div className="glimmer-line" />
    </div>
  );
}
```

서스펜스 바운더리는 동시에 UI의 부분을 보여줄지와 우선순위를 두어 컨텐츠를 보여줄지 정할 수 있게 해준다.
서스펜스 바운더리를 추가, 이동, 삭제해 나머지 앱의 동작에 영향을 끼치지 않고 트리의 어떤 위치에든 둘 수 있다.

모든 컴포넌트를 서스펜스 바운더리에 두지 마라. 서스펜스 바운더리는 사용자가 경험하기를 원하는 로딩 시퀀스보다 세분화되면 안된다.
만약 당신이 디자이너와 일한다면, 로딩 상태를 어디에 둘지 물어봐라.

### Showing stale content while fresh content is loading

예에서, SearchResults는 검색 결과를 페치하기까지 중단된다.
a를 입력한 후 결과를 기다리고, ab로 수정해라. a의 결과가 로딩 폴백으로 대체될 것이다.

```jsx
import { Suspense, useState } from "react";
import SearchResults from "./SearchResults.js";

export default function App() {
  const [query, setQuery] = useState("");
  return (
    <>
      <label>
        Search albums:
        <input value={query} onChange={(e) => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Loading...</h2>}>
        <SearchResults query={query} />
      </Suspense>
    </>
  );
}
```

일반적인 대체 UI 패턴은 이전의 결과가 새로운 결과가 준비될 때까지 유지되는 것이다.
useDeferredValue 훅은 지연된 버전의 쿼리를 전달할 수 있게 해준다.

```jsx
export default function App() {
  const [query, setQuery] = useState("");
  const deferredQuery = useDeferredValue(query);
  return (
    <>
      <label>
        Search albums:
        <input value={query} onChange={(e) => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Loading...</h2>}>
        <SearchResults query={deferredQuery} />
      </Suspense>
    </>
  );
}
```

쿼리는 즉각 업데이트 되므로 인풋은 새로운 값을 보인다. 하지만, deferredQuery는 데이터가 로드될 때까지 이전 값을 유지한다.
즉, SearchResults가 이전의 결과를 잠시 보여준다.

유저에게 좀 더 명백하게 하기 위해, 시각적 지침을 추가한 상태 결과를 보여줄 수 있다.

```jsx
import { Suspense, useState, useDeferredValue } from "react";
import SearchResults from "./SearchResults.js";

export default function App() {
  const [query, setQuery] = useState("");
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;
  return (
    <>
      <label>
        Search albums:
        <input value={query} onChange={(e) => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Loading...</h2>}>
        <div style={{ opacity: isStale ? 0.5 : 1 }}>
          <SearchResults query={deferredQuery} />
        </div>
      </Suspense>
    </>
  );
}
```

#### Note

지연된 값과 트랜지션은 서스펜스이 대체를 보여주는 것을 피하게할 수 있다.
트랜지션은 업데이트 자체를 급하지 않은 것으로 마크할 수 있다. 프레임워크나 네비게이션을 위한 라우터 라이브러리에 사용된다.
지연된 값은 반면에, 애플리케이션 코드에서 유용하며 UI의 부분을 급하지 않고 지연된 UI로 지정할 수 있다.

### Preventing already revealed content from hiding

컴포넌트가 중단 되었을 경우, 가까운 부모 서스펜스 바운더리가 폴백으로 대체한다.
이것은 이미 보여지는 컴포넌트에 대해 부조화된 유저 경험을 유발할 수 있다.

```jsx
import { Suspense, useState } from "react";
import IndexPage from "./IndexPage.js";
import ArtistPage from "./ArtistPage.js";
import Layout from "./Layout.js";

export default function App() {
  return (
    <Suspense fallback={<BigSpinner />}>
      <Router />
    </Suspense>
  );
}

function Router() {
  const [page, setPage] = useState("/");

  function navigate(url) {
    setPage(url);
  }

  let content;
  if (page === "/") {
    content = <IndexPage navigate={navigate} />;
  } else if (page === "/the-beatles") {
    content = (
      <ArtistPage
        artist={{
          id: "the-beatles",
          name: "The Beatles",
        }}
      />
    );
  }
  return <Layout>{content}</Layout>;
}

function BigSpinner() {
  return <h2>🌀 Loading...</h2>;
}
```

버튼을 누를 시, 라우터 컴포넌트는 인덱스페이지 대신 아티스트페이지를 랜더한다.
아티스트페이지의 컴포넌트는 중단되고, 가까운 서스펜스 바운더리는 폴백을 보여준다.
가장 가까운 서스펜스 바운더리는 루트와 가깝기에, 모든 사이트의 레이아웃이 빅스피너로 대체된다.

이것을 막기 위해, 네비게이션 상태 변경을 startTransition을 사용해 트랜지션으로 할 수 있다.

```jsx
function Router() {
  const [page, setPage] = useState('/');

  function navigate(url) {
    startTransition(() => {
      setPage(url);
    });
  }
  // ...
```

이것은 리액트에게 상태 트랜지션이 급한것이 아니며, 이전 페이지를 숨기지 않고 보여주는 것이 더 좋다고 알려준다.

```jsx
import { Suspense, startTransition, useState } from "react";
import IndexPage from "./IndexPage.js";
import ArtistPage from "./ArtistPage.js";
import Layout from "./Layout.js";

export default function App() {
  return (
    <Suspense fallback={<BigSpinner />}>
      <Router />
    </Suspense>
  );
}

function Router() {
  const [page, setPage] = useState("/");

  function navigate(url) {
    startTransition(() => {
      setPage(url);
    });
  }

  let content;
  if (page === "/") {
    content = <IndexPage navigate={navigate} />;
  } else if (page === "/the-beatles") {
    content = (
      <ArtistPage
        artist={{
          id: "the-beatles",
          name: "The Beatles",
        }}
      />
    );
  }
  return <Layout>{content}</Layout>;
}

function BigSpinner() {
  return <h2>🌀 Loading...</h2>;
}
```

트랜지션은 모든 컨텐츠가 로드되는 것을 기다리지 않는다. 이것은 이미 보여진 컨텐츠가 숨겨지지 않을 만큼 기다린다.
예를 들어, 레이아웃이 이미 보여졌다면, 로딩 스피너로 가리는 것은 좋지 않다. 하지만, 중첩된 서스펜스 바운더리의 앨범은 새롭기에, 트랜지션이 기다리지 않는다.

#### Note

서스펜스가 가능한 라우터는 기본적으로 네비게이션 업데이트의 트랜지션을 포함한다.

## #Indicating that a transition is happening

위의 예에서 버튼을 누르면, 네비게이션의 진행을 시각적으로 볼 수 없다.
지침을 추가하기 위해, startTransition을 isPending 값을 주는 useTransition로 교체한다.
아래 예에서 네비게이션시 헤더의 스타일이 변경된다.

```jsx
import { Suspense, useState, useTransition } from "react";
import IndexPage from "./IndexPage.js";
import ArtistPage from "./ArtistPage.js";
import Layout from "./Layout.js";

export default function App() {
  return (
    <Suspense fallback={<BigSpinner />}>
      <Router />
    </Suspense>
  );
}

function Router() {
  const [page, setPage] = useState("/");
  const [isPending, startTransition] = useTransition();

  function navigate(url) {
    startTransition(() => {
      setPage(url);
    });
  }

  let content;
  if (page === "/") {
    content = <IndexPage navigate={navigate} />;
  } else if (page === "/the-beatles") {
    content = (
      <ArtistPage
        artist={{
          id: "the-beatles",
          name: "The Beatles",
        }}
      />
    );
  }
  return <Layout isPending={isPending}>{content}</Layout>;
}

function BigSpinner() {
  return <h2>🌀 Loading...</h2>;
}
```

### Resetting Suspense boundaries on navigation

트랜지션 동안, 리액트는 이미 보여진 컨텐츠를 숨기는 것을 피할 것이다. 하지만 다른 파라미터로 이동하면, 리액트에게 다른 컨텐츠라고 알리고 싶을 것이다. 키를 통해 표현할 수 있다.

```jsx
<ProfilePage key={queryParams.id} />
```

유저의 프로필 페이지로 이동 중에 중단이 있을 경우를 상상하자. 만약 변경이 트랜지션으로 감싸졌다면, 이것은 이미 보여진 컨텐츠이기에 폴백을 일으키지 않는다. 이것은 예측된 동작이다.

하지만, 두 개의 다른 프로필로 이동하는 것을 상상하자. 이 경우, 폴백을 보여주는 것이 말이 된다. 예를 들어, 한 유저의 타임라인이 다른 유저의 타임라인과 다를 경우. 키를 특정해, 리액트에게 다른 유저의 프로필을 다른 컴포넌트로, 네비게이션 도중 서스펜스 바운더리를 초기화할 수 있도록 보장한다. 서스펜스가 통합된 라우터는 이것을 자동으로 해준다.

### Providing a fallback for server errors and server-only content

만약 서버 랜더 API를 사용하거나 프레임워크를 사용한다면, 리액트는 서버의 에러를 다룰 수 있게 한다.
만약 컴포넌트가 서버에서 에러를 발생하면, 리액트는 서버 랜더를 중단한다.
반면, 폴백을 포함한 가까운 서스펜스 컴포넌트를 찾아 서버 html을 생성한다. 유저는 스피너를 먼저 볼 것이다.

클라이언트에서, 리액트는 동일 컴포넌트의 랜더를 다시 시도할 것이다. 만약 클라이언트에 에러가 발생하면, 리액트는 에러를 발생하고 가까운 에러 바운더리에 보여준다. 하지만, 클라이언트에서 에러가 없을 시, 리액트는 결과적으로 컨텐츠가 성공적으로 표시될 때까지 에러를 보여주지 않는다.

서버에서 랜더 되는 컴포넌트에 이것을 적용할 수 있다. 이것을 실행하기 위해, 서버 환경에서 에러를 발생시키고, 서스펜스로 감싼 뒤 html을 대체해라.

```jsx
<Suspense fallback={<Loading />}>
  <Chat />
</Suspense>;

function Chat() {
  if (typeof window === "undefined") {
    throw Error("Chat should only render on the client.");
  }
  // ...
}
```

서버 html은 로딩 인디케이터를 포함한다. 이것은 클라이언트에서 챗 컴포넌트로 대체된다.

## Troubleshooting

### How do I prevent the UI from being replaced by a fallback during an update?

보이는 UI를 폴백으로 대체하는 것은 유저에게 부조화된 경험을 준다. 이것은 변경이 일어나 컴포넌트가 중단되고, 가까운 서스펜스 바운더리가 컨텐츠를 이미 보고있을 때 일어난다.

이것을 막기 위해, 변경을 급하지 않다고 startTransition으로 표시한다. 트랜지션 동안, 리액는 충분한 데이터가 로드 될 때까지 원치 않는 폴백이 등장하는 것을 방지한다.

```jsx
function handleNextPageClick() {
  // If this update suspends, don't hide the already displayed content
  startTransition(() => {
    setCurrentPage(currentPage + 1);
  });
}
```

이것은 존재하는 컨텐츠를 가리는 것을 방지한다. 하지만, 어떤 새로운 서스펜스 바운더리 랜더는 여전히 유저가 컨텐츠를 볼 수 있을 때까지 폴팩을 보여준다.

리액트는 오직 급하지 않는 변경에서만 원치 않는 폴백을 방지한다. 긴급한 변경의 랜더 결과를 미루지 않는다.
startTransition 또는 useDeferredValue를 반드시 선택해야한다.

---

## What I Learned

서스펜스와 리액트 쿼리를 사용해 비동기를 선언적으로 처리한 경험이 있다.
서스펜스는 중단 혹은 지연된 컴포넌트의 데이터가 로드 될 때까지, 대체 UI를 보여주는 컴포넌트이다.
