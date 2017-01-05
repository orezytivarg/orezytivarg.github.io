---
id: optimizing-performance
title: 퍼포먼스 최적화하기
---

내부적으로 React는 몇 가지 테크닉을 통해 UI를 업데이트할 때 필요한 DOM 조작을 최소화한다. 다른 많은 애플리케이션에 대해서 React를 사용하게되면 별다른 퍼포먼스 최적화 작업이 없이도 빠르게 반응하는 유저 인터페이스를 제공할 수 있을 것이다. 그래도 React 애플리케이션의 속도를 빠르게 하는 몇 가지 방법이 역시 존재한다.

## Production Build 사용하기

만약 당신이 벤치마킹한 React 애플리케이션에서 퍼포먼스 문제를 경험했다면, 프로덕션 빌드를 이용해서 테스트했는지 확인해보자:

* Create React App를 이용할 때에는, `npm run build` 명령어를 실행하고 다음 지시사항을 따를 필요가 있다.
* single-file 빌드 시에는, 제품 레벨에서 사용가능한 `.min.js` 버전을 제공한다.
* Browserify를 이용할 때는 , `NODE_ENV=production`과 함께 실행해야 한다.
* Webpack을 이용할 때는, 당신의 production config에 다음 플러그인을 추가해야 한다.

```js
new webpack.DefinePlugin({
  'process.env': {
    NODE_ENV: JSON.stringify('production')
  }
}),
new webpack.optimize.UglifyJsPlugin()
```

development 빌드는 애플리케이션 개발에 도움이 되는 추가적인 경고를 포함하고 있어서 느려지게하는 원인이 된다.

## 재보정 피하기(Avoid Reconciliation)

React는 렌더링된 UI를 내부적으로 따로 관리하고 있다. React가 내부적으로 관리하고 있는 모델은 당신의 컴포넌트가 return한 React element를 포함하고 있다. 이 모델을 통해 React는 DOM node를 생성하는 것을 피하고 이미 존재하는 DOM node에 대해 불필요하게 접근하는 것을 피한다. 이미 존재하는 DOM node에 대한 접근은 보통 JavaScript 오브젝트를 조작하는 것보다 느리다. 이전에는 "virtual DOM"이라고 일컬어졌지만, 이제는 React Native에서도 같은 방법으로 동작한다.

컴포넌트의 props나 state가 변경되었을 때, React는 이전에 렌더된 React element와 새로 리턴된 React element를 비교하여 실제 DOM을 갱신할 필요가 있는지를 결정한다. 이 둘이 같지 않을 경우에 React는 DOM을 갱신한다.

어떤 경우에는 컴포넌트의 라이프사이클 함수 `shouldComponentUpdate`를 전부 오버라이드해서 속도를 올릴 수 있다. 이 함수는 리렌더링 프로세스가 시작하기 직전에 트리거되는 함수다. 이 함수의 기본 구현은 `true`를 리턴하여 React로 하여금 업데이트를 수행하도록 하는 것이다:

```javascript
shouldComponentUpdate(nextProps, nextState) {
  return true;
}
```

만약 컴포넌트가 업데이트할 필요가 없는 상황인지 알고있다면, `render()`를 호출하는 것을 포함한 전체 렌더링 프로세스를 건너뛸 수 있도록 `shouldComponentUpdate` 함수에서 `false`를 리턴하면 된다.

## 실전 shouldComponentUpdate

여기에 컴포넌트 서브트리가 있다. 각각 `SCU`는 `shouldComponentUpdate`를 뜻하고, `vDOMEq`는 렌더된 React element가 동일한지를 뜻한다. 마지막으로 각 원의 색은 컴포넌트가 재조정되야하는지 아닌지 여부를 뜻한다.

<figure><img src="/assets/images/should-component-update.png" /></figure>

서브트리 C2의 `shouldComponentUpdate`가 `false`를 리턴하기 때문에 React는 C2를 렌더하려하지 않는다, 그리고 그 결과 C4와 C5의 `shouldComponentUpdate` 역시 호출하지 않는다.

For C1 and C3, `shouldComponentUpdate` returned `true`, so React had to go down to the leaves and check them. For C6 `shouldComponentUpdate` returned `true`, and since the rendered elements weren't equivalent React had to update the DOM.

The last interesting case is C8. React had to render this component, but since the React elements it returned were equal to the previously rendered ones, it didn't have to update the DOM.

Note that React only had to do DOM mutations for C6, which was inevitable. For C8, it bailed out by comparing the rendered React elements, and for C2's subtree and C7, it didn't even have to compare the elements as we bailed out on `shouldComponentUpdate`, and `render` was not called.

## Examples

If the only way your component ever changes is when the `props.color` or the `state.count` variable changes, you could have `shouldComponentUpdate` check that:

```javascript
class CounterButton extends React.Component {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  shouldComponentUpdate(nextProps, nextState) {
    if (this.props.color !== nextProps.color) {
      return true;
    }
    if (this.state.count !== nextState.count) {
      return true;
    }
    return false;
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
```

In this code, `shouldComponentUpdate` is just checking if there is any change in `props.color` or `state.count`. If those values don't change, the component doesn't update. If your component got more complex, you could use a similar pattern of doing a "shallow comparison" between all the fields of `props` and `state` to determine if the component should update. This pattern is common enough that React provides a helper to use this logic - just inherit from `React.PureComponent`. So this code is a simpler way to achieve the same thing:

```js
class CounterButton extends React.PureComponent {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
```

Most of the time, you can use `React.PureComponent` instead of writing your own `shouldComponentUpdate`. It only does a shallow comparison, so you can't use it if the props or state may have been mutated in a way that a shallow comparison would miss.

This can be a problem with more complex data structures. For example, let's say you want a `ListOfWords` component to render a comma-separated list of words, with a parent `WordAdder` component that lets you click a button to add a word to the list. This code does *not* work correctly:

```javascript
class ListOfWords extends React.PureComponent {
  render() {
    return <div>{this.props.words.join(',')}</div>;
  }
}

class WordAdder extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      words: ['marklar']
    };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    // This section is bad style and causes a bug
    const words = this.state.words;
    words.push('marklar');
    this.setState({words: words});
  }

  render() {
    return (
      <div>
        <button onClick={this.handleClick} />
        <ListOfWords words={this.state.words} />
      </div>
    );
  }
}
```

The problem is that `PureComponent` will do a simple comparison between the old and new values of `this.props.words`. Since this code mutates the `words` array in the `handleClick` method of `WordAdder`, the old and new values of `this.props.words` will compare as equal, even though the actual words in the array have changed. The `ListOfWords` will thus not update even though it has new words that shoud be rendered.

## The Power Of Not Mutating Data

The simplest way to avoid this problem is to avoid mutating values that you are using as props or state. For example, the `handleClick` method above could be rewritten using `concat` as:

```javascript
handleClick() {
  this.setState(prevState => ({
    words: prevState.words.concat(['marklar'])
  }));
}
```

ES6 supports a [spread syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator) for arrays which can make this easier. If you're using Create React App, this syntax is available by default.

```js
handleClick() {
  this.setState(prevState => ({
    words: [...prevState.words, 'marklar'],
  }));
};
```

You can also rewrite code that mutates objects to avoid mutation, in a similar way. For example, let's say we have an object named `colormap` and we want to write a function that changes `colormap.right` to be `'blue'`. We could write:

```js
function updateColorMap(colormap) {
  colormap.right = 'blue';
}
```

To write this without mutating the original object, we can use [Object.assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) method:

```js
function updateColorMap(colormap) {
  return Object.assign({}, colormap, {right: 'blue'});
}
```

`updateColorMap` now returns a new object, rather than mutating the old one. `Object.assign` is in ES6 and requires a polyfill.

There is a JavaScript proposal to add [object spread properties](https://github.com/sebmarkbage/ecmascript-rest-spread) to make it easier to update objects without mutation as well:

```js
function updateColorMap(colormap) {
  return {...colormap, right: 'blue'};
}
```

If you're using Create React App, both `Object.assign` and the object spread syntax are available by default.

## Using Immutable Data Structures

[Immutable.js](https://github.com/facebook/immutable-js) is another way to solve this problem. It provides immutable, persistent collections that work via structural sharing:

* *Immutable*: once created, a collection cannot be altered at another point in time.
* *Persistent*: new collections can be created from a previous collection and a mutation such as set. The original collection is still valid after the new collection is created.
* *Structural Sharing*: new collections are created using as much of the same structure as the original collection as possible, reducing copying to a minimum to improve performance.

Immutability makes tracking changes cheap. A change will always result in a new object so we only need to check if the reference to the object has changed. For example, in this regular JavaScript code:

```javascript
const x = { foo: "bar" };
const y = x;
y.foo = "baz";
x === y; // true
```

Although `y` was edited, since it's a reference to the same object as `x`, this comparison returns `true`. You can write similar code with immutable.js:

```javascript
const SomeRecord = Immutable.Record({ foo: null });
const x = new SomeRecord({ foo: 'bar'  });
const y = x.set('foo', 'baz');
x === y; // false
```

In this case, since a new reference is returned when mutating `x`, we can safely assume that `x` has changed.

Two other libraries that can help use immutable data are [seamless-immutable](https://github.com/rtfeldman/seamless-immutable) and [immutability-helper](https://github.com/kolodny/immutability-helper).

Immutable data structures provide you with a cheap way to track changes on objects, which is all we need to implement `shouldComponentUpdate`. This can often provide you with a nice performance boost.
