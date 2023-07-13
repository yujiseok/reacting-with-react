# \<Fragment> (<>...</>)

프래그먼트는 <>...</>로 사용되며, 래퍼 노드 없이 요소를 감싸게 해준다.

```jsx
<>
  <OneChild />
  <AnotherChild />
</>
```

## Reference

### \<Fragment>

단일 요소가 필요한 경우 프래그먼트로 요소들을 감싸 그룹화 해라.
프래그먼트로 그룹화 하는 것은 돔의 결과에 아무런 영향이 없다. 그룹화 하지 않은 것과 동일하다.
빈 jsx 태그 <></>는 <Fragment></Fragment>의 단축형이다.

**Props**

- optional key: 프래그먼트는 명시적인 <Fragment>를 사용해 키를 사용할 수 있다.

**Caveats**

- 만약 키를 프래그먼트에 사용하려면, <>...</>를 사용할 수 없다. 명시적으로 Fragment를 리액트로 가져와서 `<Fragment key={yourKey}>...</Fragment>`로 랜더 해야한다.
- 리액트는 `<><Child /></>`에서 `[<Child />]`로 또는 그 반대로 랜더하거나 `<><Child /></> `를 `<Child />`로 또는 그 반대로 랜더할 시 상태를 초기화하지 않는다. 이것은 오직 한 레벨의 깊이에서 일어난다. 예를 들어, `<><><Child /></></>`에서 `<Child />`로 가는 것은 상태를 초기화한다.

## Usage

### Returning multiple elements

프래그먼트를 사용해서 여러 요소를 그룹화해 리턴할 수 있다. 여러 요소들을 담기위해 어느 곳에서나 사용할 수 있다.
예를 들어, 오직 하나의 요소만 반환지만, 프래그먼트를 사용해 여러 요소를 함께 반환할 수 있다.

```jsx
function Post() {
  return (
    <>
      <PostTitle />
      <PostBody />
    </>
  );
}
```

프래그먼트는 레이아웃이나 스타일에 영향을 주는 컨테이너 돔 요소와 다르게 영향을 끼치지 않기에 유용하다.

### Assigning multiple elements to a variable

다른 요소와 같게, 프래그먼트 요소를 변수로 담을 수 있으며, 프랍으로 전달도 가능하다.

```jsx
function CloseDialog() {
  const buttons = (
    <>
      <OKButton />
      <CancelButton />
    </>
  );
  return (
    <AlertDialog buttons={buttons}>
      Are you sure you want to leave this page?
    </AlertDialog>
  );
}
```

### Grouping elements with text

프래그먼트로 문자를 그룹화할 수 있다.

```jsx
function DateRangePicker({ start, end }) {
  return (
    <>
      From
      <DatePicker date={start} />
      to
      <DatePicker date={end} />
    </>
  );
}
```

### Rendering a list of Fragments

명시적으로 Fragment를 사용해야 하는 경우가 있다.
여러 요소를 반복문으로 랜더할 시, 각 요소에 키를 지정해야한다.
만약 반복문의 요소가 프래그먼트일 시, 일반적인 jsx 요소를 사용해 키 속성을 전달해야한다.

```jsx
function Blog() {
  return posts.map((post) => (
    <Fragment key={post.id}>
      <PostTitle title={post.title} />
      <PostBody body={post.body} />
    </Fragment>
  ));
}
```

---

## What I Learned

키를 사용하기 위해선 명시적으로 프래그먼트를 일반적인 jsx 태그로 사용해야 한다는 것을 처음 알게 되었다.
