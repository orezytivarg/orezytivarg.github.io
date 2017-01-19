---
id: you-might-not-need-redux
title: 어쩌면 당신에게 Redux가 필요하지 않을 수도 있다
category: Redux
---
[원문보기](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367#.4ucx5na41)

# 어쩌면 당신에게 Redux가 필요하지 않을 수도 있다

사람들은 종종 필요하기도 전에 Redux를 선택한다. "Redux가 없어도 앱을 확장할 수 있을까?". 나중에 개발자들은 Redux를 간접적으로 힐난한다. "이 간단한 기능을 동작시키는 일에 파일을 3개나 손대야하지?". 정말 왜 그럴까!

사람들은 Redux, React, 함수형 프로그래밍, immutability 등등을 핑계로 자신의 불행을 탓한다. 그리고 나는 그 사람들을 이해한다. 상태를 업데이트하기 위해 보일러플레이트 코드를 요구하지 않는 접근방법들과 Redux를 비교하는 것은 당연하고, Redux가 그저 복잡하기만 하다는 결론을 내게 되는 것도 당연하다. 어떤 면에서 보면, 그걸 의도한 설계이기도 하다.

Redux는 절충안을 제안하고 있다. 당신에게 다음과 같이 요구하는 것이다:

 - 애플리케이션 상태를 플레인 오브젝트와 배열로써 기술하라
 - 시스템의 변경 사항을 플레인 오브젝트로써 기술하라
 - 변경사항을 처리하는 로직은 순수(pure)함수로 기술하라

React를 쓰든지 안쓰든지 간에 이러한 제한점이 Redux 애플리케이션을 만드는데 요구된다. 사실 이들은 꽤 강력한 제약이고, 앱의 어떤 부분에 적용하더라도 주의 깊게 생각해야만 한다.

이렇게 하는데 어떤 좋은 이유라도 있는가?

이러한 제한이 다음과 같이 앱을 만드는 것을 도와주기 때문에 매력적이다:

 - [상태를 로컬 스토리지에 저장하고, 다음에 시작할때 그것을 가지고 시작한다](https://egghead.io/lessons/javascript-redux-persisting-the-state-to-the-local-storage?course=building-react-applications-with-idiomatic-redux)
 - [서버로부터 미리 채워진 상태를 가지고 클라이언트의 HTML에 얹어서 보내고 그것을 가지고 시작한다](http://redux.js.org/docs/recipes/ServerRendering.html)
 - [사용자 액션을 직렬화하고 상태에 대한 스냅샷과 함께 첨부하여 자동 버그 보고서를 보내면 제품 개발자가 오류를 재현할 수 있게 한다](https://github.com/dtschust/redux-bug-reporter)
 - [네트워크를 통해 액션 객체를 전달하여 코드 작성 방법을 크게 변경하지 않고도 협업 환경을 구축할 수 있다](https://github.com/philholden/redux-swarmlog)
 - [코드를 크게 변경하지 않고도 undo 히스토리를 관리하거나, 낙관적인 변경을 구현할 수 있다](http://redux.js.org/docs/recipes/ImplementingUndoHistory.html)
 - [개발 중에 코드 변경 없이도 상태 히스토리간 여행을 할 수도 있고, 액션 히스토리를 통해 현재 상태를 다시 평가해볼 수 있게 한다](https://github.com/gaearon/redux-devtools)
 - [제품 개발자가 앱 전용 커스텀 툴을 만들 수 있도록 검사 및 제어 기능을 완벽히 제공한다](https://github.com/romseguy/redux-devtools-chart-monitor)
 - [대부분의 비즈니스 로직을 재사용하면서 다른 UI로 대체할 수 있게 한다](https://youtu.be/gvVpSezT5_M?t=11m51s)

만약 당신이 [확장 가능한 터미널]()이나 자바스크립트 디버거 혹은 어떤 종류의 웹앱에서 작업하는 경우라면, 적어도  이 아이디어들을 고려해보거나 실제로 시도해보는 것은 가치가 있을 것이다(별로 새로운 것은 아니다).

그러나 React를 배우는 중이라면, Redux를 첫 선택으로 만들지 말아달라.

대신 [think in React](https://facebook.github.io/react/docs/thinking-in-react.html)를 배우라. 정말로 Redux가 필요할 때, 혹은 뭔가 새로운 것이 필요할 때만 Redux로 돌아오라. 하지만 고집불통 툴을 사용하는 것처럼 조심해서 접근하라.

만약 "Redux 방식"으로 하는 것에 압박을 느낀다면, 당신이나 당신의 팀 동료들이 이를 심각하게 받아들이는 신호일 수 있다. Redux는 도구 중 하나일 뿐이며, [거친](https://www.youtube.com/watch?v=uvAXVMwHJXU) [실험](https://www.youtube.com/watch?v=xsSnOQynTHs)이다.

마지막으로, Redux를 사용하지 않고도 Redux의 아이디어를 적용할 수 있다는 것을 잊지 말아라. 예를 들자면, 로컬 상태를 가지는 React 컴포넌트를 생각해보자:

<script src="https://gist.github.com/gaearon/a9bbb73d57b6e4cc17d7b50807b62f9a.js"></script>

완벽하게 좋다. 진지하게 반복하건대,

로컬 상태는 아주 좋다.

Redux가 제공하는 절충안은 "어떻게 그것이 바뀌었는가"에서 "무엇이 일어났는가"로 분리하는 간접 참조를 추가하는 것이다.

이 절충안이 항상 좋은 것인가? 아니 절충안은 절충안일 뿐이다.

예를 들면, 컴포넌트에서 리듀서를 추출할 수 있다:

<script src="https://gist.github.com/gaearon/64e2c4adce2b4918c96c3db2b44d8f68.js"></script>

와우! 어떻게 npm install을 실행하지 않고 Redux를 사용했는지 보라.

당신의 상태적인(stateful) 컴포넌트에서도 이렇게 해야할까? 아마도 아닐 것이다. 즉, 이 추가적인 간접 지시 방식로부터 어떤 이득을 취할 계획이 있는게 아니라면 이렇게 할 필요가 없다는 것이다. 계획을 갖는 것. 그게 바로 이 글의 요점이다.

[Redux 라이브러리](http://redux.js.org/)는 하나의 글로벌 스토어 객체에다가 리듀서들을 "탑재"하는 것을 도와주는 도우미들일 뿐이다. 딱 당신이 원하는 조금의 양 만큼만 Redux를 사용해도 된다.

하지만 뭔가를 절충하기로 했다면 무언가를 확실히 얻어내야 한다.
