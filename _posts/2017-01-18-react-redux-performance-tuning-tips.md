---
layout: post
id: react-redux-performance-tuning-tips
title: React Redux 성능 튜닝 팁
tags: [React, Redux]
---

[원문보기](https://medium.com/@arikmaor/react-redux-performance-tuning-tips-cef1a6c50759#.64yt91nsu)

# React Redux 성능 튜닝 팁

복잡한 React 앱을 작성할 때면 렌더링 성능과 사투를 벌이고 있는 자신을 발견할 것이다.
이 글에서는 성능 병목 현상을 탐지하고 고치는 도구와 테크닉을 전반적으로 다룰 것이다.

## 문제는 React의 렌더링 프로세스다

컴포넌트가 `this.setState()`를 호출하면, React는 DOM을 2단계에 걸쳐 재 렌더링한다:

1.  React 내부의 가상 DOM을 재 렌더링한다
2.  이전 가상 DOM과 현재 가상 DOM을 diff하고 변경점을 계산하여 실제 DOM에 적용한다

첫 번째 단계가 너무 오래 걸리게 되는 경우에 재 렌더링은 느려질 것이다.

## pure 컴포넌트의 재 렌더링 피하기

React 렌더링 최적화의 기본 원리는 React에게 DOM 결과에 아무런 변화가 없을 것이기 때문에 다시 렌더링할 필요가 없다고 알게 하는 것이다. React 이러한 신호를 보낼 수 있는 몇 가지 방법이 있다:

### 같은 엘레멘트의 레퍼런스를 반환하기

render 메서드가 같은 레퍼런스를 반환하면, React는 변화가 없는 것으로 간주할 것이다.

```js
  class MyComponent extends Component {
    text = ""
    renderedElement = null

    _render() {
      return <div>{this.props.text}</div>
    }

    render() {
      if (!this.renderedElement || this.props.text !== this.text) {
        this.text = this.props.text
        this.renderedElement = _render()
      }
      return this.renderedElement
    }
  }
```

캐싱을 직접 구현하기 싫으면 lodash의 memoize 함수를 이용할 수도 있다.

```js
  import memoize from 'lodash/memoize'

  class MyComponent extends Component {
    _render = memoize((text) => <div>{text}</div>)

    render() {
      return _render(this.props.text)
    }
  }
```

### shouldComponentUpdate에서 false를 리턴하기

React는 재 렌더링 해야하는지를 이 메서드를 통해 확인한다. 이 메서드의 기본 구현은 항상 true를 리턴하는 것이다.
React는 shallowCompare 함수를 제공한다. (물론 다른 라이브러리에서도 이런 함수를 제공하고 있다) 이 함수는 두 오브젝트를 비교할 때 탑 레벨의 프로퍼티의 동등성을 확인한다. 이 함수를 사용하면 pure 컴포넌트를 다음과 같이 구현할 수 있다:

```js
  import shallowCompare from 'react-addons-shallow-compare'

  export default class PureComponent extends Component {
    shouldComponentUpdate(nextProps, nextState) {
      return shallowCompare(this, nextProps, nextState);
    }

    render () {...}
  }
```

재 렌더링을 할지 말지를 결정하는 당신만의 로직을 작성할 수도 있지만, 함수가 매우 빠르다는 것을 확신할 수 있어야 한다. 만약 이 함수가 느리면, 다시 처음으로 돌아가게 된다. 왜냐하면 이 함수는 React가 재 렌더링을 하려 할때마다 호출될 것이기 때문이다.

### 고차 컴포넌트(Higher Order Components)

고차 컴포넌트를 이용해 이 최적화를 구현하고 재사용할 수 있다. 사실, 우리가 사용할 수 있도록 상당한 양의 일반적인 고차함수를 포함하고 있는 [Recompose](https://github.com/acdlite/recompose)라는 라이브러리가 이미 존재한다.

```js
  // this component will re-render only when props change
  @pure
  class MyComponent extends Component {
    render() {
        ///...
    }
  }
  // this component will re-render only when prop1/prop2 changes
  // it will not re-render if prop3 changes
  @onlyUpdateForKeys(['prop1', 'prop2'])
  class MyComponent extends Component {
    render() {
        ///...
    }
  }
  // if you don't like ES7 decorators you can use them like this:
  MyComponent = pure(MyComponent)
  MyComponent = onlyUpdateForKeys(['prop1', 'prop2'])(MyComponent)
```

## Redux와 connect()

redux를 사용할 때 우리는 고차 컴포넌트인 connect()를 사용한다. 이 컴포넌트는 스토어로부터 컨텍스트를 얻어오고 상태 변화가 있을 때 mapStateToProps를 호출한다. 커넥티드 컴포넌트는 상태 변화가 있을 때 현재 연관된 값이 실제로 변경되었을 때만 재 렌더링된다(마찬가지로 shallowCompare를 이용한다). 예를 들어:

```js
  // only re-renders when prop1 changes
  connect(state => ({
      prop1: state.prop1
  }))(SomeComponent)
```

지금 뭔가 '걸려들었다', 만약 mapStateToProps 함수가 어떤 계산을 한다면 connect()의 불필요한 재 렌더링을 유발할 수 있다.

```js
  // this is ok
  connect(state => ({
      hasSomething: this.prop1 === 5 // true===true
  }))(SomeComponent)

  // this is NOT OK
  // computed data는 바뀌지 않은 경우에도 매번 다른 오브젝트다
  connect(state => ({
      computedData: {
          height: state.height,
          width: state.width
      }
  }))(SomeComponent)
```

### reselect로 이슈 해결하기

[reselect](https://github.com/reactjs/reselect)는 파생상태에 대한 의존을 선언함으로 파생 상태를 캐싱하여 memoize와 비슷하게 우리를 도와줄 수 있다:

```js
  import {createSelector} from 'reselect'
  const selectComputedData = createSelector(
      state => state.height,
      state => state.width,
      (height, width) => ({
          height,
          width
      })
  )
  connect(state => ({
      computedData: selectComputedData(state)
  }))(SomeComponent)
```

## Pure 렌더의 안티패턴

pure 컴포넌트를 사용할 때는 배열과 함수의 사용에 특히 주의해야 한다. 배열과 함수는 새로운 레퍼런스를 생성한다. 따라서 렌더 때마다가 아니라 단 한번만 생성하게 하는 것은 당신에게 달려있다.

### Functions

```js
  // 절대 이렇게 하면 안됨
  render() {
    return <MyInput onChange={this.props.update.bind(this)} />;
  }
  // 절대 이렇게 하면 안됨
  render() {
    return <MyInput onChange={() => this.props.update()} />;
  }
  // 대신 이렇게 해라
  onChange() {
      this.props.doUpdate()
  }
  render() {
    return <MyInput onChange={this.onChange}/>;
  }
```

recompose.withHandlers() 혹은 redux.connect()와 같은 고차 컴포넌트 안에서 바인딩하는 것에 의해 컴포넌트 내에서 바인딩이 일어나는 것 역시 피해야한다.

```js
  // recompose
  @withHandlers({
    onChange: props => event => {
      props.update(event.target.value)
    }
  })
  class SomeComponent extends Component {
    render() {
      return <MyInput onChange={this.props.onChange}/>;
    }
  }

  // redux
  @connect(null, (dispatch, ownProps) => {
    onChange: event => {
      dispatch(actions.updateValue(event.target.value))
    }
  })
  class SomeComponent extends Component {
    render() {
      return <MyInput onChange={this.props.onChange}/>;
    }
  }
```

### Arrays

```js
  // 절대 이렇게 하면 안된다. 아이템이 없는 경우에 SubComponent는 매번 렌더링될 것이다!
  render() {
      return <SubComponent items={this.props.items || []}/>
  }
  // 이렇게 재 렌더링을 피할 수 있다
  const EMPTY_ARRAY = []
  render() {
      return <SubComponent items={this.props.items || EMPTY_ARRAY}/>
  }
```

## 디버깅

render() 메서드에 console.log()를 추가하는 것 외에, 성능 문제의 원인을 찾는 작업은 주로 두 가지 도구를 사용한다.
다음은 간단히 요약한 것이고, 자세한 정보와 스크린샷은 Benchling의 React performance engineering [파트1](http://benchling.engineering/performance-engineering-with-react/), [파트2](http://benchling.engineering/deep-dive-react-perf-debugging/) 블로그 포스트를 통해 얻을 수 있다.

### Chrome 개발자 도구 프로파일러

크롬 개발자 도구에서 타임라인 탭을 선택하고 애플리케이션에서 작업을 수행하는 동안 기록한다. 타임 라인은 브라우저가 코드 실행에 소비한 시간과 렌더링 시간을 보여준다. 여기서 렌더링은 브라우저가 DOM을 화면에 렌더링하는 데 걸린 시간을 의미한다. React의 render() 호출은 코드 실행 시간에 포함된다.

Redux를 사용하는 경우, ReduxDevTools의 슬라이더를 사용하여 느린 재 렌더링을 유발하는 것의로 의심되는 액션만을 반복해보라. 한 번에 하나씩 측정하라.

만약 batchUpdates같은 React 함수가 총 시간은 긴데 자체 시간이 짧다는 것을 관찰하게 된다면, 화면이 너무 자주 렌더링되기 때문에 React 컴포넌트에 성능 문제가 있음을 의미한다.

이런 경우에는 ReactPerf를 사용한다.

### React Perf

React Perf addon을 설치한다.

```
  npm install react-addons-perf
```

프로젝트에 임포트하고 글로벌 컨텍스트에 노출한다:

```js
  import Perf from 'react-addons-perf'
  window.Perf = Perf
```

프로젝트에 설치한 다음에는 [React Perf Chrome Extension](https://chrome.google.com/webstore/detail/react-perf/hacmcodfllhbnekmghgdlplbdnahmhmm)을 통해 성능 관리 기능을 호출하거나, 직접 콘솔에서 호출할 수 있다.

과정은 간단하다:

1.  Perf.start()를 호출한다
2.  애플리케이션 내에서 몇가지 액션을 취한다. 이 내용은 Perf에 의해 기록될 것이다(가능하다면 ReduxDevTools의 슬라이더를 이용하라).
3.  Perf.stop()를 호출한다
4.  이제 액션이 기록되었고 3가지 유용한 함수를 호출할 수 있다:
    printInclusive() — 각 컴포넌트가 얼마나 많은 시간을 소모했는지 출력한다
    printExclusive() — 각 컴포넌트가 렌더링에 얼마나 많은 시간을 소모했는지 출력한다 (componentsWillMount, componentDidMount, props processing… 등은 포함하지 않음)
    printWasted() — 실제로 변경되지 않은 컴포넌트를 렌더링하는 데 낭비된 시간을 출력한다(렌더링이 가상 DOM 계층에서만 수행되고 브라우저 DOM에서는 변경이 발생하지 않은 시간). 이 함수는 위에서 기술한 안티패턴의 인스턴스와 pure 컴포넌트여야만 하는 컴포넌트를 보여줄 것이다.
    printOperations() — 실제 브라우저의 DOM 조작을 출력할 것이다, 브라우저가 너무 많은 시간을 렌더링에 사용한 경우에 편리하다.

## 최적화를 꼭 해야만 하나?

react-dom은 그 자체로 매우 빠르다. React Perf로 정량화 할 수 있는 문제가 있을 때만 최적화를 수행해야 할 것이다.

예를 들어, 다음과 같은 간단한 컴포넌트를 생각해보자:

```js
  const Label = ({text}) => <div className='label'>{text}</div>
```

이 컴포넌트에서 text는 아마 절대 변경되지 않을 것이다. 이 컴포넌트를 pure하게 만드는 것은 옳게 느껴지지만 아마도 눈에 띌만한 향상을 일으키진 못할 것이다. shallowCompare를 실행하면 react-dom이 같은 계산을 실행할 것이므로 컴포넌트를 pure하게 만드는 것은 그저 처리 과정을 다른 곳으로 옮긴 것 뿐이다.

다른 말로 하자면, 미래의 문제를 미리 상상하지 말고 react-dom이 충분히 좋게 느껴지지 않을 때까지 기다려야 한다. 보통은 그렇다.

이 글은 welldone-software, Mordern Software Boutique에 의해 작성되었다. Angular, Node, React, .NET, Cordova, Mobile, Cloud의 전문가인 우리에게 연락 바란다.
