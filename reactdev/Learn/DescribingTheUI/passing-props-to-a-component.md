# Passing Props to a Component

리액트 컴포넌트는 컴포넌트 간 대화를 위해 props를 사용합니다.
모든 부모 컴포넌트는 자식에게 props를 통해 정보를 전달합니다.
Props는 아마 HTML의 어트리뷰트들을 떠올리게 합니다. 하지만, 자바스크립트의 값들(객체, 배열, 함수)을 props로 전달할 수 있습니다.

## Familiar props

프랍은 JSX 태그에 전달하는 정보입니다. 예를 들어 `className`,`src`,`alt`, `width` 그리고 `height` 들은 `<img>` 태그에 전달할 수 있는 프랍입니다.

```jsx
function Avatar() {
  return (
    <img
      className="avatar"
      src="https://i.imgur.com/1bX5QH6.jpg"
      alt="Lin Lanying"
      width={100}
      height={100}
    />
  );
}
```

`<img>`에 전달할 수 있는 이미 정해져있습니다. (리액트 돔은 다음을 준수합니다. [HTML standard](https://html.spec.whatwg.org/multipage/embedded-content.html#the-img-element))

## Passing props to a component

아래의 코드에서 `Profile` 컴포넌트는 자식 컴포넌트인 `Avatar`에 어떤 프랍도 전달하지 않습니다.

```jsx
export default function Profile() {
  return <Avatar />;
}
```

다음과 같은 단계로 프랍을 전달할 수 있습니다.

### Step 1: Pass props to the child component

먼저 `Avatar` 몇 개의 프랍을 전달합니다.

```jsx
export default function Profile() {
  return (
    <Avatar person={{ name: "Lin Lanying", imageId: "1bX5QH6" }} size={100} />
  );
}
```

이제 `Avatar` 컴포넌트에서 프랍을 읽을 수 있습니다.

### Step 2: Read props inside the child component

함수 바로 뒤 `({` 와 `})`에서 프랍 목록의 `person`, `size`를 읽을 수 있습니다. 이것은 변수를 사용하는 것처럼 `Avatar` 내에서 사용할 수 있습니다.

```jsx
function Avatar({ person, size }) {
  // person and size are available here
}
```

`Avatar`에 렌더링 시 `person`, `size`를 사용하는 로직을 추가합니다.

이제 당신은 `Avatar`를 다양한 프랍을 통해 다르게 렌더 할 수 있습니다.

프랍은 부모와 자식 컴포넌트를 독립적으로 생각할 수 있게 해줍니다.
예를 들어, 당신은 `person`, `size`를 `profile` 컴포넌트 안에서 `Avatar`에서 어떻게 사용되는지 생각하지 않고 바꿔줄 수 있습니다.
비슷하게, 당신은 `Avatar`에서 `Profile`을 생각하지 않고 프랍의 사용 방법을 바꿀 수 있습니다.

프랍을 수정할 수 있는 노브라고 생각할 수 있습니다. 그들은 함수에서 인자와 같은 역할을 합니다. 사실상 프랍은 컴포넌트의 유일한 인자입니다. 리액트 컴포넌트 함수는 프랍이라는 객체를 인자로 받습니다.

```jsx
function Avatar(props) {
  let person = props.person;
  let size = props.size;
  // ...
}
```

위와 같은 방식으로 모든 `props`를 받아올 필요는 없습니다. 개별 프랍으로 구조 분해 할당할 수 있습니다.

## Specifying a default value for a prop

만약 프랍에 어떤 값도 정해지지 않아, 기본값을 주고 싶다면 `=`를 이용한 구조 분해 할당을 할 수 있습니다.

```jsx
function Avatar({ person, size = 100 }) {
  // ...
}
```

이제 `size`가 설정되지 않을 시 100으로 설정된 후 렌더 됩니다.

이 기본값은 `size`가 없을 시 혹은 `undefined`일 시에만 사용됩니다. `size={null}`또는 `size={0}`일 시에는 사용되지 않습니다.

## Forwarding props with the JSX spread syntax

때때로 프랍을 전달하는 것이 반복적일 때가 있습니다.

```jsx
function Profile({ person, size, isSepia, thickBorder }) {
  return (
    <div className="card">
      <Avatar
        person={person}
        size={size}
        isSepia={isSepia}
        thickBorder={thickBorder}
      />
    </div>
  );
}
```

이렇게 반복되는 코드는 나쁜 것이 아니며 가독성이 좋습니다. 하지만 간결함을 중요시할 수도 있습니다. 모든 프랍을 자식으로 넘겨주는 경우,
직접적으로 사용되지 않으므로 전개 연산자 문법을 사용할 수 있습니다.

```jsx
function Profile(props) {
  return (
    <div className="card">
      <Avatar {...props} />
    </div>
  );
}
```

이렇게 되면 목록을 나열하지 않고도 프랍을 전달할 수 있습니다.

전개 연산자의 사용은 제한됩니다. 만약 모든 컴포넌트에서 사용하면, 오류가 발생할 것입니다. 이것은 컴포넌트를 분리하고 JSX 자식으로 전달해야 한다는 것을 나타냅니다.

## Passing JSX as children

이것은 매우 일반적인 태그들입니다.

```html
<div>
  <img />
</div>
```

당신은 컴포넌트들을 중첩하고 싶을 수 있습니다.

```jsx
<Card>
  <Avatar />
</Card>
```

JSX 태그들 안에서 컴포넌트를 중첩하기 위해선, 컨텐트를 `children`이라는 프랍으로 받아야 합니다. 예를 들어, `Card` 컴포넌트는 `children` 프랍을 받아서 `<Avatar>`를 보여주고 div 안에서 렌더 합니다.

```jsx
import Avatar from "./Avatar.js";

function Card({ children }) {
  return <div className="card">{children}</div>;
}

export default function Profile() {
  return (
    <Card>
      <Avatar
        size={100}
        person={{
          name: "Katsuko Saruhashi",
          imageId: "YfeOqp2",
        }}
      />
    </Card>
  );
}
```

`<Card>` 안의 `<Avatar>`를 텍스트로 바꾸고 `<Card>` 컴포넌트가 어떻게 중첩된 모든 컨텐트를 감싸는지 보세요.
그 안에서 무엇이 렌더링 되는지 알 필요 없습니다. 이런 유연한 패턴을 자주 보게 될 것입니다.

`children` 프랍을 가진 컴포넌트를 부모 컴포넌트가 임의의 JSX로 채워지는 구멍을 가진 컴포넌트라고 생각할 수 있습니다.
`children` 프랍을 비주얼 래퍼에 많이 사용할 것입니다.

![](https://react.dev/images/docs/illustrations/i_children-prop.png)

## How props change over time

`Clock` 컴포넌트는 두 가지 프랍(`color`, `theme`)들을 부모로부터 전달받습니다.

```jsx
export default function Clock({ color, time }) {
  return <h1 style={{ color: color }}>{time}</h1>;
}
```

예제는 컴포넌트는 매번 다른 프랍을 받는다는 것을 보여줍니다.
프랍은 항상 똑같은 것이 아닙니다!
`time`프랍은 매초 변하고 `color` 프랍은 색을 선택했을 때 변합니다. 프랍은 컴포넌트의 데이터를 시작에만 보여주는 것이 아닌, 그 시점을 보여줍니다.

하지만 프랍은 불변성을 갖습니다. 컴포넌트가 프랍을 변경시키고 싶을 때(예. 유저가 새로운 데이터와 상호 작용할 때) 프랍은 부모 컴포넌트에게 다른 프랍 - 새로운 객체를 전달해도 되는지 물어봅니다.
오래된 프랍들은 치워지고, 자바스크립트 엔진이 그것들의 메모리를 정리합니다.

프랍을 변경하는 것을 시도하지 마세요. 유저의 입력에 반응하고 싶다면, 상태를 설정하세요.

## Recap

- JSX에 프랍을 전달하는 것은, HTML 어트리뷰트와 닮아있습니다.
- 프랍을 읽기 위해, 구조 분해를 사용합니다.
- 기본 프랍은 프랍이 없거나 `undefined`일 때 사용됩니다.
- 전개 연산자를 통해 프랍을 전달할 수 있습니다. 하지만 남용하지 마세요.
- 중첩된 JSX의 경우 `children` 프랍을 사용합니다.
- 프랍은 읽기 전용 스냅샷입니다. 모든 렌더링 시 새로운 프랍을 전달받습니다.
- 프랍은 변경이 불가능합니다. 상호작용을 원한다면 상태를 설정해야 합니다.

---

## What I Learned

JSX 태그안의 HTML 어트리뷰트들도 프랍을 받는다는 것을 매번 까먹는데, 프랍이라는 것에 대해 좀 더 명확한 지식을 얻게 되었다. 컴포넌트 간의 대화를 위해 사용된다는 부분이 흥미롭다.

<br>

출처

- https://react.dev/learn/passing-props-to-a-component
