- [Reacting to Input with State](#reacting-to-input-with-state)
  - [How declarative UI compares to imperative](#how-declarative-ui-compares-to-imperative)
  - [Thinking about UI declaratively](#thinking-about-ui-declaratively)
    - [Step 1: Identify your component’s different visual states](#step-1-identify-your-components-different-visual-states)
  - [DEEP DIVE - Displaying many visual states at once](#deep-dive---displaying-many-visual-states-at-once)
    - [Step 2: Determine what triggers those state changes](#step-2-determine-what-triggers-those-state-changes)
    - [Step 3: Represent the state in memory with useState](#step-3-represent-the-state-in-memory-with-usestate)
    - [Step 4: Remove any non-essential state variables](#step-4-remove-any-non-essential-state-variables)
  - [DEEP DIVE - Eliminating “impossible” states with a reducer](#deep-dive---eliminating-impossible-states-with-a-reducer)
    - [Step 5: Connect the event handlers to set state](#step-5-connect-the-event-handlers-to-set-state)
  - [Recap](#recap)
  - [What I Learned](#what-i-learned)

# Reacting to Input with State

리액트는 UI를 선언적으로 조작할 수 있게 해줍니다.
개별적인 UI를 직접 조작하는 대신, 컴포넌트가 가질 수 있는 다양한 상태를 설명합니다. 또, 유저의 인풋의 응답에따라 변경합니다.
이것은 디자이너가 UI를 생각하는 것과 비슷합니다.

## How declarative UI compares to imperative

UI의 상호작용을 디자인할 때, 당신은 유저의 액션에 따른 UI 변화를 생각할 것입니다. 유저가 제출하는 폼을 생각해봅시다.

- 폼에 무언가 작성 시 폼의 제출 버튼이 활성화
- 제출을 클릭 시, 폼과 버튼 비활성화 및 스피너 등장
- 네트워크 요청 성공 시, 폼은 사라지며 감사 메세지 등장
- 네트워크 요청 실패 시, 에러 메세지 등장 및 폼 활성화

명령형 프로그밍에서, 위의 내용은 상호작용을 구현하기 위해 직접적으로 일치합니다.
어떤일이 일어나는 지에 따라 UI를 조작하기 위해 정확한 명령이 필요합니다.
이것을 다르게 생각하는 방법이 있습니다.
당신이 차에 타 어디에 갈지 매번 말하는 것과 같습니다.

![Alt text](https://react.dev/images/docs/illustrations/i_imperative-ui-programming.png)

그들은 당신이 어디에 가고싶은지 모르고 명령을 따를 뿐입니다.(만약 잘못된 경로라면, 잘못된 장소에 도착합니다.)
이것을 모든 엘리먼트에 명령하였기에, 컴퓨터에게 UI를 어떻게 변경시킬지 말해주는 명령형이라고 합니다.

만약 리액트 없이, 명령형으로 UI를 그린다면 다음과 같습니다.

```js
async function handleFormSubmit(e) {
  e.preventDefault();
  disable(textarea);
  disable(button);
  show(loadingMessage);
  hide(errorMessage);
  try {
    await submitForm(textarea.value);
    show(successMessage);
    hide(form);
  } catch (err) {
    show(errorMessage);
    errorMessage.textContent = err.message;
  } finally {
    hide(loadingMessage);
    enable(textarea);
    enable(button);
  }
}

function handleTextareaChange() {
  if (textarea.value.length === 0) {
    disable(button);
  } else {
    enable(button);
  }
}

function hide(el) {
  el.style.display = "none";
}

function show(el) {
  el.style.display = "";
}

function enable(el) {
  el.disabled = false;
}

function disable(el) {
  el.disabled = true;
}

function submitForm(answer) {
  // Pretend it's hitting the network.
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (answer.toLowerCase() == "istanbul") {
        resolve();
      } else {
        reject(new Error("Good guess but a wrong answer. Try again!"));
      }
    }, 1500);
  });
}

let form = document.getElementById("form");
let textarea = document.getElementById("textarea");
let button = document.getElementById("button");
let loadingMessage = document.getElementById("loading");
let errorMessage = document.getElementById("error");
let successMessage = document.getElementById("success");
form.onsubmit = handleFormSubmit;
textarea.oninput = handleTextareaChange;
```

UI를 명령형으로 조작하는 것은 독립적인 예에서 효율적입니다. 하지만, 복잡할수록 더 어려워집니다. 페이지에 이와 같은 다른 폼을 변경한다고 생각헤보세요.
새로운 엘리먼트나 상호작용을 추가할 때, 기존의 코드에서 버그가 발생하지 않을지 살펴봐야 합니다.

리액트는 이런 문제를 해결하기 위해 만들어졌습니다.

리액트에서 당신은 직접적으로 UI를 조작할 필요없습니다.
대신, 무엇을 보여줄지 선언합니다. 그러면 리액트는 UI를 어떻게 변경할지 결정합니다.
택시에 타서 어디서 턴할지를 말하는 것 대신 어디를 가고 싶은 목적지를 말한다고 생각하세요.
기사의 일이 당신을 그곳에 데려다주는 것이며 당신이 생각하지도 못한 지름길을 알 것입니다.

![Alt text](https://react.dev/images/docs/illustrations/i_declarative-ui-programming.png)

## Thinking about UI declaratively

위에서 폼을 명령형으로 구현하는 것을 봤습니다.
리액트가 어떻게 생각하는 지 이해하기 위해, UI를 다음과 같이 재구현할 수 있습니다.

1. 컴포넌트의 다른 시각적 상태를 구별
2. 무엇이 이 상태를 변경할지 결정
3. `useState`를 사용해 메모리에 저장
4. 불필요한 상태 변수 제거
5. 상태를 설정하기 위해 이벤트 핸들러 연결

### Step 1: Identify your component’s different visual states

컴퓨터 공학에서, 당신은 상태 기계를 들어봤을 것입니다. 만약 디자이너와 일을 한다면, 다른 시각정 상태에 따른 목업을 보았을 것입니다.
리액트는 컴퓨터 공학과 디자인의 상호작용 위에 세워졌습니다. 그러니 두 아이디어가 영감의 원천입니다.

첫째로, 유저가 볼 수 있는 UI에 대한 모든 다른 상태를 시각화할 필요가 있습니다.

- Empty: 폼의 제출 버튼이 비활성화
- Typing: 폼의 제출 버튼 활성화
- Submitting: 폼이 비활성화 되고 스피너 등장
- Success: 폼 대신 감사합니다 문구 등장
- Error: Typing 상태와 동일, 에러 메세지

디자이너처럼, 로직을 추가하기 전에 다른 상태에 따른 목업을 만듭니다.
예를 들어 이것은 보여지는 부분에 대한 목업입니다.
이 목업은 기본값이 `"empty"`인 `status`라는 프랍에 의해 제어됩니다.

```jsx
export default function Form({ status = "empty" }) {
  if (status === "success") {
    return <h1>That's right!</h1>;
  }
  return (
    <>
      <h2>City quiz</h2>
      <p>
        In which city is there a billboard that turns air into drinkable water?
      </p>
      <form>
        <textarea />
        <br />
        <button>Submit</button>
      </form>
    </>
  );
}
```

당신의 프랍을 어떤 것으로든 부를 수 있습니다. ` status = 'empty'`를 `status = 'success'`로 변경시켜 성공 메세지가 나오도록 해보세요.
모킹은 로직을 구현하기 전에 UI를 반복할 수 있게 해줍니다.
`status`에 의해 제어되는 컴포넌트에 살을 붙일 수 있습니다.

```jsx
export default function Form({
  // Try 'submitting', 'error', 'success':
  status = "error",
}) {
  if (status === "success") {
    return <h1>That's right!</h1>;
  }
  return (
    <>
      <h2>City quiz</h2>
      <p>
        In which city is there a billboard that turns air into drinkable water?
      </p>
      <form>
        <textarea disabled={status === "submitting"} />
        <br />
        <button disabled={status === "empty" || status === "submitting"}>
          Submit
        </button>
        {status === "error" && (
          <p className="Error">Good guess but a wrong answer. Try again!</p>
        )}
      </form>
    </>
  );
}
```

## DEEP DIVE - Displaying many visual states at once

만약 여러 시각적 상태가 있다면, 하나의 페이지에 보여주는 것이 편합니다.

```jsx
import Form from "./Form.js";

let statuses = ["empty", "typing", "submitting", "success", "error"];

export default function App() {
  return (
    <>
      {statuses.map((status) => (
        <section key={status}>
          <h4>Form ({status}):</h4>
          <Form status={status} />
        </section>
      ))}
    </>
  );
}
```

### Step 2: Determine what triggers those state changes

당신은 두 가지 입력을 통해 상태 변화를 일으킬 수 있습니다.

- Human inputs, 버튼 클릭, 필드 입력, 링크 클릭
- Computer inputs, 네트워크 응답, 타임아웃, 이미지 로딩

두 가지 케이스에서, 당신은 반드시 상태 변수를 설정해 UI를 변경해야합니다.
폼에서 당신은 몇 개의 다른 입력으로 상태를 변경 시킬 필요가 있습니다.

- Changing the text input (human) 빈 상태를 입력된 상태로 변경하거나 그 반대로, 이것은 텍스트 박스가 비어졌는지에 따릅니다.
- Clicking the Submit button (human) 제출 상태를 변경해야합니다.
- Successful network response (computer) 성공 상태를 변경해야합니다.
- Failed network response (computer) 에러 상태를 그에 상응하는 에러 메세지와 함께 변경해야합니다.

> Note
> 인간의 입력은 보통 이벤트 핸들러를 필요로 합니다.

이 플로우를 이해하기 위해, 각각의 상태를 라벨된 원으로 종이에 그립니다.
그 후 각 변경을 화살표로 표현합니다.
당신은 이 스케치로 부터 개발전 버그를 찾아낼 수 있습니다.

[label](https://react.dev/_next/image?url%3D%2Fimages%2Fdocs%2Fdiagrams%2Fresponding_to_input_flow.dark.png%26w%3D1920%26q%3D75)

### Step 3: Represent the state in memory with useState

이런 컴포넌트의 시각적 상태를 `useState`를 활용해 메모리로 나타냅니다.
단순함이 핵심: 각 상태는 움직이는 조각입니다. 그리고 당시는 가능하면 적은 조각이 필요합니다. 복잡할수록 버그를 유발합니다.

상태가 반드시 그곳에 있는 상황에서 시작하세요. 예를 들어, 인풋의 `answer`와 `error` 상태를 저장할 수 있습니다:

```jsx
const [answer, setAnswer] = useState("");
const [error, setError] = useState(null);
```

그 후, 화면에 보여줄 시각적 상태를 원할 것입니다.
표현할 한 개 이상의 상태들이 존재하므로 실험을 해야합니다.

최선의 선택을 구현하는데 노력 중이라면, 시각적 상태를 모두 추가하는 것도 방법입니다.

```jsx
const [isEmpty, setIsEmpty] = useState(true);
const [isTyping, setIsTyping] = useState(false);
const [isSubmitting, setIsSubmitting] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);
const [isError, setIsError] = useState(false);
```

당신의 첫 아이디어는 완벽하지 않지만, 과정에 따라 리팩토링 하면 됩니다.

### Step 4: Remove any non-essential state variables

중복되는 상태를 피하고 싶을 것이므로 필수 상태를 추적합니다.
상태의 구조를 리팩토링 하는 것은 당신의 컴포넌트를 이해하기 쉽게, 중복을 줄이고, 의도하지 않은 뜻을 피하게 해줍니다.
당신의 목표는, 유저가 보는 UI에 필요하지 않은 상태를 방지하는 것입니다.
(예를 들어, 당신은 에러 메세지와 폼 비활성화를 동시에 보여주는 것과 유저가 에러를 맞추는 것을 원하지 않을 것입니다.)

상태 변수에 던질 수 있는 질문은 다음과 같습니다.

- Does this state cause a paradox? 예를 들어, `isTyping`과 `isSubmitting`은 동시에 `true` 일 수 없습니다. 역설은 상태들에 제약이 충분치 않다는 것을 뜻합니다. 두 가지 불리언은 네 가지 조합이 가능하지만, 오직 세 가지만 유효한 상태입니다. 불가능한 상태를 없애고, `'typing'`, `'submitting'` 또는 `'success'`를 갖는 `status`에 합치세요.

- Is the same information available in another state variable already? 다른 역설: `isEmpty`와 `isTyping`은 동시에 `true`가 될 수 없습니다. 분리된 상태를 만들면, 싱크를 맞추는 것이 어렵고 버그를 유발합니다. `isEmpty`를 없애고 `answer.length === 0`을 통해 체크할 수 있습니다.
- Can you get the same information from the inverse of another state variable? `isError`는 `error !== null`로 체크할 수 있기에 불필요합니다.

이런 정리후, 오직 세 가지의 필요한 상태만 남습니다.

```jsx
const [answer, setAnswer] = useState("");
const [error, setError] = useState(null);
const [status, setStatus] = useState("typing"); // 'typing', 'submitting', or 'success'
```

이것들은 제거시 제기능을 할수 없기에 필수적인 것을 알 수 있습니다.

## DEEP DIVE - Eliminating “impossible” states with a reducer

이 세 가지 상태 변수들은 폼의 상태를 표현하기에 적절합니다.
하지만, 완전히 이해가지 않는 중간 상태가 있습니다.
예를 들어, `status`가 `success`일 때, null이 아닌 `error`는 말이 되지 않습니다. 더 정확하게 하기 위해 reducer로 추출할 수 있습니다.
Reducer는 여러 상태의 관련된 로직을 하나의 객체에서 관리할 수 있게 해줍니다.

### Step 5: Connect the event handlers to set state

마지막으로, 상태를 업데이트할 이벤트 핸들러를 생성합니다. 아래는 이벤트 핸들러를 적용한 폼입니다.

```jsx
import { useState } from "react";

export default function Form() {
  const [answer, setAnswer] = useState("");
  const [error, setError] = useState(null);
  const [status, setStatus] = useState("typing");

  if (status === "success") {
    return <h1>That's right!</h1>;
  }

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus("submitting");
    try {
      await submitForm(answer);
      setStatus("success");
    } catch (err) {
      setStatus("typing");
      setError(err);
    }
  }

  function handleTextareaChange(e) {
    setAnswer(e.target.value);
  }

  return (
    <>
      <h2>City quiz</h2>
      <p>
        In which city is there a billboard that turns air into drinkable water?
      </p>
      <form onSubmit={handleSubmit}>
        <textarea
          value={answer}
          onChange={handleTextareaChange}
          disabled={status === "submitting"}
        />
        <br />
        <button disabled={answer.length === 0 || status === "submitting"}>
          Submit
        </button>
        {error !== null && <p className="Error">{error.message}</p>}
      </form>
    </>
  );
}

function submitForm(answer) {
  // Pretend it's hitting the network.
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      let shouldError = answer.toLowerCase() !== "lima";
      if (shouldError) {
        reject(new Error("Good guess but a wrong answer. Try again!"));
      } else {
        resolve();
      }
    }, 1500);
  });
}
```

비록 이 코드는 명령형 예보다 길지만, 훨씬 견고합니다.
모든 상호작용을 상태로 나타내는 것은 나중에 새로운 상태가 기존의 상태를 건드리지 않도록 해줍니다.
이것은 또한 상호작용의 로직을 변경하는 것이 아닌, 보여지는 상태를 변경해도 되게 해줍니다.

## Recap

- 선언형 프로그래밍은 UI를 미시적으로 관리하는 것이 아닌 UI를 각각의 시각적 상태로 나태내는 것입니다.
- 컴포넌트 개발 시:
  1. 모든 시각 상태를 판별
  2. 상태 변화를 일으키는 것이 인간인지 컴퓨터인지 결정
  3. 상태를 `useState`로 모델링
  4. 역설과 버그를 피하기 위해 불필요 상태 제거
  5. 상태를 설정하기 위해 이벤트 핸들러 연결

---

## What I Learned

선언형 프로그래밍과 명령형 프로그래밍이 나에게 명확하지 않은 개념이었는데, 공식 문서의 택시 예를 보고 좀 더 명확하게 알게 되었다. -> 공식 문서에는 이런 예시들이 참 잘되어 있는 것 같다.

상태라는 것이 리액트에서 얼마나 중요한지, 공부를 하면 할 수록 깨닫는다.
단계별로 상태를 판별하고 제거하는 과정이 사실 익숙치 않은데, 고려하며 개발을 해야겠다.

상태를 통해 계산될 수 있는 값은 상태가 아니라는 말을 좀 더 생각하면서!

출처

- https://react.dev/learn/reacting-to-input-with-state
