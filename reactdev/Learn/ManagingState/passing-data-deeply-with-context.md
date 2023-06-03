- [Passing Data Deeply with Context](#passing-data-deeply-with-context)
  - [The problem with passing props](#the-problem-with-passing-props)
  - [Context: an alternative to passing props](#context-an-alternative-to-passing-props)
    - [Step 1: Create the context](#step-1-create-the-context)
    - [Step 2: Use the context](#step-2-use-the-context)
    - [Step 3: Provide the context](#step-3-provide-the-context)
  - [Using and providing context from the same component](#using-and-providing-context-from-the-same-component)
  - [Context passes through intermediate components](#context-passes-through-intermediate-components)
  - [Before you use context](#before-you-use-context)
  - [Use cases for context](#use-cases-for-context)
  - [Recap](#recap)
  - [What I Learned](#what-i-learned)

# Passing Data Deeply with Context

보통, 당신은 부모에서 받은 정보를 자식 컴포넌트에 프랍으로 넘겨줄 수 있습니다.
그러나 많은 컴포넌트의 중간이나 많은 컴포넌트에 동일한 정보가 필요하다면, 프랍 전달은 장황해질 수 있습니다.
컨텍스트는 부모 컴포넌트가 정보를 트리 하위의 어떤 컴포넌트에 접근할 수 있도록 해줍니다. - 명시적은 프랍 전달 없이

## The problem with passing props

UI 트리를 통해 프랍을 사용하는 컴포넌트에 명시적으로 전달하는 데이터 파이프는 아주 좋은 방법입니다.

하지만 트리의 깊은 곳에 프랍을 전달하고 많은 컴포넌트에 사용되면 장황하고 귀찮아집니다.
데이터가 필요한 컴포넌트는 가까운 조상에서 멀어질 수 있으며, 상태를 끌어 올리는 것은 프랍 드릴링을 유발합니다.

[label](https://react.dev/_next/image?url%3D%2Fimages%2Fdocs%2Fdiagrams%2Fpassing_data_lifting_state.dark.png%26w%3D1920%26q%3D75)

[label](https://react.dev/_next/image?url%3D%2Fimages%2Fdocs%2Fdiagrams%2Fpassing_data_prop_drilling.dark.png%26w%3D1920%26q%3D75)

트리상에 데이터가 필요한 컴포넌트에 프랍을 전달하는 것 대신 텔레포트가 있으면 좋겠지 않나요?
리액트의 컨텍스트가 바로 그것입니다.

## Context: an alternative to passing props

컨텍스트는 부모 컴포넌트가 트리 하위 전체에 데이터를 제공합니다.
컨텍스트에 많은 용도가 있습니다.
헤딩 컴포넌트가 사이즈를 위한 레벨을 받는 예제가 있습니다.

```jsx
import Heading from "./Heading.js";
import Section from "./Section.js";

export default function Page() {
  return (
    <Section>
      <Heading level={1}>Title</Heading>
      <Heading level={2}>Heading</Heading>
      <Heading level={3}>Sub-heading</Heading>
      <Heading level={4}>Sub-sub-heading</Heading>
      <Heading level={5}>Sub-sub-sub-heading</Heading>
      <Heading level={6}>Sub-sub-sub-sub-heading</Heading>
    </Section>
  );
}
```

당신은 같은 사이즈의 헤딩을 갖는 섹션을 갖고 싶어합니다.

```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>Title</Heading>
      <Section>
        <Heading level={2}>Heading</Heading>
        <Heading level={2}>Heading</Heading>
        <Heading level={2}>Heading</Heading>
        <Section>
          <Heading level={3}>Sub-heading</Heading>
          <Heading level={3}>Sub-heading</Heading>
          <Heading level={3}>Sub-heading</Heading>
          <Section>
            <Heading level={4}>Sub-sub-heading</Heading>
            <Heading level={4}>Sub-sub-heading</Heading>
            <Heading level={4}>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
```

현재 당신은 레벨 프랍을 각 헤딩에 따로 전달합니다.

```jsx
<Section>
  <Heading level={3}>About</Heading>
  <Heading level={3}>Photos</Heading>
  <Heading level={3}>Videos</Heading>
</Section>
```

섹션에서 레벨을 전달하고, 헤딩에서 제거한다면 좋을 것 같습니다.
이 방식은 같은 섹션의 모든 헤딩의 크기를 같게 강제할 수 있습니다.

```jsx
<Section level={3}>
  <Heading>About</Heading>
  <Heading>Photos</Heading>
  <Heading>Videos</Heading>
</Section>
```

하지만 어떻게 헤딩 컴포넌트가 가까운 섹션의 레벨을 알까요?
자식들은 트리의 상단에 있는 데이터의 요청이 필요합니다.

당신은 프랍만으로 이것을 할 수 없습니다. 이것은 컨텍스트의 영역입니다.
3 단계에 걸쳐 할 수 있습니다.

1. 컨텍스트 생성 (당신은 `LevelContext`라고 명명할 수 있으며, 헤딩의 레벨을 담당)
2. 데이터가 필요한 컴포넌트에서 컨텍스트 사용 (헤딩은 `LevelContext`를 사용)
3. 데이터를 특정하는 컴포넌트에 컨텍스트 제공 (섹션은 `LevelContext`를 제공)

컨텍스트는 부모(심지어 더 멀리 있는 것)가 트리 안 전체에 데이터를 제공합니다.

[label](https://react.dev/_next/image?url%3D%2Fimages%2Fdocs%2Fdiagrams%2Fpassing_data_context_close.dark.png%26w%3D1920%26q%3D75)

[label](https://react.dev/_next/image?url%3D%2Fimages%2Fdocs%2Fdiagrams%2Fpassing_data_context_far.dark.png%26w%3D1920%26q%3D75)

### Step 1: Create the context

먼저, 당신은 컨텍스트를 만들 필요가 있습니다. 당신은 파일로 분리하여 컴포넌트에서 사용할 수 있습니다.

```jsx
import { createContext } from "react";

export const LevelContext = createContext(1);
```

`createContext`의 유일한 인자는 기본값입니다.
`1`은 가장 큰 헤딩 레벨이지만, 당신은 어떤 값(객체 역시)도 전달할 수 있습니다.
당신은 초기값의 중요성을 다음 스텝에서 볼 것입니다.

### Step 2: Use the context

`useContext` 훅을 리액트에서 그리고 당신의 컨텍스트를 가져옵니다.

```jsx
import { useContext } from "react";
import { LevelContext } from "./LevelContext.js";
```

현재, 헤딩 컴포넌트는 레벨을 프랍으로 읽어옵니다.

```jsx
export default function Heading({ level, children }) {
  // ...
}
```

프랍으로 가져오는 레벨을 제거한 후 값을 당신이 가져온 컨텍스트, `LevelContext`에서 읽습니다.

```jsx
export default function Heading({ children }) {
  const level = useContext(LevelContext);
  // ...
}
```

`useContext`는 훅입니다. `useState`와 `useReducer` 같이, 리액트 컴포넌트에서 즉시 호출할 수 있습니다.
`useContext`는 리액트에게 헤딩 컴포넌트가 `LevelContext`를 읽고 싶다고 알려줍니다.

이제 헤딩 컴포넌트는 레벨 프랍이 없으며, 아래의 JSX와 같이 더 이상 레벨 프랍을 넘겨줄 필요가 없습니다.

```jsx
<Section>
  <Heading level={4}>Sub-sub-heading</Heading>
  <Heading level={4}>Sub-sub-heading</Heading>
  <Heading level={4}>Sub-sub-heading</Heading>
</Section>
```

섹션이 받을 수 있도록 JSX를 수정합니다.

```jsx
<Section level={4}>
  <Heading>Sub-sub-heading</Heading>
  <Heading>Sub-sub-heading</Heading>
  <Heading>Sub-sub-heading</Heading>
</Section>
```

이 예시는 아직 동작하지 않습니다. 컨텍스트를 사용하였지만 제공하지 않았기에 모든 헤딩은 똑같은 사이즈를 갖습니다.
리액트는 어디서 가져올지 모릅니다!

당신이 컨텍스트를 제공하지 않는 다면, 리액트는 이전 단계에서 설정한 기본값을 사용합니다. 기본값으로 1을 제공하였기에, 1을 반환하고 모든 헤딩은 `h1`이 됩니다.
각 섹션이 각자의 컨텍스트를 제공하도록 수정해봅시다.

### Step 3: Provide the context

섹션 컴포넌트는 그들의 자식을 랜더합니다.

```jsx
export default function Section({ children }) {
  return <section className="section">{children}</section>;
}
```

그들을 컨텍스트 프로바이더로 감싸 `LevelContext`를 제공하도록 합니다.

```jsx
import { LevelContext } from "./LevelContext.js";

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext.Provider value={level}>{children}</LevelContext.Provider>
    </section>
  );
}
```

이것은 리액트에게 "만약 섹션안의 어떤 컴포넌트든 `LevelContext`를 요청하면 레벨을 제공해"라고 말해줍니다.
UI 트리 상위에 `<LevelContext.Provider>`가 있는 컴포넌트는 값을 사용할 수 있습니다.

이것은 기존 코드와 똑같은 결과를 보이지만, 레벨을 헤딩 컴포넌트에 전달할 필요가 없습니다.
대신, 가까운 섹션에 헤딩의 레벨을 찾게됩니다.

1. 당신은 레벨을 섹션에 전달
2. 섹션은 `<LevelContext.Provider value={level}>`로 자녀를 감쌈
3. 헤딩은 가까운 `LevelContext`의 값을 `useContext(LevelContext)`를 통해 요청

## Using and providing context from the same component

현재, 당신은 레벨을 수동적으로 특정합니다.

```jsx
export default function Page() {
  return (
    <Section level={1}>
      ...
      <Section level={2}>
        ...
        <Section level={3}>
          ...
```

컨텍스트가 컴포넌트 상위에서 정보를 읽을 수 있게 해주므로, 각 섹션은 레벨을 각 섹션 상위에서 가져오며, 레벨 + 1을 자동적으로 전달합니다.

```jsx
import { useContext } from "react";
import { LevelContext } from "./LevelContext.js";

export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext.Provider value={level + 1}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

이렇게 변경하면, 당신은 각 섹션이나 헤딩에 레벨 프랍을 전달할 필요가 없습니다.

```jsx
import Heading from "./Heading.js";
import Section from "./Section.js";

export default function Page() {
  return (
    <Section>
      <Heading>Title</Heading>
      <Section>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

이제 헤딩과 섹션은 그들이 얼마나 깊든지 상관없이 `LevelContext`를 읽어올 수 있습니다.
그리고 `LevelContext`로 자식을 감싼 섹션은 그 안에 더 깊은 레벨의 어떤 것이든 특정할 수 있습니다.

## Context passes through intermediate components

당신은 컨텍스트를 제공하는 컴포넌트 사이에 많은 컴포넌트를 추가할 수 있습니다.
이것은 당신이 만든 컴포넌트와 div와 같은 빌트인 컴포넌트도 포합됩니다.

이 예에서 같은 포스트 컴포넌트는 두 개의 다른 중첩된 레벨에서 랜더됩니다.
이 안의 헤딩은 가까운 섹션에서 레벨을 자동적으로 가져옵니다.

```jsx
import Heading from "./Heading.js";
import Section from "./Section.js";

export default function ProfilePage() {
  return (
    <Section>
      <Heading>My Profile</Heading>
      <Post title="Hello traveller!" body="Read about my adventures." />
      <AllPosts />
    </Section>
  );
}

function AllPosts() {
  return (
    <Section>
      <Heading>Posts</Heading>
      <RecentPosts />
    </Section>
  );
}

function RecentPosts() {
  return (
    <Section>
      <Heading>Recent Posts</Heading>
      <Post title="Flavors of Lisbon" body="...those pastéis de nata!" />
      <Post title="Buenos Aires in the rhythm of tango" body="I loved it!" />
    </Section>
  );
}

function Post({ title, body }) {
  return (
    <Section isFancy={true}>
      <Heading>{title}</Heading>
      <p>
        <i>{body}</i>
      </p>
    </Section>
  );
}
```

당신은 특별한 것을 한 것이 아닙니다. 섹션은 트리 안에서 컨텍스트를 특정하기에, 당신은 헤딩을 어디든 삽입할 수 있으며, 이것은 정확한 사이즈를 갖습니다.

컨텍스트는 컴포넌트에게 주위에 적응할 수 있도록 해주고 위치에 따라 그들이 랜더될 때 다르게 보여줍니다.

컨텍스트가 어떻게 작동하는 가는 CSS의 상속을 상기시킵니다.
CSS에서 div에 `color: blue`라고 특정할 수 있는데, 중간에 `color: green`이라고 덮어쓰는 것 아니면, 그 안이 얼마나 깊든 간에 색을 상속 받습니다.
리액트에서, 컨텍스트를 덮어쓰는 방법은, 다른 컨텍스트 값으로 자식을 감싸고 제공하는 것입니다.

CSS에서 다른 속성인 `color`와 `background-color`는 서로 덮어씌어지지 않습니다. 당신은 모든 div의 글자색을 배경색의 영향없이 정할 수 있습니다.
비슷하게, 리액트의 다른 컨텍스트는 서로 덮어쓰지 않습니다.
`createContext()`로 만들어진 각 컨텍스트들은 서로 분리 되어있으며, 컴포넌트들에 특정한 컨텍스트를 제공합니다.
한 컴포넌트에 많은 컨텍스트를 사용하고 제공하는 것은 문제가 되지 않습니다.

## Before you use context

컨텍스트는 사용하기 매우 쉽습니다. 하지만, 이것은 남용하기에도 쉽다는 뜻입니다.
프랍을 전달하는 단계가 단지 몇 단계라고 해서 정보를 컨텍스트에 담으라는 것이 아닙니다.

컨텍스트를 사용하기 전 생각해야할 대체 방법입니다.

1. 프랍을 전달하는 것으로 시작하기. 당신의 컴포넌트가 하찮은 것이 아니라면, 많은 컴포넌트에 많은 프랍을 전달하는 것은 드문일이 아닙니다. 어떤 컴포넌트가 어떤 데이터를 원하는 지 확실히 하는 것이 중요합니다. 당신의 코드를 유지하는 사람이 데이터 흐름을 명시적인 프랍으로 만드는 것은 환영받을 것입니다.
2. 컴포넌트로 분리해 자식으로 전달하기. 만약 당신이 데이터를 사용하지 않는 중간 컴포넌트가 많은 계층을 통해 데이터를 전달하는 경우는 대부분 컴포넌트를 추출하지 않아서 발생합니다. 예를 들어, 당신은 포스트 같은 데이터 프랍을 사용하지 않는 UI 컴포넌트에 `<Layout posts={posts} />`와 같이 전달할 수 있습니다. 대신에, 자식을 프랍으로 받는 `Layout`을 만들어 `<Layout><Posts posts={posts} /></Layout>`와 같이 랜더합니다. 이것은 데이터를 필요로하는 컴포넌트 사이의 계층의 수를 줄입니다.

만약 이런 방식이 잘 적용되지 않는 다면, 컨텍스트를 고려하세요.

## Use cases for context

- Theming: 만약 유저가 모습을 바꾸는 앱인 경우(다크 모드), 당신은 앱의 최상위에 컨텍스트를 제공할 수 있으며, 컨텍스트를 통해 컴포넌트의 보여지는 부분을 수정할 수 있습니다.
- Current account: 많은 컴포넌트는 현재 로그인된 유저를 원합니다. 유저를 컨텍스트에 담는 것은 트리 어느 곳에서 사용하기 편리합니다. 어떤 앱은 다중 계정을 허용하기도 합니다.(다른 유저로서 댓글 남기기) 이런 케이스는, UI를 서로 다른 현재 계정 값으로 감싸는 것이 좋습니다.
- Routing: 대부분의 라우팅 해결책은 현재 라우트를 유지하기 위해 내부적으로 컨텍스트를 사용합니다. 이것이 모든 링크가 활성화와 비활성화를 아는 이유입니다. 만약 당신의 라우터를 만든다면, 당신은 똑같이 할 것입니다.
- Managing state: 앱이 커지며, 많은 상태들이 앱의 상단에 모이게될 것입니다. 거리가 있는 컴포넌트는 이것을 바꾸고 싶을 것입니다. 리듀서와 컨텍스트를 함께 사용해 복잡한 상태관리와 혼란없이 먼 컴포넌트에 전달하는 것이 일반적입니다.

컨텍스트는 정적인 값에만 한정되지 않습니다. 다음 랜더에 다른 값을 전달하면, 리액트는 그 값을 사용하는 컴포넌트 모두를 변경합니다. 이것이 컨텍스트가 상태와 자주 사용되는 이유입니다.

만약 어떤 정보가 다른 트리의 먼 컴포넌트에서 사용될 경우, 컨텍스트는 좋은 지침이 될 것입니다.

## Recap

- 컨텍스트는 트리의 모든 하위에 정보를 제공
- 컨텍스트 넘기기
  - `export const MyContext = createContext(defaultValue)` 컨텍스트 생성후 내보내기
  - `useContext(MyContext)` 훅을 사용해 얼마나 깊든지, 읽어오기
  - `<MyContext.Provider value={...}>`로 부모로 제공받는 자식 감싸기
- 컨텍스트는 중간에 있는 어떤 컴포넌트도 건너 뜀
- 컨텍스트는 컴포넌트가 그들 주위에 적응하도록 해줌
- 컨텍스트 전에는, 프랍을 전달하거나 JSX를 자식으로 전달

---

## What I Learned

컨텍스트가 리액트를 처음 공부하면서 가장 어려운 부분이었다.
몇번 사용해보긴 했지만, 그래도 여전히 어렵고 명확한 부분이 없었는데, 문서를 읽으며 컨텍스트에 한발짝 가까워졌다.

특히 CSS의 상속의 예를 보며 이해가 많이 됐다.

출처

- https://react.dev/learn/passing-data-deeply-with-context
