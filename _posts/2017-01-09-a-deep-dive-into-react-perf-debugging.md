---
layout: post
id: a-deep-dive-into-react-perf-debugging
title: React 성능 디버깅 심층 연구
tags: [React]
---

[원문보기](http://benchling.engineering/deep-dive-react-perf-debugging/)

2개의 시리즈물인 React 성능 엔지니어링의 두 번째 파트다. 파트 1에서 우리는 React 성능 툴을 어떻게 쓰는지 살펴보았고, React의 일반적인 렌더링 병목현상과 몇 가지 디버깅 팁을 알아보았다. 아직 안봤다면 한 번 살펴보라!

파트 2에서는 디버깅 워크플로우에 깊게 들어가볼 것이다 - 이 모든 아이디어들을 어떻게 실제로 실행 할 수 있을까? 실생활에서 영향받은 몇 가지 예제와 Chrome의 devtools를 이용해서 성능을 진단하고 수정해볼 것이다. (다 읽은 뒤에 어떠한 제안이나 추가할만한 사항이 있다면 알려달라!)

우리는 다음 샘플 코드를 참조할 것이다 - 간단한 todo list를 React로 렌더링하는 것이다. JS fiddle 스니펫에서 "Result"를 클릭하면 성능 개선을 완료한 인터랙티브 버전을 볼 수 있다. 우리느 업데이트된 JS fiddle을 가지고 포스트할 것이다.

<script async src="//jsfiddle.net/kLv241qz/3/embed/"></script>

## Case Study #1: TodoList

위의 TodoList를 가지고 시작해보자. 최적화되지 않은 코드에서 타이핑을 해보면 얼마나 느린지 알 수 있을 것이다.

Chrome dev tools을 실행시켜 브라우저가 하는 일에 대한 상세한 프로파일링을 해주는 Timeline profiler를 시작하자: 사용자 이벤트를 핸들링하고, JS를 실행하고, 렌더링하고 페인팅하는 것 등. input에 한 글자만 타이핑하고 timeline profiler를 멈추자. 아직까지는 느린 것을 확인할 수 있다. 한 글자만 입력했기 때문이다. 하지만 이게 프로파일링에 필요한 최소 정보량을 생성하는 가장 빠른 방법이다.

![](http://benchling.engineering/content/images/2016/02/0.png)

textInput의 긴 막대를 보면 121.10ms가 스크립팅(Children)에 들어간 것을 보자. timeline 프로파일러는 느린 이유가 스타일링이나/레이아웃 재계산때문이 아니라 스크립팅 때문임을 나타낸다.

그래서 scripting으로 내려가본다. Profiles 탭으로 가자 - 타임라인은 브라우저 전반에 걸쳐 프로파일링을 하지만 Profiles 탭은 JS측에 특화된 조금 다른 모습의 시각화 툴이다. 우리 앱이 아닌 다른 앱에 대한 프로파일을 기록한 것이다:

![](http://benchling.engineering/content/images/2016/02/IipbV.png)

Heaby (Bottom Up) 부분을 보면, React의 batchedUpdates가 대부분의 시간을 차지했음을 알 수 있다. 이는 React 측에 문제가 있음을 확실히 보여주는 것이다. 반대로, function 내에서 Self 측정된 시간을 보면 child functions에서 시간은 제외되었다 - sort by Self는 어떤 특정 값비싼 functions들이 있는지 살펴볼 수 있다. 사용자측의 함수에는 특별한 병목이 없는 것 처럼 보인다. 그러므로 이제 React Perf 툴을 시도해보자.

느린 액션에 대한 측정값들을 생성하기 위해 콘솔에서 React.addons.Perf.start()를 호출한 다음 글자를 쳐서 느린 액션을 실행할 것이다. 그리고 나서 React.addons.Perf.stop()을 실행하여 측정을 마친다. 필요없는 시간 낭비가 있었는지 React.addons.Perf.printWasted()를 실행시켜서 볼 수 있다:

![](http://benchling.engineering/content/images/2016/02/2.png)

첫 번째 아이템은 Todos에 의해 렌더된 TodoItem을 나타낸다; 그러나 Pref.printWasted()는 렌더 트리를 리빌딩하지 않으면 100ms의 시간을 절약할 수 있다는 것을 알려준다. 최적화가 가장 필요한 후보처럼 보인다.

왜 TodoItem이 이렇게 많은 시간을 낭비하는지 진단하려면 WhyDidYouUpdateMixin이라는 커스텀 Minxin을 사용한다. 컴포넌트와 log를 후킹하여 update가 어디에서 왜 일어났는지 확인한다. 다음 코드를 보고 필요에 따라 적용하라:

```js
/* eslint-disable no-console */
import _ from 'underscore';

/*
Drop this mixin into a component that wastes time according to Perf.getWastedTime() to find
out what state/props should be preserved. Once it says "Update avoidable!" for {state, props},
you should be able to drop in React.addons.PureRenderMixin
React.createClass {
  mixins: [WhyDidYouUpdateMixin]
}
*/
function isRequiredUpdateObject(o) {
  return Array.isArray(o) || (o && o.constructor === Object.prototype.constructor);
}

function deepDiff(o1, o2, p) {
  const notify = (status) => {
    console.warn('Update %s', status);
    console.log('%cbefore', 'font-weight: bold', o1);
    console.log('%cafter ', 'font-weight: bold', o2);
  };
  if (!_.isEqual(o1, o2)) {
    console.group(p);
    if ([o1, o2].every(_.isFunction)) {
      notify('avoidable?');
    } else if (![o1, o2].every(isRequiredUpdateObject)) {
      notify('required.');
    } else {
      const keys = _.union(_.keys(o1), _.keys(o2));
      for (const key of keys) {
        deepDiff(o1[key], o2[key], key);
      }
    }
    console.groupEnd();
  } else if (o1 !== o2) {
    console.group(p);
    notify('avoidable!');
    if (_.isObject(o1) && _.isObject(o2)) {
      const keys = _.union(_.keys(o1), _.keys(o2));
      for (const key of keys) {
        deepDiff(o1[key], o2[key], key);
      }
    }
    console.groupEnd();
  }
}

const WhyDidYouUpdateMixin = {
  componentDidUpdate(prevProps, prevState) {
    deepDiff({props: prevProps, state: prevState},
             {props: this.props, state: this.state},
             this.constructor.displayName);
  },
};

export default WhyDidYouUpdateMixin;
```

TodoItem에 이 믹스인을 추가하고, 어떻게 되는지 봤다:

![](http://benchling.engineering/content/images/2016/02/3.png)

아하! tags가 이전과 이후에 동일한 것을 알았다 - mixin이 말하기를 동일한 객체는 아니지만 내용은 같은 객체라고 한다. 또 한편, 두 함수가 동일하다는 것을 알아내기는 힘들다. Function.bind가 생성한 새로운 펑션이 같은 아규먼트로 바인딩되었기 때문이다. 이것들은 유용한 단서가 된다 - 우리기 앞서 어떻게 tags와 deleteItem을 전달했는지 보니, TodoItem이 생성될 때마다 새로운 값을 전달했던 것 처럼 보인다.

만약 바인드되지 않은 함수를 TodoItem 전달한다면, 그리고 tags를 상수로 저장한다면 이런 문제를 피할 수 있다:

<script src="https://gist.github.com/joshma/8c0b2a3b60844efea2d5.js"></script>

WhyDidYouUpdateMixin은 prevProps와 new props가 shallow equal함을 보인다. PureRenderMixin을 사용하면 이런 상황에서 업데이트를 건너뛸 수 있따.

profiler를 다시 해보니 35ms 정도만 걸렸음을 알 수 있다(이전 속도의 4배):
When we run the profiler again, we see that it only takes about 35ms (4x faster than before):

![](http://benchling.engineering/content/images/2016/02/hSPOL.png)

조금 나아졌지만 아직 이상적인 상황은 아니다. input에 타이핑하는 것은 오버헤드가 되어선 안된다. 우리는 여전히 O(목록에 있는 항목 수)로 동작하고 있다. 단순히 상수를 줄이기만 했기 때문에 각 항목에 대해 shallow compare 수행이 필요하다.

1000개의 항목을 추가하는 극단적인 상황을 가정했을때, 30ms가 적당한 속도라고 가정하자. 그러나 수천의 항목이 예상되는 경우에는 60fps(프레임당 16ms - 눈에 띄는 지연시간)가 이상적이다.

컴포넌트를 쪼개서 여러개로 나누는 것은 합리적인 다음 단계라고 볼 수 있다(유효한 첫 번째 단계이다). Todos 컴포넌트가 2개로 나눠질만한 서브컴포넌트로 이루어져 있음을 알 수 있다. AddTaskForm 컴포넌트는 input과 버튼을 포함하고, TodoItems 컴포넌트는 항목들에 대한 목록을 포함한다.

각각의 리팩토링은 지속적인 성능 향상을 기대할 수 있다:

* 만약 PureRenderMixin을 이용해서 TodoItems를 생성한다면 각 아이템들에 대해 다시 랜더링을 하는 O(n) 작업을 피할 수 있다. prevProps.items === this.props.items와 같은 작업을 하기 때문이다.
* 만약 AddTaskForm 컴포넌트를 생성하고 입력된 text에 대한 상태를 유지하자. 텍스트가 변경되어도 Todos 컴포넌트는 다시 랜더링되지 않는다.(O(n) 렌더링 작업을 피할 수 있다)

두 개를 합쳐서 키 입력당 10ms의 속도를 달성했다!

## Case Study #2:

시나리오: 한 사용자가 너무 많은(3000개 이상) 태스크를 가지고 있다면 경고를 렌더링하기를 원하고, 또한 각 todo 항목이 각각 배경색을 가질 수 있도록 하고싶다.

구현:

* todo list 예제의 TodoItems 구현과 비슷하게 구현한다 - 이 예제에서는 input의 text를 최상위 컴포넌트의 state로 저장한다.
* 태스크의 개수에 따라 메시지를 렌더하는 TaskWarning 컴포넌트를 생성한다. 컴포넌트 내에 로직을 캡슐화하기 위해, 렌더링을 하지 않아야 할 때는 null을 리턴하도록 한다.
* div:nth-child(even)에 대해 회색 배경색을 가지도록 CSS를 추가한다.

관찰: input에 빠르게 타아핑하면 page가 약간 렉이 걸린다(3000개 미만에서). 만약 이 상태에서 3000번째 항목을 추가하면 렉이 사라진다. 놀랍게도 더 많은 작업을 추가하니 문제가 해결된 것이다!

디버깅: timeline 프로파일이 뭔가 매우 흥미로운 것을 보여준다:

![](http://benchling.engineering/content/images/2016/02/6.png)

어떤 이유에서인지 한 글자를 타이핑하기만 해도 30ms나 잡아먹는 큰 규모의 스타일 재계산이 일어난다(이것이 30ms/글자수 이상의 속도로 타이핑할 때 렉이 나타나는 이유다, [jank](http://jankfree.org/)를 통해 관찰했다)

위 이미지의 아래쪽에서 나타난 First invalidated 섹션을 보라. Danger.dangerouslyReplaceNodeWithMarkup이 레이아웃 invalidation을 유발하는 것을 알 수 있다. 그리고 이것이 스타일 재계산으로 인도한다. `react-with-addons.js:2301:`를 보라

```js
oldChild.parentNode.replaceChild(newChild, oldChild);
```

어떤 이유로 React는 DOM 노드를 완전히 새로운 DOM 노드로 갈아치운다! DOM 조작은 매우 값비싸다는 것을 상기시켜보자. Perf.printDOM()을 사용해서 React의 DOM 조작을 알아볼 수 있다:

어트리뷰트의 업데이트는 TaskWarning이 없을 때 input에 abc를 타이핑하는 것을 반영한 것이다. 그러나, 동일한 virtual DOM임 것처럼 보이는데도 React가 DOM을 터치하기로 결정하여 DOM 노드의 치환이 나타난다.

밝혀졌듯이, React(<=v0.13)는 noscript 태그를 사용하여 "no component"를 렌더링하지만 두 개의 noscript 태그를 같지 않은 것으로 잘못 처리한다. noscript는 불필요하게 다른 noscript로 교체된다. *또한* 회색 배경의 모든 div 들을 생각해보라. CSS 때문에 3000 개의 항목 노드 중 하나의 렌더링은 그 앞의 형제 노드에 종속적이가 된다. noscript 태그를 바꿀 때마다 후속 DOM 노드의 스타일이 전부 다시 계산된다.

이 이슈를 해결하기 위해 다음과 같이 한다:
* TaskWarning은 empty div를 리턴한다
* TaskWarning 컴포넌트를 이동시켜서 div 내에서 해당 노드의 CSS selector에 영향을 미치지 않도록 한다.
* React를 업그레이드 한다 :-)

하지만 이것은 요점에서 빗나갔다. 중요한 것은 timeline 프로파일러에서 직접 진단할수 있다는 것이다.

## 결론

React 성능 이슈가 dev tool에서 어떻게 나타나는지 보여주는데 유용했으면 좋겠다 - Timeline과 Profiles, React의 Perf 툴의 조합해서 먼 길을 떠나자.

몇천개의 항목과 임의적 컬러링이 있는 Todo list가 인위적으로 보일수도 있지만, 실제로 개발 도중 커다란 문서나 스프레드시트를 렌더링할 때 매우 비슷한 문제와 맞닥뜨린 적이 있다. 그리고 맞다, 우리 팀은 아직 성장하고 있고 복잡한 React 앱이 당신을 흥분시킨다면 연락바란다.
