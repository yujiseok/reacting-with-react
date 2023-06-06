# You Might Not Need an Effect

이펙트는 리액트 패러다임의 탈출구입니다.
그들은 리액트 밖으로 나오게 해주는데, 리액트 위젯이 아닌 네트워크, 또는 브라우저 돔과 컴포넌트가 동기화할 수 있도록 해줍니다.
만약 외부 시스템과 관련이 없다면, 이펙트가 필요없습니다.
불필요한 이펙트를 제거하면, 코드가 더 읽기 쉬우며 빠르고 에러 발생이 적습니다.

## How to remove unnecessary Effects

이펙트가 필요하지 않은 일반적인 두 상황이 있습니다.

- 랜더링에 필요한 데이터를 변형하기 위해 이펙트는 필요하지 않습니다. 리스트를 보여주기 전에 필터하는 예가 있다. 리스트가 변경되었을 때, 변경시키는 이펙트를 작성하고 싶을 것이다. 하지만, 이것은 효율적이지 않다. 상태를 업데이트 하고 싶을 때, 리액트는 먼저 컴포넌트 함수를 호출해 화면에 무엇이 보일지 계산한다. 그 후 리액트는 돔에 변경 사항을 커밋해 화면을 업데이트한다. 그 후 리액트는 이펙트를 실행한다. 만약 당신의 이펙트가 상태를 업데이트하면, 이것은 처음부터 재시작할 것이다. 불필요한 랜더를 피하기 위해, 모든 데이터를 변형하는 것은 컴포넌트 최상단에서 해야한다. 그 코드는 당신이 프랍이나 상태를 변경하면 자동으로 재 실행된다.
- 이벤트를 다루기 위해 이펙트가 필요없다. 예를 들어, POST 요청을 보낸 후 알림을 줄 때. 구매 버튼 클릭 핸들러에서, 당신은 무엇이 일어날지 압니다. 이펙트에서 처리하면, 당신은 유저가 무엇을 했는지 모릅니다. 이것이 이벤트를 상응하는 이벤트 핸들러로 다뤄야하는 이유입니다.

외부 시스템과 싱크를 맞출 때 이펙트가 필요하다.
예를 들어, 리액트 상태와 싱크를 맞추기 위해 이펙트로 제이쿼리 위젯을 유지하기 위해 이펙트를 작성한다. 당신은 또한 이펙트와 데이터를 페치할 수 있다: 현재 검색 쿼리와 검색 결과의 싱크를 맞추는 예.
모던 프레임워크들은 직접 이펙트를 적는 것보다, 더 좋은 빌트인 데이터 페칭 메커니즘을 제공한다.

### Updating state based on props or state

이름과 성을 갖는 상태가 있다. 당신은 풀네임을 그들로 계산하고 싶어한다.
게다가, 풀네임이 이름이나 성이 바뀔대 변경하고 싶다.
당신은 아마 풀네임 상태를 추가하고 이펙트로 업데이트할 것이다.

```jsx
function Form() {
  const [firstName, setFirstName] = useState("Taylor");
  const [lastName, setLastName] = useState("Swift");

  // 🔴 Avoid: redundant state and unnecessary Effect
  const [fullName, setFullName] = useState("");
  useEffect(() => {
    setFullName(firstName + " " + lastName);
  }, [firstName, lastName]);
  // ...
}
```

이것은 필요 이상으로 복잡하다. 또 비효율적이다.
이것은 풀네임에 대한 유효하지 않은 값으로 전체 랜더를 수행후, 다음 업데이트된 값으로 즉시 리랜더링한다. 상태와 이펙트를 지워라.

```jsx
function Form() {
  const [firstName, setFirstName] = useState("Taylor");
  const [lastName, setLastName] = useState("Swift");
  // ✅ Good: calculated during rendering
  const fullName = firstName + " " + lastName;
  // ...
}
```

기존의 프랍과 상태로 계산이 가능하면 상태로 두지마라. 대신, 랜더 중 계산해라.
이것은 당신의 코드를 빠르고, 간결하며 다른 상태 변수들과 싱크를 맞춰야하는 에러 발생을 줄인다.

### Caching expensive calculations

이 컴포넌트는 todos를 filter프랍으로 필터링 하여 visibleTodos를 계산한다.

```jsx
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState("");

  // 🔴 Avoid: redundant state and unnecessary Effect
  const [visibleTodos, setVisibleTodos] = useState([]);
  useEffect(() => {
    setVisibleTodos(getFilteredTodos(todos, filter));
  }, [todos, filter]);

  // ...
}
```

앞선 예처럼, 이것은 둘다 불필요하고 비효율적이다. 먼저 상태와 이펙트를 지운다.

```jsx
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState("");
  // ✅ This is fine if getFilteredTodos() is not slow.
  const visibleTodos = getFilteredTodos(todos, filter);
  // ...
}
```

이것은 코드는 괜찮다. 하지만 `getFilteredTodos()`가 느리거나 많은 `todos`가 있을 경우.
당신은 `getFilteredTodos()`를 연관이 없는 `newTodo`가 변경되었다고 다시 계산하고 싶지 않을 것이다.

당신은 비싼 계산을 `useMemo` 훅으로 감싸 캐시하거나 메모이즈할 수 있다.

```jsx
import { useMemo, useState } from "react";

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState("");
  const visibleTodos = useMemo(() => {
    // ✅ Does not re-run unless todos or filter change
    return getFilteredTodos(todos, filter);
  }, [todos, filter]);
  // ...
}
```

이것은 리액트에게 todos나 filter가 바뀌지 않은 이상 안의 함수가 재실행되는 것을 막도록한다.
리액트는 todos나 filter가 다를 때까지 `getFilteredTodos()`의 값을 기억한다.
만약 그들이 같다면 마지막에 저장한 값을 다르다며 안의 함수를 다시 호출한다.

이 `useMemo`는 랜더 도중에 실행되므로 순수 계산에서만 동작한다.

#### DEEP DIVE - How to tell if a calculation is expensive?

사실상, 천개가 넘는 객체를 반복문으로 만드는 것은 비용이 많이들지 않는다.
콘솔로그로 걸리는 시간을 측정해볼수 있다.

```jsx
console.time("filter array");
const visibleTodos = getFilteredTodos(todos, filter);
console.timeEnd("filter array");
```

당신의 상호작용을 측정하면, 당신은 `filter array: 0.15ms`와 같은 콘솔을 볼 것이다. 만약 1ms 보다 이상인 중대한 시간이 걸리면 메모이즈 하는 것은 말이된다. 실험으로 계산을 `useMemo`로 감싸서 시간이 감소했는지 측정할 수 있다.

```jsx
console.time("filter array");
const visibleTodos = useMemo(() => {
  return getFilteredTodos(todos, filter); // Skipped if todos and filter haven't changed
}, [todos, filter]);
console.timeEnd("filter array");
```

`useMemo`는 첫 랜더를 빠르게 하지 않는다. 이것은 오직 불필요한 업데이트만 무시한다.

기계는 유저보다 빠르다는 것을 기억하고 테스트시 속도를 줄여야한다.
예를 들어 크롬은 CPU 쓰로틀링 옵션을 제공한다.

그리고 개발 환경은 정확한 퍼포먼스 측정 결과를 보여주지 않는다.
정확한 시간을 알고 싶다면, 배포하고 테스트를 진행해야한다.

### Resetting all state when a prop changes

프로필페이지 컴포넌트는 userId 프랍을 받는다.
페이지는 댓글 인풋을 포함하는데, comment 상태로 값을 들고있는다.
어느날, 당신은 문제를 발견한다.
다른 프로필로 이동해도 comment 상태는 초기화되지 않는다. 그 결과로 다른 유저 프로필에 댓글을 달 위험이 있다.
이것을 고치기 위해, userId가 바뀌면 comment를 초기화해야한다.

```jsx
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState("");

  // 🔴 Avoid: Resetting state on prop change in an Effect
  useEffect(() => {
    setComment("");
  }, [userId]);
  // ...
}
```

이것은 비효율적인데 프로필페이지와 자식은 첫 랜더시 불필요한 상태 값을 갖고 랜더한다. 이것은 또한 복잡한데 당신은 프로필이미지 내부의 상태를 갖는 모든 컴포넌트에 적용해야한다. 예를 들어, 댓글 UI가 중첩되면, 중첩된 댓글 상태도 초기화 시켜야한다.

대신에, 명시적인 키를 제공해 컨셉적으로 서로 다른 프로필이라고 알려준다.
컴포넌트를 두 개로 나눈 후 키 어트리뷰트를 밖에서 안으로 전달한다.

```jsx
export default function ProfilePage({ userId }) {
  return <Profile userId={userId} key={userId} />;
}

function Profile({ userId }) {
  // ✅ This and any other state below will reset on key change automatically
  const [comment, setComment] = useState("");
  // ...
}
```

일반적으로, 리액트는 같은 장소에 같은 컴포넌트가 랜더되면 상태를 보존한다.
userId를 프로필 컴포넌트의 키로 넘김으로, 리액트에게 프로필 컴포넌트를 어떤 상태도 공유하지 않는 두 컴포넌트로 다루도록 요청한다.
키가 변경될 시, 리액트는 돔을 다시 생성하고 프로필 컴포넌트와 모든 하위 자식의 상태를 초기화한다. 이제 comment 필드는 프로필을 오갈 때 초기화된다.

이 예에서 프로필페이지 컴포넌트만 내보내고 프로젝트의 다른 파일에 표시한다.
프로필페이지를 랜더링하는 컴포넌트는 키를 넘길 필요 없다. 그들은 userId를 프랍으로 넘긴다. 프로필페이지가 이것을 키로써 프로필 컴포넌트에 전달하는 것이 구현 디테일입니다.

### Adjusting some state when a prop changes

때로, 당신은 프랍의 변경에 따라 상태를 변경하거나 초기화하고 싶을 수 있지만, 일부는 불가하다.

리스트 컴포넌트는 리스트의 아이템들을 프랍으로 받고, 선택 상태로 선택된 아이템을 유지한다. 아이템 프랍을 다르게 받았을 경우 당신은 선택을 null로 초기화 하고 싶다.

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 🔴 Avoid: Adjusting state on prop change in an Effect
  useEffect(() => {
    setSelection(null);
  }, [items]);
  // ...
}
```

이것은 이상적이지않다. 아이템이 변했을 경우, 리스트와 자식 컴포넌트들은 불필요한 선택 값과 우선 랜더된다.
그 후 리액트는 돔을 업데이트하고 이펙트를 실행한다.
마지막으로, setSelection(null) 이 또 다른 리랜더를 유발하고, 이 과정을 되풀이한다.

이펙트를 지우고, 랜더 중 상태를 직접 변경한다.

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // Better: Adjust the state while rendering
  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}
```

이전 랜더에서 정보 저장은 이해하기 어렵지만, 이펙트안에서 업데이트 하는 것보단 좋다. 위 예에서, `setSelection`은 랜더 중에 직접 호출된다. 리액트는 리스트는 즉시 리턴문 이후의 것들을 리랜더 한다.
리액트는 리스트의 자식이나 돔을 업데이트 하지않는다. 즉 이것은 리스트 자식이 불필요한 선택 값을 무시한다.

랜더 중 컴포넌트를 업데이트할 경우 리액트는 반환된 JSX를 버리고, 랜더를 재시도합니다. 느린 계단식 재시도를 피하기 위해, 리액트는 오직 같은 컴포넌트만을 랜더 중 업데이트하는 것을 허용한다.
items !== prevItems 같은 조건은 반복을 피하기위해 필요하다.
상태는 이렇게 조정할 수 있지만, 컴포넌트는 순수해야하므로 다른 사이드 이펙트는 이벤트 핸들러나 이펙트안에 살아야한다.

이런 패턴은 이펙트보다 훨씬 효율적이므로, 대부분의 컴포넌트는 이것을 필요로한다.
어떻든간에, 상태를 프랍이나 다른 상태에 따라 조정하는 것은 데이터 플로우를 복잡하게 하고 디버깅을 어렵게한다. 항상 키로 상태를 리셋하거나 랜더 중 계산하는 것을 생각해라. 예를 들어, 당신은 선택된 아이템의 아이디를 저장할 수 있다.

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // ✅ Best: Calculate everything during rendering
  const selection = items.find((item) => item.id === selectedId) ?? null;
  // ...
}
```

이제 더이상 상태를 조정하는 것은 필요없다. 만약 선택된 아이디의 아이템이 리스트에 존재하면, 선택된 상태로 남는다.
그렇지 않다면 selection은 랜더 중 계산되고 맞는 것이 없으므로 null을 반환한다.
이 동작은 다르지만, selection에 아이템의 변경이 보존되기에 거의 틀림없이 나은 방법이다.

### Sharing logic between event handlers

두 버튼이 있는 상품을 구매할 수 있는 프로덕트 페이지가 있다.
당신은 카트에 상품이 담기면 알림을 보여주고 싶다.
두 버튼의 showNotification()이 중복이므로 당신은 로직을 이펙트에 두려한다.

```jsx
function ProductPage({ product, addToCart }) {
  // 🔴 Avoid: Event-specific logic inside an Effect
  useEffect(() => {
    if (product.isInCart) {
      showNotification(`Added ${product.name} to the shopping cart!`);
    }
  }, [product]);

  function handleBuyClick() {
    addToCart(product);
  }

  function handleCheckoutClick() {
    addToCart(product);
    navigateTo("/checkout");
  }
  // ...
}
```

이 이펙트는 불필요하다. 그리고 이것은 버그를 유발한다.
예를 들어, 당신의 앱이 페이지가 리로드될 시 쇼핑 카트를 기억한다고 하자.
당신이 카트에 상품을 추가하고 새로고침을 하면 알림이 다시 보일 것이다.
이것은 프로덕트 페이지를 매번 새로고침할 시 보인다. product.isInCart가 페이지가 로드될 시 이미 true 이기 때문에 이펙트는 showNotification()을 호출한다.

당신이 어떤 로직이 이펙트에 존재해야 할지 이벤트 핸들러에 존재할지 모르겠다면, 자신에게 왜 코드가 실행돼야 하는지 물어봐라. 오직 유저에 컴포넌트가 보여졌을 시에만 이펙트를 사용해라. 이 예에서 알림은 페이지가 보여서가 아니라, 유저가 버튼을 눌렀기에 발생한다.
이펙트를 지운 후 공통된 로직을 갖는 함수를 각 이벤트 핸들러에 추가해 호출한다.

```jsx
function ProductPage({ product, addToCart }) {
  // ✅ Good: Event-specific logic is called from event handlers
  function buyProduct() {
    addToCart(product);
    showNotification(`Added ${product.name} to the shopping cart!`);
  }

  function handleBuyClick() {
    buyProduct();
  }

  function handleCheckoutClick() {
    buyProduct();
    navigateTo("/checkout");
  }
  // ...
}
```

이것은 불필요한 이펙트와 버그 둘다 제거한다.

### Sending a POST request

이 폼 컴포넌트는 두 POST 요청을 보낸다.
마운트 되었을 시 분석 이벤트를 보낸다.
폼을 채우고 제출 버튼을 누를 시, `/api/register` 엔드포인트에 POST 요청을 보낸다.

```jsx
function Form() {
  const [firstName, setFirstName] = useState("");
  const [lastName, setLastName] = useState("");

  // ✅ Good: This logic should run because the component was displayed
  useEffect(() => {
    post("/analytics/event", { eventName: "visit_form" });
  }, []);

  // 🔴 Avoid: Event-specific logic inside an Effect
  const [jsonToSubmit, setJsonToSubmit] = useState(null);
  useEffect(() => {
    if (jsonToSubmit !== null) {
      post("/api/register", jsonToSubmit);
    }
  }, [jsonToSubmit]);

  function handleSubmit(e) {
    e.preventDefault();
    setJsonToSubmit({ firstName, lastName });
  }
  // ...
}
```

이전 예의 기준을 다시 적용한다.

분석 POST 요청은 이펙트 안에 존재해야한다. 분석 이벤트는 폼이 보여졌을 때 보내져야하기 때문이다.

하지만 `/api/register` POST 요청은 보여졌을 때 발생하는 것이 아니다.
당신은 오직 특정한 때에만 요청을 보내야한다:유저가 버튼을 눌렀을 경우
이것은 오직 특정한 상호작용에만 동작해야한다.
두 번째 이펙트를 지우고 POST 요청을 이벤트 핸들러안으로 옮긴다.

```jsx
function Form() {
  const [firstName, setFirstName] = useState("");
  const [lastName, setLastName] = useState("");

  // ✅ Good: This logic runs because the component was displayed
  useEffect(() => {
    post("/analytics/event", { eventName: "visit_form" });
  }, []);

  function handleSubmit(e) {
    e.preventDefault();
    // ✅ Good: Event-specific logic is in the event handler
    post("/api/register", { firstName, lastName });
  }
  // ...
}
```

로직을 이벤트 핸들러에 넣을지 이펙트에 넣을지 고민이라면, 유저의 관점에서 어떤 로직이 오는지 대답하는 것이다.
만약 이런 로직이 특정한 상호작용에서 온다면 이벤트 핸들러로, 만약 유저가 컴포넌트가 화면에 보이는 것으로 발생한다면 이펙트 안에 두어라.

### Chains of computations

가끔 당신은 상태에 기반해 다른 상태를 조정하도록 이펙트를 연속시키고 싶을 수 있다.

```jsx
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);
  const [isGameOver, setIsGameOver] = useState(false);

  // 🔴 Avoid: Chains of Effects that adjust the state solely to trigger each other
  useEffect(() => {
    if (card !== null && card.gold) {
      setGoldCardCount(c => c + 1);
    }
  }, [card]);

  useEffect(() => {
    if (goldCardCount > 3) {
      setRound(r => r + 1)
      setGoldCardCount(0);
    }
  }, [goldCardCount]);

  useEffect(() => {
    if (round > 5) {
      setIsGameOver(true);
    }
  }, [round]);

  useEffect(() => {
    alert('Good game!');
  }, [isGameOver]);

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    } else {
      setCard(nextCard);
    }
  }

  // ...
```

이 코드는 두 가지 문제가 있다.

첫 번째 문제는 비효율적이라는 것이다. 컴포넌트는 각 `set` 호출 시 리랜더를 일으킨다. 위 예에서, 최악의 상황은(setCard → render → setGoldCardCount → render → setRound → render → setIsGameOver → render) 하위 트리에 불필요한 세 번의 리랜더가 일어나는 것이다.

느릴 뿐아니라, 코드가 체인에 연관되어있을 시, 새로운 요구사항을 충족시키지 못한다. 이동 기록을 추가하고 싶다고 생각해보자. 과거에서 각 상태를 업데이트 해서 구현할 것이다. 하지만, 과거의 카드 상태를 설정한다면 이펙트를 실행하고 당신이 보는 데이터를 바꾼다. 이런 코드는 경직되었고 부서지기 쉽다.

이 예에서 랜더 중 계산하는 것과 상태를 이벤트 핸들러로 조정하는 것이 낫다.

```jsx
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);

  // ✅ Calculate what you can during rendering
  const isGameOver = round > 5;

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    }

    // ✅ Calculate all the next state in the event handler
    setCard(nextCard);
    if (nextCard.gold) {
      if (goldCardCount <= 3) {
        setGoldCardCount(goldCardCount + 1);
      } else {
        setGoldCardCount(0);
        setRound(round + 1);
        if (round === 5) {
          alert('Good game!');
        }
      }
    }
  }

  // ...
```

이것은 더 효율적이다. 또, 기록을 구현할 때 모든 값을 변경하는 이펙트를 실행 시킬 필요 없이 할 수 있다.
만약 로직을 재활용하고 싶다면 함수로 뽑아내 이벤트 핸들러에서 사용할 수 있다.

이벤트 핸들러 안의 상태는 스냅샷과 같다.
예를 들어, `setRound(round + 1)`를 호출한 뒤라도, `round`는 유저가 클릭한 시점의 값을 보여준다.
만약 당신이 다음 계산 값을 알고 싶다면,`const nextRound = round + 1`로 정의해라.

어떤 경우에, 당신은 이벤트 핸들러에서 다음 상태를 직접 계산할 수 없을 것이다.
예를 들어, 이전에 선택된 드롭다운에 따르는 여러 드롭다운이 있는 폼을 생각해보자. 그러면, 네트워크와 동기화하기 위해 이펙트를 체인하는 게 올바르다.

### Initializing the application

어떤 로직은 앱이 로드되면 오직 한 번만 실행되어야 한다.
당신은 아마 최상위 컴포넌트에 이펙트를 추가할 것이다.

```jsx
function App() {
  // 🔴 Avoid: Effects with logic that should only ever run once
  useEffect(() => {
    loadDataFromLocalStorage();
    checkAuthToken();
  }, []);
  // ...
}
```

하지만 당신은 두 번 실행되는 것을 알것이다.
이것은 문제가 된다.- 예를 들어, 이것은 함수가 두 번 호출되도록 디자인되지 않았기에 인증 토큰을 무력화할 것이다.
보편적으로, 당신의 컴포넌트는 리마운트 될 동안 원상복귀된다.
최상위 `App`도 포함된다.

비록 배포 환경에서 리마운트는 일어나지 않지만, 다음과 같은 제약을 주는 것이 코드를 더 쉽고 재사용을 쉽게 해준다.
만약 어떤 로직이 컴포넌트가 마운트 될때가 아닌 로드될 때 실행되어야 한다면, 최상위 변수를 추가해 이미 실행되었는지 추적하도록 한다.

```jsx
let didInit = false;

function App() {
  useEffect(() => {
    if (!didInit) {
      didInit = true;
      // ✅ Only runs once per app load
      loadDataFromLocalStorage();
      checkAuthToken();
    }
  }, []);
  // ...
}
```

당신은 앱이 랜더하기 전에 모듈 초기화 단계에서 실행시킬 수도 있다.

```jsx
if (typeof window !== "undefined") {
  // Check if we're running in the browser.
  // ✅ Only runs once per app load
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

컴포넌트가 임포트 되었을 시, 최상위에서 동작하는 코드가 한 번 실행된다. - 랜더 되지 않는 경우도 마찬가지다.
임의의 컴포넌트를 가져올 때, 속도 저하와 예상치 못한 동작을 피하기 위해, 이런 패턴을 남용하지마라.
초기화 로직은 `App.js` 같은 루트 컴포넌트나 애플리케이션의 진입점에 유지한다.

### Notifying parent components about state changes

isOn(true or false) 상태를 갖는 토글 컴포넌트를 작성한다고 하자.
토글하는 몇 개의 방식이 있다.
당신은 내부 토글 상태 변화를 부모에게 알리기 위해 `onChange`이벤트를 이펙트에서 호출한다.

```jsx
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  // 🔴 Avoid: The onChange handler runs too late
  useEffect(() => {
    onChange(isOn);
  }, [isOn, onChange]);

  function handleClick() {
    setIsOn(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      setIsOn(true);
    } else {
      setIsOn(false);
    }
  }

  // ...
}
```

이전과 같이, 이것은 이상적이지 않다. 토글은 상태를 먼저 변경하고, 리액트가 화면을 변경한다. 그 후 리액트는 이펙트를 실행하는데, 부모에서 받아온 `onChange`를 실행한다.
이제 부모는 그들의 상태를 변경하고 또 다른 랜더를 일으킨다.
한 번에 모든 것을 처리하는 것이 더 좋다.

이펙트를 지우고 동일한 이벤트 핸들러에서 두 상태를 모두 변경한다.

```jsx
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  function updateToggle(nextIsOn) {
    // ✅ Good: Perform all updates during the event that caused them
    setIsOn(nextIsOn);
    onChange(nextIsOn);
  }

  function handleClick() {
    updateToggle(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      updateToggle(true);
    } else {
      updateToggle(false);
    }
  }

  // ...
}
```

이런 접근은, 토글과 부모 컴포넌트가 상태를 이벤트를 통해 변경하게된다.
리액트는 배치 업데이트를 하기에 오직 한 번의 랜더가 일어난다.

당신은 상태를 지우고 부모에서 `isOn`을 받아올 수 있다.

```jsx
// ✅ Also good: the component is fully controlled by its parent
function Toggle({ isOn, onChange }) {
  function handleClick() {
    onChange(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      onChange(true);
    } else {
      onChange(false);
    }
  }

  // ...
}
```

상태를 끌어 올리는 것은 부모 컴포넌트가 토글 컴포넌트를 부모의 상태로 완전히 제어한다는 것을 뜻한다.
이것은 부모가 로직을 더 소유한다는 것이지만, 신경쓸 상태가 적어진다.
당신이 두 상태를 동기화하길 시도한다면, 상태를 끌어올려보자.

### Passing data to the parent

자식 컴포넌트는 데이터를 페치해 이펙트를 통해 부모 컴포넌트로 넘겨준다.

```jsx
function Parent() {
  const [data, setData] = useState(null);
  // ...
  return <Child onFetched={setData} />;
}

function Child({ onFetched }) {
  const data = useSomeAPI();
  // 🔴 Avoid: Passing data to the parent in an Effect
  useEffect(() => {
    if (data) {
      onFetched(data);
    }
  }, [onFetched, data]);
  // ...
}
```

리액트에서, 데이터 플로우는 부모에서 자식이다. 화면에 오류가 발생하면, 당신은
어떤 컴포넌트가 잘못된 프랍이나 상태를 전달했는지 상위 컴포넌트로 체이닝하면서 추적할 수 있다.
자식 컴포넌트가 부모 컴포넌트의 상태를 이펙트 안에서 변경하면 추적하기 어려워진다.
두 컴포넌트가 데이터가 필요하면 부모에서 페치한 후 자식에게 넘겨줘야한다.

```jsx
function Parent() {
  const data = useSomeAPI();
  // ...
  // ✅ Good: Passing data down to the child
  return <Child data={data} />;
}

function Child({ data }) {
  // ...
}
```

이것은 간단하고 데이터 흐름도 부모에서 자식으로 예측 가능하다.

### Subscribing to an external store

가끔 당신의 컴포넌트는 리액트 상태 밖의 데이터를 구독할 때가 있다.
이 데이터는 서드 파티 라이브러리나 빌트인 브라우저 api에서 올 수 있다.
이런 데이터는 리액트가 모르게 변경 되므로, 수동적으로 싱크를 맞춰야한다.
이것은 종종 이펙트로 구현한다.

```jsx
function useOnlineStatus() {
  // Not ideal: Manual store subscription in an Effect
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function updateState() {
      setIsOnline(navigator.onLine);
    }

    updateState();

    window.addEventListener("online", updateState);
    window.addEventListener("offline", updateState);
    return () => {
      window.removeEventListener("online", updateState);
      window.removeEventListener("offline", updateState);
    };
  }, []);
  return isOnline;
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```

컴포넌트는 외부 데이터 스토어(`navigator.onLine`)를 구독한다.
api가 서버에 존재하지 않으므로, 상태는 true로 설정된다.
브라우저의 데이터 스토어가 변할 때마다 컴포넌트는 상태를 변경한다.

비록 보통 이펙트를 사용하지만, 리액트는 외부 스토어를 구독하기위한 선호되는 빌트인 훅이 존재한다.
이펙트를 지우고 `useSyncExternalStore`를 호출한다.

```jsx
function subscribe(callback) {
  window.addEventListener("online", callback);
  window.addEventListener("offline", callback);
  return () => {
    window.removeEventListener("online", callback);
    window.removeEventListener("offline", callback);
  };
}

function useOnlineStatus() {
  // ✅ Good: Subscribing to an external store with a built-in Hook
  return useSyncExternalStore(
    subscribe, // React won't resubscribe for as long as you pass the same function
    () => navigator.onLine, // How to get the value on the client
    () => true // How to get the value on the server
  );
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```

이 접근 방식은 직접적으로 이펙트를 사용해 상태를 뮤터블 데이터와 싱크를 맞추는 것보다 에러를 줄입니다.
`useOnlineStatus()` 커스텀 훅으로 사용해 각 컴포넌트에서 반복할 필요 없게할 수 있습니다.

### Fetching data

많은 앱들은 데이터 페치를 위해 이펙트를 사용한다. 다음과 같이 이펙트와 데이터 페치하는 것은 보편적이다.

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);

  useEffect(() => {
    // 🔴 Avoid: Fetching without cleanup logic
    fetchResults(query, page).then((json) => {
      setResults(json);
    });
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

당신은 이 페치를 이벤트 핸들러로 옮길 필요 없습니다.

이것은 이벤트 핸들러에 로직을 옮겼던 이전 예와 모순이 될 수도 있다.
하지만, 타이핑 이벤트가 페치를 일으키는 이유가 아니다.
검색 인풋은 보통 URL에 많고 인풋을 건드리지 않고 유저는 앞뒤로 이동한다.

페이지와 쿼리가 어디서 왔는지 상관없다.
컴포넌트가 보일때, results를 현재 페이지와 쿼리의 네트워크에서 받아온 데이터와 동기화시키길 원한다. 그러므로 이것이 이펙트에 있는 이유다

하지만 hello를 빠르게 타입하면 쿼리는 h에서 he, hel, hell 그리고 hello로 변할 것이다. 이것은 다른 페치를 일으키고, 어떤 응답이 올지 보장이 없다. 예를 들어, hell 응답이 hello 보다 늦게 올 수 있다.
setResults()를 마지막에 호출해, 당신은 잘못된 검색 결과를 보일 수 있다.
이것을 경쟁 상태(두 요청이 경쟁해 기대와 다른 응답이 오는 것)라고 부른다.

이런 경쟁 상태를 해결하기 위해, 불필요한 응답을 무시하기 위해 정리 함수를 추가해야한다.

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);
  useEffect(() => {
    let ignore = false;
    fetchResults(query, page).then((json) => {
      if (!ignore) {
        setResults(json);
      }
    });
    return () => {
      ignore = true;
    };
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

이것은 마지막 요청의 응답을 제외한 모든 응답이 무시되는 것을 보장한다.

경쟁 상태는 페치에서만 나타나는 어려움은 아니다.
당신은 응답 캐시, 서버에서 데이터를 가져오는 것, 네트워크 폭포를 피하는 것을 생각할 것이다.

이것들은 리액트 뿐 아니라 모든 UI 라이브러리의 문제이다.
이것을 해결하는 것은 사소하지 않다. 그러므로 모던 프레임워크들이 이펙트에서 데이터를 페치하는 것보다 더 효율적인 빌트인 데이터 페칭 메커니즘을 제공하는 이유다.

프레임워크를 사용하지 않는 다면, 페치하는 커스텀 훅으로 뽑아내는 것을 고려해야한다.

```jsx
function SearchResults({ query }) {
  const [page, setPage] = useState(1);
  const params = new URLSearchParams({ query, page });
  const results = useData(`/api/search?${params}`);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}

function useData(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
    let ignore = false;
    fetch(url)
      .then((response) => response.json())
      .then((json) => {
        if (!ignore) {
          setData(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [url]);
  return data;
}
```

당신은 또한 에러 핸들링 로직을 추가하거나 로딩을 추가할 수 있다.
당신은 커스텀 훅을 만들수 있고 리액트 생태계에서 많은 해결책 중 하나를 선택할 수 있다. 이것만으로 프레임워크의 빌트인 데이터 페칭 메커니즘을 사용하는 것보다 효율적이지 않지만, 커스텀 훅으로 만들면 나중에 효율적으로 데이터 페칭 전략을 채택하기 쉬워진다.

이펙트를 작성할 때, 커스텀 훅으로 추출하는 것을 고려해라.
커스텀 훅은 더 선언적이며 목적을 위해 만들어진 API `useData` 같다.
컴포넌트 안의 적은 `useEffect`는 당신의 애플리케이션을 유지 보수하기 쉽게해준다.

## Recap

- 만약 랜더 중 계산이 가능하면, 이펙트가 불필요
- 비싼 계산일 경우 `useEffect` 대신 `useMemo` 사용
- 전체 컴포넌트 트리의 상태를 초기화하기 위해선, 다른 키를 추가
- 특정 프랍에 따라 상태를 변경하려면, 랜더 중에 설정
- 코드가 컴포넌트가 화면에 보이면서 실행될 경우 이펙트 안에, 나머지는 이벤트 안에
- 여러 상태에서 상태를 업데이트할 경우, 단일 이벤트에서 처리하는 것이 효율
- 다른 컴포넌트에서 상태를 동기화시킬 경우, 상태 끌어올리기를 고려
- 이펙트 안에서 데이터 페치할 수 있지만, 경쟁 상태를 피하기 위해 정리 함수를 구현

---

## What I Learned

일단 제법 어렵다. 여러번 읽어봐야할 것 같다.

컴포넌트가 화면에 보이면서 어떤 로직이 실행될 경우 -> 이펙트 안에서
그 외의 나머지 로직 실행의 경우 (e.g 상호작용) -> 이벤트 핸들러
또 랜더 중에 계산할 수 있다면 굳이 이펙트안에서 처리하지 말기.
상태를 끌어올려 싱크 맞추기.
경쟁 상태를 피하기 위해 클린업 함수를 추가하자.
데이터 페칭 라이브러리를 사용하는 것도 좋은 선택지.

지금 껏 코드를 작성하면서, 이펙트가 필요 없는 데 사용한 코드가 상당할 것 같다는 느낌이 들었다.

몇 번 더 읽어보면서 이해해야지.

출처

- https://react.dev/learn/you-might-not-need-an-effect
