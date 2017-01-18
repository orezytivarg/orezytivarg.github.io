---
id: optimizing-performance
title: React 성능 최적화하기
category: React
---

[원문보기](https://facebook.github.io/react/docs/optimizing-performance.html)

내부적으로 React는 몇 가지 테크닉을 통해 UI를 업데이트할 때 필요한 DOM 조작을 최소화한다. 다른 많은 애플리케이션에 대해서 React를 사용하게되면 별다른 성능 최적화 작업이 없이도 빠르게 반응하는 유저 인터페이스를 제공할 수 있을 것이다. 그래도 React 애플리케이션의 속도를 빠르게 하는 몇 가지 방법이 역시 존재한다.

## Production Build 사용하기

만약 벤치마킹한 React 애플리케이션에서 성능 문제를 경험했다면, 프로덕션 빌드를 이용해서 테스트했는지 확인해보자:

* Create React App를 이용할 때에는, `npm run build` 명령어를 실행하고 다음 지시사항을 따를 필요가 있다.
* single-file 빌드 시에는, 제품 레벨에서 사용가능한 `.min.js` 버전을 제공한다.
* Browserify를 이용할 때는, `NODE_ENV=production`과 함께 실행해야 한다.
* Webpack을 이용할 때는, production config에 다음 플러그인을 추가해야 한다.

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

React는 렌더링된 UI를 내부적으로 다른 식으로 관리하고 있다. React가 내부적으로 관리하고 있는 이 모델은 컴포넌트가 리턴한 React element를 포함하고 있다. 이 모델을 통해 React는 DOM node를 생성하는 것을 피하고 이미 존재하는 DOM node에 대해 불필요하게 접근하는 것을 피한다. 이미 존재하는 DOM node에 대한 접근은 JavaScript 오브젝트를 조작하는 것보다 종종 느릴 수 있다. 이전에는 "virtual DOM"이라고 일컬어졌지만, 이제는 React Native에서도 같은 방법으로 동작한다.

컴포넌트의 props나 state가 변경되었을 때, React는 이전에 렌더된 React element와 새로 리턴된 React element를 비교하여 실제 DOM을 갱신할 필요가 있는지를 결정한다. 이 둘이 같지 않을 경우에 React는 DOM을 갱신한다.

어떤 경우에는 컴포넌트의 라이프사이클 함수 `shouldComponentUpdate`를 전부 오버라이드해서 속도를 올릴 수 있다. 이 함수는 리렌더링 프로세스가 시작하기 직전에 트리거되는 함수다. 이 함수의 기본 구현은 `true`를 리턴하여 React로 하여금 업데이트를 수행하도록 하는 것이다:

```javascript
shouldComponentUpdate(nextProps, nextState) {
  return true;
}
```

만약 컴포넌트가 업데이트할 필요가 없는 상황인지 알고있다면, `render()`를 호출하는 것을 포함한 전체 렌더링 프로세스를 건너뛸 수 있도록 `shouldComponentUpdate` 함수에서 `false`를 리턴하면 된다.

## 실전 shouldComponentUpdate

아래에 컴포넌트와 그 서브트리가 있다. `SCU`는 `shouldComponentUpdate`를 뜻하고, `vDOMEq`는 렌더된 React element가 동일한지를 뜻한다. 각 원의 색은 컴포넌트가 재조정되야하는지 아닌지 여부를 뜻한다.

<figure><img src="https://facebook.github.io/react/img/docs/should-component-update.png" /></figure>

서브트리 C2의 `shouldComponentUpdate`가 `false`를 리턴하기 때문에 React는 C2를 렌더하려하지 않는다, 그리고 그 결과 C4와 C5의 `shouldComponentUpdate` 역시 호출하지 않는다.

C1과 C3은 `shouldComponentUpdate`가 `true`를 리턴하기 때문에, React는 아래쪽을 쭉 따라 내려가면서 확인해야만 한다. C6의 `shouldComponentUpdate`가 `true`를 리턴하고 따라서 렌더링된 엘레멘트가 동일하지 않게 되기 때문에 React는 DOM을 갱신해야만 한다.

마지막으로 흥미로운 것은 C8이다. React는 이 컴포넌트를 렌더해야만 하지만 React elements가 이전에 리턴된 React elements와 동일한 것이기 때문에, DOM을 갱신해야만 하는 것은 아니다.

React는 피할수없는 C6만을 위해 DOM을 변경해야만 한다는 것을 유의하자. C8은 렌더된 React elements와의 비교를 통해 구제되며 C2의 서브트리와 C7에서는 비교할 필요조차 없이 `shouldComponentUpdate`에서 구제되며, `render`는 호출되지 않았다.

## 예제

만약 컴포넌트를 변경하는 유일한 방법이 `props.color`나 `state.count`를 변경하는 것이라면 `shouldComponentUpdate`에서는 다음과 같이 확인하도록 작성하면 된다:

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

이 코드에서, `shouldComponentUpdate`는 그저 `props.color`나 `state.count`에 변화가 있는지만 확인한다. 만약 값이 변하지 않았다면 컴포넌트를 갱신하지 않는다. 만약 컴포넌트가 조금 더 복잡한 경우에는 컴포넌트 업데이트를 결정하기 위해 `props`와 `state`의 모든 필드에 대해 "shallow comparison"이라는 패턴을 사용할 수 있다. 이 패턴은 React에서 이 로직을 사용하기 위한 헬퍼를 제공할 정도로 일반적이다 - 그냥 `React.PureComponent`만 상속하면 된다. 따라서 이 코드는 더 간단한 방법으로 위 코드와 같은 일을 할수 있다:

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

대부분의 경우 `shouldComponentUpdate`를 작성하는 대신 `React.PureComponent`를 사용할 수 있다. 다만 이것은 shallow comparison을 할 것이므로, props나 state에 mutate를 하는 경우에는 shallow comparison은 제대로 동작하지 않을 것이다.

이는 더 복잡한 데이터 구조에서 문제가 될 수 있다. 예를 들어, 콤마로 분리된 단어 목록을 보여주는 `ListOfWords`라는 컴포넌트가 있다고 하고, 버튼을 클릭하여 단어를 추가하는 부모인 `WordAdder` 컴포넌트가 있다고 하자. 이 코드는 올바르게 동작하지 *않는다*:

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

문제는 `PureComponent`가 단순히 `this.props.words`의 이전 값과 새 값을 비교할 뿐이라는 것이다. `WordAdder`의 메서드인 `handleClick`이 `words`를 mutate하는 방식이기 때문에, 배열 내의 실제 단어들이 변경되었더라도, `this.props.words`의 이전값과 새 값은 동일한 값이라고 비교될 것이다. 따라서 `ListOfWords`는 새 단어가 추가되더라도 갱신되지 않을 것이다.

## 가변적이지 않은 데이터의 힘(The Power Of Not Mutating Data)

이 문제를 피할 수 있는 가장 간단한 방법은 바로 props와 state의 값을 mutating하지 않는 것이다. 예를 들어 위의 `handleClick` 메서드를 다음과 같이 `concat`을 사용하도록 재작성할 수 있다:

```javascript
handleClick() {
  this.setState(prevState => ({
    words: prevState.words.concat(['marklar'])
  }));
}
```

ES6는 배열을 쉽게 조작할 수 있도록 [spread syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator)를 제공한다. 만약 Create React App을 사용한다면 기본적으로 이 문법이 사용가능하다.

```js
handleClick() {
  this.setState(prevState => ({
    words: [...prevState.words, 'marklar'],
  }));
};
```

또한 오브젝트를 mutate하는 코드를 mutate하지 않도록 재작항 할 수 있다. 예를 들어, `colormap`이라는 오브젝트가 있고 `colormap.right`를 `blue`로 변경하고 싶다고 하자. 이를 다음과 같이 작성할 수 있다:

```js
function updateColorMap(colormap) {
  colormap.right = 'blue';
}
```

원본 오브젝트를 mutating하는 방법이다. 이를 원본 오브젝트를 mutating하지 않는 방법으로 작성하기 위해서  [Object.assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) 메서드를 사용하면 된다:

```js
function updateColorMap(colormap) {
  return Object.assign({}, colormap, {right: 'blue'});
}
```

`updateColorMap`은 이제 mutating된 이전 오브젝트가 아닌 새로운 오브젝트를 리턴한다. ES6의 `Object.assign`은 polyfill이 필요하다.

자바스크립트에 추가될 제안 중 [object spread properties](https://github.com/sebmarkbage/ecmascript-rest-spread) 또한 mutation 없이 간단히 오브젝트를 업데이트할 수 있는 방법을 제공한다:

```js
function updateColorMap(colormap) {
  return {...colormap, right: 'blue'};
}
```

만약 Create React App을 사용한다면 `Object.assign`과 object spread 문법을 둘다 기본적으로 사용 가능하다.

## 불변 데이터 구조 사용하기(Using Immutable Data Structures)

[Immutable.js](https://github.com/facebook/immutable-js) 는 이 문제를 해결하기 위한 또다른 방법이다. 이 라이브러리는 내부적으로 구조를 공유하도록 작성된 immutable하고 persistent한 컬렉션을 제공한다.

* *Immutable*: 일단 한번 생성되면, 컬렉션은 변경될 수 없다.
* *Persistent*: set과 같은 컬렉션은 이전 컬렉션으로부터 새로운 컬렉션이 생성될 수 있다. 원본 컬렉션은 새로운 컬렉션이 생성된 이후에도 아직 유효하다.
* *Structural Sharing*: 오리지널 컬렉션으로부터 생성된 새로운 컬렉션은 가능한한 오리지널 컬랙션과 같은 구조를 갖게 되며 성능 향상을 위해 카피를 줄인다.

불변성은 값 변화에 대한 추적비용을 감소시킨다. 값 변화는 항상 새로운 오브젝트를 리턴하기 때문에 오브젝트의 레퍼런스가 변경되었는지만 확인하면 된다. 예를 들면 다음과 같은 보통 자바스크립트 코드에서는:

```javascript
const x = { foo: "bar" };
const y = x;
y.foo = "baz";
x === y; // true
```

`y`가 편집되더라도 `x`와 같은 레퍼런스를 유지하기 때문에 비교 결과는 `true`이다. immutable.js를 이용한 비슷한 코드를 작성해보자:

```javascript
const SomeRecord = Immutable.Record({ foo: null });
const x = new SomeRecord({ foo: 'bar'  });
const y = x.set('foo', 'baz');
x === y; // false
```

이 경우에는 `x`를 mutate할 경우 새로운 레퍼런스를 리턴하기 때문에, 안전하게 `x`가 변경되었다고 가정할 수 있다.

불변 데이터를 지원할 수 있도록 도와주는 두 개의 다른 라이브러리들이 있다.
 [seamless-immutable](https://github.com/rtfeldman/seamless-immutable) 와 [immutability-helper](https://github.com/kolodny/immutability-helper).

불변 데이터 구조는 `shouldComponentUpdate`에 필요한 object 변경에 대한 추적을 쉽게 해준다. 아마도 훌륭한 성능 부스터로 쓰일 수 있을 것이다.
