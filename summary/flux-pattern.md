- [Flux 패턴](#flux-패턴)
  - [Flux 패턴이 등장하게된 배경](#flux-패턴이-등장하게된-배경)
  - [MVC](#mvc)
    - [문제점](#문제점)
  - [Flux](#flux)
    - [Action](#action)
    - [Dispatcher](#dispatcher)
    - [Store](#store)
    - [View](#view)

# Flux 패턴

## Flux 패턴이 등장하게된 배경

Flux는 MVC 패턴의 복잡성과 에러의 예측이 어려워지는 이유에서 등장하게 되었습니다.

## MVC

MVC 패턴은 Model(데이터)-View(UI)-Controller(데이터를 다루는 로직)로 구성된 패턴입니다.

![](https://media.geeksforgeeks.org/wp-content/uploads/20210629165722/mvc.png)

### 문제점

MVC 패턴의 문제점은 다음과 같습니다. 클라이언트 앱의 규모가 커지면 아래와 같이 수많은 Model과 View가 생기게 됩니다.

![](https://www.freecodecamp.org/news/content/images/2020/04/Screenshot-2020-04-16-at-6.38.14-PM.png)

이런 관계는 컴포넌트의 복잡도를 높이게 됩니다. 또한 이런 복잡도로 인해 앱은 예측할 수 없게 됩니다.

이런 문제점을 해결하기 위해 페이스북 팀은 Flux 패턴을 고안했습니다.

## Flux

Flux는 MVC 패턴의 복잡성을 해결하기 위해 등장한 패턴입니다.
Flux는 단방향 데이터 흐름을 통해 복잡성을 해소하였습니다.

![](https://www.freecodecamp.org/news/content/images/2020/04/Flux-3.png)

각 단계를 살펴보면 다음과 같습니다.

### Action

액션은 이벤트를 다룹니다. 이 이벤트는 뷰에서 발생됩니다. 보통 이 단계에서는 API 호출이 일어납니다. 액션이 발생되면 디스패쳐를 통해 디스패치됩니다. 액션은 포스트를 작성하거나, 삭제하는 등의 유저 상호작용이 될 수 있습니다.

### Dispatcher

디스패쳐는 액션의 페이로드를 받아 스토어에 디스패치합니다.
즉 액션을 받아서 실제로 어떤 동작을 취할지 결정하는 곳입니다.

### Store

앱의 상태를 저장하는 역할로, 디스패쳐와 연결시켜 스토어에 접근할 수 있도록 해줍니다. 이 상태를 뷰에서 반영합니다.

### View

UI 렌더링을 담당하는 역할로, 액션을 발생시킵니다.
스토어에 변화가 발생하면 리렌더링이 일어납니다.

---

출처

- https://ko.reactjs.org/blog/2014/05/06/flux.html
- https://www.geeksforgeeks.org/benefit-of-using-mvc/
- https://www.freecodecamp.org/news/how-to-use-flux-in-react-example/
