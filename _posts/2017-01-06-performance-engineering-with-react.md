---
id: performance-engineering-with-react
title: React 퍼포먼스 엔지니어링
category: React
---
[원문보기](http://benchling.engineering/performance-engineering-with-react/)

이 포스트는 React 퍼포먼스 엔지니어링 시리즈의 첫 번째 파트다. Part 2 - [A Deep Dive into React Perf Debugging](http://benchling.engineering/deep-dive-react-perf-debugging/) 도 올라왔다!

이 포스트는 복잡한 React 애플리케이션을 작성한 사람들을 위한 것이다. 만약 단순한 것을 작성 중이라면 퍼포먼스에 포커싱하는 일은 별로 필요하지 않을 것이다. 섣부른 최적화는 금물! 만드는 것이 먼저다!

하지만, DNA 디자인 도구나, 젤 형태 이미지 분석 소프트웨어, rich-text 에디터, full-feature 스프레드시트 등을 개발 중이라면 아마도 성능 병목 현상과 마주칠 것이고, 이를 해결해야만 한다. 우리는 Benchling에서 이 성능 병목 현상을 마주쳤고, 우리가 배운 것들 중 일부를 공유하려고 이 포스트를 작성했다. 그래서 Benchling 사람들과 인터넷 상의 사람들을 대상으로 작성했다.(그리고 맞다! 우리는 이런 종류의 문제를 좋아하는 사람을 고용 중이다!)

이 포스트에서는 React의 Perf 도구 이용의 기초, React 렌더링 병목현상의 일반적인 이슈를 다루며, 또한 디버깅 도중 염두해 둘만한 팁들을 다룬다.

## React 기초(Baseline React)

브라우저 성능을 세 문장으료 요약하면: 초당 60프레임으로 렌더링하고 프레임 당 16.7ms를 남겨 두는 것이 이상적이다. 앱이 느리다는 것은 사용자 이벤트에 응답하는 것이 오래 걸리거나, 데이터를 처리는데 시간이 오래 걸리거나, 새 데이터를 다시 렌더링하는 것이 오래 걸리는 것이다. 대다수의 경우에는 데이터를 처리하는 것이 아니라 다시 렌더링하는 것에 시간을 낭비하고 있다.

React를 사용하면 별다른 작업 없이도 즉시 성능향상을 이뤄낼 수 있다.

왜냐하면 React가 모든 DOM 조작을 다루기 때문이다. 따라서 DOM을 파싱하고 레이아웃하는 이슈를 크게 회피할 수 있다. 장막 뒤에서는 React가 자바스크립트 내에서 virtual DOM을 관리하고 있으며, 원하는 상태의 문서를 만들어내는데 필요한 최소한의 변화만을 빠르게 결정하여 사용한다.

왜냐하면 React 컴포넌트의 상태는 자바스크립트에 저장되어 있기 때문에 DOM에 직접 접근하는 것을 피할 수 있다. 고전적인 성능 이슈는 DOM을 부적절한 순간에 접근하기 때문이다. 일반적으로 이런 부적절한 순간 문제란 강제로 layout 동기화 같은 문제(예: someNode.style.left를 읽으면 브라우저는 강제로 프레임을 렌더링한다)다.

다음과 같이 하는 대신에:

```js
someNode.style.left = parseInt(someNode.style.left) + 10 + "px";  
```

우리는 선언적으로 <SomeComponent style={{left: this.state.left}} />과 같이 DOM 상태를 읽지 않고도 컴포넌트가 움직이도록 간단하게 업데이트할 수 있다:

```js
this.setState({left: this.state.left + 10}).  
```

더 명확히 하자면, 이런 최적화는 React 없이도 가능하다 - 하지만 말하고자 하는 바는 바로 React가 이런 문제를 미리 해결하는 경향이 있다는 것이다.

단순한 애플리케이션에서는 이 퍼포먼스 최적화가 React를 사용하는 것만으로 충분하다 - 나는 그것이 선언적 프레임워크가 실현될 수 있는 최소한의 작업이라고 생각한다. 그러나 보다 복잡한 뷰들을 개발하고, 관리하고, virtual DOM을 비교하는 것은 비용이 많이 드는 작업이 될 수 있다. 다행히도, React는 성능 문제가 존재하는 곳을 감지하고 이를 방지하기 위한 수단을 몇 가지 툴을 통해 제공한다.

## 디버깅으로 인한 성능 이슈(Performance issues caused by debugging)

조심! - 디버깅하는 것 자체만으로 오버헤드가 생길 수 있고 제품에서는 생기지도 않는 디버깅 세션의 혼란을 야기할 수도 있다.

### Elements pane

 Elements pane은 어떤 것이 다시 렌더링되는지 보여주는 훌륭하고 단순한 방법이다 - 속성이 변경되거나 갱신/추가/치환되는 DOM node를 깜빡이는 컬러로 보여준다. 그러나 이 깜빡임이 바로 성능에 영향을 준다! 나는 종종 Console pane으로 전환해서 FPS에 대한 정확한 감각을 유지한다.

### PropTypes

개발 빌드의 React에서는 컴포넌트를 렌더링할 때 PropType의 유효성 검사가 일어난다 - 컴포넌트가 전달받는 props를 확인해서 디버깅과 개발을 돕는다. 크롬의 JS 프로파일러를 사용할 때 보면, React component가 validate 메서드에서 가장 많은 시간을 소비하는 것을 볼 수 있을 것이다.

개발 빌드에서 나타나는 경고들은 디버깅시에는 유용하고, 그 코스트는 제품에는 반영되지 않는다. 나는 개발 빌드에서의 느린 반응속도에 대한 잘못된 감각을 무시하기 위해 가끔 React의 제품 빌드로 전환한다. (제품 빌드를 사용하려면 NODE_ENV를 production으로 세팅한다: https://facebook.github.io/react/downloads.html#npm)

## React.addons.Perf와 성능 이슈 식별하기

일반적인 수정사항에 들어가기에 앞서, 측정할 수 있었던 문제에 대해서만 시간을 투자해야 한다는 것을 강조하는 것이 중요하다. 훈련하지 않았다면 어둠 속에서 측정을 마치기 일쑤다 - 다시 말하자면 개발에 주력하고 핵심 성능 병목 현상을 해결하는 데만 시간을 투자하자.

표준적인 디버깅 도구를 이용해서 병목 현상을 식별하는 것은 여전히 가능하지만 도구가 React측 코드에 시간을 소비할 수 있으므로 데이터를 해석하기가 어렵다. (예: 빠르게 실행되도록 작성한 복잡한 렌더 메서드를 사용하면 가상 DOM에 대한 계산 결과가 훨씬 비싸진다.) 그래서 React측에서 가시적인 병목 현상을 유발한 코드가 무엇인지 식별하기 어렵게 된다.

다행히도 React는 React의 개발 빌드에서 사용할 수 있는 몇 가지 perf 도구들과 함께 번들로 제공됩니다. 0.13에서는 `React.addons.Perf`에서 찾을 수 있고 0.14 이상에서는 자체적인 `react-addons-perf` 패키지에서 찾을 수 있다.

### 사용법

Perf를 사용하려면 콘솔에서 Perf.start()를 호출하면 된다. 그리고 나서 기록하고 싶은 행동을 하고, 다시 Perf.stop()을 선언하면 된다. 그리고 나서 다음 메서드들 중 하나를 호출해서 측정값을 출력해서 확인하면 된다.

퍼포먼스 디버깅 모드에서 나는, 간단하게 start/stop 레코딩 버튼을 만들어서 퍼포먼스를 측정한다. (코드는 정말 간단하다 - 컴포넌트를 화면 한 쪽에 놓고 React.addons.Perf를 호출하도록 한다.) React DevTools 처럼 [Chrome Extension](https://github.com/facebook/react-devtools/issues/71)으로 사용할 수도 있다. Jeff가 start/stop에 단축키를 바인드하는 환상적인 팁을 알려줬다.

### Perf.printWasted()

Perf.printWasted()는 가장 유용하다. 최종적으로 DOM 수정이 없는 경우인데도 render 트리를 생성하고 virtual DOM 비교를 하는 작업에 얼마나 많은 시간을 낭비했는지 찾아서 알려준다. 여기에 나타난 컴포넌트는 PureRenderMixin이나 다른 테크닉으로 수정되야할 주요 후보들이다.

### Perf.printInclusive() / Perf.printExclusive()

이 출력 함수들은 컴포넌트를 렌더링 하는데 얼마나 많은 시간이 들었는지를 보여준다. 나는 렌더링 병목현상이 렌더링하지 않음으로 렌더링이 빨라지는 경우에 의해 해결되는 경우가 잦아서 이 함수들의 유용함을 찾지 못했었다. 그러나, 라이프사이클 메서드들 중 컴포넌트의 계산 성능이 많이 요구되는지 찾는 데에 도움이 될 수 있다. 나는 보통 printWasted 이슈를 해결한 후, 내 애플리케이션의 코드가 성능요구가 많다는 것을 알게되었다. 이 시즘에서는 Chrome DevTool의 표준 JS Profiler를 사용하고 가장 비싼 함수 호출이 무엇인지 직접 살펴보는 것이 좋다.

### Perf.printDOM()

Perf.printDOM()은 React tree를 렌더링할 때 발생하는 모든 DOM 연산을 리턴한다. 내 경험상, 정확히 무엇이 일어났는지 설명하는 긴 항목이므로 각 속성 변경과 각각의 DOM 삽입에 대한 해석/시각화가 어렵다. 그리고 만약 애플리케이션이 충분히 복잡하다면 출력 내용은 굉장히 큰 변화로 나타날 것이다.

처음 컴포넌트가 렌더링된 이후에, 향후의 렌더링에서 기존 DOM 노드를 다시 사용하거나 업데이트를 하고 새로운 DOM 노드를 생성하지 않기를 기대하고 결국 이것이 React의 virtual DOM이 제공하는 최적화입니다.

나는 가끔 이 함수를 사용해서 이상한 브라우저 버그를 발견하거나, 예기치 못한 대량의 DOM 수정을 발견했습니다.

## shouldComponentUpdate로 렌더링 피하기

React는 값비싼 DOM 연산을 피하기 위해서 virtual DOM 표현을 유지하는 놀라운 일을 하지만, virtual DOM 표현을 유지하는 것 역시 비용이 많이 든다. 아주 크고 복잡한 렌더 트리를 상상해보자. 만약 어떤 노드의 props라도 갱신하게되면, React는 렌더 트리상의 모든 leaf노드까지 내려가면서 virtual DOM 비교를 위한 계산을 다시 해야 한다. 운 좋게도 React는 이 재 계산을 피할 수 있는 shouldComponentUpdate 라는 이름의 메커니즘을 제공한다. 이 메서드에서 false를 리턴하면 렌더링을 위해 이 컴포넌트의 전체 서브트리를 괴롭히는 일은 하지 않게 된다. 우리는 어떻게/언제 false를 리턴해야 하는지만 알아내면 된다.

이 이점을 취하는 가장 간단한 방법은 render 메서드를 pure하게 유지하는 것이다 - 컴포넌트를 state와 props에만 의존하여 렌더하도록 하는 것(반대로는 DOM을 읽거나, 쿠키 혹은 다른 어떤 것을 읽는 것이다)이다. 이 "pure rendering" 테크닉은 꽤나 자주 언급되곤 하지만 컴포넌트의 존재 이유에 대해 알기 쉽게 하는 좋은 습관이므로 다시 한번 강조해도 무방하다. 그래도 종종 외부에 상태를 갖게 될 때가 있다 - 외부 상태에 의존하는 몇몇 컴포넌트를 독립적으로 유지하고 나머지는 pure하게 유지하도록 노력하라.

이렇게 함으로써 컴포넌트는 [PureRenderMixin](https://facebook.github.io/react/docs/pure-render-mixin.html)을 사용할 수 있다. [소스를 보면](https://github.com/facebook/react/blob/master/src/addons/ReactComponentWithPureRenderMixin.js) mixin은 바로 shallowCompare를 호출한다.(만약 ES6 클래스를 사용하는 경우에는 [shallowCompare](https://facebook.github.io/react/docs/shallow-compare.html)를 직접 사용하는 것도 좋다.)

```js
var ReactComponentWithPureRenderMixin = {  
  shouldComponentUpdate: function(nextProps, nextState) {
    return shallowCompare(this, nextProps, nextState);
  },
};
```

만약 props/state에 변화를 감지하지 않았다면, 다시 렌더링하지 않을 것이다 컴포넌트의 올바른 동작을 위해서, 컴포넌트는 반드시 다음과 같아야한다:

> render() 는 반드시 props와 state에만 의존해야 한다. 즉 어떠한 전역 상태로부터 값을 읽어오는 일이 없어야 한다.
> props와 state는 절대 mutate되서는 안된다 - shallowCompare가 최상위 props에 대해서만 동등성 검사를 하므로, 어떠한 변화라도 반드시 새로운 변수가 생성되어야 한다. [react-addons-update](https://facebook.github.io/react/docs/update.html)이 불변 업데이트를 도와줄 것이다. 또한 `Object.assign`/`_.extend`과 같은 간단한 경우에도 마찬가지다. [ImmutableJS](https://facebook.github.io/immutable-js/)는 더 중대한 변화가 요구되지만 PureRenderMixin을 쉽게 사용할 수 있다. this.state.myItem.stars++와 같은 일을 하고 싶은 충동에 주의하라. 상태를 직접적으로 변경하고 있다는 일은 잊기 쉽고 특히 다른 상태가 변경되면서 변경이 함께 일어나는 경우가 있다.

만약 pure components를 고수한다면 병목 현상을 발견했을때 PureRenderMixin을 사용하기가 훨씬 쉬워진다.

### 작은 유의점

PureRenderMixin을 사용한다면 성능 향상에 대한 잘못된 감각을 가질 수 있습니다 - 이것은 자식 컴포넌트들의 propType 유효성 검사 또한 회피하기 때문이다. 어차피 이 propType 유효성 검사는 제품 빌드에서는 PureRenderMixin 없이도 건너뛰는 것들이다.

### 더 큰 유의점

더욱 엄격한 정책을 고수하더라도, PureRenderMixin의 혜택을 즉시 누리지 못할 수도 있다. 상술한 바와 같이, React는 재 렌더링의 필요성을 결정하기 위해 deep 비교가 아닌 shallow-equal 비교를 수행한다. shallow equal이 아니라 의도치 않게 deep-equal 비교를 해버리는 너무나 많은 방법들이 있다.(나중에 더 설명함)

한가지 빠른 방법은 `_.isEqual`을 사용하는 것이다.

```js
shouldComponentUpdate(nextProps, nextState) {
  return !_.isEqual(this.props, nextProps) ||
    !_.isEqual(this.state, nextState);
}
```

대부분의 props를 재사용한 경우 `_.isEqual`이 처음에 shallow 비교를 하기 때문에, 성능은 괜찮아 보였다. 실제로 `_.isEqual`로 충분한 경우에는 deep compare와 성능상의 이슈를 발견하지는 못했다.

또한 컴포너트에 맞게 재단된 custom shouldComponentUpdate를 작성해도 되지만, 나는 단순한 컴포넌트에만 이를 적용했다. 만약 이 custom 메서드가 적절하게 관리되지 않는다면, 실제로 갱신이 필요한데도 갱신이 되지 않는 경우가 발생한다.

## Optimizing for shallow-equal props

새 객체를 만들지 않는 best practice를 사용하면, 렌더링 최적화에 자연스럽게 도움이 되는 경우가 종종 있다.

### Function.bind() / inline (anonymous) functions

`Function.bind`는 컴포넌트의 메서드를 맥락에 맞게 호출할 수 있는 편리한 방법이다. 불행히도, `Function.bind`의 호출은 새로운 함수를 생성한다:

```js
console.log.bind(null, 'hi') === console.log.bind(null, 'hi')
false
function(){console.log(‘hi');} === function(){console.log(‘hi');}
false
// New function each time
render() {
  return <MyComponent onClick={() => this.setState(...)} />
}
```

prop 검사는 더이상 도움이 되지 않으며 컴포넌트는 항상 다시 렌더링된다. ([react/jsx-no-bind](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-no-bind.md) eslint rule을 통해 bind나 arrow 함수를 jsx의 props로 전달하는 일을 막을 수 있다)

우리가 찾은 가장 간단한 해결방법은 bind 되지않은 함수를 전달하고 필요한 인자를 instance 메서드를 사용해 전달하는 것이다. 예를 들면:

```js
const TodoItem = React.createClass({
  deleteItem() {
    this.props.deleteItem(this.props.index);
  },
});
```
서브 컴포넌트가 index를 되돌려주는 제약사항과 함께 더 general한 메서드를 노출하는 것은 이상한 일이기 때문에, 우리는 id와 같은 인자를 컨텍스트에 바인딩하려는 목적으로 IntermediateBinder를 사용한다. IntermediateBinder는 id를 prop으로 취하고 자체적인 method를 바인딩해서 자식 컴포넌트에 이 바인딩된 메서드를 전달한다.

```js
const React = require('react/addons');

const IntermediateBinder = React.createClass({
  displayName: 'IntermediateBinder',
  propTypes: {
    boundArg: React.PropTypes.any.isRequired,
    children: React.PropTypes.func.isRequired,
  },
  _rebindFns(props, bindAll) {
    const newFns = {};
    for (const name in props) {
      const value = props[name];
      if (name !== 'boundArg' && name !== 'children') {
        if (bindAll || value !== this.props[name]) {
          newFns[name] = value.bind(null, props.boundArg);
        } else {
          newFns[name] = this._boundFns[name];
        }
      }
    }
    this._boundFns = newFns;
  },
  componentWillMount() {
    this._rebindFns(this.props, true);
  },
  componentWillReceiveProps(nextProps) {
    this._rebindFns(nextProps, this.props.boundArg !== nextProps.boundArg);
  },
  render() {
    return this.props.children(this._boundFns);
  },
});

module.exports = IntermediateBinder;
```

이는 다음과 같이 작성하는 것을 허용한다:

```js
<IntermediateBinder
  deleteItem={this.deleteItem}
  boundArg={item.id}
>
  {(boundProps) => <TodoItem deleteItem={boundProps.deleteItem} />}
</IntermediateBinder>
```

(우리가 조사한 또 다른 가능한 방법은, 실제로 변경되지 않은 bind된 함수들을 찾는 더 나은 check 함수와 함수 자체에 메타 데이터를 저장하는 custom bind 함수를 조합해서 사용하는 것이다. 그러나 이는 우리 취향과 명백히 맞지 않았다.)

### 리터럴 array/object 생성

간단하지만 종종 묵과된다. array 리터럴은 종종 PureRenderMixin을 깨트린다.

```js
['important', 'starred'] === ['important', 'starred']
false
```

만약 이 오브젝트가 변경되지 않을 것으로 기대된다면, 모듈의 상수나 컴포넌트의 static 변수로 이동시키면 된다.

```js
const TAGS = ['important', 'starred'];
```

### Subcomponents

Defining content boundaries between a component and its subcomponent often lends itself to easy performance optimizations - well-encapsulated component interfaces lend naturally to performant updates. Refactoring out intermediate components can help improve where you can use PureRenderMixin and save updates:

```js
<div>
  <ComplexForm props={this.props.complexFormProps} />
  <ul>
    <li prop={this.props.items[0]}>item A</li>
    ...1000 items...
  </ul>
</div>
```

In this case, if complexFormProps and items come from the same store, typing in the ComplexForm might lead to store updates, and each store update leads to re-rendering the entire <ul>. Virtual DOM diffing is great, but it still has to check every <li>. Instead, refactor out <ul> into its own subcomponent that takes in this.props.items, and only update if this.props.items changes:

```js
<div>
  <CustomList items={this.props.items} />
  <ComplexForm props={this.props.complexFormProps} />
</div>
```

### Cache expensive computations

This goes against the "single source of state" principle, but if computations on a prop are expensive you can cache them on the component. Instead of directly using doExpensiveComputation(this.prop.someProp) in the render method, we can wrap the call that caches the value if the prop is unchanged:

```js
getCachedExpensiveComputation() {
  if (this._cachedSomeProp !== this.prop.someProp) {
    this._cachedSomeProp = this.prop.someProp;
    this._cachedComputation = doExpensiveComputation(this.prop.someProp);
  }
  return this._cachedComputation;
}
```

Candidates for this optimization would be best discovered using the JS Profiler.

### Link State

React's Two Way Binding Helpers can be very useful for simple inversion of control, allowing a child component to communicate new state to the parent. If only used with valueLink for a React form component, it isn't so bad as the React form inputs are very simple. But if you start threading it through more components like we were doing, you may run into issues. linkState is implemented as follows:

```js
linkState(key) {
  return new ReactLink(
    this.state[key],
    ReactStateSetters.createStateKeySetter(this, key)
  );
}
```

Every call to linkState returns a new object, even if the the state hasn't changed! This means shallowCompare will never work. Our workaround is unfortunately simply not to use a linkState. If you instead flatten the linkState into a getter prop and a setter prop, we avoid creating a new object, e.g. nameLink={this.linkState(‘name')} could be replaced with name={this.state.name} setName={this.setName}. (We've considered writing a linkState that caches itself...)

## Compiler Optimizations

Newer versions of Babel and React support inlining React elements and automatically hoisting constant React elements. We haven't played too much with this yet, unfortunately, but they will help with reducing calls to React.createElement and in speeding up DOM reconciliation, respectively.

## Wrapping Up

We went through a lot just now (you should've seen the original list!), but the key point to take away is that you should 1) get comfortable with profiling and 2) shouldComponentUpdate will get you a long way. We hope this has been useful!

Any suggestions, comments, or things we've missed? Let us know - saif at benchling.com.

Stay tuned for part 2, where we'll discuss our React debugging workflows, dive into real examples of non-performant code, and subsequently fix them.
