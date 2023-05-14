- [Updating Arrays in State](#updating-arrays-in-state)
  - [Updating arrays without mutation](#updating-arrays-without-mutation)
    - [Pitfall](#pitfall)
    - [Adding to an array](#adding-to-an-array)
    - [Removing from an array](#removing-from-an-array)
    - [Transforming an array](#transforming-an-array)
    - [Replacing items in an array](#replacing-items-in-an-array)
    - [Inserting into an array](#inserting-into-an-array)
    - [Making other changes to an array](#making-other-changes-to-an-array)
  - [Updating objects inside arrays](#updating-objects-inside-arrays)
    - [Write concise update logic with Immer](#write-concise-update-logic-with-immer)
  - [Recap](#recap)
  - [What I Learned](#what-i-learned)

# Updating Arrays in State

배열은 자바스크립트에서 변경 가능하지만, 상태에서 불변하게 취급해야합니다..
객체와 마찬가지로 배열을 상태에 저장할 때, 새로운 배열을 만들거나 복사해야합니다.

## Updating arrays without mutation

자바스크립트에서 배열은 객체의 다른 모습입니다.
객체와 마찬가지로 배열 역시 읽기 전용 상태로 취급해야합니다.
이것은 배열의 아이템을 `arr[0] = 'bird'` 처럼 재할당을 하지말아야 하는 것과 `push()`와 `pop()` 같은 배열을 mutate하는 메서드를 사용하지 말하야하는 것을 뜻합니다.

대신, 배열을 변경하고 싶다면, 새로운 배열을 세터에 넘겨줍니다.
새로운 배열을 만들기 위해선, non-mutate 메서드인 `filter()`와 `map()`을 사용합니다. 그러면 새로운 배열을 상태로 설정할 수 있습니다.

리액트의 상태를 다룰 때, 왼쪽 컬럼은 지양하고 오른쪼 컬럼을 사용하도록 합니다.

<img width="472" alt="image" src="https://github.com/yujiseok/yujiseok.blog/assets/83855636/e175abbb-ab15-4c9e-b645-99e1974130e2">

또한 Immer를 사용할 수 있습니다.

### Pitfall

`slice`와 `splice`는 매우 비슷하게 생겼지만 전혀 다릅니다.

- `slice`는 배열의 복사본 또는 부분을 제공
- `splice`는 배열을 mutate(삽입, 삭제) 합니다.

리액트에서는 `slice`를 많이 사용하게 될 것입니다. 객체나 배열을 mutate 하면 안되기 때문입니다.

### Adding to an array

`push()`는 당신이 원치 않는데 배열을 변경합니다.

대신에, 기존의 아이템을 포함하고 새로운 아이템을 마지막에 추가하는 새로운 배열을 생성하는 합니다.
여러 방법이 있지만, 가장 쉬운 방식은 전개 연산자를 사용하는 것입니다.

```jsx
import { useState } from "react";

let nextId = 0;

export default function List() {
  const [name, setName] = useState("");
  const [artists, setArtists] = useState([]);

  return (
    <>
      <h1>Inspiring sculptors:</h1>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <button
        onClick={() => {
          setArtists([...artists, { id: nextId++, name: name }]);
        }}
      >
        Add
      </button>
      <ul>
        {artists.map((artist) => (
          <li key={artist.id}>{artist.name}</li>
        ))}
      </ul>
    </>
  );
}
```

배열의 전개 연산자는 기존 아이템 앞에 추가하는 것도 가능하게 해줍니다.

이 방법으로, `push()`와 `unshift()` 두 가지 메서드를 대체할 수 있습니다.

### Removing from an array

배열에서 아이템을 삭제하는 가장 쉬운 방식은 필터하는 것입니다.
다른 말로, 그 아이템을 포함하지 않는 새로운 배열을 생성하는 것입니다.
`filter` 메서드를 사용해 구현합니다.

```jsx
import { useState } from "react";

let initialArtists = [
  { id: 0, name: "Marta Colvin Andrade" },
  { id: 1, name: "Lamidi Olonade Fakeye" },
  { id: 2, name: "Louise Nevelson" },
];

export default function List() {
  const [artists, setArtists] = useState(initialArtists);

  return (
    <>
      <h1>Inspiring sculptors:</h1>
      <ul>
        {artists.map((artist) => (
          <li key={artist.id}>
            {artist.name}{" "}
            <button
              onClick={() => {
                setArtists(artists.filter((a) => a.id !== artist.id));
              }}
            >
              Delete
            </button>
          </li>
        ))}
      </ul>
    </>
  );
}
```

삭제 버튼을 눌러보고 이벤트 핸들러의 다음 코드를 읽어보세요.

```jsx
setArtists(artists.filter((a) => a.id !== artist.id));
```

`artists.filter(a => a.id !== artist.id)`는 `artists`를 `artist.id`가 다른 새로운 배열을 만들라는 뜻입니다. 다른 말로, 각각의 아티스트는 삭제를 누를시 필터되어 배열에서 사라지고 리랜더를 요청합니다.
`filter`는 기존 배열을 변경하지 않습니다.

### Transforming an array

만약 배열의 아이템들을 전부 또는 일부를 변경하고 싶을때, `map()`을 사용해 새로운 배열을 생성합니다. `map`에 전달된 함수는 각 아이템 별로(순서나 데이터에 따라) 어떤 동작을 취할지 정해줍니다.

이 예에서, 배열은 원과 사각형의 정보를 갖고 있습니다.
버튼을 눌렀을 때, 원들만 아래로 50픽셀 내려갑니다.
이것은 `map()`으로 새로운 배열을 만들어 수행합니다.

```jsx
import { useState } from "react";

let initialShapes = [
  { id: 0, type: "circle", x: 50, y: 100 },
  { id: 1, type: "square", x: 150, y: 100 },
  { id: 2, type: "circle", x: 250, y: 100 },
];

export default function ShapeEditor() {
  const [shapes, setShapes] = useState(initialShapes);

  function handleClick() {
    const nextShapes = shapes.map((shape) => {
      if (shape.type === "square") {
        // No change
        return shape;
      } else {
        // Return a new circle 50px below
        return {
          ...shape,
          y: shape.y + 50,
        };
      }
    });
    // Re-render with the new array
    setShapes(nextShapes);
  }

  return (
    <>
      <button onClick={handleClick}>Move circles down!</button>
      {shapes.map((shape) => (
        <div
          key={shape.id}
          style={{
            background: "purple",
            position: "absolute",
            left: shape.x,
            top: shape.y,
            borderRadius: shape.type === "circle" ? "50%" : "",
            width: 20,
            height: 20,
          }}
        />
      ))}
    </>
  );
}
```

### Replacing items in an array

여러개 또는 한개의 배열 아이템을 교체하는 것은 일반적입니다.
`arr[0] = 'bird`는 원본 배열을 수정하는 것이므로 `map`을 사용합니다.

아이템을 교체하려면, `map`으로 새로운 배열을 생성합니다.
`map` 호출 시, 아이템의 인덱스를 두 번째 인자로 받습니다.
이것을 활용해 원본 아이템을 반환 받을지 아닐지를 정합니다.

```jsx
import { useState } from "react";

let initialCounters = [0, 0, 0];

export default function CounterList() {
  const [counters, setCounters] = useState(initialCounters);

  function handleIncrementClick(index) {
    const nextCounters = counters.map((c, i) => {
      if (i === index) {
        // Increment the clicked counter
        return c + 1;
      } else {
        // The rest haven't changed
        return c;
      }
    });
    setCounters(nextCounters);
  }

  return (
    <ul>
      {counters.map((counter, i) => (
        <li key={i}>
          {counter}
          <button
            onClick={() => {
              handleIncrementClick(i);
            }}
          >
            +1
          </button>
        </li>
      ))}
    </ul>
  );
}
```

### Inserting into an array

가끔 특정 아이템을 시작과 끝 같은 특정 위치 삽입하고 싶을 경우가 있을 것입니다.
전개 연산자와 `slice()` 메서드를 이용해 할 수 있습니다.
`slice()`는 배열을 자르도록 해줍니다.
삽입을 위해선, 먼저 삽입 전 위치에 전개 연산자로 배열을 복사합니다. 그리고 새로운 아이템을 추가하고 나머지 배열을 둡니다.

아래의 예에서, 삽입 버튼은 항상 첫 번째 인덱스에 추가합니다.

```jsx
import { useState } from "react";

let nextId = 3;
const initialArtists = [
  { id: 0, name: "Marta Colvin Andrade" },
  { id: 1, name: "Lamidi Olonade Fakeye" },
  { id: 2, name: "Louise Nevelson" },
];

export default function List() {
  const [name, setName] = useState("");
  const [artists, setArtists] = useState(initialArtists);

  function handleClick() {
    const insertAt = 1; // Could be any index
    const nextArtists = [
      // Items before the insertion point:
      ...artists.slice(0, insertAt),
      // New item:
      { id: nextId++, name: name },
      // Items after the insertion point:
      ...artists.slice(insertAt),
    ];
    setArtists(nextArtists);
    setName("");
  }

  return (
    <>
      <h1>Inspiring sculptors:</h1>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <button onClick={handleClick}>Insert</button>
      <ul>
        {artists.map((artist) => (
          <li key={artist.id}>{artist.name}</li>
        ))}
      </ul>
    </>
  );
}
```

### Making other changes to an array

전개 연산자, `map()` 그리고 `filter()`와 같은 non-mutating 메서드를 가지고 할 수 없는 것들이 있습니다.
예를 들어 배열을 정렬하거나 역순으로 만드는 것입니다.
자바스크립트 `reverse()`와 `sort()`는 기존 배열을 수정하므로 직접 사용하면 안됩니다.

하지만, 복사를 먼저한 후 변경시킬 수 있습니다.

```jsx
import { useState } from "react";

let nextId = 3;
const initialList = [
  { id: 0, title: "Big Bellies" },
  { id: 1, title: "Lunar Landscape" },
  { id: 2, title: "Terracotta Army" },
];

export default function List() {
  const [list, setList] = useState(initialList);

  function handleClick() {
    const nextList = [...list];
    nextList.reverse();
    setList(nextList);
  }

  return (
    <>
      <button onClick={handleClick}>Reverse</button>
      <ul>
        {list.map((artwork) => (
          <li key={artwork.id}>{artwork.title}</li>
        ))}
      </ul>
    </>
  );
}
```

`[...list]` 전개 연산자를 사용해 원본 배열의 복사본을 만듭니다.
복사본이 있으니,`nextList.reverse()`, `nextList.sort()`와 같은 변경 메서드와 심지어 `nextList[0] = "something"`와 같이 개별 할당도 사용할 수 있습니다.

하지만, 복사본이라도, 존재하는 아이템을 직접적으로 변경할 수 없습니다.
그 이유는 복사는 얕은 복사(새로운 배열이 원본 배열과 동일한 아이템을 갖기에)이기 때문입니다. 만약 복사한 배열안의 객체를 변경한다면, 기존의 상태를 변경하는 것입니다.

```jsx
const nextList = [...list];
nextList[0].seen = true; // Problem: mutates list[0]
setList(nextList);
```

비록 `nextList`와 `list`는 다른 배열이지만 `nextList[0]`와 `list[0]`은 똑같은 객체를 가리킵니다.
`nextList[0].seen`를 변경하면, `next[0].seen` 역시 변경하는 것입니다.
이것은 피해야할 상태의 변경입니다. 각 아이템을 복사해 변경하는 것으로 해결할 수 있습니다.

## Updating objects inside arrays

사실상 객체들은 배열안에 존재하는 것이 아닙니다. 그들은 코드안에서 보입니다. 하지만 배열안 각각의 객체들은 배열이 가리키는 분리된 값입니다.
이것이 중첩된 `list[0]`을 변경할 때, 조심해야하는 이유입니다.
다른 사람의 작품 목록이 같은 아이템을 가리킬 수 있습니다.

중첩된 상태를 변경하기 위해, 당신은 변경하기 위한 곳을 탑레벨까지 복사본으로 만들어야합니다.

다음 예에서, 두 작품 목록은 동일한 상태를 같습니다. 그들은 독립적으로 보이지만, 변경을 통해 그들은 상태를 공유합니다.

```jsx
import { useState } from "react";

let nextId = 3;
const initialList = [
  { id: 0, title: "Big Bellies", seen: false },
  { id: 1, title: "Lunar Landscape", seen: false },
  { id: 2, title: "Terracotta Army", seen: true },
];

export default function BucketList() {
  const [myList, setMyList] = useState(initialList);
  const [yourList, setYourList] = useState(initialList);

  function handleToggleMyList(artworkId, nextSeen) {
    const myNextList = [...myList];
    const artwork = myNextList.find((a) => a.id === artworkId);
    artwork.seen = nextSeen;
    setMyList(myNextList);
  }

  function handleToggleYourList(artworkId, nextSeen) {
    const yourNextList = [...yourList];
    const artwork = yourNextList.find((a) => a.id === artworkId);
    artwork.seen = nextSeen;
    setYourList(yourNextList);
  }

  return (
    <>
      <h1>Art Bucket List</h1>
      <h2>My list of art to see:</h2>
      <ItemList artworks={myList} onToggle={handleToggleMyList} />
      <h2>Your list of art to see:</h2>
      <ItemList artworks={yourList} onToggle={handleToggleYourList} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map((artwork) => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={(e) => {
                onToggle(artwork.id, e.target.checked);
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
  );
}
```

문제가 되는 코드는 다음과 같습니다.

```jsx
const myNextList = [...myList];
const artwork = myNextList.find((a) => a.id === artworkId);
artwork.seen = nextSeen; // Problem: mutates an existing item
setMyList(myNextList);
```

비록 `myNextList` 배열은 새롭지만, 그 아이템들은 `myList`와 동일합니다.
`artwork.seen`를 변경하는 것은 원본을 변경하는 것입니다.
`yourList` 안의 작품 아이템 역시 버그를 유발합니다.
이런 버그들은 어렵지만, 상태를 변경하는 것을 피하면 사라집니다.

`map`을 사용해 이전 아이템을 변경 없이 대체합니다.

```jsx
setMyList(
  myList.map((artwork) => {
    if (artwork.id === artworkId) {
      // Create a *new* object with changes
      return { ...artwork, seen: nextSeen };
    } else {
      // No changes
      return artwork;
    }
  })
);
```

이런 접근은, 기존의 상태를 변경시키지 않고, 버그를 고칩니다.

```jsx
import { useState } from "react";

let nextId = 3;
const initialList = [
  { id: 0, title: "Big Bellies", seen: false },
  { id: 1, title: "Lunar Landscape", seen: false },
  { id: 2, title: "Terracotta Army", seen: true },
];

export default function BucketList() {
  const [myList, setMyList] = useState(initialList);
  const [yourList, setYourList] = useState(initialList);

  function handleToggleMyList(artworkId, nextSeen) {
    setMyList(
      myList.map((artwork) => {
        if (artwork.id === artworkId) {
          // Create a *new* object with changes
          return { ...artwork, seen: nextSeen };
        } else {
          // No changes
          return artwork;
        }
      })
    );
  }

  function handleToggleYourList(artworkId, nextSeen) {
    setYourList(
      yourList.map((artwork) => {
        if (artwork.id === artworkId) {
          // Create a *new* object with changes
          return { ...artwork, seen: nextSeen };
        } else {
          // No changes
          return artwork;
        }
      })
    );
  }

  return (
    <>
      <h1>Art Bucket List</h1>
      <h2>My list of art to see:</h2>
      <ItemList artworks={myList} onToggle={handleToggleMyList} />
      <h2>Your list of art to see:</h2>
      <ItemList artworks={yourList} onToggle={handleToggleYourList} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map((artwork) => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={(e) => {
                onToggle(artwork.id, e.target.checked);
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
  );
}
```

즉, 당신은 당신이 새로 만든 객체만을 변경해야합니다. 만약 새로운 작품을 삽입하고 싶다면, 변경할 수 있지만, 이미 상태가 존재한다면 복사본을 만들어야합니다.

### Write concise update logic with Immer

중첩된 배열을 변경시키지 않는 것은 제법 반복적입니다.

- 보통, 깊은 상태를 변경할 필요는 없습니다. 만약 당신의 객체가 매우 깊다면, 상태를 재설정할 것입니다.
- 만약 상태의 구조를 바꾸기 싫다면 Immer를 선호할 것입니다.

Immer를 사용한 코드입니다.

```jsx
import { useState } from "react";
import { useImmer } from "use-immer";

let nextId = 3;
const initialList = [
  { id: 0, title: "Big Bellies", seen: false },
  { id: 1, title: "Lunar Landscape", seen: false },
  { id: 2, title: "Terracotta Army", seen: true },
];

export default function BucketList() {
  const [myList, updateMyList] = useImmer(initialList);
  const [yourList, updateYourList] = useImmer(initialList);

  function handleToggleMyList(id, nextSeen) {
    updateMyList((draft) => {
      const artwork = draft.find((a) => a.id === id);
      artwork.seen = nextSeen;
    });
  }

  function handleToggleYourList(artworkId, nextSeen) {
    updateYourList((draft) => {
      const artwork = draft.find((a) => a.id === artworkId);
      artwork.seen = nextSeen;
    });
  }

  return (
    <>
      <h1>Art Bucket List</h1>
      <h2>My list of art to see:</h2>
      <ItemList artworks={myList} onToggle={handleToggleMyList} />
      <h2>Your list of art to see:</h2>
      <ItemList artworks={yourList} onToggle={handleToggleYourList} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map((artwork) => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={(e) => {
                onToggle(artwork.id, e.target.checked);
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
  );
}
```

Immer를 사용하면 `artwork.seen = nextSeen`와 같은 변형이 가능합니다.

```jsx
updateMyTodos((draft) => {
  const artwork = draft.find((a) => a.id === artworkId);
  artwork.seen = nextSeen;
});
```

이것은 Immer에서 제공 받은 `draft`라는 특별한 객체를 변형하는 것이지, 원본 상태를 변형하는 것이 아닙니다. `draft`에 `push()`와 `pop()`과 같은 변형 메서드를 사용할 수 있습니다.

Immer는 `draft`를 사용해 변경한 것을 다음 상태로 구성합니다.
이것이 이벤트 핸들러를 간결하게 해줍니다.

## Recap

- 배열을 상태에 둘 수 있지만, 변경해서는 안됩니다.
- 배열을 변경하는 것 대신, 새로운 배열을 만들어 상태를 변경합니다.
- 새로운 아이템을 포함한 새로운 배열을 전개 연산자를 통해 만들 수 있습니다.
- `filter()`와 `map()`를 사용해 필터되거나 이동된 새로운 배열을 만들 수 있습니다.
- Immer를 사용해 간결한 코드를 작성할 수 있습니다.

---

## What I Learned

Immer를 사용해서 mutate 하는 것을 시도해 봐야겠다.
mutate라는 개념이 정말 헷갈렸는데, 좀 더 명확해졌다.

출처

- https://react.dev/learn/updating-arrays-in-state
