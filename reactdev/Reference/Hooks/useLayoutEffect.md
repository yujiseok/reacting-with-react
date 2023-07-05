# useLayoutEffect

#### Pitfall

useLayoutEffect는 퍼포먼스 문제를 일으킨다. 가능하면 useEffect를 사용해라.

useLayoutEffect는 useEffect의 다른 버전으로 브라우저가 리페인트 되기 전에 실행된다.

```jsx
useLayoutEffect(setup, dependencies?)
```

## Reference

### useLayoutEffect(setup, dependencies?)

브라우저가 리페인트 되기전 레이아웃 측정을 위해 useLayoutEffect를 호출해라.

```jsx
import { useState, useRef, useLayoutEffect } from 'react';

function Tooltip() {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0);

  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setTooltipHeight(height);
  }, []);
  // ...
```

**Parameters**

- setup: 이펙트 로직의 함수. 셋업 함수는 선택적으로 정리 함수를 반환한다. 컴포넌트가 돔에 추가되기 전에, 리액트는 셋업 함수를 실행한다. 의존성의 변경으로 리랜더 된 후, 리액트는 우선 이전 값과 정리 함수를 실행하고 새로운 값과 셋업 함수를 실행한다. 돔에서 컴포넌트가 사라지기 전에, 리액트는 정리 함수를 실행한다.
- optional dependencies: 셋업 코드가 참조하는 반응 값의 리스트. 반응 값은 컴포넌트 내부의 모든 값을 포함한다.

**Returns**
useLayoutEffect는 undefined를 반환한다.

**Caveats**

- useLayoutEffect 내부의 코드는 상태 변경을 스케줄해, 브라우저가 리페인트 하는 것을 막는다. 지나치게 사용하면, 앱이 느려질 수 있다. 가능하면 useEffect를 사용해라.

## Usage

### Measuring layout before the browser repaints the screen

대부분의 컴포넌트는 그들의 위치나 사이즈를 화면에 그리는 것을 결정할 때 알필요 없다.
그들은 오직 jsx만 반환한다. 그리고 브라우저가 레이아웃을 계산하고 화면에 리페인트 한다.

가끔, 이것은 충분치 않다. 호버시 어떤 요소 옆에 나타나는 툴팁을 생각하자. 만약 충분한 공간이 있다면, 요소 위에 생길 것이고, 아닐 경우 아래에 생길 것이다.
올바른 툴팁의 최종 위치를 랜더하기 위해선, 높이를 알아야한다.

이렇게 하려면, 두 패스로 랜더해야한다.

1. 아무데서 툴팁을 랜더
2. 높이를 재고, 어디에 툴팁을 둘지 결정
3. 정확한 위치에 툴팁 랜더

모든 동작은 브라우저가 화면을 리페인트 하기 전에 일어나야한다. 유저가 툴팁의 이동을 보는 것은 원치 않을테니.
useLayoutEffect를 호출해 브라우저가 화면에 리페인트 되기전 레이아웃을 잰다.

```jsx
function Tooltip() {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0); // You don't know real height yet

  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setTooltipHeight(height); // Re-render now that you know the real height
  }, []);

  // ...use tooltipHeight in the rendering logic below...
}
```

1. 툴팁은 tooltipHeight이 0인 상태에서 랜더 됨
2. 리액트는 돔에 그것을 두고, useLayoutEffect을 실행
3. useLayoutEffect는 툴팁 컨텐츠의 높이를 재고 즉각적인 리랜더를 일으킴
4. 툴팁은 진짜 tooltipHeight와 랜더
5. 리액트가 돔을 변경하고, 브라우저는 툴팁을 최종적으로 화면에 보여줌

툴팁 컴포넌트가 두 패스로 랜더하는 것을 명심해라. 오직 최종 결과만을 보게된다.
이것이 useLayoutEffect가 필요한 이유이다.

#### Note

두 패스로 랜더하는 것은 브라우저 성능을 저하시킨다. 이것을 가능한 피하도록 해라.

## Troubleshooting

#### I’m getting an error: ”useLayoutEffect does nothing on the server”

useLayoutEffect의 목적은 컴포넌트가 랜더에 레이아웃 정보를 사용할 수 있도록 하는 것이다.

1. 초기 컨텐츠를 랜더
2. 브라우저가 리페인트하기 전 레이아웃 측정
3. 레이아웃 정보를 통해 최종 컨텐츠 랜더

서버 랜더링을 사용하는 프레임워크를 사용하면, 리액트는 html을 서버에서 초기 랜더한다. 이것은 자바스크립트가 로드 되기전에 html을 보여주도록 한다.

문제는 서버에 레이아웃 정보가 없다는 것이다.

위의 예에서 툴팁은 useLayoutEffect를 호출해 컨텐츠의 높이에 따른 위치를 정확히한다.
만약 툴팁을 서버 html에서 랜더한다면, 이것을 결정하는 것은 불가능하다. 서버에서, 레이아웃이 없기 때문이다.
즉 서버에서 랜더 되더라도, 이것의 위치는 클라이언트로 점프 되고 자바스크립트가 로드되고 실행된다.

보통, 레이아웃 정보에 기반한 컴포넌트는 서버에서 랜더될 필요가 없다.
예를 들어, 툴팁을 초기 랜더에 보여주는 것은 말이 안된다. 이것은 클라이언트 상호작용에서 일어난다.

하지만, 이 문제가 발생하면, 몇 가지 다른 옵션이 있다.

- useLayoutEffect를 useEffect로 대체. 이것은 리액트에게 초기 랜더의 결과를 페인트를 블록하지 않고 보여줘도 된다고 말해준다.
- 오직 클라이언트 위한 컴포넌트로 만들기. 이것은 서버 랜더 중 가자 가까운 서스펜스 바운더리까지 컨텐츠를 대체하도록 한다.
- 하이드레이션 이후에만 useLayoutEffect와 랜더할 수 있다. isMounted 불리언 상태를 false로 초기화해 이펙트 호출 내부에서 true로 변경한다. 랜더 로직은 다음과 같다. return isMounted ? <RealContent /> : <FallbackContent />. 하이드레이션과 서버 동안, 유저는 FallbackContent를 보고, useLayoutEffect를 호출하지 않는다. 그 후 리액트는 RealContent로 대체하고 useLayoutEffect를 실행한다.
- 컴포넌트를 외부 데이터 스토어와 연결할 때 다른 레이아웃을 측정하기 위해 useLayoutEffect에 의존한다면, 서버 랜더를 지원하는 useSyncExternalStore를 사용하는 것을 고려해라.

---

## What I Learned

useLayoutEffect를 사실 사용해 본적이 없다. 아직도 어떤 경우에 사용해야하는지 모호하다.
이펙트의 차이는 레이아웃이펙트는 리페인트 전에 동작한다는 것이고, 레이아웃 측정 정보가 사용될 때 사용된다는 것이다.

하지만, 많이 사용하게 되면 브라우저가 리페인트 되는 것을 막기에 퍼포먼스 측면에서 좋지않다.
대부분의 경우에서 이펙트를 사용해 처리하자.
