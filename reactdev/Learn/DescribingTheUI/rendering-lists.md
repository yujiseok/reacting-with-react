- [Rendering Lists](#rendering-lists)
  - [Rendering data from arrays](#rendering-data-from-arrays)
  - [Filtering arrays of items](#filtering-arrays-of-items)
    - [Pitfall](#pitfall)
  - [Keeping list items in order with key](#keeping-list-items-in-order-with-key)
    - [DEEP DIVE](#deep-dive)
    - [Where to get your key](#where-to-get-your-key)
    - [Rules of keys](#rules-of-keys)
    - [Why does React need keys?](#why-does-react-need-keys)
    - [Pitfall](#pitfall-1)
  - [Recap](#recap)
  - [What I Learned](#what-i-learned)

# Rendering Lists

여러 개의 컴포넌트를 하나의 데이터 컬렉션에서 보여주고 싶을 수 있습니다. 이때 데이터 배열을 조작하기 위해 자바스크립트의 배열 메서드를 사용할 수 있습니다. 이 페이지에서, `filter()`와 `map()`을 사용해 데이터 배열을 컴포넌트 배열로 필터링하고 변형시킬 수 있습니다.

## Rendering data from arrays

```jsx
<ul>
  <li>Creola Katherine Johnson: mathematician</li>
  <li>Mario José Molina-Pasquel Henríquez: chemist</li>
  <li>Mohammad Abdus Salam: physicist</li>
  <li>Percy Lavon Julian: chemist</li>
  <li>Subrahmanyan Chandrasekhar: astrophysicist</li>
</ul>
```

와 같은 리스트가 있다고 생각해 보세요.

리스트 아이템의 다른 것은 오직 데이터입니다. 아마 다른 데이터를 사용하는 똑같은 컴포넌트를 인터페이스에 보여줘야 하는 경우가 많을 것입니다.
이럴 때, 데이터를 객체나 배열에 담고 `filter()`와 `map()` 사용해 컴포넌트로 렌더 할 수 있습니다.

1. 데이터를 배열로 옮깁니다.

```js
const people = [
  "Creola Katherine Johnson: mathematician",
  "Mario José Molina-Pasquel Henríquez: chemist",
  "Mohammad Abdus Salam: physicist",
  "Percy Lavon Julian: chemist",
  "Subrahmanyan Chandrasekhar: astrophysicist",
];
```

2. `people` 멤버를 map을 이용해 JSX 노드의 새 배열로 만듭니다.

```jsx
const listItems = people.map((person) => <li>{person}</li>);
```

3. `listItems`를 `<ul>`안에 반환합니다.

```jsx
return <ul>{listItems}</ul>;
```

`Warning: Each child in a list should have a unique “key” prop.` 오류가 발생하는 데 후에 해결하는 법을 다룹니다.

## Filtering arrays of items

데이터는 다음과 같이 보일 수도 있습니다.

```js
const people = [
  {
    id: 0,
    name: "Creola Katherine Johnson",
    profession: "mathematician",
  },
  {
    id: 1,
    name: "Mario José Molina-Pasquel Henríquez",
    profession: "chemist",
  },
  {
    id: 2,
    name: "Mohammad Abdus Salam",
    profession: "physicist",
  },
  {
    name: "Percy Lavon Julian",
    profession: "chemist",
  },
  {
    name: "Subrahmanyan Chandrasekhar",
    profession: "astrophysicist",
  },
];
```

직업이 `chemist`인 사람만 보여주고 싶은 경우가 있을 수 있습니다. 자바스크립트의 `filter()`를 사용하여 조건에 부합하는 사람을 반환합니다. 이 메서드는 배열을 받아와 테스트를 합니다. 조건에 부합하면 새로운 배열을 반환합니다.

직업이 `chemist`인 사람만 필터 하는 방법은 다음과 같습니다. 테스트 함수 `(person)=> person.profession === "chemist"`

1. `chemist`인 사람들의 배열을 만들어 줍니다. `filter()` 메서드를 호출하여 조건에 맞게 필터 합니다.

```jsx
const chemists = people.filter((person) => person.profession === "chemist");
```

2. `chemists`를 map 합니다.

```jsx
const listItems = chemists.map((person) => (
  <li>
    <img src={getImageUrl(person)} alt={person.name} />
    <p>
      <b>{person.name}:</b>
      {" " + person.profession + " "}
      known for {person.accomplishment}
    </p>
  </li>
));
```

3. `listItems`를 반환합니다.

```jsx
return <ul>{listItems}</ul>;
```

### Pitfall

화살표 함수는 암시적으로 리턴합니다. 그러니 `return`문을 작성하지 않아도 됩니다.

```jsx
const listItems = chemists.map(
  (person) => <li>...</li> // Implicit return!
);
```

하지만 `=>` 뒤에 중괄호를 사용하면 반드시 명시적인 `return`문을 작성해야 합니다.

```jsx
const listItems = chemists.map((person) => {
  // Curly brace
  return <li>...</li>;
});
```

화살표 함수에서 `{}` 는 블록 바디로 한 줄 이상의 코드를 적을 수 있게 해주는 것입니다. 만약 반환을 원하면 `return`을 적어줘야 합니다. 생략한다면, 아무것도 반환되지 않습니다!

## Keeping list items in order with key

`Warning: Each child in a list should have a unique “key” prop.`

다음과 같은 에러를 보았을 것입니다.

배열 아이템에 유일한 `key`를 제공해 줘야 합니다.

```jsx
<li key={person.id}>...</li>
```

`map()` 바로 안의 JSX 요소에는 키가 항상 필요합니다.

키들은 각 컴포넌트와 배열 아이템이 상승하다는 것을 알려줍니다. 이것은 배열의 아이템들이 이동, 삽입, 삭제될 때 매우 중요합니다. 잘 설정된 키는 리액트가 어떤 것이 일어났는지 정확히 알려줍니다. 그리고 DOM 트리를 정확하게 업데이트합니다.

키를 만들기보단, 데이터 안에 넣으세요.

### DEEP DIVE

**Displaying several DOM nodes for each list item**

여러 개의 DOM 노드를 렌더 하고 싶으면 어떻게 해야 할까요?

`<>...</>` 프래그먼트 문법에 키를 넘길 수 없습니다. `<div>`를 활용해 그룹을 짓거나, 명시적이 `<Fragment>` 문법을 사용할 수 있습니다.

```jsx
import { Fragment } from "react";

// ...

const listItems = people.map((person) => (
  <Fragment key={person.id}>
    <h1>{person.name}</h1>
    <p>{person.bio}</p>
  </Fragment>
));
```

프래그먼트들은 DOM에서 사라져 `<h1>`, `<p>`, `<h1>`, `<p>` 리스트를 만듭니다.

### Where to get your key

다른 소스의 데이터는 다른 소스의 키를 제공한다.

- **Data from database** 데이터가 데이터베이스에서 온다면, 데이터베이스의 키나 아이디를 사용
- **Locally generated data** 데이터가 생성되고 로컬에 지속된다면, 카운터, `crypto.randomUUID()` 또는 `uuid` 패키지를 사용

### Rules of keys

- **키는 반드시 유일해야 합니다.** 하지만, 다른 배열에서 같은 키는 괜찮습니다.
- **키는 변하지 말아야 합니다.** 그러면 그들의 목적을 져버립니다. 렌더링 중에 생성하지 마세요!

### Why does React need keys?

데스크톱에 있는 파일들에 이름 없이, 파일들을 1번 파일, 2번 파일로 참조한다고 상상해 보세요.
하지만 파일을 지울 시, 혼란을 야기합니다. 2번 파일이 1번 파일이 되고 3번 파일이 2번 파일이 됩니다.

폴더 안의 파일 이름과 JSX의 키는 비슷한 목적을 갖습니다. 그들은 형제들과 달리 식별할 수 있도록 도와줍니다. 잘 지정된 키는 순서 말고도 더 많은 정보를 제공합니다. 위치를 재조정하더라도, 키들은 리액트가 생명주기 동안 식별할 수 있도록 해줍니다.

### Pitfall

배열의 인덱스를 키로 설정할 수 있을 것입니다. 사실 그것은 리액트가 키를 설정하지 않았을 시 하는 행동입니다. 하지만 아이템이 렌더링 될 때 추가되고 삭제될 경우 인덱스 키는 버그를 유발합니다.

비슷하게, 렌더 시에 키를 생성하지 마세요. 이 키들은 렌더링 시 절대 맞춰질 수 없으며 매번 생성되게 합니다. 느릴 뿐 아니라, 유저의 인풋 역시 잃게 만듭니다. 고정된 아이디 기반의 데이터를 사용하세요.

컴포넌트 자체는 `key`를 프랍으로 받지 못합니다. 만약 아이디를 받고 싶다면 `<Profile key={id} userId={id} />`과 같이 사용하세요.

## Recap

- 배열이나 객체의 데이터를 컴포넌트로 옮기는 법
- `map()`을 이용한 컴포넌트 생성
- `filter()`을 이용한 필터 된 컴포넌트 생성
- 리액트가 `key`를 통해 각 컴포넌트를 추적하는 법

---

## What I Learned

리액트를 공부하고 사용하면서 제일 좋아하는 부분이 바로 이 리스트를 렌더링 하는 부분인데, 불확실했던 키의 역할을 제법 확실하게 알게 되었다. 키는 즉, 리액트가 각 컴포넌트를 식별할 수 있게 해주는 고유한 이름의 역할을 한다.

리액트를 처음 배우고 자바스크립트도 익숙지 않았을 때, `map()`으로 배열을 순회하여도 반환되는 것이 없어서 엄청나게 고민을 했던 기억이 있는데, `return`을 안 해줘서 였다는 것을 알았을 때의 허무함이 갑자기 생각이 났다. 그런 고민의 시간들 덕분에 내가 성장한 게 아닐까 싶다! 열심히 오류와 마주하고 열심히 해결하면서 성장해야지.

출처

- https://react.dev/learn/rendering-lists
