- [Your First Component](#your-first-component)
  - [Components: UI building blocks](#components-ui-building-blocks)
  - [Defining a component](#defining-a-component)
    - [Step 1: Export the component](#step-1-export-the-component)
    - [Step 2: Define the function](#step-2-define-the-function)
    - [Step 3: Add markup](#step-3-add-markup)
  - [Using a component](#using-a-component)
    - [What the browser sees](#what-the-browser-sees)
    - [Nesting and organizing components](#nesting-and-organizing-components)
  - [DEEP DIVE Components all the way down](#deep-dive-components-all-the-way-down)
  - [Recap](#recap)
  - [What I Learned](#what-i-learned)

# Your First Component

컴포넌트들은 리액트의 코어 컨셉입니다.
그들은 UI의 초석입니다. 최적의 장소에 두는 것이 리액트의 시작입니다.

## Components: UI building blocks

웹에서 HTML은 각종 태그들로 우리가 잘 구조화된 문서를 만들 수 있게 해줍니다.

```html
<article>
  <h1>My First Component</h1>
  <ol>
    <li>Components: UI Building Blocks</li>
    <li>Defining a Component</li>
    <li>Using a Component</li>
  </ol>
</article>
```

위 마크업은 `<article>`로 아티클을 보여주고 `<h1>` 태그로 헤딩을 보여줍니다. 그리고 컨텐츠를 `<ol>`을 통해 보여줍니다. 이런 마크업은 CSS, 자바스크립트와 결합되어 사이드바, 아바타, 모달, 드롭 다운 등 모든 웹에서 보이는 요소입니다.

리액트는 재사용 가능한 UI 엘리먼트인 컴포넌트에 마크업 CSS, 자바스크립트를 한데 모을 수 있게 해줍니다.
위의 예제의 코드는 `<TableOfContents>` 컴포넌트가 되어 모든 페이지에 렌더 될 수 있습니다. 내부적으로 HTML 태그 사용이 가능합니다.

HTML 태그와 마찬가지로 컴포넌트를 조합하고 중첩시킬 수 있습니다.

```jsx
<PageLayout>
  <NavigationHeader>
    <SearchBar />
    <Link to="/docs">Docs</Link>
  </NavigationHeader>
  <Sidebar />
  <PageContent>
    <TableOfContents />
    <DocumentationText />
  </PageContent>
</PageLayout>
```

프로젝트가 커지며, 대다수의 디자인에 대하여 이미 작성한 컴포넌트를 재사용할 수 있음을 알 수 있으며 개발 속도가 빨라질 것입니다.

## Defining a component

전통적으로 웹 페이지를 만들 때, 마크업을 짜고 자바스크립트를 추가하여 상호작용하도록 하였습니다.
이것은 상호작용이 웹에서 좋을 때 잘 작동하였습니다. 요즘의 많은 사이트와 앱에서 상호작용이 기대됩니다.
리액트는 위와 똑같이 동작하지만 상호작용을 우선으로 생각합니다. 리액트 컴포넌트는 마크업이 추가된 자바스크립트의 함수입니다.

컴포넌트를 만드는 법은 다음과 같습니다.

### Step 1: Export the component

`export default`는 다른 파일에서 import 해와서 사용할 수 있도록 해주는 자바스크립트의 문법입니다.

### Step 2: Define the function

대문자를 사용해 함수를 정의합니다. `function Profile() { }`

> 리액트의 컴포넌트는 자바스크립트의 함수지만 대문자를 사용해야 합니다.

### Step 3: Add markup

예제 컴포넌트는 `<img/>` 태그를 반환합니다.
`<img/>`는 HTML과 비슷하게 작성되지만, 이것은 실제로 JSX 문법입니다. 이것은 자바스크립트 안에서 마크업을 작성할 수 있게 해줍니다.

마크업이 `return` 키워드와 같은 줄이 아니라면 반드시 ()로 감싸줘야 합니다.

```jsx
return (
  <div>
    <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
  </div>
);
```

> 만약 ()로 감싸지 않는다면 `return` 키워드 아래 코드들은 무시될 것입니다.

## Using a component

이렇게 만들어진 컴포넌트는 다른 컴포넌트 안에 중첩해서 사용할 수 있습니다. 예를 들어 `Gallery` 컴포넌트 안에 `Profile` 컴포넌트를 여러 번 사용할 수 있습니다.

### What the browser sees

리액트의 컴포넌트는 대문자로 시작하는데, 브라우저에서는 HTML 태그로 보입니다.

### Nesting and organizing components

컴포넌트는 자바스크립트의 함수로, 같은 파일에 여러 개의 컴포넌트를 소유할 수 있습니다.
이것은 컴포넌트가 서로 밀접하게 관련되어 있는 경우 편합니다. 만약 파일이 어지럽다면, 다른 파일로 옮길 수 있습니다.

`Profile`가 `Gallery` 안에서 렌더링 되므로 우리는 `Gallery`는 부모 컴포넌트, `Profile`은 자식 컴포넌트라고 부를 수 있습니다. 한번 컴포넌트를 정의하면 여러 곳에서 여러 번 재사용할 수 있습니다.

> 컴포넌트는 다른 컴포넌트들 안에서 렌더링 될 수 있지만, 중첩해서 정의하면 안 됩니다.
> 중첩해서 작성하면 매우 느리면 버그를 일으킵니다. 모든 컴포넌트는 최상위 레벨에 정의되어야 합니다.
> 만약 자식 컴포넌트가 데이터가 필요하면 props로 넘겨 줄 수 있습니다.

## DEEP DIVE Components all the way down

리액트 애플리케이션은 루트 컴포넌트에서 시작됩니다. 새로운 프로젝트를 만들었을 때 자동을 생성됩니다(CRA, CodeSandbox).

대부분의 리액트 앱들은 컴포넌트를 사용합니다. 이것은 재사용 가능한 버튼 같은 컴포넌트뿐 아니라 더 큰 사이드바, 리스트 그리고 심지어 페이지 역시 사용할 수 있다는 뜻입니다.
단 한 번 사용된다 하더라도 컴포넌트는 UI를 잘 구성할 수 있는 좋은 방법입니다.

리액트 기반의 프레임워크들은 빈 HTML 파일을 만드는 것이 아닌 HTML 파일을 생성합니다. 이것은 자바스크립트가 로드되기 전 컨텐츠를 보여줄 수 있습니다.

만은 웹사이트들이 리액트를 사용하는데, 루트 컴포넌트를 여러 개 둘 수 있습니다. 필요에 의해 리액트를 많게 혹은 적게 사용 가능합니다.

## Recap

- 리액트는 재사용이 가능한 UI인 컴포넌트를 만들어 줍니다.
- 리액트 앱에서 모든 UI들은 컴포넌트입니다.
- 리액트 컴포넌트는 예외가 있는 자바스크립트의 함수입니다.
  - 대문자로 시작
  - JSX를 리턴

---

## What I Learned

리액트로 개발을 하면서, 가끔 컴포넌트가 함수라는 사실을 간과할 때가 있는데, 확실히 머릿속에 각인을 시킨 것 같다. UI들을 재사용할 수 있다는 점은 참 매력적인 것 같다.

</br>

출처

- https://react.dev/learn/your-first-component
