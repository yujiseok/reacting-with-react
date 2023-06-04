- [Scaling Up with Reducer and Context](#scaling-up-with-reducer-and-context)
  - [Combining a reducer with context](#combining-a-reducer-with-context)
    - [Step 1: Create the context](#step-1-create-the-context)
    - [Step 2: Put state and dispatch into context](#step-2-put-state-and-dispatch-into-context)
    - [Step 3: Use context anywhere in the tree](#step-3-use-context-anywhere-in-the-tree)
  - [Moving all wiring into a single file](#moving-all-wiring-into-a-single-file)
    - [Note](#note)
  - [Recap](#recap)
  - [What I Learned](#what-i-learned)

# Scaling Up with Reducer and Context

리듀서는 컴포넌트 내부의 상태 변경 로직을 통합합니다. 컨텍스트는 정보를 깊숙한 컴포넌트에 전달하도록 해줍니다.
당신은 리듀서와 컨텍스트를 합쳐 복잡한 화면의 상태를 관리할 수 있습니다.

## Combining a reducer with context

리듀서를 소개하는 예에서 상태는 리듀서에 의해 관리됩니다. 리듀서는 모든 상태 변경 로직을 포함하며, 파일 아래에 선언되어있습니다.

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
      <h1>Day off in Kyoto</h1>
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
  { id: 0, text: "Philosopher’s Path", done: true },
  { id: 1, text: "Visit the temple", done: false },
  { id: 2, text: "Drink matcha", done: false },
];
```

리듀서는 이벤트 핸들러를 짧고 간결하게 해줍니다.
하지만 앱이 커지면, 당신은 또 다른 문제에 직면합니다.
현재, 태스크 상태와 디스패치 함수는 오직 태스크앱 컴포넌트에서만 사용 가능합니다.
다른 컴포넌트들이 리스트를 읽고 변경 시키기위해, 당신은 프랍으로 전달해야합니다.

예를 들어, 태스크앱 태스크와 이벤트 핸들러를 태스크리스트에 전달합니다.

```jsx
<TaskList
  tasks={tasks}
  onChangeTask={handleChangeTask}
  onDeleteTask={handleDeleteTask}
/>
```

그 후 태스크리스트는 이벤트 핸들러를 태스크로 넘겨줍니다.

```jsx
<Task task={task} onChange={onChangeTask} onDelete={onDeleteTask} />
```

이런 작은 예에서, 이것은 잘 작동하지만, 만약 중간에 백개 이상의 컴포넌트가 있다면 꽤 좌절스러울 것입니다.

이것이 프랍으로 전달하는 것이 아닌, 태스크와 디스패치 함수를 컨텍스트에 넣고 싶은 이유입니다.
이 방식으로 태스크앱 하위 트리의 모든 컴포넌트는 프랍을 전달하는 프랍 드릴링 없이 태스크와 디스패치 액션에 접근할 수 있습니다.

리듀서와 컨텍스트를 같이 사용하는 법은 다음과 같습니다.

1. 컨텍스트 생성
2. 상태와 디스패치를 컨텍스트에 주입
3. 컨텍스트를 트리 어느곳에서 사용

### Step 1: Create the context

`useReducer` 훅은 현재 태스크와 디스패치를 반환 합니다.

```js
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

트리 아래로 전달하기 위해 두 개의 컨텍스트를 생성:

- `TasksContext`는 현재 태스크들의 리스트를 제공
- `TasksDispatchContext` 컴포넌트가 액션을 디스패치하는 함수 제공

다른 파일에서 내보내 다른 파일에서 불러와서 사용합니다.

```jsx
import { createContext } from "react";

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```

현재 당신은 `null`을 두 컨텍스드들에 기본값으로 설정하였습니다.
실제값은 태스크앱 컴포넌트에서 제공받습니다.

### Step 2: Put state and dispatch into context

이제 태스크앱 컴포넌트에서 두 컨텍스트를 불러 올 수 있습니다.
`useReducer()`의 반환 값인 태스크와 디스패치를 트리 하위에 제공합니다.

```jsx
import { TasksContext, TasksDispatchContext } from "./TasksContext.js";

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
  // ...
  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        ...
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
```

현재, 당신은 프랍과 컨텍스트 둘다 전달하였습니다.

```jsx
import { useReducer } from "react";
import AddTask from "./AddTask.js";
import TaskList from "./TaskList.js";
import { TasksContext, TasksDispatchContext } from "./TasksContext.js";

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
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        <h1>Day off in Kyoto</h1>
        <AddTask onAddTask={handleAddTask} />
        <TaskList
          tasks={tasks}
          onChangeTask={handleChangeTask}
          onDeleteTask={handleDeleteTask}
        />
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
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
  { id: 0, text: "Philosopher’s Path", done: true },
  { id: 1, text: "Visit the temple", done: false },
  { id: 2, text: "Drink matcha", done: false },
];
```

다음 단계에서, 프랍을 제거할 것입니다.

### Step 3: Use context anywhere in the tree

이제 당신은 태스크와 이벤트 핸들러를 트리 아래로 전달할 필요가 없습니다.

```jsx
<TasksContext.Provider value={tasks}>
  <TasksDispatchContext.Provider value={dispatch}>
    <h1>Day off in Kyoto</h1>
    <AddTask />
    <TaskList />
  </TasksDispatchContext.Provider>
</TasksContext.Provider>
```

대신, 태스크 리스트가 필요한 어떤 컴포넌트에서든 `TaskContext`에서 읽어올 수 있습니다.

```jsx
export default function TaskList() {
  const tasks = useContext(TasksContext);
  // ...
```

태스크 리스트를 변경하기 위해, 어떤 컴포넌트에서든 `dispatch` 함수를 컨텍스트에서 읽어올 수 있습니다.

```jsx
export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useContext(TasksDispatchContext);
  // ...
  return (
    // ...
    <button onClick={() => {
      setText('');
      dispatch({
        type: 'added',
        id: nextId++,
        text: text,
      });
    }}>Add</button>
    // ...
```

태스크앱 컴포넌트와 태스크리스트 컴포넌트는 어떠한 이벤트 핸들러도 태스크 컴포넌트에 전달하고 있지않습니다. 각 컴포넌트는 필요한 컨텍스트를 읽습니다.

```jsx
import { useState, useContext } from "react";
import { TasksContext, TasksDispatchContext } from "./TasksContext.js";

export default function TaskList() {
  const tasks = useContext(TasksContext);
  return (
    <ul>
      {tasks.map((task) => (
        <li key={task.id}>
          <Task task={task} />
        </li>
      ))}
    </ul>
  );
}
```

상태는 여전히 최상단인 태스크앱 컴포넌트에 존재하며, `useReducer`로 관리됩니다.
하지만 태스크와 디스패치는 트리 하위의 모든 컴포넌트에서 컨텍스트를 통해 접근할 수 있습니다.

## Moving all wiring into a single file

이렇게 할 필요는 없지만, 정리를 위해 리듀서와 컨텍스트를 한 파일로 옮길 수 있습니다.
현재 `TasksContext.js`는 오직 두 컨텍스트 선언만 포함합니다.

```jsx
import { createContext } from "react";

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```

이제 이 파일은 붐빌 것입니다. 리듀서를 옮긴 후, `TasksProvider`를 같은 파일에 선언합니다. 이 컴포넌트는 모든 것을 묶을 것입니다.

1. 이것은 리듀서로 상태를 관리
2. 이것은 하위 컴포넌트에 컨텍스트 제공
3. 이것은 children 프랍을 받아 JSX를 반환

```jsx
export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
```

이것은 태스크앱 컴포넌트의 복잡성을 줄입니다.

```jsx
import AddTask from "./AddTask.js";
import TaskList from "./TaskList.js";
import { TasksProvider } from "./TasksContext.js";

export default function TaskApp() {
  return (
    <TasksProvider>
      <h1>Day off in Kyoto</h1>
      <AddTask />
      <TaskList />
    </TasksProvider>
  );
}
```

또한 컨텍스트를 사용하는 함수도 내보낼 수 있습니다.

```jsx
export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}
```

컴포넌트에서 컨텍스트를 읽고 싶은 경우 위 함수를 사용합니다.

```jsx
const tasks = useTasks();
const dispatch = useTasksDispatch();
```

이것은 어떤 동작도 변화시키지 않지만, 컨텍스트를 좀 더 분리하고 로직을 추가할 수 있게 해줍니다.
이제 모든 컨텍스트와 리듀서는 `TasksContext.js`에 정리되었습니다.
이것은 컴포넌트를 명확하고 정돈해주며, 데이터를 어디서 가져오는 지 보단, 무엇을 보여줄지에 초점을 맞춰줍니다.

```jsx
import { useState } from "react";
import { useTasks, useTasksDispatch } from "./TasksContext.js";

export default function TaskList() {
  const tasks = useTasks();
  return (
    <ul>
      {tasks.map((task) => (
        <li key={task.id}>
          <Task task={task} />
        </li>
      ))}
    </ul>
  );
}

function Task({ task }) {
  const [isEditing, setIsEditing] = useState(false);
  const dispatch = useTasksDispatch();
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={(e) => {
            dispatch({
              type: "changed",
              task: {
                ...task,
                text: e.target.value,
              },
            });
          }}
        />
        <button onClick={() => setIsEditing(false)}>Save</button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>Edit</button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={(e) => {
          dispatch({
            type: "changed",
            task: {
              ...task,
              done: e.target.checked,
            },
          });
        }}
      />
      {taskContent}
      <button
        onClick={() => {
          dispatch({
            type: "deleted",
            id: task.id,
          });
        }}
      >
        Delete
      </button>
    </label>
  );
}
```

당신은 `TasksProvider`를 태스크를 어떻게 다룰지 아는 스크린이라고 생각할 수 있으며, `useTasks`로 그들을 읽고 `useTasksDispatch`로 트리 하위의 어떤 컴포넌트에서든 변경할 수 있습니다.

#### Note

`useTasks`나 `useTasksDispatch`은 커스텀훅이라 불립니다.
만약 커스텀훅이라 생각 된다면, `use`와 함께 이름을 붙이세요. 이것은 다른 훅을 `useContext`와 같은 다른 훅 안에서 사용할 수 있게 해줍니다.

당신의 앱이 커지면, 당신은 많은 쌍의 컨텍스트-리듀서가 있을 것입니다.
이것은 당신의 앱을 잘 정돈하는 강력한 방법이며 상태를 끌어올리는 것 없이 컴포넌트가 깊든지 데이터를 접근할 수 있게 해줍니다.

## Recap

- 당신은 리듀서와 컨텍스트를 합쳐 어떤 컴포넌트에서든 읽고 변경하기를 가능하게 합니다.
- 상태와 디스패치 함수를 하위 컴포넌트에 제공
  1. 두 컨텍스트 생성(상태와 디스패치 함수)
  2. 리듀서를 사용하는 컴포넌트에 컨텍스트 제공
  3. 필요한 컴포넌트에서 컨텍스트를 사용
- 당신은 하나의 파일로 정리할 수 있습니다.
  - 컨텍스트를 제공하는 `TaskProvider`와 같은 컴포넌트로 분리
  - `useTasks` 그리고 `useTasksDispatch`와 같은 커스텀훅으로 읽기
- 많은 쌍의 컨텍스트-리듀서를 앱에서 가질 수 있음

---

## What I Learned

예전에는 컨텍스트는 전역 상태 관리를 위한 것인 줄로만 알았다.
하지만 공부를 하다보니 상태 관리를 위해서는 리듀서와 함께 사용하면 합이 잘맞는 것을 알게 되었다.

리듀서와 컨텍스트를 같이 사용하는 것에도 익숙해져야겠다.

출처

- https://react.dev/learn/scaling-up-with-reducer-and-context
