# Writing Markup with JSX

JSX는 JavaScript 내부에서 HTML스러운 마크업이 가능하게 해주는 문법입니다.
다른 방식으로 컴포넌트를 생성하는 방식도 있지만, 대부분의 리액트 개발자들은 이 문법을 사용합니다.

## JSX: Putting markup into JavaScript

웹은 HTML, CSS 그리고 JavaScript로 만들어졌었습니다. 많은 기간 동안, 웹 개발자들은 HTML에 컨텐트를, 디자인은 CSS로 로직은 JavaScript로 각각 다른 파일에 두었습니다.

```html
// HTML

<div>
  <p></p>
  <form></form>
</div>
```

```js
// JavaScript

isLoggedIn() {}
onClick() {}
onSubmit() {}
```

그러나 웹이 점점 더 상호적이게 되면서, 로직이 컨텐트를 결정하게 되었습니다. JavaScript가 HTML을 담당하게 된 것처럼 말이죠!
리액트에서 렌더링 로직과 마크업이 한 공간에서 존재하게 된 이유입니다.

```jsx
Sidebar() {
  if(isLoggedIn()) {
    <p>Welcome</p>
  } else {
    <Form />
  }
}

Form() {
  onClick() {...}
  onSubmit() {...}

  <form onSubmit>
    <input onClick/>
    <input onClick/>
  </form>
}
```

버튼의 렌더링 로직과 마크업을 함께 두면, 변화마다 싱크를 맞추는 것을 보장합니다.
반대로, 버튼과 사이드바의 마크업이 관계가 없다면, 서로 독립적이므로 변화시키는 것에 있어서 안전합니다.

각각의 리액트 컴포넌트는 브라우저에 렌더링 되는 마크업을 갖춘 JavaScript 함수입니다.
리액트의 컴포넌트는 마크업을 위해 JSX라는 문법을 사용합니다. JSX는 HTML가 매우 유사하게 생겼지만, 좀 더 엄격하며, 동적인 정보를 보여줍니다.

> JSX와 리액트는 서로 다릅니다. 그들은 종종 함께 쓰이지만, 따로 사용할 수 있습니다. JSX는 확장된 문법입니다.

## Converting HTML to JSX

당신은 문법적으로 완벽한 HTML이 있습니다.

```html
<h1>Hedy Lamarr's Todos</h1>
<img src="https://i.imgur.com/yXOvdOSs.jpg" alt="Hedy Lamarr" class="photo" />
<ul>
  <li>Invent new traffic lights</li>
  <li>Rehearse a movie scene</li>
  <li>Improve the spectrum technology</li>
</ul>
```

그리고 이것으로 컴포넌트를 만들고 싶어 합니다.

```jsx
export default function TodoList() {
  return (
    // ???
  )
}
```

만약 복사 붙여넣기 한다면, 이것은 제대로 동작하지 않을 것입니다.

```jsx
export default function TodoList() {
  return (
    // This doesn't quite work!
    <h1>Hedy Lamarr's Todos</h1>
    <img
      src="https://i.imgur.com/yXOvdOSs.jpg"
      alt="Hedy Lamarr"
      class="photo"
    >
    <ul>
      <li>Invent new traffic lights
      <li>Rehearse a movie scene
      <li>Improve the spectrum technology
    </ul>
  );
}
```

```bash
/App.js: Adjacent JSX elements must be wrapped in an enclosing tag. Did you want a JSX fragment <>...</>? (5:4)

```

이것은 JSX가 그냥 HTML보다 더 엄격하기 때문입니다. 에러 메시지를 확인하면 어떻게 고쳐야 하는지 알 수 있습니다. 또는 아래의 가이드를 따라 할 수도 있습니다.

> 대부분의 리액트 에러 메시지들은 문제 해결에 도움을 줍니다.

## The Rules of JSX

### 1. Return a single root element

여러 개의 요소들을 컴포넌트에서 반환하려면, 그들을 부모 태그로 감싸야 합니다.

예를 들어, `<div>` 태그를 사용할 수 있습니다.

```jsx
<div>
  <h1>Hedy Lamarr's Todos</h1>
  <img
    src="https://i.imgur.com/yXOvdOSs.jpg"
    alt="Hedy Lamarr"
    class="photo"
  >
  <ul>
    ...
  </ul>
</div>
```

만약 추가적인 `<div>`가 싫다면, `<></>`로 대신할 수 있습니다.

```jsx
<>
  <h1>Hedy Lamarr's Todos</h1>
  <img
    src="https://i.imgur.com/yXOvdOSs.jpg"
    alt="Hedy Lamarr"
    class="photo"
  >
  <ul>
    ...
  </ul>
</>
```

이 빈 태그는 Fragment라고 불립니다. Fragment는 HTML 트리에 어떤 흔적도 남기지 않고 그룹화를 해줍니다.

### DEEP DIVE Why do multiple JSX tags need to be wrapped?

JSX는 HTML과 닮아있는데, 내부적으로는 자바스크립트의 객체로 변환됩니다. 두 가지 객체는 배열로 감싸기 전에는 함수의 반환값이 될 수 없습니다. JSX 태그들이 부모 태그 없이 못 쓰이는 이유이기도 합니다.

### Close all the tags

JSX는 명시적으로 닫혀야 합니다. 닫힌 태그인 `<img>` 태그도 반드시 `<img/>`와 같이 작성되어야 합니다. 감싸는 태그 역시 마찬가지입니다.

### 3. camelCase ~~all~~ most of the things!

JSX는 자바스크립트로 변환되며, JSX의 속성들은 자바스크립트 객체의 키가 됩니다.
컴포넌트 안에서 당신은 종종 이런 속성들을 변수로 읽고 싶을 것입니다.
하지만 자바스크립트는 변수명에 제한이 있습니다. 예를 들어, `-`나 예약어의 경우 변수명으로 사용할 수 없습니다.

위의 이유가 리액트에서 HTML, SVG 속성 등이 카멜 케이스로 작성되는 이유입니다. 예를 들어, `stroke-width` 대신 `strokeWidth`를 사용하는 것처럼. `class`가 예약어이기에, 리액트는 `className`을 사용합니다.

> 역사적인 이유로 `aria-*`와 `data-*` 속성은 `-`를 사용합니다.

### Pro-tip: Use a JSX Converter

기존에 작성된 마크업을 바꾸는 것은 지루합니다! 이 [컨버터](https://transform.tools/html-to-jsx)를 사용하면 HTML과 SVG를 쉽게 JSX로 변환할 수 있습니다.

## Recap

이제 당신은 JSX의 존재와 컴포넌트에서 어떻게 사용하는지 알게 되었습니다.

- 리액트 컴포넌트 그룹은 마크업과 렌더링이 서로 연관되어 있어 로직을 같이 수행합니다.
- JSX는 HTML과 비슷하지만, 몇 가지 차이가 있습니다.
- 에러 메시지는 마크업 에러를 집어줍니다.
