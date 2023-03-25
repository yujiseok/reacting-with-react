- [Importing and Exporting Components](#importing-and-exporting-components)
  - [The root component file](#the-root-component-file)
  - [Exporting and importing a component](#exporting-and-importing-a-component)
  - [DEEP DIVE Default vs named exports](#deep-dive-default-vs-named-exports)
  - [Recap](#recap)
  - [What I Learned](#what-i-learned)

# Importing and Exporting Components

컴포넌트의 마법은 다른 컴포넌트를 조합하여 재사용 가능한 새로운 컴포넌트를 만들 수 있다는 것입니다.
그러나 중첩될수록, 다른 파일로 분리하는 것이 좋습니다.
그것은 유지 보수를 쉽게 할 수 있게 해주며, 가독성을 높여줍니다.

## The root component file

만약 CRA로 프로젝트를 만들었다면, 당신의 앱은 `src/App.js`에 존재할 것입니다. 당신의 설정에 따라 루트 컴포넌트는 다른 파일에 존재할 수 있습니다. 만약 파일 시스템 기반의 프레임워크(넥스트)를 사용한다면 루트 컴포넌트가 페이지별로 다를 것입니다.

## Exporting and importing a component

만약 랜딩 화면을 과학 책의 목록으로 보이게 하고 싶으면 어떻게 해야 할까요? 혹은 프로필을 다른 곳에 두고 싶다면? 이것은 컴포넌트(`Gallery`, `Profile`)를 루트 컴포넌트에서 분리하는 것이 말이 된다는 것을 뜻합니다. 이런 과정은 더 모듈화 시키고 다른 파일에서 재사용할 수 있도록 해줍니다.
3 단계에 거쳐서 컴포넌트를 분리할 수 있습니다.

1. 새로운 자바스크립트 파일을 만들고 컴포넌트를 옮깁니다.
2. 함수형 컴포넌트를 export 합니다.
3. 컴포넌트를 사용할 파일에서 import 해옵니다.

## DEEP DIVE Default vs named exports

자바스크립트에는 두 가지 export 방식이 있는데, default export의 경우 한 파일에서 오직 한 번만 가능하며, named exports는 사용하고 싶은 만큼 사용할 수 있습니다.

default export는 import 해올 시 이름을 바꿀 수 있습니다. 대부분 사람들은 한 컴포넌트를 export 시킬 때는 default export를 여러 컴포넌트를 export 시키고 싶을 때는 named export를 사용합니다.

## Recap

- 루트 컴포넌트는 프로젝트의 시작점
- 어떻게 컴포넌트를 import 하고 export 하는지
- 언제 default export와 named export를 사용하는지
- 어떻게 여러 컴포넌트를 export 하는지

---

## What I Learned

가끔 export 방식이 헷갈릴 때가 있는데 이제는 안 헷갈릴 것 같다.
</br>

출처

- https://react.dev/learn/importing-and-exporting-components
