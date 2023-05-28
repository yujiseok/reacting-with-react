- [Sharing State Between Components](#sharing-state-between-components)
  - [Lifting state up by example](#lifting-state-up-by-example)
    - [Step 1: Remove state from the child components](#step-1-remove-state-from-the-child-components)
    - [Step 2: Pass hardcoded data from the common parent](#step-2-pass-hardcoded-data-from-the-common-parent)
    - [Step 3: Add state to the common parent](#step-3-add-state-to-the-common-parent)
  - [DEEP DIVE - Controlled and uncontrolled components](#deep-dive---controlled-and-uncontrolled-components)
  - [A single source of truth for each state](#a-single-source-of-truth-for-each-state)
  - [Recap](#recap)
  - [What I Learned](#what-i-learned)

# Sharing State Between Components

가끔 두 컴포넌트의 상태를 동시에 변경하고 싶을 때가 있을 것입니다. 이것을 하기 위해, 상태를 제거한 후 가장 가까운 부모 컴포넌트로 옮긴 후, 프랍으로 넘겨줍니다.
이것은 상태 lifting이라고 불리며, 리액트를 사용하면서 아주 보편적인 일입니다.

## Lifting state up by example

이 예에서, 부모 아코디언 컴포넌트는 두 개의 패널을 랜더합니다.

- 아코디언
  - 패널
  - 패널

각 패널은 컨텐츠가 보이도록 하는 불리언 상태인 `isActive`를 갖고 있습니다.

각 패널은 다른 패널에 영향을 주지 않습니다. - 상호 독립

[label](https://react.dev/_next/image?url%3D%2Fimages%2Fdocs%2Fdiagrams%2Fsharing_state_child.dark.png%26w%3D640%26q%3D75)

[label](https://react.dev/_next/image?url%3D%2Fimages%2Fdocs%2Fdiagrams%2Fsharing_state_child_clicked.dark.png%26w%3D640%26q%3D75)

그러나, 오직 하나의 패널만이 열리도록 바꾸고 싶을 수 있습니다. 이 디자인에서, 두 번째 패널이 열리면, 첫 번재 패널을 닫혀야합니다. 어떻게 구현할 수 있을까요?

두 패널을 구현하기 위해, 상태를 세 단계에 거쳐 부모 컴포넌트로 올려야 합니다.

1. 자식 컴포넌트의 상태를 제거
2. 부모 컴포넌트에서 하드 코딩된 데이터 전달
3. 부모 컴포넌트에 상태 추가와 이벤트 핸들러와 함께 전달

이것은 아코디언 컴포넌트의 패널이 한 번만 열리게 해줍니다.

### Step 1: Remove state from the child components

패널을 제어하는 `isActive`를 부모 컴포넌트로 전달합니다. 이것은 부모 컴포넌트가 `isActive`를 패널에 프랍으로 전달하는 것을 뜻합니다. 패널 컴포넌트에서 이 라인을 지우는 것부터 시작하세요.

```jsx
const [isActive, setIsActive] = useState(false);
```

대신 패널의 프랍에 `isActive`를 추가합니다.

```jsx
function Panel({ title, children, isActive }) {
```

이제 패널의 부모 컴포넌트가 `isActive`를 프랍으로 전달해 제어할 수 있습니다. 반대로, 패널은 이제 `isActive`를 제어할 수 없습니다.

### Step 2: Pass hardcoded data from the common parent

상태를 올리기 위해선, 가까운 자식 컴포넌트를 갖는 부모 컴포넌트에 상태를 위치시켜야합니다.

- 아코디언
  - 패널
  - 패널

이 예에서 아코디언 컴포넌트는 각 패널을 프랍으로 제어하며, 어떤 패널이 활성화 됐는지 알 수 있게 해줍니다.
아코디언 컴포넌트에서 하드 코드된 `isActive` 값을 패널들에 내려줍니다.

하드 코드된 `isActive` 값을 변경하고 결과를 보세요.

### Step 3: Add state to the common parent

상태를 올리는 것은 종종 무엇을 상태로 갖을지 변화시킵니다.

이 경우에서, 오직 하나의 패널만이 활성화 됩니다.
이것은 부모 컴포넌트인 아코디언이 어떤 패널이 활성화 됐는지 추적해야합니다.
불리언 값 대신, 이것은 패널의 인덱스인 숫자를 상태로 갖을 수 있습니다.

```jsx
const [activeIndex, setActiveIndex] = useState(0);
```

`activeIndex`가 `0`일 시, 첫 번째 패널이 펴지고, `1`일 시에는 두 번째가 펴집니다.

각 패널의 show 버튼을 누르는 것은 아코디언에 있는 활성화 인덱스를 바꿔야합니다.
패널은 `activeIndex`가 아코디언에서 설정되었기에, 직접적으로 설정할 수 없습니다. 아코디언 컴포넌트는 이벤트 핸들러를 프랍으로 패널에 전달해 명시적으로 바꿀 수 있도록 허용해야합니다.

패널 안의 버튼은 `onShow` 프랍을 클릭 이벤트 핸들러로 사용할 수 있습니다.

이것은 상태 올리기를 완성시킵니다.
상태를 부모 컴포넌트로 옮겨 두 패널을 조정할 수 있도록 합니다.
플래그 변수 대신 활성화 인덱스를 사용하는 것이 하나의 패널이 활성화 되는 것을 보장합니다.
그리고 이벤트 핸들러를 프랍으로 넘겨 자식에서 부모의 상태를 변경할 수 있게 해줍니다.

[label](https://react.dev/_next/image?url%3D%2Fimages%2Fdocs%2Fdiagrams%2Fsharing_state_parent.dark.png%26w%3D640%26q%3D75)

[label](https://react.dev/_next/image?url%3D%2Fimages%2Fdocs%2Fdiagrams%2Fsharing_state_parent_clicked.dark.png%26w%3D640%26q%3D75)

## DEEP DIVE - Controlled and uncontrolled components

지역 상태를 갖는 컴포넌트를 비제어 컴포넌트라고 부르는 것이 일반적입니다.
예를 들어, 패널 컴포넌트의 `isActive` 상태는 비제어입니다. 왜냐하면 부모는 패널이 활성화 됐거나, 아니거나에 영향이 없습니다.

반대로, 지역 상태를 갖지 않고 중요한 정보가 프랍으로 넘겨진 컴포넌트들은 제어 컴포넌트라고 부릅니다.
이것은 부모 컴포넌트가 그 것의 행동을 특정합니다.
`isActive`를 프랍으로 갖는 최종 패널 컴포넌트는 아코디언 컴포넌트에 제어됩니다.

비제어 컴포넌트들은 설정할 것이 없기에 그들의 부모에서 사용되는 것이 용이합니다.
하지만 그들은 같이 사용할 때 덜 유연합니다.
제어 컴포넌트는 매우 유연하지만, 프랍으로 설정해줄 부모 컴포넌트를 필요로합니다.

실제로, 제어와 비제어는 엄격하지 않은 기술 명칭입니다. 각 컴포넌트는 종종 지역 상태와 프랍을 갖기 때문입니다. 하지만, 이것은 컴포넌트가 어떻게 디자인 되었는지 혹은 어떤 성능을 제공하는 지 이야기 하기에 유용합니다.

컴포넌트를 작성할때 어떤 정보가 제어되고 비제어되어야 하는지 고려하세요. 하지만 항상 바꿀 수 있으며 리팩토링 할 수 있습니다.

## A single source of truth for each state

리액트 애플리케이션에서, 많은 컴포넌트들은 그들의 상태를 갖습니다.
어떤 상태는 컴포넌트 트리의 아래에 존재하고, 어떤 상태는 최상위에 존재합니다.
예를 들어 클라이언트 사이드 라우팅 라이브러리들은 현재 라우트를 리액트의 상태에 저장하고 프랍으로 넘겨줍니다.

각각의 상태들을 어떤 컴포넌트가 소유할지 정해야합니다. 이런 원칙은 단일 진실 공급원이라고 알려져있습니다. 이것은 모든 상태가 하나의 장소에 존재해야하는 것을 의미하는 것이 아닌, 특정 컴포넌트들이 각 상태를 소유하는 것을 말합니다.
복사해 컴포넌트들에 공유하는 것이 아닌, 부모로 올린 후 필요한 자식에 넘겨주는.

당신의 앱은 변화할 것입니다. 그것은 일반적이면 당신은 상태를 어디에 존재하게 할지 정할 것입니다.

## Recap

- 두 컴포넌트들을 조정하고 싶다면, 상태를 부모 컴포넌트로 올리세요.
- 그 후, 프랍으로 전달하세요.
- 최종적으로, 이벤트 핸들러를 프랍으로 전달해 부모의 상태를 변경하세요.
- 제어 컴포넌트(프랍)와 비제어 컴포넌트(상태)를 고려하는 것은 유용합니다.

---

## What I Learned

상태를 다루는 것에 좀 더 알 수 있게 되었다.

출처

- https://react.dev/learn/sharing-state-between-components
