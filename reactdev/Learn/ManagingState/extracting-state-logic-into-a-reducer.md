- [Extracting State Logic into a Reducer](#extracting-state-logic-into-a-reducer)
  - [Consolidate state logic with a reducer](#consolidate-state-logic-with-a-reducer)
    - [Step 1: Move from setting state to dispatching actions](#step-1-move-from-setting-state-to-dispatching-actions)
    - [Note](#note)
    - [Step 2: Write a reducer function](#step-2-write-a-reducer-function)
      - [Note](#note-1)
      - [DEEP DIVE - Why are reducers called this way?](#deep-dive---why-are-reducers-called-this-way)
    - [Step 3: Use the reducer from your component](#step-3-use-the-reducer-from-your-component)
  - [Comparing useState and useReducer](#comparing-usestate-and-usereducer)
  - [Writing reducers well](#writing-reducers-well)
  - [Writing concise reducers with Immer](#writing-concise-reducers-with-immer)
  - [Recap](#recap)
  - [What I Learned](#what-i-learned)

# Extracting State Logic into a Reducer

컴포넌트의 많은 상태와 이벤트 핸들러들은 압도될 수 있습니다.
이런 경우에, 당신은 리듀서라는 함수를 통해 상태를 변경시키는 로직을 컴포넌트 외부에 통합할 수 있습니다.

## Consolidate state logic with a reducer

컴포넌트가 복잡해짐에따라, 상태가 변경되는 것을 한 눈에 알아보긴 힘듭니다.
예를 들어, 태스크앱 컴포넌트는 태스크 배열을 추가, 삭제, 수정 등의 이벤트 핸들러를 사용합니다.

```jsx
import { useState } from "react";
import AddTask from "./AddTask.js";
import TaskList from "./TaskList.js";

export default function TaskApp() {
  const [tasks, setTasks] = useState(initialTasks);

  function handleAddTask(text) {
    setTasks([
      ...tasks,
      {
        id: nextId++,
        text: text,
        done: false,
      },
    ]);
  }

  function handleChangeTask(task) {
    setTasks(
      tasks.map((t) => {
        if (t.id === task.id) {
          return task;
        } else {
          return t;
        }
      })
    );
  }

  function handleDeleteTask(taskId) {
    setTasks(tasks.filter((t) => t.id !== taskId));
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: "Visit Kafka Museum", done: true },
  { id: 1, text: "Watch a puppet show", done: false },
  { id: 2, text: "Lennon Wall pic", done: false },
];
```

모든 이벤트 핸들러들은 `setTasks`를 호출해 상태를 변경합니다.
이 컴포넌트가 커지면, 이런 상태 로직들이 도처에 뿌려집니다.
이런 복잡성을 줄이고 로직을 한 눈에 파악하기 위해, 당신은 상태 로직을 리듀서라 불리는 하나의 함수로 컴포넌트에서 분리할 수 있습니다.

리듀서는 상태를 다루는 다른 방식입니다. `useState`를 `useReducer`로 3 단계에 거쳐 통합할 수 있습니다.

1. 상태를 설정하는 것을 액션 디스패칭으로
2. 리듀서 함수 작성
3. 컴포넌트 내부에서 리듀서 사용

### Step 1: Move from setting state to dispatching actions

당신의 이벤트 핸들러는 현재 무엇을 할지를 상태를 설정해 특정합니다.

모든 상태를 설정하는 로직을 제거하세요. 당신이 남겨야할 것은 세 개의 이벤트 핸들러입니다.

- `handleAddTask(text)`는 유저가 추가 버튼을 눌렀을 때 호출
- `handleChangeTask(task)`는 유저가 태스크를 토글하거나 저장을 눌렀을 때 호출
- `handleDeleteTask(taskId)`는 유저가 삭제를 눌렀을 때 호출

리듀서로 상태를 관리하는 것은 직접적으로 상태를 설정하는 것과 다릅니다.
리액트에게 무엇을 할지를 말하는 것이 아닌, 유저가 무엇을 할지를 이벤트 핸들러를 통한 액션을 디스패치해 특정합니다.즉, 이벤트 핸들러로 태스크들을 설정하는 것이 아닌, 추가/수정/삭제 액션을 디스패치 하는 것입니다.
이것은 유저의 의도를 명확히 합니다.

```jsx
function handleAddTask(text) {
  dispatch({
    type: "added",
    id: nextId++,
    text: text,
  });
}

function handleChangeTask(task) {
  dispatch({
    type: "changed",
    task: task,
  });
}

function handleDeleteTask(taskId) {
  dispatch({
    type: "deleted",
    id: taskId,
  });
}
```

디스패치에 넘겨준 객체는 액션이라 불립니다.

```jsx
function handleDeleteTask(taskId) {
  dispatch(
    // "action" object:
    {
      type: "deleted",
      id: taskId,
    }
  );
}
```

이것은 평범한 자바스크립트 객체입니다. 당신이 무엇을 넣을지 결정합니다. 하지만 일반적으로 무엇이 일어날지에 대한 최소한의 정보를 담습니다.

### Note

액션 객체는 어떤 모양이든 될 수 있습니다.

컨벤션으로, 문자열로 무엇이 일어날지에 대한 `type`을 전달하고, 추가 정보를 다른 필드에 전달합니다. `type`은 컴포넌트 별로 다르며, 이 예에서 `"added"` 또는 `"added_task"` 둘다 괜찮습니다.
무슨 일이 일어날지에 대한 이름을 고르세요.

```jsx
dispatch({
  // specific to component
  type: "what_happened",
  // other fields go here
});
```

### Step 2: Write a reducer function

리듀서 함수는 당신의 상태 로직을 둘 장소입니다.
이것은 현재 상태와 액션 객체 두 인자를 받고 다음 상태를 반환합니다.

```jsx
function yourReducer(state, action) {
  // return next state for React to set
}
```

리액트는 리듀서로 반환 받은 상태를 설정할 것입니다.

이벤트 핸들러에서 상태 로직을 리듀서로 옮기는 예는 다음과 같습니다.

1. 현재 상태(`tasks`)를 첫 번째 인자로 선언
2. 두 번째 인자로 액션 객체 선언
3. 리듀서에서 다음 상태를 반환

리듀서로 이주된 상태 로직입니다.

```js
function tasksReducer(tasks, action) {
  if (action.type === "added") {
    return [
      ...tasks,
      {
        id: action.id,
        text: action.text,
        done: false,
      },
    ];
  } else if (action.type === "changed") {
    return tasks.map((t) => {
      if (t.id === action.task.id) {
        return action.task;
      } else {
        return t;
      }
    });
  } else if (action.type === "deleted") {
    return tasks.filter((t) => t.id !== action.id);
  } else {
    throw Error("Unknown action: " + action.type);
  }
}
```

리듀서 함수가 상태를 인자로 받기에, 컴포넌트 외부에 선언할 수 있습니다.
이것은 인덴테이션 레벨을 줄이고 코드를 읽기 쉽게 해줍니다.

#### Note

위의 코드는 리듀서 안에서 if/else 문을 사용하지만, 컨벤션으로는 스위치문을 사용합니다.
결과는 같지만, 스위치문은 한눈에 보기 쉽습니다.

```js
function tasksReducer(tasks, action) {
  switch (action.type) {
    case "added": {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case "changed": {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case "deleted": {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error("Unknown action: " + action.type);
    }
  }
}
```

우리는 각 `case` 블락을 `{}`로 감싸는 것을 추천하는데, 중괄호 안의 선언된 변수는 다른 `case`와 충돌을 일으키지 않기 때문입니다.
또한,`case`는 반드시 `return`으로 끝나야 합니다.
만약 반환을 까먹으면 다음 `case`로 넘어가게 되고 실수를 유발합니다.

만약 스위치문이 익숙하지 않으면 if/else 문을 사용해도 문제 없습니다.

#### DEEP DIVE - Why are reducers called this way?

리듀서는 코드를 누산할 수 있기에, 사실상 `reduce()` 배열에서 사용하는 함수에서 따왔습니다.

`reduce()` 함수는 배열의 각 요소를 축적할 수 있게 해줍니다.

```js
const arr = [1, 2, 3, 4, 5];
const sum = arr.reduce((result, number) => result + number); // 1 + 2 + 3 + 4 + 5
```

`reduce`에 넘기는 함수는 리듀서라 알려져있습니다.
이것은 결과와 현재 값을 받은 후, 다음 결과를 반환합니다.
리액트 리듀서는 동일한 아이디어의 예로: 그들은 상태와 액션을 받아서 다음 상태를 반환합니다.
이런 방식으로, 그들은 액션을 상태로 축적합니다.

`reduce()` 메서드를 초기 상태를 배열로 설정해 사용할 수도 있습니다.

```js
let actions = [
  { type: "added", id: 1, text: "Visit Kafka Museum" },
  { type: "added", id: 2, text: "Watch a puppet show" },
  { type: "deleted", id: 1 },
  { type: "added", id: 3, text: "Lennon Wall pic" },
];

let finalState = actions.reduce(tasksReducer, initialState);

const output = document.getElementById("output");
output.textContent = JSON.stringify(finalState, null, 2);
```

이렇게 할 필요는 없지만, 이것은 리액트가 하는 것과 비슷합니다.

### Step 3: Use the reducer from your component

마지막으로, `tasksReducer`를 컴포넌트 안에서 훅 업 해야합니다.
`useReducer` 훅을 리액트에서 가져옵니다.

```jsx
import { useReducer } from "react";
```

그 후, `useState`를 대체합니다.

```jsx
const [tasks, setTasks] = useState(initialTasks);
```

`useReducer`와 함께

```jsx
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

`useReducer` 훅은 `useState`와 비슷합니다.- 초기값을 넘긴 후 상태를 반환하고 설정하는 방식이
하지만 차이점이 있습니다.

`useReducer` 훅은 두 인자를 받습니다:

1. 리듀서 함수
2. 초기 상태

그리고 반환합니다.

1. 상태 값
2. 디스패치 함수

이제 완료 되었습니다.

```jsx
import { useReducer } from "react";
import AddTask from "./AddTask.js";
import TaskList from "./TaskList.js";

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({
      type: "added",
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: "changed",
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: "deleted",
      id: taskId,
    });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case "added": {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case "changed": {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case "deleted": {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error("Unknown action: " + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: "Visit Kafka Museum", done: true },
  { id: 1, text: "Watch a puppet show", done: false },
  { id: 2, text: "Lennon Wall pic", done: false },
];
```

만약 원한다면, 다른 파일로 리듀서를 분리할 수 있습니다.

컴포넌트 로직은 이렇게 관심사를 분리 시키면 가독성이 올라갑니다.
이제 이벤트 핸들러는 오직 디스패치로 무엇이 발생할지만 특정하고, 리듀서 함수가 상태를 어떻게 변경할지를 결정합니다.

## Comparing useState and useReducer

- Code size: 일반적으로 `useState`는 적은 코드를 사용합니다. `useReducer`를 사용하면, 리듀서 함수와 액션 디스패치를 작성해야 합니다. 하지만 `useReducer`는 많은 이벤트 핸들러가 상태를 비슷한 방식으로 변경할 시 코드양을 줄여줍니다.

- Readability: `useState`는 상태 변경이 간단하면 가독성이 높습니다. 복잡해질수록, 컴포넌트의 코드를 팽창시키며 읽기 어렵게 만듭니다. 이런 경우, `useReducer`는 명확하게 이벤트 핸들러에서 무엇이 일어나는 지를 변경 로직으로부터 분리해줍니다.

- Debugging: `useState`를 사용하면, 어떤 상태가 잘못 설정 됐는지 알기 어렵습니다. `useReducer`는, 모든 상태 변경이 일어날때 콘솔로그를 리듀서에 추가할 수 있습니다. 만약 액션이 맞다면, 리듀서 로직의 오류를 알 수 있습니다. 하지만 `useState`는 더 많은 코드가 필요합니다.

- Testing: 리듀서는 순수 함수로 컴포넌트에 의존하지 않습니다. 즉 독립적으로 테스트할 수 있습니다. 일반적으로 현실적인 환경에서 테스트 하는 것이 좋지만, 복잡한 상태 로직의 경우 리듀서가 특정 초기 상태 및 작업에 대해 특정 상태를 반환한다고 주장하는 것이 유용할 수 있습니다.

- Personal preference: 어떤 사람은 리듀서를 좋아하고, 어떤 사람은 아닐 수 있습니다. 당신은 `useState`와 `useReducer`를 변환할 수 있습니다.

우리는 몇 컴포넌트 내에서 맞지 않는 상태 변경이 있을 시, 그리고 코드를 좀 더 구조화 하고 싶은 경우 리듀서를 추천합니다. 리듀서를 모든 곳에 사용할 필요는 없으며, 믹스매치에 자유입니다. 같은 컴포넌트 안에서 `useState`와 `useReducer`를 동시에 사용할 수 있습니다.

## Writing reducers well

리듀서를 작성할 때 두 가지를 기억하세요.

- Reducers must be pure. 업데이터 함수처럼, 리듀서는 랜더링 중에 작동합니다. 이것은 리듀서가 반드시 순수해야함을 뜻합니다. 타임아웃된 요청을 보내지 말아야하며, 어떤 사이드 이펙트도 처리하면 안됩니다. 그들은 객체와 배열을 불변하게 변경해야합니다.

- Each action describes a single user interaction, even if that leads to multiple changes in the data. 예를 들어 폼을 초기화 할때 다섯 개의 `set_field` 액션을 디스패치 하는 것 보다, 하나의 `reset_form`을 디스패치하는 게 더 말이 됩니다. 리듀서의 모든 액션에 로그를 찍어보면, 어떻게 재구조화할 지 알 수 있게됩니다. 이것은 디버깅에 도움이 됩니다.

## Writing concise reducers with Immer

객체와 배열 상태를 변경하는 것과 같이 당신은 Immer를 사용해 리듀서를 더 간결하게 작성할 수 있습니다. `useImmerReducer`는 상태를 뮤테이트 하게 해줍니다.

리듀서는 반드시 순수해야 하므로, 그들은 상태를 뮤테이트할 수 없습니다.
하지만 Immer는 객체를 뮤테이트해도 안전한 특별한 `draft` 제공합니다.
이것이 `useImmerReducer`로 관리되는 리듀서가 첫 인자를 뮤테이트할 수 있고 상태를 반환하지 않아도 되는 이유입니다.

## Recap

- `useState`를 `useReducer`로 바꾸는 법
  1. 이벤트 핸들러로 부터 액션을 디스패치
  2. 주어진 상태와 액션으로 다음 상태를 반환하는 리듀서 함수 정의
  3. `useState`를 `useReducer`로 대체
- 리듀서는 많은 코드가 요구 되지만, 디버깅과 테스를 쉽게 해줍니다.
- 리듀서는 순수 해야합니다.
- 각 액션은 하나의 유저 상호작용을 설명해야합니다.
- Immer를 사용해 리듀서를 뮤테이트

---

## What I Learned

리액트를 처음 공부할 때, 리듀서라는 개념이 정말 너무 모호했던 기억이 있다. 지금도 100% 이해한것은 아니지만, 그래도 제법 가까워진 느낌이 들고 문서가 무슨 말을 하는 지 알것 같다.

상태를 변경하는 로직들을 컴포넌트 밖에 정의할 수 있어서 가독성이 높아지는 점이 정말 좋은 것 같다. 관심사의 분리..!

출처

- https://react.dev/learn/extracting-state-logic-into-a-reducer
