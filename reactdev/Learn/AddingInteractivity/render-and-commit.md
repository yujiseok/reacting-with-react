- [Render and Commit](#render-and-commit)
  - [Step 1: Trigger a render](#step-1-trigger-a-render)
    - [Initial render](#initial-render)
    - [Re-renders when state updates](#re-renders-when-state-updates)
  - [Step 2: React renders your components](#step-2-react-renders-your-components)
  - [Pitfall](#pitfall)
  - [DEEP DIVE - Optimizing performance](#deep-dive---optimizing-performance)
  - [Step 3: React commits changes to the DOM](#step-3-react-commits-changes-to-the-dom)
  - [Epilogue: Browser paint](#epilogue-browser-paint)
  - [Recap](#recap)
  - [What I Learned](#what-i-learned)

# Render and Commit

당신의 컴포넌트가 화면에 보여지기 전에, 그들은 리액트에 의해 반드시 렌더링 되어야합니다.
이 과정을 이해하는 것은 당신의 코드가 어떻게 실행되고 동작되는지 설명해줍니다.

컴포넌트를 주방에서 재료들로 만들어지는 맛있는 식사라고 생각해보세요.
이 경우에서, 리액트는 손님들로 부터 요청을 전달하고 식사를 가져다 주는 웨이터입니다.
이 요청과 UI의 서빙은 3 단계를 거칩니다.

1. 렌더를 트리거 (손님의 주문을 주방에 전달)
2. 컴포넌트 렌더링 (주방에서 주문 준비)
3. 돔에 커밋 (주문을 테이블에 전달)

![image](https://user-images.githubusercontent.com/83855636/236820505-7c7f8365-0075-4b1e-8ccc-ad088c38389a.png)

## Step 1: Trigger a render

컴포넌트는 2가지 이유에서 렌더링 됩니다.

1. 컴포넌트의 첫 번째 렌더
2. 컴포넌트 혹은 조상의 상태가 변경 되었을 때

### Initial render

앱이 시작할 때, 첫 번째 렌더링을 트리거 해야합니다. 프레임워크들 또는 샌드박스들은 가끔씩 이런 코드를 숨깁니다. 하지만 이것은 타겟이 될 돔 노드에 `createRoot`를 호출하고 `render` 메서드로 구현할 수 있습니다.

### Re-renders when state updates

첫 번째 렌더링이 일어난 후, 당신은 set 함수로 상태를 변경해 추가적인 렌더링을 일으킬 수 있습니다.
컴포넌트의 상태를 변경시키는 것은 렌더를 자동적으로 큐에 등록합니다. (식당의 손님이 차를 시키고, 디저트를 시키는 모든 종류의 것들이 첫 번째 주문을 한 후에 그들에 목마름이나 배고픈 상태에 따릅니다. )

![image](https://user-images.githubusercontent.com/83855636/236822219-fc5e9480-1461-499f-993f-a6e6e6139b74.png)

## Step 2: React renders your components

렌더를 일으킨 후, 리액트는 화면에 보여줄 컴포넌트들을 호출을 통해 찾아냅니다.
렌더링은 리액트가 컴포넌트를 호출하는 것을 뜻합니다.

- 첫 번째 렌더, 리액트는 루트 컴포넌트를 호출
- 이후 렌더, 리액트는 상태가 변경되어 렌더를 일으키는 컴포넌트를 호출

이 과정은 계속됩니다. 만약 변경된 컴포넌트가 다른 컴포넌트를 반환할 경우, 리액트는 그 컴포넌트를 렌더합니다. 그리고 그 컴포넌트가 무언가 반환하면, 리액트는 계속해서 그 컴포넌트를 다음으로 렌더합니다. 이 과정은 중첩된 컴포넌트가 없을 경우와 리액트가 정확히 어떤 것을 화면에 나타낼지 알때 까지 계속 됩니다.

```jsx
export default function Gallery() {
  return (
    <section>
      <h1>Inspiring Sculptures</h1>
      <Image />
      <Image />
      <Image />
    </section>
  );
}

function Image() {
  return (
    <img
      src="https://i.imgur.com/ZF6s192.jpg"
      alt="'Floralis Genérica' by Eduardo Catalano: a gigantic metallic flower sculpture with reflective petals"
    />
  );
}
```

- 첫 번째 렌더에서, 리액트는 `<section>`, `<h1>`, 그리고 `<img>` 태그들을 위한 돔 노드를 생성합니다.
- 리렌더시, 리액트는 그들의 속성들이 이전 렌더에 비해 변경이 있는지 계산합니다. 커밋 단계로 넘어가기 전까지 어떤 행동도 하지 않습니다.

## Pitfall

렌더링은 반드시 순수해야 합니다.

- 똑같은 입력과 똑같은 출력. 컴포넌트는 반드시 똑같은 JSX를 반환해야 합니다.
- 자신들의 일에 신경 써야 합니다. 렌더 전에 존재한 객체또는 변수를 변경해서는 안됩니다.

그렇지 않으면, 혼란스러운 버그들과 코드가 커질 수록 예상치 못한 결과를 마주합니다. 개발시 엄격 모드는 리액트가 함수를 두 번 호출하여 순수하지 않은 함수를 밝혀냅니다.

## DEEP DIVE - Optimizing performance

만약 변경될 컴포넌트가 중첩되어 트리의 상단에 있을 경우 기본적으로 퍼포먼스적으로 최적화 되지않습니다. 만약 퍼포먼스 이슈가 있다면, 퍼포먼스 섹션에서 최적화하는 법을 다룹니다.
섣불리 최적화하지 마세요!

## Step 3: React commits changes to the DOM

당신의 컴포넌트가 렌더링 된 후, 리액트는 돔을 수정합니다.

- 첫 번째 렌더, 리액트는 `appendChild()` 돔 API를 활용해 돔 노드를 추가합니다.
- 리렌더 시, 리액트는 최신의 렌더링에서 가장 최소한의 필요에 따른 작업을 적용합니다.

리액트는 렌더 사이에 달라진 돔 노드만을 변경합니다.

## Epilogue: Browser paint

렌더링이 끝난 후, 리액트가 돔을 변경하고 브라우저는 화면을 리페인트합니다. 비록 이 과정은 브라우저 렌더링이라고 알려져 있지만, 리액트는 혼동을 피하기 위해 페인팅이라고 참조합니다.
![Alt text](https://react.dev/images/docs/illustrations/i_browser-paint.png)

## Recap

- 리액트 앱에서 어떤 업데이트든 3 단계를 거칩니다.

  1. Trigger
  2. Render
  3. Commit

- 엄격 모드를 통해 실수를 잡아낼 수 있습니다.
- 만약 돔이 이전 렌더와 동일하다면, 리액트는 변경시키지 않습니다.

---

## What I Learned

이전 렌더링과 동일한 돔은 변경시키 않는 다는 것이 신기하다. 렌더와 커밋 단계를 식당에 비유한 예시 덕분에 쉽게 이해가 되었다.

출처

- https://react.dev/learn/render-and-commit
