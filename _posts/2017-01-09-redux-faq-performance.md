---
id: redux-faq-performance
title: Redux FAQ 성능편
category: Redux
---

[원문보기](http://redux.js.org/docs/faq/Performance.html)

## 목차

- [Redux를 어떻게 성능과 아키텍쳐 측면에서 잘 "확장""할 수 있나요?](#performance-scaling)
- [각 액션에 대해서 "내 모든 리듀서들"을 호출하는 것이 느리지 않나요?](#performance-all-reducers)
- [리듀서에서 내 상태를 deep-clone해야만 하나요? 상태를 복사하는 것으로 인해 느려지지는 않나요?](#performance-clone-state)
- [어떻게 해야 스토어를 업데이트하는 이벤트의 수를 줄일 수 있나요?](#performance-update-events)
- ["하나의 상태 트리"가 메모리 문제를 일으키지 않나요? 많은 액션들을 디스패칭하는 것은 많은 메모리가 필요하지 않나요?](#performance-state-memory)


## 성능

<a id="performance-scaling"></a>
### Redux를 어떻게 성능과 아키텍쳐 측면에서 잘 "확장""할 수 있나요?

이 문제에 대해서 하나의 확실한 대답을 할 수 없지만, 대부분의 경우 이 문제는 염려되지 말아야 한다.

Redux에 의한 작업 완료는 몇 구역으로 나뉘어진다: 미들웨어와 리듀서들에서 액션 처리(immutable한 업데이트를 위한 오브젝트 복제를 포함), 액션이 디스패치되고 나서 구독자들에게 알림 보내기, 상태 변화에 따라 UI 컴포넌트를 업데이트하기. 충분히 복잡한 상황에서는 이들 각각이 성능 우려를 만들만한 *가능성* 이 있지만, Redux가 구현되는 방식에 본질적으로는 느리거나 비효율적인 것은 없다. 사실, React-Redux는 불필요한 재 렌더링을 줄이기 위해 많이 최적화되어 있으며 React-Redux v5는 이전 버전보다 눈에 띄는 개선 사항이 많다.

Redux는 다른 라이브러리와 비교할 때 효율적이지는 않다. React 애플리케이션의 렌더링 성능을 극대화하려면 상태를 정규화 된 모양으로 저장해야 하며, 많은 개별 컴포넌트들을 스토어에 연결해야 하고, 연결된 목록 컴포넌트는 item ID들을 하위 리스트 항목들에게 전달해야 한다.(자신만을 위한 데이터들을 ID를 통해서 찾을 수 있도록) 이렇게 하면 전체 렌더링 양이 최소화된다. 메모이즈드 셀렉터 함수를 사용하는 것도 중요한 성능 고려사항이다.

아키텍쳐에 관해서 일화적인 증거는 Redux가 다양한 프로젝트 및 팀 크기에 대해 잘 작동하고 있다는 것이다. Redux는 현재 NPM에서 수십만 개의 월간 설치로 수천 명의 기업과 수천 명의 개발자가 사용하고 있고, 한 개발자가 다음과 같이 보고했다.

> 규모면에서, 우리는 ~500개의 액션 타입과, 400개의 리듀서 케이스, ~150개의 컴포넌트, 5개의 미들웨어, ~200개의 액션, 2300개의 테스트를 가지고 있다.

#### 더 읽어보기

**Documentation**

- [레시피: 리듀서 구조화 하기 - 상태 모양의 정규화](http://redux.js.org/docs/recipes/reducers/NormalizingStateShape.md)


**Articles**

- [React 애플리케이션을 확장하는 법](https://www.smashingmagazine.com/2016/09/how-to-scale-react-applications/) (연관된 이야기: [React 애플리케이션 확장하기](https://vimeo.com/168648012))
- [고성능 Redux](http://somebody32.github.io/high-performance-redux/)
- [React와 Redux의 성능을 Reselect로 향상시키기](http://blog.rangle.io/react-and-redux-performance-with-reselect/)
- [Redux 상태 트리 캡슐화하기](http://randycoulman.com/blog/2016/09/13/encapsulating-the-redux-state-tree/)
- [React/Redux Links: Performance - Redux](https://github.com/markerikson/react-redux-links/blob/master/react-performance.md#redux-performance)

**Discussions**

- [#310: 누가 Redux를 사용하는가?](https://github.com/reactjs/redux/issues/310)
- [#1751: 대규모 콜렉션에 대한 성능 이슈](https://github.com/reactjs/redux/issues/1751)
- [React Redux #269: 커스텀 구독 메서드를 사용하여 연결하다](https://github.com/reactjs/react-redux/issues/269)
- [React Redux #407: 발전된 API를 제공하도록 connect를 재작성하다](https://github.com/reactjs/react-redux/issues/407)
- [React Redux #416: 더 나은 성능과 확장성을 위해 connect를 재작성하다](https://github.com/reactjs/react-redux/pull/416)
- [Redux vs MobX TodoMVC 벤치마크: #1](https://github.com/mweststrate/redux-todomvc/pull/1)
- [Reddit: 초기상태 저장을 위한 최고의 장소는 어디인가?](https://www.reddit.com/r/reactjs/comments/47m9h5/whats_the_best_place_to_keep_the_initial_state/)
- [Reddit: SPA를 위한 Redux 상태 디자인에 대한 도움말 ](https://www.reddit.com/r/reactjs/comments/48k852/help_designing_redux_state_for_a_single_page/)
- [Reddit: 대규모 상태 오브젝트에 대한 Redux 성능 이슈들?](https://www.reddit.com/r/reactjs/comments/41wdqn/redux_performance_issues_with_a_large_state_object/)
- [Reddit: 초 거대 스케일 앱을 위한 React/Redux](https://www.reddit.com/r/javascript/comments/49box8/reactredux_for_ultra_large_scale_apps/)
- [Twitter: Redux 스케일링](https://twitter.com/NickPresta/status/684058236828266496)
- [Twitter: Redux vs MobX 벤치마크 그래프 - Redux 상태 모양 문제](https://twitter.com/dan_abramov/status/720219615041859584)
- [Stack Overflow: 중첩된 컴포넌트를 위한 props 업데이트는 어떻게 최적화 하는가?](http://stackoverflow.com/questions/37264415/how-to-optimize-small-updates-to-props-of-nested-component-in-react-redux)
- [Chat log: React/Redux perf - 10000개 Todo list를 업데이트하기](https://gist.github.com/markerikson/53735e4eb151bc228d6685eab00f5f85)
- [Chat log: React/Redux perf - single connection vs many connections](https://gist.github.com/markerikson/6056565dd65d1232784bf42b65f8b2ad)

<a id="performance-all-reducers"></a>
### 각각의 액션마다 "내 모든 리듀서"를 호출하는 것은 느리지 않나요?

Redux 스토어는 실제로 하나의 리듀서 함수만을 가진다는 것에 유의하는 것이 중요하다. 스토어는 하나의 리듀서 함수에 현재 상태를 전달하고 액션을 디스패치하하고 리듀서는 이를 적절히 다루도록 하자.

명백히, 모든 가능한 액션을 하나의 함수에서 다루려고 시도하면 함수 크기와 가독성 측면에서 단순하게 확장되지 않으므로, 실제 작업을 최상위 레벨의 리듀서에서 호출할 수 있는 별도의 함수로 분할하는 것이 좋다. 특히, 일반적으로 특정 키를 가진 특정 상태 조각에 대한 업데이트를 관리하는 하위 리듀서 함수를 갖는 패턴을 제안하고 있다. Redux와 함께 제공되는 `combineReducers()`는 이를 달성할 수 있는 많은 방법 중 하나다. 또한 스토어 상태를 가능한 평평하게 정규화 하는 것이 좋다. 궁극적으로, 어떤 방식의 로직으로든 리듀서를 조직할 책임은 당신에게 있다.

그러나, 다른 리듀서 함수가 함께 조합되고 심하게 중첩된 상태를 가지고서도 리듀서의 속도는 문제가 되지 않는다. 자바스크립트 엔진은 초당 매우 많은 수의 함수 호출을 실행할 수 있으며 대부분의 리듀서는 `switch` 문을 사용하고 default에서 기존 state를 반환한다.

만약 정말로 reducer의 성능이 우려된다면 [redux-ignore](https://github.com/omnidan/redux-ignore)나  [reduxr-scoped-reducer](https://github.com/chrisdavies/reduxr-scoped-reducer)를 이용하여 특정 액션에 하나의 리듀서만 반응하도록 할 수 있다. 또한 [redux-log-slow-reducers](https://github.com/michaelcontento/redux-log-slow-reducers)를 이용해 성능 벤치마킹을 할 수 있다.

#### 더 읽어보기

**Discussions**

- [#912: 제안: 액션 필터 유틸리티](https://github.com/reactjs/redux/issues/912)
- [#1303: 빈번한 갱신이 발생하는 대규모 스토어에서 Redux 성능](https://github.com/reactjs/redux/issues/1303)
- [Stack Overflow: Redux 앱의 상태는 리듀서 이름을 가진다](http://stackoverflow.com/questions/35667775/state-in-redux-react-app-has-a-property-with-the-name-of-the-reducer/35674297)
- [Stack Overflow: Redux는 어떻게 깊게 중첩된 모델을 다루는가?](http://stackoverflow.com/questions/34494866/how-does-redux-deals-with-deeply-nested-models/34495397)


<a id="performance-clone-state"></a>
### 리듀서에서 내 상태를 deep-clone해야만 하나요? 상태를 복사하는 것으로 인해 느려지지는 않나요?

Immutabl하게 상태를 갱신하는 것은 일반적으로 deep 카피가 아니라 shallow 카피를 뜻한다. shallow 카피는 딥 카피보다 훨씬 빠르다, 왜나하면 적은 오브젝트와 필드만 카피하고, 포인터를 효과적으로 움직이게 된다.

그러나 영향을 줘야하는 레벨의 중첩까지는 각각 오브젝트를 생성하고 카피해야할 필요가 있다. 특히 비싸지 않아도 되지만, 이는 state를 가능한한 얕게 정규화된 채 유지해야할 또다른 이유다.

> 일반적인 Redux에 대한 오해: state를 deep copy해야 한다. 실제: 내부가 바뀌지 않으면 참조를 동일하게 유지해야 한다.

#### 더 읽어보기

**Documentation**

- [Recipes: 리듀서 구조화 - 선행 컨셉](http://redux.js.org/docs/recipes/reducers/PrerequisiteConcepts.md)
- [Recipes: 리듀서 구조화 - immutable한 업데이트 패턴](http://redux.js.org/docs/recipes/reducers/ImmutableUpdatePatterns.md)

**Discussions**

- [#454: 리듀서에서 커다란 상태를 다루기](https://github.com/reactjs/redux/issues/454)
- [#758: 상태는 왜 mutate될 수 없나?](https://github.com/reactjs/redux/issues/758)
- [#994: 중첩된 entities를 업데이트할 때의 보일러 플레이트는?](https://github.com/reactjs/redux/issues/994)
- [Twitter: 일반적인 오해 - deep cloning](https://twitter.com/dan_abramov/status/688087202312491008)
- [자바스크립트에서 오브젝트 클론하기](http://www.zsoltnagy.eu/cloning-objects-in-javascript/)


<a id="performance-update-events"></a>
### 어떻게 해야 스토어를 업데이트하는 이벤트의 수를 줄일 수 있나요?

리덕스는 각 액션을 성공적으로 디스패치한 다음 구독자들에게 알린다(즉, 스토어에 도달한 액션이 리듀서에서 처리된 경우). 어떤 경우에는, 구독자가 호출된 횟수를 줄이는 것이 유용할 수 있다. 특히, 액션 생성자가 여러 개의 서로 다른 액션을 연속적으로 발송하는 경우에 유용하다.

React를 사용한다면 `ReactDOM.unstable_batchedUpdates()`에 여러개의 동기적인 디스패치를 감싸서 성능을 향상시킬 수 있지만 이 API는 실험적이고 React 릴리즈에서 제거될 수 있으니 많이 의존하지는 말아라. [redux-batched-actions](https://github.com/tshelburne/redux-batched-actions) (여러 개의 액션을 하나의 액션으로 감싸고 리듀서에서 그것들을 "unpack"하는 고차 리듀서), [redux-batched-subscribe](https://github.com/tappleby/redux-batched-subscribe) (여러번 디스패치되는 호출을 디바운스하게 하는 스토어 인핸서), 혹은 [redux-batch](https://github.com/manaflair/redux-batch) (여러 개의 액션들을 하나의 구독자 알림으로 다루는 스토어 인핸서)을 참고하라.

#### 더 읽어보기

**Discussions**

- [#125: 계단식 렌더를 피하는 전략](https://github.com/reactjs/redux/issues/125)
- [#542: 아이디어: 액션 일괄수행하기](https://github.com/reactjs/redux/issues/542)
- [#911: 액션 일괄수행하기](https://github.com/reactjs/redux/issues/911)
- [#1813: 배열 디스패칭을 지원하는 루프를 사용하라 ](https://github.com/reactjs/redux/pull/1813)
- [React Redux #263: 수백개의 액션을 수행할 때의 커다란 성능 이슈](https://github.com/reactjs/react-redux/issues/263)

**Libraries**

- [Redux Addons Catalog: Store - Change Subscriptions](https://github.com/markerikson/redux-ecosystem-links/blob/master/store.md#store-change-subscriptions)


<a id="performance-state-memory"></a>
### "하나의 상태 트리"가 메모리 문제를 일으키지 않나요? 많은 액션들을 디스패칭하는 것은 많은 메모리가 필요하지 않나요?

우선 메모리 사용 측면에서 Redux는 다른 자바스크립트 라이브러리와 다르지 않다. 유일한 차이점은 Backbone처럼 다양한 독립적인 모델 인스턴스를 저장하는 것이 아니라, 여러 오브젝트 레퍼런스가 중첩되어 하나의 트리를 이룬다는 것이다. 두번째로는 전형적인 Redux 앱은 Backbone 앱에 비해 *적은* 메모리를 사용할 것이라는 것이다. 왜냐하면 Redux는 모델이나 컬렉션의 인스턴스를 생성하기 보다는 plain 자바스크립트 오브젝트와 배열을 사용하는 것을 독려하기 때문이다. 마지막으로, Redux는 한 번에 하나의 상태 트리 레퍼런스만을 가지고 있다. 트리 내에서 더 이상 레퍼런스되지 않는 오브젝트들은 보통 가비지 콜렉션 대상이 된다.

Redux는 액션 자체에 대한 기록을 저장하지 않는다. Redux DevTools는 리플레이할 수 있도록 액션들을 저장하지만 일반적으로 Redux DevTools는 제품 모드가 아닌 개발 모드에서만 활성화한다.

#### 더 읽어보기

**Documentation**

- [Docs: 비동기 액션들 ](http://redux.js.org/docs/advanced/AsyncActions.md)

**Discussions**

- [Stack Overflow: Redux가 메모리를 해제하기 위해서 상태를 커밋하는 다른 방법이 있나?](http://stackoverflow.com/questions/35627553/is-there-any-way-to-commit-the-state-in-redux-to-free-memory/35634004)
- [Reddit: 초기 상태를 유지할만한 최고의 장소는 어디인가?](https://www.reddit.com/r/reactjs/comments/47m9h5/whats_the_best_place_to_keep_the_initial_state/)
