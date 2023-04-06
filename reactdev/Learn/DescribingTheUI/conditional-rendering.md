- [Conditional Rendering](#conditional-rendering)
  - [Conditionally returning JSX](#conditionally-returning-jsx)
  - [Conditionally returning nothing with `null`](#conditionally-returning-nothing-with-null)
  - [Conditionally including JSX](#conditionally-including-jsx)
    - [Conditional (ternary) operator (? :)](#conditional-ternary-operator--)
    - [Are these two examples fully equivalent?](#are-these-two-examples-fully-equivalent)
    - [Logical AND operator (\&\&)](#logical-and-operator-)
    - [Pitfall](#pitfall)
  - [Conditionally assigning JSX to a variable](#conditionally-assigning-jsx-to-a-variable)
  - [Recap](#recap)
  - [What I Learned](#what-i-learned)

# Conditional Rendering

당신의 컴포넌트는 때로는 다른 조건에 따른 화면을 보여준다.
리액트에서 조건부 렌더링을 하기 위해선 `if`, `&&`, `? :` 연산자를 사용한다.

## Conditionally returning JSX

`Item`들을 렌더링하는 `PackingList` 컴포넌트가 있다고 생각해 보자.

```jsx
function Item({ name, isPacked }) {
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item isPacked={true} name="Space suit" />
        <Item isPacked={true} name="Helmet with a golden leaf" />
        <Item isPacked={false} name="Photo of Tam" />
      </ul>
    </section>
  );
}
```

몇 `Item` 컴포넌트들의 `isPacked` 프랍에 `true`로 지정해 ✔마크를 표시하고 싶다면 다음과 같이 작성할 수 있다.

```jsx
if (isPacked) {
  return <li className="item">{name} ✔</li>;
}
return <li className="item">{name}</li>;
```

만약 `isPacked` 프랍이 `true`일 경우 **다른 JSX 트리**를 반환한다.
이런 변화로 몇 아이템들은 체크마크를 갖게 될 것이다.

```jsx
function Item({ name, isPacked }) {
  if (isPacked) {
    return <li className="item">{name} ✔</li>;
  }
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item isPacked={true} name="Space suit" />
        <Item isPacked={true} name="Helmet with a golden leaf" />
        <Item isPacked={false} name="Photo of Tam" />
      </ul>
    </section>
  );
}
```

리액트에서 제어 흐름(조건문과 같은)는 자바스크립트를 통해 다뤄진다.

## Conditionally returning nothing with `null`

어떤 상황에서는, 아무것도 반환하고 싶지 않을 수 있다. 예를 들어 챙긴 아이템을 보여주기 싫을 때.
컴포넌트는 반드시 무언가를 반환해야 한다. 이때, `null`을 반환할 수 있다.

```jsx
if (isPacked) {
  return null;
}
return <li className="item">{name}</li>;
```

만약 `isPacked`가 참이면 컴포넌트는 아무것도 반환하지 않을 것이다. 거짓이면 JSX를 반환할 것이다.

```jsx
function Item({ name, isPacked }) {
  if (isPacked) {
    return null;
  }
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item isPacked={true} name="Space suit" />
        <Item isPacked={true} name="Helmet with a golden leaf" />
        <Item isPacked={false} name="Photo of Tam" />
      </ul>
    </section>
  );
}
```

예제에서 `null`을 반환하는 것은 개발자가 렌더링을 시킬 시 놀랄 수 있기에 일반적인 것은 아니다.
더 자주, 부모 컴포넌트의 JSX에서 조건을 추가해 컴포넌트를 포함하고 제외할 것이다.

## Conditionally including JSX

이전의 예에서 조건에 따라 JSX 트리가 반환되게 했을 것이다. 아마 렌더 결과에 중복이 있다는 것을 눈치챘을 것이다.

```jsx
<li className="item">{name} ✔</li>
```

와

```jsx
<li className="item">{name}</li>
```

는 매우 닮아있다.

각각의 조건 브랜치들은 `<li className="item">...</li>`를 반환한다.

이런 중복은 안 좋은 것은 아니지만, 코드의 유지 보수를 어렵게 한다. 클래스 네임을 바꾸고 싶더라도 두 곳에서 고쳐야 한다. JSX 안에서 조건 처리를 할 수 있다.

### Conditional (ternary) operator (? :)

자바스크립트는 조건 처리를 위한 간단한 문법을 가지고 있는데, 그것은 "삼 항 연산자"이다.

```jsx
if (isPacked) {
  return <li className="item">{name} ✔</li>;
}
return <li className="item">{name}</li>;
```

대신

```jsx
return <li className="item">{isPacked ? name + " ✔" : name}</li>;
```

와 같이 작성할 수 있다.

### Are these two examples fully equivalent?

OOP에 익숙하다면 `<li>`의 서로 다른 두 개의 인스턴스를 생성할 수 있기 때문에 두 예제는 다르다고 추측할 것이다. 하지만 JSX 요소는 인스턴스가 아니다. 왜냐하면 실제로 내부적인 상태를 유지하지 않고 실제 돔노드가 아니기 때문에. 그들은 가벼운 청사진이다. 그래서 위 두 예제는 완벽하게 같다.

다른 HTML 태그 `<del>`과 같은 태그로 감싸고 싶을 때, 중괄호를 활용해 쉽게 추가할 수 있다.

```jsx
function Item({ name, isPacked }) {
  return <li className="item">{isPacked ? <del>{name + " ✔"}</del> : name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item isPacked={true} name="Space suit" />
        <Item isPacked={true} name="Helmet with a golden leaf" />
        <Item isPacked={false} name="Photo of Tam" />
      </ul>
    </section>
  );
}
```

위 같은 스타일은 간단한 조건에는 좋지만, 잘 절제해야 한다. 컴포넌트가 너무 많은 조건으로 어지러워지면, 자식 컴포넌트로 추출하는 것을 고려해라. 리액트에서 마크업은 코드의 한 부분이다. 복잡한 표현식을 변수나, 함수를 이용할 수 있다.

### Logical AND operator (&&)

다른 숏컷으로는 자바스크립트의 논리 연산자이다. 리액트에서 조건이 true 일 때만 JSX를 리턴하고 아닐 시 아무것도 반환하지 않는 경우가 많다. `&&`를 활용해 오직 참일 경우에만 체크 마크를 렌더 할 수 있다.

```jsx
return (
  <li className="item">
    {name} {isPacked && "✔"}
  </li>
);
```

위의 코드를 해석하면, 만약 `isPacked`가 참이면 체크 마크를 렌더하고 아닐 시 아무것도 렌더 하지 마와 같다.

`&&` 논리 연산자는 왼쪽 조건이 참일 경우 오른쪽의 값을 반환한다. 만약 조건이 거짓일 시, 모든 표현식이 거짓이 된다. 리액트에서 거짓은 JSX 트리의 `null` 또는 `undefined`와 같이 구멍을 뜻하며 아무것도 렌더링 하지 않는다.

### Pitfall

**Don’t put numbers on the left side of &&.**

자바스크립트는 조건을 시험하기 위해 왼쪽 부분을 불린으로 변형시킨다. 하지만, 만약 왼쪽이 `0`일경우 리액트는 `0`을 렌더한다.

예를 들어, 자주 하는 실수로 `messageCount && <p>New messages</p>`와 같이 쓰는 것이다.
이것은 쉽게 `messageCount`가 0 일때 아무것도 렌더링 하지 않기를 기대하지만, 실제로 `0` 그 자체를 렌더 한다.

위를 고치려면 `messageCount > 0 && <p>New messages</p>`와 같이 왼쪽 부분을 불린형으로 변형 시켜야 한다.

## Conditionally assigning JSX to a variable

이런 숏컷들이 코드를 작성하는 데 방해가 될 때 `if`문과 변수를 활용해라. `let`을 이용해 변수를 재할당할 수 있다.

```js
let itemContent = name;
```

`if`문을 활용해, JSX를 재할당 시켜라.

```js
if (isPacked) {
  itemContent = name + " ✔";
}
```

중괄호에 변수를 적고, 계산된 식을 JSX 트리에 반환 시켜라.

```jsx
<li className="item">{itemContent}</li>
```

이런 스타일은 가장 장황한 코드지만, 유연하다.

```jsx
function Item({ name, isPacked }) {
  let itemContent = name;
  if (isPacked) {
    itemContent = name + " ✔";
  }
  return <li className="item">{itemContent}</li>;
}

let itemContent = name;
if (isPacked) {
  itemContent = <del>{name + " ✔"}</del>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item isPacked={true} name="Space suit" />
        <Item isPacked={true} name="Helmet with a golden leaf" />
        <Item isPacked={false} name="Photo of Tam" />
      </ul>
    </section>
  );
}
```

텍스트뿐 아니라, 임의의 JSX도 사용 가능하다.

## Recap

- 리액트에서 자바스크립트를 활용해 브랜치 로직을 조작할 수 있다.
- `if`문을 활용해 JSX를 반환할 수 있다.
- 변수에 조건에 따른 JSX를 저장해 중괄호에 사용할 수 있다.
- JSX에서 삼 항 연산자를 사용할 수 있다.
- JSX에서 논리 연산자를 사용할 수 있다.
- 숏컷들은 매우 흔하지만, `if`문을 선호하면 사용해도 된다.

---

## What I Learned

조건부 렌더링을 처리하면서 중첩이 많이 되거나, 가독성이 떨어질 때가 많았는데, 변수를 활용하는 방법을 시도해 봐야겠다. 어떤 포스트에서 `&&`를 사용하지 말라고 했던 걸 본적이 있는데, 0을 반환하는 것 아니면 사용해도 될 것 같다.

출처

- https://react.dev/learn/conditional-rendering
