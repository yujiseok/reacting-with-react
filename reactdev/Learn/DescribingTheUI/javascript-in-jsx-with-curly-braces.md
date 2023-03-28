- [JavaScript in JSX with Curly Braces](#javascript-in-jsx-with-curly-braces)
  - [Passing strings with quotes](#passing-strings-with-quotes)
  - [Using curly braces: A window into the JavaScript world](#using-curly-braces-a-window-into-the-javascript-world)
    - [Where to use curly braces](#where-to-use-curly-braces)
  - [Using “double curlies”: CSS and other objects in JSX](#using-double-curlies-css-and-other-objects-in-jsx)
  - [More fun with JavaScript objects and curly braces](#more-fun-with-javascript-objects-and-curly-braces)
  - [Recap](#recap)
  - [What I Learned](#what-i-learned)

# JavaScript in JSX with Curly Braces

JSX는 자바스크립트 내에서 렌더링 로직과 컨텐트를 가지며 HTML과 비슷한 마크업을 가능하게 해줍니다.
당신은 가끔 자바스크립트 로직을 추가하고 싶거나 동적인 요소를 추가하고 싶을 수 있습니다.
이 상황에 `{}`를 사용해, 자바스크립트 로직을 사용할 수 있습니다.

## Passing strings with quotes

JSX에 문자로된 어트리뷰트를 넘길 때는 홑 또는 쌍따옴표를 사용합니다.

만약 동적인 값을 추가하고 싶으면 `"`또는 `"`를 `{`와 `}`로 대체합니다.

```jsx
export default function Avatar() {
  const avatar = "https://i.imgur.com/7vQD0fPs.jpg";
  const description = "Gregorio Y. Zara";
  return <img className="avatar" src={avatar} alt={description} />;
}
```

CSS의 클래스명을 나타내는 `className="avatar"`와 이미지의 자바스크립트의 변수 `avatar`를 사용한 `src={avatar}`는 서로 다르다는 것을 알아차렸을 것입니다.
중괄호가 마크업에서 자바스크립트를 사용할 수 있게 해줍니다.

## Using curly braces: A window into the JavaScript world

JSX는 자바스크립트를 작성하는 특별한 방식입니다. 이것은 자바스크립트를 중괄호를 사용해 내부적으로 사용할 수 있다는 것을 뜻합니다.

모든 자바스크립트 식은 중괄호 안에서 동작합니다.

```jsx
const today = new Date();

function formatDate(date) {
  return new Intl.DateTimeFormat("en-US", { weekday: "long" }).format(date);
}

export default function TodoList() {
  return <h1>To Do List for {formatDate(today)}</h1>;
}
```

### Where to use curly braces

두 가지 방법으로만 JSX 안에서 중괄호를 사용할 수 있다.

1. JSX 태그 안에 직접적으로 들어가는 텍스트로, `<h1>{name}'s To Do List</h1>`는 동작하지만, `<{tag}>Gregorio Y. Zara's To Do List</{tag}>`는 동작하지 않습니다.
2. `=` 연산자 뒤에 바로 오는 어트리뷰트들, `src={avatar}`는 `avatar` 변수로 읽히지만 `src="{avatar}"`의 경우 `"avatar"` 문자로 읽힙니다.

## Using “double curlies”: CSS and other objects in JSX

문자, 숫자, 자바스크립트 표현식 또한 객체 역시 JSX에 넘겨줄 수 있습니다.
객체는 이미 중괄호로 표현되어 있기 때문에, 객체를 넘겨줄 때는 객체를 중괄호로 한 번 더 감싸줍니다.
`person={{ name: "Hedy Lamarr", inventions: 5 }}`

아마 인라인 스타일링을 할 때 봤을 것입니다.
리액트는 인라인 스타일을 권장하지 않습니다. 하지만 당신이 인라인 스타일을 원한다면, `style` 어 트리뷰트에 객체를 넘겨줍니다.

```jsx
export default function TodoList() {
  return (
    <ul
      style={{
        backgroundColor: "black",
        color: "pink",
      }}
    >
      <li>Improve the videophone</li>
      <li>Prepare aeronautics lectures</li>
      <li>Work on the alcohol-fuelled engine</li>
    </ul>
  );
}
```

이제 JSX에서 `{{` `}}` 를 보면 객체 그 이상도 아니라는 것을 알 게 되었습니다.

## More fun with JavaScript objects and curly braces

객체 안으로 다양한 표현식을 옮길 수 있습니다. 그리고 JSX 안에서 참조하도록 합니다.

```jsx
const person = {
  name: "Gregorio Y. Zara",
  theme: {
    backgroundColor: "black",
    color: "pink",
  },
};

export default function TodoList() {
  return (
    <div style={person.theme}>
      <h1>{person.name}'s Todos</h1>
      <img
        className="avatar"
        src="https://i.imgur.com/7vQD0fPs.jpg"
        alt="Gregorio Y. Zara"
      />
      <ul>
        <li>Improve the videophone</li>
        <li>Prepare aeronautics lectures</li>
        <li>Work on the alcohol-fuelled engine</li>
      </ul>
    </div>
  );
}
```

위의 예에서 `person` 은 `name`과 `theme` 객체를 포함한 객체입니다.

컴포넌트 안에서 `person`은 다음과 같이 사용 가능합니다.

```jsx
<div style={person.theme}>
<h1>{person.name}'s Todos</h1>
```

JSX는 자바스크립트를 사용해 데이터와 로직을 구성할 수 있는 템플릿(very minimal) 언어입니다.

## Recap

- JSX 어트리뷰트들의 따옴표는 문자로 넘겨집니다.
- 중괄호는 마크업에 자바스크립트 로직과 변수를 사용할 수 있게 해줍니다.
- 중괄호는 JSX 태그의 컨텐트 또는 어트리뷰트 `=` 뒤에 사용할 수 있습니다.
- `{{` `}}` 는 특별한 문법이 아닌 JSX 안의 객체를 나타냅니다.

---

## What I Learned

가끔씩 JSX 안에서 중괄호를 쓰며 어떤 값을 넣어야할 지 헷갈릴 때가 있었는 데, 값으로 표현되는 표현식만 가능하다는 것을 확실히 알게 되었다.

<br>

출처

- https://react.dev/learn/javascript-in-jsx-with-curly-braces
