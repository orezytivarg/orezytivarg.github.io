---
layout: post
id: modular-reducers-and-selectors
title: 모듈형 리듀서와 셀렉터
tags: [React, Redux]
---
[원문보기](http://randycoulman.com/blog/2016/09/27/modular-reducers-and-selectors/)

# 모듈형 리듀서와 셀렉터

최근에 Redux 상태 트리를 캡슐화 하는 방법을 이야기했다. 이전 포스트에서는 combineReducers를 사용할 때의 리듀서와 셀렉터의 비대칭에 대해 알아보았고 Rails-스타일의 프로젝트 조직화에서는 잘 동작하는 접근방법을 알아보았다. 하지만 다른 모듈형 구조(domain-스타일)을 사용하려고 할 때, 이슈가 나타났다.

## 모듈형 프로젝트 구조란 무엇인가?

이전 시간이 언급했던, Redux FAQ에서 얘기했던 프로젝트를 구조화하는 몇 가지 다른 방법들은 다음과 같다:

> - Rails-스타일: "actions", "constants", "reducers", "containers", "components"로 분리된 폴더를 두는 스타일
> - Domain-스타일: 피처나 도메인 별로 폴더를 두고 파일 타입에 따라 서브 폴더를 두는 스타일
> - Ducks: 도메인 스타일과 비슷하지만, 액션과 리듀서를 같은 파일에 명시적으로 함께 두는 스타일

이 포스트에서는 "Ducks"라는 스타일만 다룰 것이다. 왜냐하면 리듀서와 셀렉터에 관련된 차이가 없기 때문이다. 나는 이러한 스타일을 일반적으로 "모듈형"이라고 부른다. 애플리케이션을 별도의 모듈로 쪼개기 때문이다.

Redux FAQ에 많은 링크들이 있지만 개인적으로 Jask Hsu의 [(Redux)애플리케이션 구조화 법칙](http://jaysoo.ca/2016/02/28/organizing-redux-application/)을 가장 좋아한다.

이 글에서 Jack은 프로젝트 구조화에 대해 3가지 법칙을 정립했다:

1.  피쳐에 따른 조직: 애플리케이션의 각 도메인이나 피처는 자신만의 "모듈" 혹은 폴더를 가지고 있어야 한다. 관련된 액션들, 리듀서들, 컴포넌트들, 셀렉터들은 모두 이 모듈에 위치한다.
2.  엄격한 모듈 경계 생성: 각각의 모듈은 퍼블링 인터페이스를 명시적으로 정의한다. 탑 레벨의 `index.js` 파일만이 다른 모듈에 노출시켜야 하는 부분이다. 모듈은 절대로 다른 모듈로부터 "deep imports"(예: `import Component from 'modules/foo/compoents/Compoent'`)를 하지 말아야 한다. 만약 `import { Component } from 'modules/foo'`가 불가능하다면 컴포넌트는 외부 모듈에서 사용할 수 있는 컴포넌트가 아닌 것이다.
3.  순환 의존관계 피하기: 모듈 A가 모듈 B에 의존한다면(즉, B의 퍼블릭 인터페이스의 무언가를 임포트한다면), 모듈 B는 A의 어떤 것에도 의존하지 말아야 한다. 이것은 이전에 관련 [포스트](http://randycoulman.com/blog/2014/02/04/packaging-principles-part-2/)를 올린 적 있었던 엉클 밥 마틴의 [Acyclic Dependencies Principle(ADP)](https://en.wikipedia.org/wiki/Acyclic_dependencies_principle)를 말한다.

## 그래서 문제가 뭐야?

모듈형 프로젝트 구조에서는 리듀서 및 로컬라이즈된 셀렉터가 모듈 내에 있다(지난 포스트의 `todos` 예제처럼). 그 부분은 양호하다.

리듀서는 `todos`를 탑레벨 모듈(이 포스트에서는 앱이라고 부를 것이다)에서 임포트하고, 다른 모듈의 리듀서를 `combineReducers`를 사용해 조합한다. 지금까지는 꽤 괜찮다.

썽크 액션 생성자와 컨테이너 컴포넌트도 `todos` 모듈에 있다. 그러나 이들은 글로벌화된 셀렉터를 필요로 한다.

그래서 글로벌 셀렉터는 어디에 있을까? 글로벌 셀렉터는 두 가지가 필요하다.

-로컬 셀렉터
-로컬 셀렉터가 바라는 전체 상태 트리의 루트로부터 잘라내진(sliced) 상태 트리

로컬 셀렉터는 `todos`에 있다. path 정보는 메인 앱 리듀서와 함께 작성된 `app` 모듈에 있다.

`app` 모듈이 우리가 필요한 정보의 일부를 갖고 있고, 이미 `todos`의 리듀서에 의존하고 있기 때문에 글로벌 셀렉터가 `app` 모듈에 있어야 한다는 것은 꽤 논리적인 것 같다. 문제가 해결되었다. 맞나?

속단하지 말고, 썽크 액션 생성자와 컨테이너는 글로벌 셀렉터가 필요하며, `todos` 모듈에 있음을 기억하자.

글로벌 셀렉터가 `app` 모듈에 있다면 `todos` 모듈은 썽크 액션 생성자와 컨테이너가 사용할 수 있도록 임포트 해야할 필요가 있다. 하지만 `app` 모듈은 이미 `todos`에 의존하고 있기 때문에 종속성 싸이클이 생성된다.(app -> todos -> app). Jack의 세 번째 규칙인 Acyclic Dependencies Principle(ADP)를 따르고 있다면 이렇게 할 수 없다.

## 이제 뭘 해야 할까?

스포일러 경고: 아직 이 문제에 대한 실제 솔류션을 찾아내지 못했다.

내가 생각해내고 실제로 하고 있는 몇 가지 선택지에 대해 말할 수 있다.

### ADP를 잊어라

ADP를 잊어버리고 종속성 싸이클을 허용할 수 있다. 문제는 한번 예외를 허용하면 다른 예외들도 쉽게 허용할 수 있다는 것이다. 이는 결국 유지보수가 불가능한 복잡한 스파게티 코드와 함께 끝나게 될 것이다. 모듈식 구조를 통해 우리가 처음부터 피하려고 했던 것이다.

더 실제적으로 보면 빌드/번들링 도구가 의존성 싸이클을 제대로 처리하지 못할 것이다. 나는 웹팩을 사용할 때, 그리고  React Native 패키저를 사용할 때 모두 의존성 싸이클을 만든 적이 있고, 두 경우 모두 정말 행복하지 못했다.

ADP를 잊는 것은 좋은 해답이 아닌 것처럼 보인다.

## 모듈형 구조를 잊어라

모듈형 구조를 포기하고 Rails 스타일로 돌아갈 수 있다. 어쩌면 모듈형 구조가 실제로 좋은 접근 방법이 아닐 수 있다는 표시일 수도 있다.

그러나 나는 모듈형 구조가 훨씬 행복한 구조라는 것을 알게되었다. 뭔가를 찾는 것이 더 쉽고, 애플리케이션의 동작에 대해 추론하는 것이 더 쉬울 뿐더러, 코드가 더 깨끗해진다. 실제로 뭔가가 어디에 속해야 하는지 생각하기가 더 쉽기 때문이다.

다시, 모듈형 구조의 요점은 큰 스파게티 코드를 피하는 것이다.

모듈형 구조를 잊는 것도 역시 좋은 해답이 아닌 것처럼 보인다.

## 싸이클을 깨라

"교과서(엉클 밥이 쓴 애자일 소프트웨어 개발: 원칙, 패턴, 관행)"적인 방법은 두 가지 메커니즘 중 하나를 사용하여 싸이클을 깨는 것이다.

1.  의존성 반전 원리를 적용하라. todos 내에서 app과 todos 양쪽에서 쓰이는 부분을 추출하는 것이다. 다음 섹션에서 살펴볼 것이다.
2.  app과 todos가 모두 의존하는 새 모듈을 만든다.

이러한 접근법에 대해서 여러번 생각해보았으나 만족할만한 추출법을 발견하지는 못했다.

아이디어 하나는 상태트리 모양에 대한 설명을 빼내는 것이다. 애플리케이션의 메인 리듀서는 리듀서를 결합하는데 이 설명을 사용하고 todos는 이를 사용하여 셀렉터를 글로벌화한다.

그러나 이것은 지나치게 복잡해 보인다. 그래서 아직 이쪽 방향으로 나아가진 않았다. 그러나 의존성 싸이클을 깨는 다른 방법에 대해서는 열려있다.

## 글로벌 셀렉터를 모듈로 이동하라

내가 고려해본 최종 해결책은 글로벌 셀렉터를 todos 모듈로 옮기는 것이다. 이렇게 하려면 todos 모듈은 상태 트리의 루트에서 로컬 서브-섹션을 알아내야 한다.

앱 리듀서가 이 경로를 생성하기 때문에 서브섹션을 알아내는 방법을 실제로는 가질수 없고, 복제된 형태로만 가질 수 있다.

이를 수행하는 몇 가지 방법이 있다.

## 모듈이 상태 설치 위치를 제어하게 하라

Jack의 글에서 말하기를:

> todos 모듈에게 자신만의 state에 대한 설치 위치를 제어할 수 있도록 하여 이 문제를 해결할 수 있다

이 방법에서 각 모듈은 `reducerKey` 혹은 `moduleName` 혹은 이런 비슷한 상수를 정의한다.

상수는 글로벌 셀렉터를 생성하는 데 사용된다.

Globalizing a Selector

```js
import { moduleName } from './constants'
import * as fromTodos from './localSelectors'

export const allTodos = state => fromTodos.allTodos(state[moduleName])
```

또한 메인 리듀서가 리듀서들을 조합할 때도 사용된다:

Combining Reducers

```js
import { combineReducers } from 'redux'
import { reducer as todosReducer, moduleName } from './todos'

export default combineReducers({
  [moduleName]: todosReducer
})
```

## 복제와 함께 살아가기

또다른 선택지는 셀렉터를 조금 복제하는 방법이다. 나는 현재 이 접근법을 사용하고 있지만, Jack의 방법으로 스위칭할 것을 고려중이다.

셀렉터를 리듀서로부터 분리된 파일에 유지하고, 셀렉터 파일에서는 로컬 셀렉터를 이름 지어 익스포트하고, 글로벌 셀렉터를 포함하는 default로 익스포트한다.

다음과 같이 보일 것이다:

Living With Duplication

```js
const localState = state => state.todos // the duplication!

export const allTodos = state => {
  // extract all todos from the local state
}

export default {
  allTodos: state => allTodos(localState(state))
}
```

실제 세계에서는 [이전에 언급](http://randycoulman.com/blog/2016/02/16/using-ramda-with-redux/)했던 [Ramda](http://ramdajs.com/)를 이용한다. 조금 더 깨끗해 보일 것이다:

Using ramda

```js
import { compose, prop } from 'ramda'

const localState = prop('todos')

export const allTodos = state => {
  // extract all todos from the local state
}

export default {
  allTodos: compose(allTodos, localState)
}
```

내 리듀서와 그 테스트에서는 개별 셀렉터를 임포트하고 다른 곳에서는 디폴트를 임포트해서 글로벌 셀렉터를 임포트한다. 지금은 이게 혼동되는지 잘 모르겠지만, 시간이 지난 후에는 따라잡기 어려울 수 있다.

## 결론

말했다시피 아직 완전히 만족스러운 해결책은 찾지 못했다. 나는 현재 복제를 이용하여 살아가고 있다.

그리고 모듈이 상태 트리에 설치되는 곳을 제어하는 Jack의 접근법으로 전환을 진지하게 고려중이다.

혹시 당신도 이런 문제가 발생했나? 그렇다면 어떻게 해결했는지? 내 방법보다 나은 방법이 있는가?
