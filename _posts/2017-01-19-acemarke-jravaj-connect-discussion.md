---
id: acemarke-jravaj-connect-discussion
title: connected 컴포넌트에 대한 토론
category: Redux
---
[원문보기](https://gist.github.com/markerikson/6056565dd65d1232784bf42b65f8b2ad)

**May 12, 2016**

[8:23 PM] **jrajav**:: 헤이, <Provider> 컴포넌트가 실제로 어떤 기능인지 궁금해. 완벽히 이해하려는 건 아니고 - 왜 `store.subscribe( () => render( <App state={store.getState()} />, document.getElementById('root') ) )`처럼 그냥 전달하지 않지?

[8:24 PM] **jrajav**:: dispatch 함수도 전달하고

[8:24 PM] **jrajav**:: 다음엔 각 하위 컴포넌트에 상태를 적절하게 나누어서 분할하여 전달하는 거지

[8:25 PM] **jrajav**:: 만약 모든 컴포넌트가 순수한, stateless 함수형 컴포넌트라면 이 접근방법은 여전히 성능이 좋아야만 하겠지. 안그래?

[9:58 PM] **acemarke**: `@jrajav` : 관용적인 Redux 사용법은 약간의 의존성 주입이 일어나. 이상적으로는 컴포넌트와 액션 생성자가 직접 store를 참조하는 일은 없어
[9:58 PM] **acemarke**: 그게 코드를 더 재사용할 수 있게 하고 테스트 하기도 쉽지

[9:59 PM] **acemarke**: 이걸 봐바 http://redux.js.org/docs/FAQ.html#store-setup-multiple-stores

[10:00 PM] **acemarke**: 또한 네가 작성한 스니펫은 App 컴포넌트가 모든 필요한 상태를 아래로 전달하는 것 처럼 보여  
[10:00 PM] **acemarke**: 만약 상태가 하나의 탑레벨 컴포넌트에 둘 수 있을 정도로 작으면 아마도 Redux가 필요할 것 같지도 않네 - 그냥 React의 컴포넌트 상태를 사용해

[10:01 PM] **acemarke**: 하지만, React Redux에서는 <Provider> 안쪽의 어떤 컴포넌트라도 필요한 state의 조각을 구독할 수 있어. 하위 컴포넌트들이 필요한 데이터를 알고있는 탑레벨 컴포넌트가 없더라도 말이지

[10:25 PM] **jrajav**:: 맞아 `@acemarke`, 근데 한편으론 어떤 컴포넌트 트리에서 어떤 데이터를 필요로하는지 알 수 없잖아. 깊이 중첩된 컴포넌트에서 마술처럼 상태조각을 가져가버리는 건 덜 기능적이고 덜 관리가능해 보여

[10:26 PM] **jrajav**:: 만약 중첩된 컨테이너를 사용해서 상태를 구독해버리면 말이야

[11:56 PM] **acemarke**: 어.. "마술"이란건 없는데

[11:57 PM] **acemarke**: connect()를 사용해서 정의된 어떤 컨테이너 클래스라도 필요한 상태 조각을 명확하게 지정할 수 있어

[11:57 PM] **acemarke**: 여기 전형적인 예제가 있어

[11:57 PM] **acemarke**: 상태 안에 여러명의 users가 있고

[11:58 PM] **acemarke**: 탑 레벨의 프레젠테이셔널 컴포넌트를 가지고 있고, LeftSidebar와 RightMainPanel이라는 두개의 프레젠테이셔널 컴포넌트를 렌더링한다고 하자

[11:58 PM] **Francois Ward**: I think he means that looking at your folder structure, without grepping, stuff can come from anywhere.

[11:58 PM] **acemarke**: LeftSidebar에서는 connected 컴포넌트인 UsersList를 렌더링한다고 하자

[11:58 PM] **acemarke**: UserItems의 목록을 표시하는 List지

[11:58 PM] **acemarke**: 아마도 이름이나 몇몇 세부사항을 표시하겠지

[11:58 PM] **Francois Ward**: which is a pretty common criticism of that model. Cyclejs/Elm do it very much the "everything starts from the top and gets wired down". Its a compromise of pragmatism vs ease of following/purity/testability.(edited)

[11:59 PM] **acemarke**: 그러고나서, 만약 UserListItem을 선택하기 위해 클릭하면, RightMainPanel에 UserDetails 컴포넌트가 보여지길 바랄거야

[11:59 PM] **acemarke**: 역시 connected 컴포넌트지 user에 대한 세부 사항을 보는

**May 13, 2016**

[12:00 AM] **acemarke**: 하지만 TopLevelComponentㄱ LeftSidebar와 RightMainPanel을 렌더링하지만 그 세부사항에 대해 알 필요는 없어

[12:00 AM] **acemarke**: 만약 모든 데이터에 대한 책임을 탑레벨 컴포넌트에게 지우고 싶다면 그럴 수 있어

[12:01 AM] **acemarke**: 하지만 이 시점에서 일반적으로 인정되는 모범사례는 컴포넌트 트리의 하위 부분에 여러 개의 연결 포인트가 있는 것이야 데이터가 실제 필요한 부분에 가깝게 말이지

[12:02 AM] **acemarke**: 원래 질문에 답하자면 Provider를 사용하는 이유는 하나의 탑레벨 컴포넌트에만 연결 포인트를 두고 그 아래는 pure한 stateless 컴포넌트로 두는 것에 반대하는 것이지

[12:03 AM] **acemarke**: 오 그리고 `@jrajav` : 네 질문에 대한 특정한 대답인데, 그게 더 성능이 좋지는 않아

[12:03 AM] **acemarke**: 하나의 탑 컴포넌트를 연결하는 것은 모든 스토어 변경사항에 대해서 재 렌더링을 할거야

[12:03 AM] **acemarke**: 그리고 그 하위의 모든 stateless 컴포넌트도 재 렌더링할거야

[12:04 AM] **acemarke**: 하위 컴포넌트를 여러개 연결하는 것은 서브렌더 트리를 더 작게 만들지

[12:04 AM] **acemarke**: 그리고 React Redux는 얼마나 많은 연결된 컴포넌트가 실제로 재 렌더링해야하는지에 대한 최적화로 많은 작업들을 해

[12:19 AM] **jrajav**:: 맞아 `@Francois Ward` 그게 내가 뜻한 바야

[12:20 AM] **jrajav**:: connect() 자체의 "마술" 부분이지, 임의의 장소에서 상태를 가져와서 노출시키는 것 말야

[12:21 AM] **jrajav**:: 그리고 @acemarke 설명해줘서 고마워 - 재 렌더링이 뜻하는 바를 정확히 물어봐도 될까?

[12:22 AM] **jrajav**:: 네가 말하는 바가 React가 실제로 light DOM에 모든 싱글 컴포넌트를 재 렌더링한다는거야?

[12:22 AM] **jrajav**:: 아니면 그냥 shouldComponentUpdate에 의해 "숏-서킷"되지 않고 render()함수가 재실행 된다는 거야?
[12:22 AM] **acemarke**: ...반대로?

[12:23 AM] **acemarke**: 만약 네가 mapStateToProps 함수가 state.some.very.nested.field를 파야한다고 걱정되면 "selector"를 이용해서 상태 트리의 특정 필요 부분을 가져오는 식으로 캡슐화하면 돼
[12:24 AM] **jrajav**:: 아냐, 나는 React + redux + immutable.js가 더 걱정돼 그리고 상태 오브젝트 전체를 <App /> 컴포넌트에 전달하는 패턴이 하나의 상태 필드만 고쳤을 때도 전체 <App />을 DOM에 재 렌더링하는 결과를 내는 것이 더 걱정돼

[12:24 AM] **jrajav**:: 혹은 React의 virtual DOM diff를 대부분의 파트에서 실행한다거나 하는 것 말야

[12:25 AM] **jrajav**:: diffing이 어떤식으로 일어나는지는 정확히는 모르지만 말야

[12:27 AM] **jrajav**:: 예를들어 <Page1 />과 <Page2 />가 있다고 하고 전체 상태 오브젝트의 서브 프로퍼티인 pageOne을 1로 pageTwo를 2로 전달하려고 해. pageTwo의 중첩된 몇몇 서브 프로퍼티가 변경되서 <Page2 />가 바뀌고, 재 렌더링된다고 하자. pageOne의 어떤 것도 바뀌지 않았기 때문에 <Page1 />은 같은 파라미터와 함께 호출되서 같은 virtual DOM 트리로 결과가 나타나지

[12:28 AM] **jrajav**:: 나는 모든 React가 최적화되지 않았기 때문에 피할수 있는 모든 render()가 호출된다고는 생각하지 않아 (맞지?) stateless functional 컴포넌트들 말야

[12:28 AM] **jrajav**:: 하지만 최소한, virtual DOM diffing의 결과에 따라 <Page1 />의 light DOM은 아무것도 갱신되지 않겠지.. 맞지?

[12:29 AM] **acemarke**: render 함수는 재실행되고 그에 맞는 virtual DOM도 re-diffed 될거야

[12:29 AM] **acemarke**: 주어진 컴포넌트의 인풋이 아무것도 변경되지 않더라도, 즉 헛된 노력이지

[12:30 AM] **jrajav**:: 맞아 - 그래도 DOM 업데이트에 비해서 값싼 것이지

[12:30 AM] **jrajav**:: memoizing 래퍼에 의해 잠재적으로 해결될 수 있고

[12:30 AM] **acemarke**: 아 잠깐, 네가 쓴걸 좀 읽어볼게

[12:30 AM] **acemarke**: (wifi를 실행시켜서 약간 연결이 끊기네)

[12:30 AM] **jrajav**:: 괜찮음

[12:31 AM] **jrajav**:: 그리고 더 명확히 하자면 light DOM은 "actual DOM"을 이야기한거야

[12:31 AM] **jrajav**:: (주로 Polymer로 작업할때 shadow DOM 때문에 light DOM이라고 말하곤 하지)

[12:32 AM] **acemarke**: 이론적으론 맞아

[12:32 AM] **acemarke**: 하지만 추가적으로 알려줄게 있어

[12:32 AM] **acemarke**: connect()가 여러가지 편리하게 해주는 것들이 있는데

[12:32 AM] **acemarke**: 다른 것들 중에서도, 그건 shallow equality check를 하지 mapStateToProps가 리턴한 오브젝트에 대해서 이전에 뭘 리턴했는지랑 말야

[12:33 AM] **acemarke**: 그래서 컴포넌트가 MSTP로부터 같은 내용을 연달아 리턴하게 된다면 재 렌더링을 넘어갈 수 있어 포장된 컴포넌트의 재 렌더링 말야

[12:34 AM] **acemarke**: 위쪽에 말야

[12:34 AM] **acemarke**: 네 Page1/Page2 예제에서

[12:34 AM] **jrajav**:: 아 그래.

[12:35 AM] **acemarke**: connect()를 사용하지 않았다고 가정하면, 그리고 직접 shouldComponentUpdate를 구현하지 않았다고 하면, Page2에 관한 상태 트리의 아주 작은 부분이라도 변한다면 Page1 Page2 양쪽과 그 모든 하위 컴포넌트에 대해 적어도 render / diff 사이클을 유발할거야

[12:36 AM] **jrajav**:: shouldComponentUpdate의 자체 구현에 의한게 네가 말한 memoizing 래퍼 컴포넌트 맞지?

[12:36 AM] **jrajav**:: 만약 서브컴포넌트를 memoize했다면 VirtualDOM에 대한 diffing을 피할 수 있지?

[12:39 AM] **acemarke**: data->props 관리하는 것을 추가적으로 해주기 때문에 React Redux의 connect()는 사용하는 것만으로 좋은 성능 향상이 일어나지

[12:39 AM] **acemarke**: 아.. 아니 보통 SCU는 prop 동등성 체크를 하지

12:41 AM] **jrajav**:: 음 shallow comparison은 보통 memoizing 래퍼들이 할 건데

[12:41 AM] **jrajav**:: 난 shouldComponentUpdate가 render()가 같은 컴포넌트를 리턴하는 것과 같은 방식으로 동작하는 건지 알지 못하겠네

[12:41 AM] **acemarke**: 일반적으로 SCU는 this.props와 nextProps에 대해 shallow 동등성 체크를 해

[12:41 AM] **acemarke**: 이상적으로는

[12:41 AM] **acemarke**: 메모이제이션은 안하지

[12:42 AM] **acemarke**: SCU는 컴포넌트를 리턴하지 않아

[12:42 AM] **acemarke**: boolean을 리턴하지

[12:42 AM] **jrajav**:: 그럼 stateless functional 컴포넌트는 중복을 막지 못한다는거야?

[12:42 AM] **acemarke**: 그건 말 그대로 "이봐 컴포넌트, 여기 네 props가 이렇게 들어왔어. 재 렌더링하길 원해? 예/아니오"

[12:43 AM] **acemarke**:

```js
shouldComponentUpdate(nextProps) {
    // single-field example.  extrapolate for more props, or do all of them
    return this.props.someField !== nextProps.someField;
}
```

[12:43 AM] **acemarke**: 아니

[12:44 AM] **acemarke**: 네가 React 라이프사이클 메서드를 원한다면 React.createClass()나 MyClass extends React.Component를 사용해야해

[12:44 AM] **acemarke**: 지금으로써는 functional stateless components는 간편하고 명확할 뿐이지

[12:45 AM] **acemarke**: 어떤 성능 최적화도 하지 않아(문서에서는 곧 할거라고는 하지만), 효과적으로 재 렌더링이 일어나지 않게할 방법이 없으므로 성능 손실이 발생하지

[12:46 AM] **acemarke**: 즉, connect()에 어떤 타입이라도 전달한다면 이득을 볼수 있다는 거지

[12:47 AM] **acemarke**: 왜냐면 그건 네 "진짜" 컴포넌트를 랩핑하는 것이니까

[12:47 AM] **jrajav**:: 이론적으로는 상태를 추출하지 않아도 connect()를 사용할 수 있다는거네?

[12:48 AM] **jrajav**:: 그리고 그건 SCU의 shallow 프로퍼티 최적화(Immutable로 충분한)를 해줄 거라는 거고

[12:48 AM] **jrajav**:: 혹은 내 고차 컴포넌트 함수를 비슷하게 써도 되고

[12:49 AM] **jrajav**:: 하지만 stateless function에 대해서는 최적화 방법이 없다는 거지

[12:50 AM] **acemarke**: 어떤 도움이 된건가?

[12:50 AM] **jrajav**:: 아직 명확하지 않은 것 한 가지가 있어. 모든 것을 virtual DOM으로 재 렌더링하는 비용을 제거하고 싶어

[12:50 AM] **jrajav**:: virtual DOM diffing은 실제 DOM을 업데이트하는 것을 대부분 막아주지 맞지?

[12:51 AM] **jrajav**:: 그래서 <Page1 />의 어떤 실제 DOM 부분도 절대 변하지 않는다는 거지? 최적화되지 않은 stateless 컴포넌트라도 예를 들자면말야

[1:00 AM] **acemarke**: 기본적으로, 맞아

[1:02 AM] **acemarke**: 맞아 확실해

[1:02 AM] **acemarke**: 아무 필요없는 비교를 하는 데 꽤 많은 CPU 싸이클을 낭비하겠지

[1:03 AM] **acemarke**: 컴포넌트가 몇개 없거나 상대적으로 Redux Store의 크기가 작다면 신경쓸 필요 없어

[1:04 AM] **acemarke**: 이전 시대의 격언인 "일단 동작하게 하고, 그다음 올바르게 만들고, 그다음 빠르게 하라"는 말로 충분하지

[1:06 AM] **acemarke**: 반대로 꽤 많은 컴포넌트가 필요하다는 것을 알고 있고, 액션도 많다면(특히 controlled input을 store를 업데이트하는 데 사용한다면), 정말 이런 낭비를 막을 필요가 있을거야

[1:07 AM] **acemarke**: 잘 시간이 지났네. 도움이 됐길 바라 :)

[1:07 AM] **jrajav**:: 그래, React가 stateless 컴포넌트를 최적화 할때까진 connect()나 memoizing 레이어를 이용해야 한다는 거네

[1:07 AM] **jrajav**:: 맞아 엄청 도움됐어 `@acemarke`

[1:07 AM] **jrajav**:: 시간을 많이 써줘서 고마워

[1:08 AM] **jrajav**:: (그런데 실제 memoizing이 아니라, React-short-circuiting이나 뭔가 있지 않나)

9:30 AM] **jrajav**:: `@acemarke` https://github.com/acdlite/recompose 의 pure() 함수가 react-redux의 connect()같은 일을 한다는것을 알아냈어

[9:30 AM] **jrajav**:: 그럼 stateless 컴포넌트에 대해서 connect()를 선호하는 이유가 더 있을까?

[9:31 AM] **jrajav**:: 만약 통과하는 props에만 의존한다면?

[9:31 AM] **jrajav**:: 다른 성능 최적화를 제공한다거나 하나?

9:49 AM] **acemarke**: 맞아, 조금 더 해. 네가 말하는 접근법을 감안하면 아마도 recompose를 사용할 것 같군. 더 명확한 의도로

[9:52 AM] **acemarke**: (분명히, 난 아직 네가 connect를 여러 군대 사용하는데 사용하는 이유를 반대하는 것 같아)

[10:06 AM] **jrajav**:: 난 그냥 더 functional한 방법을 선호할 뿌닝야. 데이터 의존성이 명확니까, 합성이나 테스팅도 엄청 쉽지.

[10:07 AM] **jrajav**:: 내가 더 봐야할게 뭐지? 네가 말한 방식이 recompose가 제공하는 것들에도 있나?

[7:58 PM] **acemarke**: `@jrajav` : connect()를 사용하면 내장된 두 가지 최적화를 제공하고 더 많은 최적화를 할 수있는 공간을 제공해. 최근의 스토어 변경이 컴포넌트에 영향을 미치는지 알기 위해서 mapStateToProps가 리턴하는 오브젝트에 대해서 shallow-comparison을 하지. (부모로부터 직접 전달되는 prop은 아니고); 그리고 잘 알려지지 않은 최적화인데 map/dispatch/merged props 중 어떤 것도 변경되지 않은 경우에 React.createElement가 정확히 같은 인스턴스를 반환하는 거야; 그리고 이 모든 것 위에 너만의 shouldComponentUpdate를 추가로 구현할 수도 있어.

Dan은 connect()의 소스코드 대부분을 휼륭한 비디오 코스를 통해서 그게 어떻게 구현되었는지 알려줬지. 지금은 코드가 조금 진화했겠지만, 여전히 좋은 정보야. 여기 그 비디오 링크야 https://youtu.be/VJ38wSFbM3A.

[9:11 PM] **jrajav**:: @acemarke 그래서 만약 우리가 주입된 상태나 dispatch를 사용하지 않고 그저 immutable.js props로 모든 것을 명시적으로 전달한다면, connect()는 recompose의 pure()가 하는 일 외에 아무 것도 추가하지 않는다는 거지?

[9:11 PM] **jrajav**:: 맞아?

[9:11 PM] **jrajav**:: 왜냐면 immutable.js props만을 가지고서 shouldComponentUpdate를 shallow-compare하기면 이 모든 것(pure()가 하는 것)을 덮을 수 있으니까

[9:13 PM] **acemarke**: 거의, 맞아

[9:38 PM] **acemarke**: 흠. 이봐, `@jrajav`. 이 모든 것 말고 고려해야 할 하나가 뭔지 알아? 컴포넌트 트리를 어떻게 두었냐에 따라 이 shouldComponentUpdate의 모든 이득을 취하지 못할 것이라고 생각해.

[9:38 PM] **acemarke**: 만약 수동으로 위에서 아래로 모든 props를 전달하려고 하면, 트리 윗부분의 몇 개의 컴포넌트가 항상 다시 렌더링될거라는 것을 의미하지

[9:38 PM] **acemarke**: 아래쪽 근처에 있는 것들은 그렇지 않겠지만

[9:39 PM] **acemarke**: 하지만 많은 props를 하위 컴포넌트들에게 위임하는 것들은 마찬가지야

[9:39 PM] **acemarke**: 항상 깊게 중첩된 하위 컴포넌트가 어떤 props를 필요로 하는지 추적해야 하고 A > B > C > D > E로 전달될 수 있도록 생각해야만 해

**May 14, 2016**

[1:24 AM] **jrajav**:: 루트로부터 리프까지 경로가 모두 재 렌더링해야할 필요가 있다는 것은 이치에 맞네. 그게 정확이 무슨 일이 일어나고 있는지 반영하는게 아닌가? 어떤 방식의 조직화가 그런걸 피할 수 있지?

[1:27 AM] **jrajav**:: 그리고 맞아 `@acemarke` 나는 그 props들을 추적하는 것이 얼마나 번거로운지 알아. 개인적으로는 다음과 같이 생각하고 있어: D가 E를 포함하고 E는 렌더가 필요한 부분이라고 하자. 그러면 D가 E가 필요로 하는 프로퍼티를 필요로하는게 이치에 맞아. 왜냐면 그 프로퍼티는 실제로 D가 렌더링 될 때 필요한 프로퍼티기 때문이지. 그리고 다른 것들도 마찬가지겠지... 나는 오히려 컴포넌트의 함수 시그니처에 렌더를 목적으로 필요한 파라미터를 정확히 요구해야하는 문제라고 봐. 컴포넌트가 마술처럼 스토어의 임의 장소에서 상태를 가져오는 것이 아니라 말이야. 내게는 전역 변수와 싱글톤 클래스로 state를 갖고 올 수 있는게 편할 수 있지만, 코드는 어디서 그것들이 오는지 추론하기 힘들어지고, 테스트하기 어렵고, 검증하기 어렵고, 오류가 발생하기 쉬울 거야.

[1:28 AM] **jrajav**:: 그리고 react/redux에서는 함수형 패러다임을 사용하는 것처럼 보이는데도 그런 컨테이너/ 컴포넌트를 사용하는 방법이 "모범 사례"라는 것이 나에게 매우 혼란스러워.

11:37 AM] **acemarke**: @jrajav : 내가 봤을때는, "container" 패턴이 몇 가지 장점을 갖고 있어. 우선, 개념적 유형에 따른 분리를 허용하지. 그런 방식으로 바라본다면 말야 - 컴포넌트가 레이아웃과 조직화에 중점을 두는 것들과 컴포넌트가 데이터를 가져오는 데 책임을 갖는 것들 (Redux store루부터든지 서버로부터든지 등등).

[11:41 AM] **acemarke**: Second, you get performance improvements, because a lot fewer components are forced to re-render compared to a totally top-down approach.  Most of your more layout-focused presentational components probably wouldn't be re-rendering at all, and the use of connect() on a component effectively starts a new update tree-ish thing, especially if that connected component isn't being directly passed any props by its parent.  So, if the store updates, and the update didn't change any values that component cared about in mapStateToProps, that component will skip re-rendering, and so will its entire subtree.  Easy win.

[11:43 AM] **acemarke**: Third, I would argue that having a connected component pull in the data it needs, right there, makes it a lot easier to trace the actual data flow, as there's fewer layers to go through.  If I want to know why I'm getting something wrong in my UserListItem, I'd rather just look at its parent UserList rather than have to trace through TopLevelComponent > MainLayout > LeftSidebar > UserList > UserListITem

[11:44 AM] **acemarke**: Fourth, the connected component itself becomes a lot more reusable within the context of the application - if I decided I want to move it from the LeftSidebar to the BottomPanel, I can just do so, without having to change the flow of props coming down from the top component

[11:46 AM] **acemarke**: Fifth, the advice for testing connected components is to export the connected version as the default export, but export the "plain" component as a named export, and only really test the unconnected version.  What you care about is how that component responds to its props and lifecycle.  How it gets those props shouldn't matter.  So, you feed it some props in your test code and verify its behavior there, and maybe test your mapStateToProps function as well if you want, but you can safely assume that connect() will correctly call mapStateToProps when needed, and pass those props to your component when something changes.

[11:50 AM] **acemarke**: Sixth: yes, in a sense Redux is a "global variable", but on the flip side, idiomatic Redux code never actually references it directly.  Everything gets dependency injected.  connect() gives you props from the store and a reference to dispatch(), middleware get dispatch() and getState() injected, and since thunks and such are actually middleware, your action creators can get those injected as well.  No direct references to the store anywhere in your actual app code other than the two lines where you create your store, and pass it to <Provider>.  That also makes using something like redux-mock-store feasible for testing.

[11:50 AM] **acemarke**: Finally, I'm kinda confused: what are your concerns with "containers" vs "functional"?

[11:52 AM] **acemarke**: Ultimately, sounds like we have some differing philosophies on what constitutes good architecture, which is okay.  I'm just trying to understand what all the differences in point of view are.

[2:03 PM] **Francois Ward**: "Fifth, the advice for testing connected components is to export the connected version as the default export, but export the "plain" component as a named export, and only really test the unconnected version. "

[2:04 PM] **Francois Ward**: Careful about that one, its not that simple... if you have a dumb component that contains a connected component, they now MUST provide a store somehow, OR mock the connected component. Not a problem with Jest, or when you can shallow render.

[2:04 PM] **Francois Ward**: but it's not always an option.

[2:04 PM] **acemarke**: yeah, that caveat probably oughta get added to the docs at some point

[2:04 PM] **Francois Ward**: connected components as children are a pain in the rear. Its a compromise, but its not simply "connect everywhere, its free!"

[2:05 PM] **Francois Ward**: there's a strong benefit, AND a high cost to it.



**May 15, 2016**

[11:39 AM] **jrajav**:: @acemarke Thanks for the very detailed response! And yes, as you mentioned I think a lot of what this comes down to is a difference of opinion on best practices. You think it's easier to reason about and reuse a component that knows how to pull its own properties out of the global state (or a close parent of it does, at least), and I would actually concede that at the component-level but I'd also say that dealing with global state at arbitrary points makes the composition of multiple components and the app itself more difficult to reason about, more difficult to compose (like putting together a puzzle), and more error-prone. The extra work involved in explicitly declaring the data dependencies (in this case, by manually passing parameters down) is good work that avoids all that and makes your application simpler and more maintainable. That's the way that I see it, at least.

[11:41 AM] **jrajav**:: @acemarke Also, enforcing that React is always just the presentation layer, and has no knowledge of the store at all, and that all components are "dumb", pure, and have no real lifecycle to speak of - they are just functions of their input - that makes them much easier to develop, test, and reuse

[11:42 AM] **jrajav**:: And can I ask what you meant by this?(edited)

[11:42 AM] **jrajav**:: > Second, you get performance improvements, because a lot fewer components are forced to re-render compared to a totally top-down approach.  Most of your more layout-focused presentational components probably wouldn't be re-rendering at all, and the use of connect() on a component effectively starts a new update tree-ish thing, especially if that connected component isn't being directly passed any props by its parent.  So, if the store updates, and the update didn't change any values that component cared about in mapStateToProps, that component will skip re-rendering, and so will its entire subtree.  Easy win.

[11:44 AM] **jrajav**:: From what I understood, if you have a shouldComponentUpdate over a subtree composed of fully dump props and all of the parameters are equal (probably with the aid of Immutable.js to handle deep objects), then that subtree will not be rendered, so I can't understand how this is different with either methodology

[11:48 AM] **jrajav**:: And to directly answer your question, my concern with "containers" vs. "functional" is that, following from my principles, React should just be the presentation layer, just one large function of the state resulting in the corresponding UI - all the way up to the <App /> itself.

[11:50 AM] **jrajav**:: And thanks for all the counter-opinions, @acemarke! I obviously have my own set of preconceptions already but I'm very new to React and cautious about the nuances I haven't considered. This is very valuable to me.

[11:52 AM] **acemarke**: @jrajav : sure, good reasonable technical differences of opinion and discussion are usually valuable :)

[11:53 AM] **acemarke**: so let's see.  Per your comments:

[11:54 AM] **acemarke**: in my case, my opinions are generally influenced by the numerous discussions I've read, as well as the particular needs of my own app

[11:55 AM] **acemarke**: for example, Dan Abramov's advice at the very beginning of Redux was indeed to only connect your top component, or maybe top couple

[11:57 AM] **acemarke**: but that advice has distinctly changed as time has gone on, and now the suggested usage is to connect deeper wherever it makes sense.  In fact, in his recent optimization pass for the MobX vs Redux benchmark, he found that the "list with 10K items" scenario works best if the parent list gets the IDs of all 10K items, passes the ID to each rendered child item, and the ListItems themselves are also connected and responsible for updating themselves.  That way, an update to one list item doesn't modify the set of IDs, therefore the parent list doesn't have to re-render, etc.

[12:00 PM] **acemarke**: another thought is that while "React is just the view layer" is true, the issue here is that we're trying to figure out how to hook up the data layer and the view layer.  A typical connected component will define its mapStateToProps and the plain component in the same file right next to each other, then export default connect(mapStateToProps)(MyComponent).  So, it's pretty easy to see what the data needs are for a given connected component.

[12:01 PM] **acemarke**: I would also argue that even with multiple connected components, the overall UI is still a "function of the state"

[12:01 PM] **acemarke**: let's see.  the performance tree thing...

[12:01 PM] **jrajav**:: In this case "re-render" just meaning that the list element still needs to iterate over every list item just to find out most of them pass the SCU, right?

[12:02 PM] **acemarke**: yeah

[12:02 PM] **acemarke**: parent renders -> children check SCU -> children may or may not re-render

[12:02 PM] **acemarke**: but the point is that SCU still gets called even if the props are literally identical

[12:03 PM] **jrajav**:: Right

[12:03 PM] **jrajav**:: But if the SCU is just an Immutable.js equality check, that boils down to a single === op per call right?

[12:03 PM] **jrajav**:: Factor in the function calls and yes it's not exactly nothing but not very expensive either

[12:03 PM] **acemarke**: one per field in props, anyway

[12:04 PM] **jrajav**:: Right

[12:04 PM] **jrajav**:: But yes, I can see how this lets you skip over subtrees for rendering

[12:05 PM] **acemarke**: lemme see if I can clarify my point on the subtrees bit you were asking about

[12:06 PM] **acemarke**: so as I understand it, in the "one top level component to rule them all" scenario, you're basically doing const mapStateToProps = (state) => state;

[12:06 PM] **jrajav**:: I'm not entirely sure what you mean by that

[12:06 PM] **jrajav**:: It's more like there is no mapStatetoProp anywhere

[12:06 PM] **acemarke**: well, SOMEONE has to use it to grab something from the store...

[12:07 PM] **acemarke**: or the moral equivalent thereof

[12:07 PM] **jrajav**:: <App dispatch={store.dispatch} state={store.getState()} />(edited)

[12:07 PM] **acemarke**: yeah, that's just connect() without actually calling connect

[12:08 PM] **acemarke**: I assume you're doing, roughly:

[12:08 PM] **jrajav**:: The difference being that it's only at this one root element that we ever access the state

[12:08 PM] **acemarke**: so the same as doing connect(state => state)(App)

[12:09 PM] **jrajav**:: Well sure, but when it's that simple either way it seems preferable to do it the more obvious, explicit way

[12:09 PM] **jrajav**:: Otherwise you're using an abstracted function just to do state={store.getState()} once in the whole app

[12:09 PM] **acemarke**: yeah.  anyway, point is, top level component gets the whole state every time, either way

[12:09 PM] **jrajav**:: Correct

[12:09 PM] **acemarke**: so.  Let's go with my favorite hypothetical app structure

[12:10 PM] **jrajav**:: Todo? ??

[12:10 PM] **acemarke**: some kind of LeftSidebar and RightMainPanel(edited)

[12:10 PM] **jrajav**:: Aww

[12:10 PM] **acemarke**: NO.  NO MORE TODOS.

[12:11 PM] **acemarke**: and we'll say that the left sidebar shows.... a CurrentUserInfo component, plus a ListOfThings component

[12:11 PM] **acemarke**: when you select a ThingListItem in the list, the MainPanel shows a details form for that Thing

[12:11 PM] **acemarke**: which you can edit

[12:12 PM] **acemarke**: so our state would look like, say: { currentUser: {}, things : { byId : {}, order : [] }, thingBeingEdited : {} }

[12:13 PM] **acemarke**: cool so far?

[12:13 PM] **jrajav**:: Yep, I'm following you

[12:14 PM] **acemarke**: oh, we might also want to have a currentSelectedThingId field in there

[12:14 PM] **acemarke**: maybe.

[12:14 PM] **acemarke**: gimme a sec to think through how the sequence for this would work

[12:14 PM] **jrajav**:: No prob

[12:15 PM] **acemarke**: so to start, we'll say the user's already logged in, and we've fetched our data, so both currentUser and things are filled up appropriately

[12:16 PM] **acemarke**: so our ThingList is displaying all its ThingListItems

[12:17 PM] **acemarke**: so I click on ThingListItem #1 to select it, and we dispatch {type : SELECT_THING, payload : {id : 1} }

[12:17 PM] **acemarke**: reducer copies the values from things.byId[1] over to thingBeingEdited

[12:18 PM] **acemarke**: and we want to pass all that down to our form over in the RightMainPanel

[12:19 PM] **acemarke**: so, TopLevelComponent renders `<RightMainPanel thingBeingEdited={state.thingBeingEdited} />`, which renders `<ThingEditForm thingBeingEdited={this.props.thingBeingEdited} />`

[12:19 PM] **acemarke**: which renders a bunch of inputs

[12:20 PM] **acemarke**: and I'm assuming we're going with controlled inputs here

[12:20 PM] **jrajav**:: Controlled inputs meaning that their values are reflected in the state?

[12:20 PM] **acemarke**: other way around technically - their values come from the state

[12:21 PM] **acemarke**: "uncontrolled" inputs means you let the user manipulate them at will, the value is stored by the browser itself per normal, and at some point in time (like form submit) you ask the actual input element itself for its value

[12:22 PM] **acemarke**: "controlled" inputs means you always specify a "value" prop for every input, forcing it to use that value

[12:22 PM] **acemarke**: which means you must also specify an onChange handler, look at the new proposed value in the event, put that in state somewhere, and re-render with the newly accepted modified value

[12:24 PM] **acemarke**: so.  now I put my cursor in the "Thing Name" input field, which currently has a value of "Thing #1", and start typing

[12:24 PM] **acemarke**: and I just try to add "asdf" to the end of "Thing #1"

[12:25 PM] **acemarke**: each keystroke will trigger the onChange handler I've attached to the input

[12:25 PM] **acemarke**: and we'll assume for purposes of the scenario that I'm storing all this still in Redux state, not component state

[12:26 PM] **acemarke**: so the onChange handler dispatches {type : EDIT_THING_FIELD, payload : {name : "Thing #1a"} } for the first keystroke

[12:27 PM] **acemarke**: the thingBeingEdited reducer will update the thingBeingEdited.name field with the new value

[12:27 PM] **acemarke**: so thingBeingEdited.name, thingBeingEdited, and state are all new references

[12:27 PM] **acemarke**: scenario still clear so far?

[12:28 PM] **acemarke**: (again, making this all up off the top of my head, so hopefully not screwing things up :)  )

[12:28 PM] **jrajav**:: Yes, clear so far

[12:28 PM] **jrajav**:: And I'm starting to see where you're heading, too ??

[12:28 PM] **acemarke**: so.  TopLevelComponent gets the new state reference

[12:28 PM] **acemarke**: and will re-render:

[12:30 PM] **acemarke**:


```js
render() {
    const {state} = this.props;
    return (
        <div>
           <LeftSidebar currentUser={state.currentUser} things={state.things} selectedThingId={state.selectedThingId} />
           <RightMainPanel thingBeingEdited={state.thingBeingEdited} />
        </div>
    )
}
```


[12:30 PM] **acemarke**: RightMainPanel, ThingEditForm, and the "name" input obviously re-render

[12:31 PM] **acemarke**: the LeftSidebar will attempt to re-render

[12:31 PM] **acemarke**: so at this point, the question is just how many SCU implementations you've slapped on the rest of your components

[12:32 PM] **acemarke**: in theory, LeftSidebar is a purely presentational component

[12:32 PM] **acemarke**: all it really cares about is rendering CurrentUserDetails and ThingList

[12:32 PM] **jrajav**:: Aren't they both purely presentational if we're using controlled inputs?

[12:32 PM] **acemarke**: ?

[12:33 PM] **jrajav**:: RightMainPanel as well I mean

[12:34 PM] **acemarke**: yeah, sure.  the semi-point of the scenario, I guess, is that the state change really only "affects" one small input, buried deep down inside RightMainPanel

[12:34 PM] **acemarke**: but, because we're doing things purely top-down, every keystroke is going to try to re-render LeftSidebar

[12:34 PM] **jrajav**:: Right

[12:34 PM] **acemarke**: now, maybe we did hook up an SCU implementation for LeftSidebar

[12:35 PM] **acemarke**: but that's still work that doesn't have to be done

[12:35 PM] **jrajav**:: Well

[12:35 PM] **jrajav**:: If everything really is a pure dumb component

[12:35 PM] **jrajav**:: And we use Immutable.js for the full state tree

[12:35 PM] **jrajav**:: Then wrap every dumb component in a recompose pure()

[12:35 PM] **jrajav**:: It's pretty much all done for us, isn't it?

[12:35 PM] **jrajav**:: Without writing any extra code (except the pure() call), LeftMainSidebar would SCU correctly and not re-render

[12:36 PM] **acemarke**: if you really do do that for every component, mostly, yeah

[12:36 PM] **acemarke**: so let's invert the scenario a bit

[12:36 PM] **acemarke**: let's say that TopLevelComponent, LeftSidebar, and RightMainPanel are 100% presentational components, no connections at all

[12:36 PM] **acemarke**: CurrentUserDetails, ThingsList, and ThingEditForm are all connected

[12:37 PM] **acemarke**: and each one has a simple mapStateToProps that is just grabbing their appropriate top-level chunk of state

[12:37 PM] **acemarke**: once again, I place my cursor in the "Name" field after "Thing #1" and type "a"

[12:38 PM] **acemarke**: edit action dispatches, thingBeingEdited.name is updated, store notifies subscribers

[12:38 PM] **acemarke**: CurrentUserDetails will re-run mapStateToProps and return {currentUser : state.currentUser}

[12:39 PM] **acemarke**: connect() will shallow-compare that return value vs the last return value, see things haven't changed, and skip re-rendering

[12:39 PM] **acemarke**: ditto for ThingsList

[12:39 PM] **acemarke**: ThingEditForm will see that thingBeingEdited has changed and re-render

[12:39 PM] **acemarke**: but TopLevelComponent, LeftSidebar, and RightMainPanel never even had to do anything

[12:40 PM] **acemarke**: they do not know, and they do not care, that anything happened with the state at all

[12:40 PM] **jrajav**:: I'm definitely seeing the point here, but - are we not just trading one linear set of compare operations for another?

[12:40 PM] **jrajav**:: I do see the argument that the connect()'ed set may be smaller

[12:41 PM] **jrajav**:: But if the state tree is organized sensibly w.r.t. the UI they may not be that dissimilar

[12:41 PM] **acemarke**: yeah, I'm not saying that the overall intent and behavior are drastically different

[12:42 PM] **jrajav**:: In this case specifically they're almost identical - we seem to be running three comparison funcs outside of the "real" re-render (the form) in either case

[12:42 PM] **acemarke**: but depending on how nested your UI is, and how much data you've got, you could certainly be cutting out a number of intermediate layers

[12:43 PM] **jrajav**:: But you're trading those intermediate layers for however many container components you have

[12:43 PM] **acemarke**: it also is, as you said, a question of whether or not you want to explicitly tag all your more layout-y components with SCU

[12:43 PM] **acemarke**: and deal with the props management

[12:45 PM] **acemarke**: for me personally, I would rather do a lot less typing, get the same or better performance, and have less data flow to track



12:45 PM] **jrajav**:: But it's not explicit if we rely on recompose's pure() implementing it for us

[12:46 PM] **jrajav**:: You do have to remember to do that though...

[12:47 PM] **acemarke**: yeah, that's what I meant.  just having to actually include it in each component definition

[12:47 PM] **jrajav**:: Right

[12:48 PM] **acemarke**: so lemme just give a general description of my own app, which is obviously an influence on my thinking

[12:48 PM] **acemarke**: rewriting the client for an app I built a few years back in GWT

[12:50 PM] **acemarke**: basically a map planning tool that uses a 3D globe, lets the user interact with the globe.  left pane has some control buttons and a treeview with all items in the project.  bottom pane has tabs to show attribute forms for each different data type in the tree.  data items are shown in the tree, can be viewed and edited in the forms, and are displayed on the globe.

[12:50 PM] **acemarke**: visually, the app has a marked resemblance to Google Earth Desktop's layout: http://elmcip.net/sites/default/files/platform_images/launch_google_earth.jpg

12:50 PM] **acemarke**: bump up the globe and put a tabbed section with forms inside in the bottom third of the right pane

[12:52 PM] **acemarke**: so for my particular app, I've got some pretty heavy nesting inside that tree, need to show/edit items in the forms, and need to render them on the globe, and update all of the above on selection, editing, etc

[12:52 PM] **acemarke**: my top couple layers are very much purely presentational and dumb

[12:52 PM] **acemarke**: my TreeView is itself also presentational and dumb at the top, but displays a "ConnectedFolder" tree item for each data type

[12:53 PM] **acemarke**: the individual nested tree items then follow that "grab data for self, pass just ID to children" paradigm

[12:53 PM] **acemarke**: and only render their children if expanded

12:54 PM] **acemarke**: so, there's some pretty serious nesting going on, but if I do something like click on a tree item to make it the currently selected thing, only that tree item, the appropriate attributes form, and the corresponding individual globe display component will really update

12:55 PM] **acemarke**: I don't have my tree item components optimized to use per-component-instance memoized selectors yet, but so far performance hasn't been an issue.  always something I can throw in later.

[12:56 PM] **acemarke**: so, that sort of structure is obviously an influence on my thought process.

12:58 PM] **acemarke**: as well as Dan Abramov's recent optimization pass at the MobX vs Redux benchmark: https://twitter.com/dan_abramov/status/720219615041859584 , https://github.com/mweststrate/redux-todomvc/pulls?q=is%3Apr+is%3Aclosed

Dan Abramov@dan_abramov

Red is unoptimized Redux app. Orange is an optimized one. Green is MobX. Takeaway: in Redux, state shape means a lot https://t.co/qq3NlzbQpv

GitHub

mweststrate/redux-todomvc

redux-todomvc - Redux todoMVC, used to do some benchmarking

[12:59 PM] **acemarke**: and finally, a great presentation on optimizing Redux that demonstrates the various approaches to connect: http://somebody32.github.io/high-performance-redux/

[1:00 PM] **acemarke**: so.  all that said: I personally would find the pure top-down approach annoying and verbose.  BUT, if it does work better for you mentally, go for it.

[1:11 PM] **jrajav**:: I feel like I understand the situation a lot better now @acemarke, thanks again for the very detailed discussion

[1:12 PM] **acemarke**: sure.  Wall O' Texts my specialty! :)

[1:13 PM] **jrajav**:: I think I would still like to try this current application in the purely-functional style, out of curiosity more than anything else. Perhaps there are performance concerns but this application isn't too performance bound. It does, however, have a rather large state tree, so I'm interested to see if that can be effectively organized with subproperty destructuring at each level of the tree, and just how messy it would really get

[1:14 PM] **acemarke**: I guess I never did ask: what sort of app do you have, and what's the data like?

[1:15 PM] **jrajav**:: A relatively simple search-and-display app

[1:15 PM] **jrajav**:: Main page will be a search view, with advanced features like a facet filtering sidebar, autocomplete, pagination, term highlighting

[1:16 PM] **jrajav**:: Then the search results will link to a large details view for each item, with a few dozen mostly static detail properties, spread into different tabs and views

[1:18 PM] **jrajav**:: So the state tree will mostly be a 'search' object with the query and facets, a 'searchResults' object (which I'm considering nesting in 'search'), then a separate 'details' object with the single selected item once you route to that

[1:19 PM] **jrajav**:: It's probably too simple of an app to really run into big issues with state management or performance, but it is complex enough to get a good sense for the organization and passing down properties at least

[1:22 PM] **acemarke**: gotcha.  so, not entirely dissimilar from my hypothetical ThingEditor example, then :)

[1:22 PM] **acemarke**: and yeah, doesn't sound overly perf-intensive
