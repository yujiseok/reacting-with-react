- [Keeping Components Pure](#keeping-components-pure)
  - [Purity: Components as formulas](#purity-components-as-formulas)
  - [Side Effects: (un)intended consequences](#side-effects-unintended-consequences)
  - [Local mutation: Your component’s little secret](#local-mutation-your-components-little-secret)
  - [Where you can cause side effects](#where-you-can-cause-side-effects)
  - [DEEP DIVE - Why does React care about purity?](#deep-dive---why-does-react-care-about-purity)
  - [Recap](#recap)
  - [What I Learned](#what-i-learned)

# Keeping Components Pure

몇 자바스크립트의 함수는 순수합니다. 순수 함수는 오직 연산을 수행합니다.
컴포넌트를 순수하게 작성하는 것은, 이해할 수 없는 버그와 예측 불가능한 행동을 막아줍니다.
이런 이점을 얻기 위해, 반드시 따라야 하는 룰이 있습니다.

## Purity: Components as formulas

컴퓨터 공학에서(특히 함수형 프로그래밍에서) 순수 함수는 다음과 같이 정의됩니다.

- 오직 자신의 일만 신경 씁니다. 호출되기 전에 존재하는 어떤 객체나 변수를 변화시키지 않습니다.
- 같은 입력과 같은 출력을 항상 보장합니다.

아마 수학에서 비슷한 순수 함수의 예제들을 보았을 것입니다.

y= 2x 라는 공식을 생각해 보세요.

만약 x = 2 이면 항상 y = 4의 결과를 갖습니다.
x = 3 이면 항상 y = 6의 결과를 갖습니다.

x = 3 일 때 y는 시간에 따르거나 주식 시장의 상태에 따라 9이거나 -1 또는 2.5일 수 없습니다.

y = 2x이고 x = 3일 때, y는 무조건 6입니다.

만약 자바스크립트로 함수를 만든다면, 다음과 같습니다.

```js
function double(number) {
  return 2 * number;
}
```

위의 `double` 함수는 순수 함수입니다. 만약 `3`을 넘겨주면 항상 `6`을 리턴합니다.

리액트는 이와 같은 컨셉으로 디자인되었습니다. 리액트는 우리가 적은 모든 코드가 순수하다는 가정합니다.
즉 우리가 작성한 리액트 컴포넌트는 항상 같은 JSX를 리턴해야 합니다.

```JSX
function Recipe({ drinkers }) {
  return (
    <ol>
      <li>Boil {drinkers} cups of water.</li>
      <li>Add {drinkers} spoons of tea and {0.5 * drinkers} spoons of spice.</li>
      <li>Add {0.5 * drinkers} cups of milk to boil and sugar to taste.</li>
    </ol>
  );
}

export default function App() {
  return (
    <section>
      <h1>Spiced Chai Recipe</h1>
      <h2>For two</h2>
      <Recipe drinkers={2} />
      <h2>For a gathering</h2>
      <Recipe drinkers={4} />
    </section>
  );
}
```

`drinkers={2}`를 `Recipe`에 넘겨주면 `2 cups of water`를 갖는 JSX를 항상 리턴합니다.
`drinkers={4}`를 넘겨주면 `4 cups of water`를 갖는 JSX를 항상 리턴합니다.

수학 공식과 같이.

우리는 리액트의 컴포넌트를 레시피로 생각할 수 있다. 새로운 재료를 요리 과정에 추가하지 않는다면, 항상 같은 음식을 만들 수 있듯이. 그 음식이 리액트가 렌더링 할 JSX입니다.

![](https://react.dev/images/docs/illustrations/i_puritea-recipe.png)

## Side Effects: (un)intended consequences

리액트의 렌더링 과정은 항상 순수해야 합니다. 컴포넌트는 오직 JSX만을 리턴해야 하며, 렌더링 이전에 객체나 변수들을 바꿔서는 안됩니다. 이런 것들은 컴포넌트를 비순수하게 합니다.

컴포넌트의 순수성을 해친 예입니다.

```JSX
let guest = 0;

function Cup() {
  // Bad: changing a preexisting variable!
  guest = guest + 1;
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup />
      <Cup />
      <Cup />
    </>
  );

```

이 컴포넌트는 외부에서 선언된 `guest`를 읽고 수정합니다. 이것은 컴포넌트를 여러 번 호출할 시 다른 JSX를 만들어 낸다는 것을 의미합니다. 거기다, 다른 컴포넌트에서 `guest`를 읽을 시 또 다른 JSX를 컴포넌트가 렌더링 될 시 만들어 냅니다. 이것은 예측 가능하지 않습니다.

우리의 수학 공식으로 돌아가 봅시다. y = 2x는 이제 x = 2 일 때여도, y = 4라는 것을 확신하지 못합니다. 우리의 테스트는 실패했고, 유저는 당황하며, 비행기는 추락합니다. 이런 비순수성이 얼마나 큰 혼란을 야기하는지 아시겠나요?

다음과 같이 프랍을 넘겨줘, 코드를 수정할 수 있습니다.

```jsx
function Cup({ guest }) {
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup guest={1} />
      <Cup guest={2} />
      <Cup guest={3} />
    </>
  );
}
```

이제 우리의 컴포넌트는 프랍에 의존해 JSX를 리턴하므로 순수합니다.

컴포넌트가 특정한 순서에 따라 렌더링 된다는 것을 기대하지 말아야합니다. y = 2x를 y = 5x 보다 먼저 호출하든 늦게 호출하든 각각의 공식은 독립적으로 해결됩니다. 컴포넌트에서도 똑같이, 각각의 컴포넌트는 오직 자신들만 생각합니다. 그리고 렌더링 중 다른 컴포넌트를 조정하거나 의존하지 않습니다. 렌더링은 학교의 시험과 같습니다. 각각의 컴포넌트는 각각의 JSX를 계산해냅니다.

## Local mutation: Your component’s little secret

위의 예제의 문제는 렌더링 시 기존에 존재하던 변수를 컴포넌트가 변경해서입니다. 이런 것들은 종종 **"뮤테이션"**이라고 불립니다. 순수 함수는 함수 밖 스코프의 변수와 호출전에 생성된 객체를 뮤테이트하지 않습니다. 그것들은 비순수를 만들기 때문입니다.

하지만, 렌더링 도중에 변수를 변경하거나 객체를 생성하는 것은 괜찮습니다.
예를 들어, `[]` 배열을 만들어 `cups` 변수에 할당합니다. 그리고 12개의 컵을 `push`합니다.

```JSX
function Cup({ guest }) {
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaGathering() {
  let cups = [];
  for (let i = 1; i <= 12; i++) {
    cups.push(<Cup key={i} guest={i} />);
  }
  return cups;
}
```

만약 `cups` 변수가 `TeaGathering` 함수 밖에서 선언되었다면 이것은 아주 큰 문제입니다. 기존에 존재한 배열을 바꾸는 것입니다.

하지만, 똑같은 렌더링에 `TeaGathering` 내부에서 생성되므로 괜찮습니다. `TeaGathering` 밖의 코드들은 무슨 일이 일어나는지 알 수 없습니다. **"로컬 뮤테이션"**이라고 불리는 컴포넌트의 작은 비밀입니다.

## Where you can cause side effects

함수형 프로그래밍이 순수성에 크게 의존하지만, 동시에, 어디선가, 무언가가 바뀔 수 있습니다.
이것이 프로그래밍의 핵심입니다. 이런 변화들 - 화면을 바꾸거나, 애니메이션이 시작되거나, 데이터를 바꾸는 - 이런 것들은 사이드 이펙트라고 불립니다. 이런 것들은 렌더링 중 발생하는 것이 아닌, 어떤 사이드에서 일어납니다.

리액트에서 사이드 이펙트는 대부분 이벤트 핸들러에 내부에 속합니다. 이벤트 핸들러는 어떤 액션을 취했을 때 실행하는 함수들로 예를 들면, 버튼 클릭이 있습니다. 이벤트 핸들러가 컴포넌트 안에서 선언되었지만, 그들은 렌더링 도중 실행되지 않습니다. 그러니 이벤트 핸들러는 순수할 필요가 없습니다.

만약 사이드 이펙트를 처리할 마땅할 이벤트 핸들러를 찾지 못했다면, `useEffect`를 활용해 해결할 수 있습니다. 이것은 리액트에게 렌더링 이후에 그것을 호출하도록 합니다. 하지만 이런 접근은 최후의 보루입니다.

## DEEP DIVE - Why does React care about purity?

순수 함수를 작성하는 것은 몇 가지 룰이 있습니다. 하지만 이것은 놀라운 이점을 줍니다.

- 예를 들어 서버에서 컴포넌트가 실행될 때, 그들은 항상 똑같은 입력과 출력을 하기에 많은 유저의 요청을 처리할 수 있습니다.
- 입력이 없을 경우 렌더링을 넘겨 성능을 향상시킬 수 있습니다. 순수 함수는 항상 같은 결과를 리턴하므로 캐시 하기에 안전합니다.
- 만약 컴포넌트 트리의 깊숙한 곳의 데이터가 바뀌었다면, 리액트는 outdated된 렌더에 시간 낭비를 하지 않고 렌더링을 다시 시작합니다.

모든 리액트의 새로운 기능들이 순수성의 이점을 얻습니다. 데이터 페칭부터 퍼포먼스 하기 위한 애니메이션까지 컴포넌트를 순수하게 유지하는 것은 리액트 패러다임에 힘을 해제합니다.

## Recap

- 컴포넌트는 순수해야 한다는 것의 의미

  - 오직 자신의 일만 신경 씁니다. 호출되기 전에 존재하는 어떤 객체나 변수를 변화시키지 않습니다.
  - 같은 입력과 같은 출력을 항상 보장합니다.

- 렌더링은 아무 때나 일어날 수 있으므로, 컴포넌트는 각각의 렌더링에 의존하고 있지 않습니다.
- 렌더링을 위한 입력값(프랍, 상태, 컨텍스트)을 뮤테이트하면 안됩니다. 화면을 업데이트하기 위해 기존의 객체를 수정하는 것이 아닌 상태를 set 합니다.
- JSX를 리턴하기 위한 로직을 작성하고, 변화를 원한다면 이벤트 핸들러를 사용합니다. 최후의 보루로 `useEffect`를 사용합니다.
- 순수 함수를 작성하는 것은 연습이 필요하지만, 리액트 패러다임의 힘을 해제합니다.

---

## What I Learned

순수성이라는 개념은 나에게 항상 명확하지 않은 개념이었는데, 완벽히는 아니더라도 제법 감을 잡게 된 느낌이다.
수학 공식이라고 생각하니까 이해가 제법 잘 되었고, 음식의 레시피 예시가 나에게 딱이었다.
항상 같은 입력과 같은 출력을 보장해야 한다는 것이 순수 함수의 핵심인 것 같다. 생각해 보면 리액트 컴포넌트들은 공식 문서의 말처럼 항상 같은 값을 받아 같은 JSX를 리턴했던 것 같다. 순수성이라는 개념을 좀 더 공부해 봐야 할 것 같다.

<br>

출처

- https://react.dev/learn/keeping-components-pure
