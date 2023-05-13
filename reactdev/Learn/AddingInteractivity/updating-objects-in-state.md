- [Updating Objects in State](#updating-objects-in-state)
  - [What’s a mutation?](#whats-a-mutation)
  - [Treat state as read-only](#treat-state-as-read-only)
  - [DEEP DIVE - Local mutation is fine](#deep-dive---local-mutation-is-fine)
  - [Copying objects with the spread syntax](#copying-objects-with-the-spread-syntax)
  - [DEEP DIVE - Using a single event handler for multiple fields](#deep-dive---using-a-single-event-handler-for-multiple-fields)
  - [Updating a nested object](#updating-a-nested-object)
  - [DEEP DIVE - Objects are not really nested](#deep-dive---objects-are-not-really-nested)
  - [Write concise update logic with Immer](#write-concise-update-logic-with-immer)
  - [DEEP DIVE - How does Immer work?](#deep-dive---how-does-immer-work)
  - [DEEP DIVE - Why is mutating state not recommended in React?](#deep-dive---why-is-mutating-state-not-recommended-in-react)
  - [Recap](#recap)
  - [What I Learned](#what-i-learned)

# Updating Objects in State

상태는 자바스크립트의 객체를 포함한 어떤 값이든 가질 수 있습니다.
하지만 직접적으로 객체를 변경해서는 안됩니다.
대신 객체를 변경하기 위해선 새로운 객체를 만들 거나 기존의 것을 복사해 변경할 수 있습니다.

## What’s a mutation?

당신은 자바스크립트의 어떤 값이든 상태에 저장할 수 있습니다.

```jsx
const [x, setX] = useState(0);
```

숫자, 문자 그리고 불리언들을 다룬적이 있을 것입니다. 이런 자바스크립트의 값을 읽기전용의 변경할수 없는 불변하다고 합니다. 당신은 값을 대체해 리랜더를 일으킬 수 있습니다.

```jsx
setX(5);
```

x의 상태는 0에서 5로 변경되는 것이지, 0은 변경되지 않습니다.
내장된 숫자, 스트링 그리고 불리언과 같은 원시 값들을 바꾸는 것은 불가능 합니다.

이제 객체를 생각해봅시다.

```jsx
const [position, setPosition] = useState({ x: 0, y: 0 });
```

기술적으로, 객체의 값을 바꾸는 것은 가능합니다. 이것을 mutation이라고 합니다.

```jsx
position.x = 5;
```

하지만, 비록 객체가 가변하다고 해도 리액트에서는 숫자, 문자 그리고 불린과 같이 불변하게 다뤄야합니다. 그들을 변경하는 것이 아닌 항상 대체해야합니다.

## Treat state as read-only

다른 말로, 상태에 두는 객체는 읽기 전용으로 다뤄야한다는 것입니다.

이 예제의 상태는 현재 위치를 객체로 갖습니다.
빨간 점은 이전 공간에서 터치나 이동에 따라 움직입니다.
그러나 상태는 초기 위치에 존재합니다.

```jsx
import { useState } from "react";
export default function MovingDot() {
  const [position, setPosition] = useState({
    x: 0,
    y: 0,
  });
  return (
    <div
      onPointerMove={(e) => {
        position.x = e.clientX;
        position.y = e.clientY;
      }}
      style={{
        position: "relative",
        width: "100vw",
        height: "100vh",
      }}
    >
      <div
        style={{
          position: "absolute",
          backgroundColor: "red",
          borderRadius: "50%",
          transform: `translate(${position.x}px, ${position.y}px)`,
          left: -10,
          top: -10,
          width: 20,
          height: 20,
        }}
      />
    </div>
  );
}
```

이 코드가 문제의 원인입니다.

```jsx
onPointerMove={e => {
  position.x = e.clientX;
  position.y = e.clientY;
}}
```

이 코드는 이전 랜더에서 위치를 바꾸려합니다. 그러나 상태를 설정하는 함수 없이는, 리액트는 객체가 변경되었는 지 알 수 없습니다.
그래서 리액트는 아무런 반응이 없습니다.
이것은 이미 식사를 마치고 주문을 바꾸는 것과 같습니다.
상태를 mutating 시키는 것은 어떤 상황에서 될지도 모르지만, 우리는 추천하지 않습니다.
당신은 상태값을 읽기 전용으로 취급해야합니다.

리랜더를 일으키기 위해선, 새로운 객체를 만들고 세터 함수에 넘겨줘야합니다.

```jsx
onPointerMove={e => {
  setPosition({
    x: e.clientX,
    y: e.clientY
  });
}}
```

`setPosition`를 사용하며 리액트에 다음을 시킵니다.

- position을 새로운 객체로 교체
- 이 컴포넌트 다시 랜더

## DEEP DIVE - Local mutation is fine

다음과 같은 코드는 기존재하는 객체를 수정하기에 문제를 야기합니다.

```jsx
position.x = e.clientX;
position.y = e.clientY;
```

그러나 다음과 같은 코드는 새로 만들어진 객체를 mutating 하기에 완전하게 괜찮습니다.

```jsx
const nextPosition = {};
nextPosition.x = e.clientX;
nextPosition.y = e.clientY;
setPosition(nextPosition);
```

사실상, 이것은 다음과 같이 작성하는 것과 같습니다.

```jsx
setPosition({
  x: e.clientX,
  y: e.clientY,
});
```

Mutation은 상태에 이미 존재하는 객체를 변경할 때만 문제를 일으킵니다.
당신이 만든객체를 Mutating 하는 것은 어떤 코드도 참조하지 않기에 전혀 문제될 것이 없습니다.
이것을 변경하는 것은 그것에 의존한 어떤 것에 효과를 미치지 못합니다.
이것을 local mutation이라 부릅니다. 랜더 도중에도 local mutation을 할 수 있습니다.

## Copying objects with the spread syntax

이전의 예에서, position 객체는 현재 커서의 위치에 따라 새롭게 생성되었습니다.
하지만, 대부분 당신은 기존의 존재했던 데이터를 새로 만들어진 객체에 포함하길 원합니다.
예를 들어, 다른 필드는 그대로 두고 폼의 한 필드만 업데이트 하고 싶은 경우

이 인풋 필드들은 작동하지 않습니다. `onChange` 핸들러로 상태를 mutate하기 때문에

```jsx
import { useState } from "react";

export default function Form() {
  const [person, setPerson] = useState({
    firstName: "Barbara",
    lastName: "Hepworth",
    email: "bhepworth@sculpture.com",
  });

  function handleFirstNameChange(e) {
    person.firstName = e.target.value;
  }

  function handleLastNameChange(e) {
    person.lastName = e.target.value;
  }

  function handleEmailChange(e) {
    person.email = e.target.value;
  }

  return (
    <>
      <label>
        First name:
        <input value={person.firstName} onChange={handleFirstNameChange} />
      </label>
      <label>
        Last name:
        <input value={person.lastName} onChange={handleLastNameChange} />
      </label>
      <label>
        Email:
        <input value={person.email} onChange={handleEmailChange} />
      </label>
      <p>
        {person.firstName} {person.lastName} ({person.email})
      </p>
    </>
  );
}
```

이 코드는 과거의 랜더의 상태를 mutate 하고 있습니다.

```jsx
person.firstName = e.target.value;
```

당신이 원하는 동작은 새로운 객체를 생성해 `setPerson`에 넘겨주는 것입니다.
하지만, 당신은 하나의 필드만 변경하기 위해 기존재하는 데이터를 복사하길 원할수 있습니다.

```jsx
setPerson({
  firstName: e.target.value, // New first name from the input
  lastName: person.lastName,
  email: person.email,
});
```

전개 연산자 `...` 문법을 통해 각 프로퍼티를 복사할 필요가 없습니다.

```jsx
setPerson({
  ...person, // Copy the old fields
  firstName: e.target.value, // But override this one
});
```

이제 폼이 동작합니다!

각 필드별 상태 변수를 선언하지 않았다는 것을 알 수 있습니다.
폼이 커질 수록 데이터를 객체로 관리하는 것은 매우 편리합니다.

```jsx
import { useState } from "react";

export default function Form() {
  const [person, setPerson] = useState({
    firstName: "Barbara",
    lastName: "Hepworth",
    email: "bhepworth@sculpture.com",
  });

  function handleFirstNameChange(e) {
    setPerson({
      ...person,
      firstName: e.target.value,
    });
  }

  function handleLastNameChange(e) {
    setPerson({
      ...person,
      lastName: e.target.value,
    });
  }

  function handleEmailChange(e) {
    setPerson({
      ...person,
      email: e.target.value,
    });
  }

  return (
    <>
      <label>
        First name:
        <input value={person.firstName} onChange={handleFirstNameChange} />
      </label>
      <label>
        Last name:
        <input value={person.lastName} onChange={handleLastNameChange} />
      </label>
      <label>
        Email:
        <input value={person.email} onChange={handleEmailChange} />
      </label>
      <p>
        {person.firstName} {person.lastName} ({person.email})
      </p>
    </>
  );
}
```

`...`는 오직 한뎁스의 얕은 복사를 실시합니다. 이것은 빠르지만, 중첩된 객체가 있다면, 여러번 사용해야 합니다.

## DEEP DIVE - Using a single event handler for multiple fields

`[]`를 사용해 특정 프로퍼티의 동적 이름을 가져올 수 있습니다.
3개의 이벤트 핸들러가 아닌 하나의 이벤트 핸들러로 다른 것들을 처리할 수 있습니다.

```jsx
import { useState } from "react";

export default function Form() {
  const [person, setPerson] = useState({
    firstName: "Barbara",
    lastName: "Hepworth",
    email: "bhepworth@sculpture.com",
  });

  function handleChange(e) {
    setPerson({
      ...person,
      [e.target.name]: e.target.value,
    });
  }

  return (
    <>
      <label>
        First name:
        <input
          name="firstName"
          value={person.firstName}
          onChange={handleChange}
        />
      </label>
      <label>
        Last name:
        <input
          name="lastName"
          value={person.lastName}
          onChange={handleChange}
        />
      </label>
      <label>
        Email:
        <input name="email" value={person.email} onChange={handleChange} />
      </label>
      <p>
        {person.firstName} {person.lastName} ({person.email})
      </p>
    </>
  );
}
```

`e.target.name`은 `<input>` 돔 엘리먼트의 `name` 프로퍼티를 뜻합니다.

## Updating a nested object

다음과 같이 중첩된 객체가 있다고 생각해보세요

```jsx
const [person, setPerson] = useState({
  name: "Niki de Saint Phalle",
  artwork: {
    title: "Blue Nana",
    city: "Hamburg",
    image: "https://i.imgur.com/Sd1AgUOm.jpg",
  },
});
```

만약 `person.artwork.city`를 변경시키고 싶은 경우
mutation을 사용하면 쉽지만,

```jsx
person.artwork.city = "New Delhi";
```

리액트에서는, 상태를 불변하게 취급해야합니다. `city`를 변경하기 위해, 새로운 `artwork` 객체를 생성해야하며 `artwork`를 참조하는 새로운 `person` 객체를 생성합니다.

```jsx
const nextArtwork = { ...person.artwork, city: "New Delhi" };
const nextPerson = { ...person, artwork: nextArtwork };
setPerson(nextPerson);
```

혹은 하나의 함수에 작성합니다.

```jsx
setPerson({
  ...person, // Copy other fields
  artwork: {
    // but replace the artwork
    ...person.artwork, // with the same one
    city: "New Delhi", // but in New Delhi!
  },
});
```

이것은 장황해 보이지만, 많은 경우에서 제대로 동작합니다.

```jsx
import { useState } from "react";

export default function Form() {
  const [person, setPerson] = useState({
    name: "Niki de Saint Phalle",
    artwork: {
      title: "Blue Nana",
      city: "Hamburg",
      image: "https://i.imgur.com/Sd1AgUOm.jpg",
    },
  });

  function handleNameChange(e) {
    setPerson({
      ...person,
      name: e.target.value,
    });
  }

  function handleTitleChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        title: e.target.value,
      },
    });
  }

  function handleCityChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        city: e.target.value,
      },
    });
  }

  function handleImageChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        image: e.target.value,
      },
    });
  }

  return (
    <>
      <label>
        Name:
        <input value={person.name} onChange={handleNameChange} />
      </label>
      <label>
        Title:
        <input value={person.artwork.title} onChange={handleTitleChange} />
      </label>
      <label>
        City:
        <input value={person.artwork.city} onChange={handleCityChange} />
      </label>
      <label>
        Image:
        <input value={person.artwork.image} onChange={handleImageChange} />
      </label>
      <p>
        <i>{person.artwork.title}</i>
        {" by "}
        {person.name}
        <br />
        (located in {person.artwork.city})
      </p>
      <img src={person.artwork.image} alt={person.artwork.title} />
    </>
  );
}
```

## DEEP DIVE - Objects are not really nested

다음과 같은 객체는 중첩된 것처럼 보입니다.

```jsx
let obj = {
  name: "Niki de Saint Phalle",
  artwork: {
    title: "Blue Nana",
    city: "Hamburg",
    image: "https://i.imgur.com/Sd1AgUOm.jpg",
  },
};
```

하지만, 중첩은 객체가 어떻게 동작하는 지에 부정확한 표현법입니다.
코드가 실행될 때, 중첩된 객체라는 것은 없습니다. 정확히 다른 두 객체를 보는 것 입니다.

```jsx
let obj1 = {
  title: "Blue Nana",
  city: "Hamburg",
  image: "https://i.imgur.com/Sd1AgUOm.jpg",
};

let obj2 = {
  name: "Niki de Saint Phalle",
  artwork: obj1,
};
```

`obj1`는 `obj2`안에 있는 것이 아닙니다. 예를 들어 `obj3`은 `obj1`을 가리킵니다.

```jsx
let obj1 = {
  title: "Blue Nana",
  city: "Hamburg",
  image: "https://i.imgur.com/Sd1AgUOm.jpg",
};

let obj2 = {
  name: "Niki de Saint Phalle",
  artwork: obj1,
};

let obj3 = {
  name: "Copycat",
  artwork: obj1,
};
```

만약 당신이 `obj3.artwork.city`을 mutate 한다면 이것은 `obj2.artwork.city`와 `obj1.artwork.city`에 영향을 미칩니다.
그것은 `obj3.artwork`, `obj2.artwork` 그리고 `obj1`이 같은 객체이기 때문입니다.
이러한 이유 때문에 중첩되었다고 생각하기 어려운 이유입니다. 그들은 대신 서로 분리된 객체이면서, 서로 다른 프로퍼티를 포함하며 가리킵니다.

## Write concise update logic with Immer

만약 상태가 깊게 중첩 되었다면, 당신은 flat하게 하고 싶을 것입니다. 하지만 상태의 구조는 바꾸고 싶지 않을 경우, 중첩된 전개 연산자를 사용할 것입니다.
immer는 간편한 방법으로 mutate 문법을 제공하는 유명한 라이브러리입니다.
immer를 사용하면, 룰을 어기고 객체를 mutate 하는 것 처럼 보입니다.

```jsx
updatePerson((draft) => {
  draft.artwork.city = "Lagos";
});
```

일반적인 mutation과 달리 이것은 과거의 상태를 오버라이트 하지 않습니다.

## DEEP DIVE - How does Immer work?

Immer가 제공하는 `draft`는 [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)라 불리는 특별한 타입의 객체로 당신이 그것으로 무엇을 하는지 기록합니다.
이것이 mutate를 자유롭게할 수 있는 이유입니다.
Immer는 `draft`의 어떤 부분이 변경되었는 지 찾고, 변경을 포함한 완전히 새로운 객체를 생성합니다.

Immer 사용법

1. `npm install use-immer`를 의존성에 추가합니다.
2. `import { useState } from 'react'`를 `import { useImmer } from 'use-immer'`로 대체합니다.

Immer를 활용한 코드입니다.

```jsx
import { useImmer } from "use-immer";

export default function Form() {
  const [person, updatePerson] = useImmer({
    name: "Niki de Saint Phalle",
    artwork: {
      title: "Blue Nana",
      city: "Hamburg",
      image: "https://i.imgur.com/Sd1AgUOm.jpg",
    },
  });

  function handleNameChange(e) {
    updatePerson((draft) => {
      draft.name = e.target.value;
    });
  }

  function handleTitleChange(e) {
    updatePerson((draft) => {
      draft.artwork.title = e.target.value;
    });
  }

  function handleCityChange(e) {
    updatePerson((draft) => {
      draft.artwork.city = e.target.value;
    });
  }

  function handleImageChange(e) {
    updatePerson((draft) => {
      draft.artwork.image = e.target.value;
    });
  }

  return (
    <>
      <label>
        Name:
        <input value={person.name} onChange={handleNameChange} />
      </label>
      <label>
        Title:
        <input value={person.artwork.title} onChange={handleTitleChange} />
      </label>
      <label>
        City:
        <input value={person.artwork.city} onChange={handleCityChange} />
      </label>
      <label>
        Image:
        <input value={person.artwork.image} onChange={handleImageChange} />
      </label>
      <p>
        <i>{person.artwork.title}</i>
        {" by "}
        {person.name}
        <br />
        (located in {person.artwork.city})
      </p>
      <img src={person.artwork.image} alt={person.artwork.title} />
    </>
  );
}
```

이벤트 핸들러가 얼마나 간결해졌는지 알 수 있습니다.
하나의 컴포넌트 안에서 `useState`와 `useImmer`를 혼용해서 사용할 수 있습니다. Immer는 중첩된 객체나 객체를 복사하는 반복적인 코드가 있을시 이벤트 핸들러를 간결하게 해주는 좋은 방법입니다.

## DEEP DIVE - Why is mutating state not recommended in React?

몇 가지 이유가 있습니다.

- Debugging: 만약 콘솔 로그와 상태를 mutate 하지 않았을 시, 과거의 로그들은 최근 상태 변화에 영향을 받지 않습니다. 즉 명확하게 랜더시 상태 변화를 확인할 수 있습니다.
- Optimizations: 리액트의 최적화 방식은 이전 프랍이나 상태가 같을 시 넘기는 것에 의존합니다. 만약 상태를 mutate 하지 않는 다면, 빠르게 변화를 찾아냅니다.
  만약 `prevObj === obj`일 시 내부에서 어떤 변화도 일어나지 않았다는 것을 알 수 있습니다.
- New Features: 리액트의 새로운 기능은 상태를 스냅샷처럼 다루는 것에 기반합니다. 만약 이전 상태를 mutate 한다면, 이것은 새로운 기능을 사용하는 것을 방지할 것 입니다.
- Requirement Changes: 몇 애플리케이션의 기능들 취소/재실행은 변화의 기록을 보여주거나, 폼을 초기화 하는 것은 mutate 없이 진행하는 것이 쉽습니다.
  예전 상태를 메모리에 복사해 두고 재사용할 수 있기 때문입니다. mutate를 활용한다면, 이런 기능은 어려울 것입니다.
- Simpler Implementation: 리액트가 mutation에 의존하지 않기에, 객체에 특별한 것을 할 필요 없다. 프로퍼티를 하이잭하거나, 프록시들로 감싸거나 혹은 반응적인 해결책들이 하듯 초기화를 하거나 하는 것이 필요 없습니다. 이것이 리액트가 상태에 객체(크기가 커도)를 둘 수 있는 이유입니다.

종종, 리액트의 상태를 mutate 하겠지만, 새로운 리액트의 기능을 사용하기 위해 하지말길 추천합니다.

## Recap

- 리액트의 모든 상태를 불변하게 다뤄야합니다.
- mutate로 객체의 상태를 변경하는 것은 리랜더를 일으키지 않습니다. 그것은 이전 랜더의 스냅샷을 변경합니다.
- 객체를 mutate 하지말고 새로운 객체를 생성해 리랜더를 일으키세요.
- 전개 연산자를 이용해 객체의 복사본을 만들 수 있습니다.
- 전개 연산자는 얕은 복사로 오직 한뎁스만 복사합니다.
- 중첩된 객체를 변경하기 위해 변경을 위한 곳의 모든 복사본을 만들어야 합니다.
- 반복되는 코드를 줄이기 위해 Immer를 사용하세요

---

## What I Learned

객체 상태를 변경하기 위해서는 반드시 새로운 객체를 만들거나 복사본을 사용해야한다.
객체를 뮤테이트 해서 변경하는 것이 리랜더를 일으키지 않는 이유는 이전 랜더의 스냅샷을 변경했기 때문이다.

객체와 상태를 좀 더 공부해야겠다.

출처

- https://react.dev/learn/updating-objects-in-state#whats-a-mutation
