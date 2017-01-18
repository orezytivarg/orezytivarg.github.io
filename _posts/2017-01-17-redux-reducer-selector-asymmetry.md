---
id: redux-reducer-selector-asymmetry
title: Redux 리듀서와 셀렉터간의 비대칭성
category: Redux
---
[원문보기](http://randycoulman.com/blog/2016/09/20/redux-reducer-selector-asymmetry/)

# Redux 리듀서와 셀렉터간의 비대칭성

이전 포스트에서 Redux 상태 트리를 캡슐화하는 액션과 리듀서와 셀렉터의 사용에 대해서 이야기 했다. 그 포스트에서는 하나의 탑레벨 리듀서에 대해서 잘 동작한다는 것을 보였으나, 분해된 리듀서를 어떻게 다루어야 하는지에 대해서는 보여주지 못했다. 이제 그에 대해서 이야기하자.

## 무엇이 문제인가?

리듀서를 작은 조각으로 나누는 몇 가지 방법들이 있다. Mark Erikson은 Redux 문서의 pull request에서 이 선택사항에 대해서 잘 설명했다.

가장 일반적인 접근방법은 Redux의 combineReducers 함수를 이용하여 state를 분리 독립적인 "슬라이스"로 나누는 것이다. 각 슬라이스는 서브-리듀서에 의해 처리된다. 예를 들면:

combineReducers

```js
import calendarReducer from './calendarReducer'
import todosReducer from './todosReducer'
import usersReducer from './usersReducer'

export default combineReducers({
  calendar: calendarReducer,
  todos: todosReducer,
  users: usersReducer
})
```

이 예제에서는 상태트리 안의 각각의 섹션을 처리하는 리듀서들로 조합된 싱글 탑레벨 리듀서를 만들었다. calendarReducer, todosReducer, usersReducer는 그들만의 권리를 갖고 있다.

우리는 상태 트리의 각 섹션들이 애플리케이션의 나머지 부분으로부터 그 모양을 숨긴 채로 유지하기를 원한다. 어떻게 그렇게 할 수 있을까?

Redux 문서의 FAQ는 이렇게 말하고 있다:

> 일반적으로 리듀서와 셀렉터를 함께 정의하고 익스포트하기를 제안한다. 리듀서 파일 안애 상태 트리의 실제 모양에 대해서 알아야 하는 코드를 모두 함께 두어서 이를 다른 곳(mapStateToProps 함수들, 비동기 액션 생성자, sagas)에서 재사용하도록 하는 것이다.

셀렉터와 리듀서를 함께 두고 정의한다면 리듀서가 상태 트리의 서브셋에서 동작하는 경우 셀렉터는 뭘 해야 할까?

## 어떤 선택지가 있는가?

실제로 단 2개의 셀렉터를 위한 선택지가 있다:

1. 리듀서와 평행적으로 셀렉터를 만들어야 한다. 리듀서들과 마찬가지로 제한적인 서브셋에 대해 동작해야만 한다.

2. "글로벌" 셀렉터를 만들어야 한다. 즉, 셀렉터는 전체 상태 트리의 루트를 가지고 동작하기를 기대하는 셀렉터를 말한다.

## 어떤 선택지가 가장 좋은가?

셀렉터를 사용하는 곳을 살펴보자. 지난번에 몇몇 장소를 식별했었다:

- 컨테이너 컴포넌트의 mapStateToProps 함수들
- 썽크 액션 생성자들
- 리듀서 테스트들

추가적으로 셀렉터를 사용할 수 있는 다른 곳들을 생각해봤다:

- 리듀서 자체. 가끔 리듀서는 상태 (서브)트리의 또다른 부분을 참조할 필요가 있다. 비록 리듀서가 이미 상태 트리의 모양에 결합되어 있다고는 하지만, 이미 정의된 셀렉터를 이용하는 것이 종종 편리할 때가 있다.

셀렉터의 이러한 용도를 살펴본 결과, 글로벌 상태 트리 혹은 로컬의 제한된 상태 트리를 사용하는 것 중 어떤 것이 좋을까?

mapStateToProps는 항상 글로벌 상태 트리를 호출한다. 썽크 액션 생성자의 getState 함수는 글로벌 상태트리를 리턴하는 것 같다. 그러나 리듀서와 리듀서 테스트는 로컬 상태 트리와 함께 작동한다.

50대 50이다. 별로 도움이 되지는 않는다.

## 둘 다 할 수는 없나?

글로벌 상태와 로컬 상태 양 쪽 모두에 대해 작동하는 셀렉터를 가질 수 있는 방법이 있을까?
몇 가지 방법이 있다.

### 하이브리드

우선 "하이브리드"라고 부르는 접근 방법이다.

이 접근법에서는 모든 셀렉터는 로컬 섹션과 함께 동작하도록 정의한다.

상위레벨(즉, 메인 리듀서)에서의 셀렉터는 로컬 상태 트리를 갖고 온다. 글로벌 상태가 필요한 셀렉터를 호출하는 경우에는 먼저 탑레벨 셀렉터를 적용하고난 다음 로컬 셀렉터를 적용한다. 결곡 다음과 같이 보일 것이다:

Hybrid selectors

```js
// In todosReducer.js:
const allTodos = state => {
 // get todos from local state
}

// In appReducer.js:
const todosState = state => state.todos

// In a container somewhere:
import { allTodos } from './todosReducer'
import { todosState } from './appReducer'

const mapStateToProps = state => ({
  todos: allTodos(todosState(state))
})
```

동작은 하겠지만, 끔찍하게 많은 반복적인 코드가 모든 곳에 위치하게 된다. 더 잘 할 수 있지 않을까?

## 위임

Dan Abramov의 비디오, [Redux: Colocating Selectors with Reducers](https://egghead.io/lessons/javascript-redux-colocating-selectors-with-reducers)에서는 위임하는 방법을 사용한다.

이 접근법은 하이브리드 방법과 비슷하지만 로컬 셀렉터와 상태를 쪼개는 셀렉터를 조합하여 appReducer 파일에 둔다.

위와 같이 todosReducer.js는 로컬(todos-only) 상태에 대해서 동작하는 allTodos 셀렉터를 익스포트한다. 그 다음 메인 리듀서 파일에서 다음과 같이 한다:

Making a Global Selector

```js
import * as fromTodos from './todosReducer'

export const allTodos = state => fromTodos.allTodos(state.todos)
```

todos 서브 섹션의 상태 트리를 추출한 다음 로컬-상태버전의 allTodos를 호출하는 글로벌-상태 버전의 allTodos를 정의한다.

리듀서와 리듀서 스펙에서는 todosReducer가 익스포트한 로컬 버전의 allTodos를 임포트한다. 컨테이너와 액션 생성자에서는 메인 리듀서가 익스포트하는 글로벌 버전의 allTodos를 임포트한다.

추가 함수를 정의하는 비용으로, 셀렉터의 두 가지 버전을 갖게된다. 로컬 상태와 동작하는 첫 번째, 그리고 첫 번째 버전을 이용하여 글로벌 상태와 동작하도록 만든 두 번째 버전이다.

이 접근 방식의 장범은 상태 트리의 모양에 대한 모든 지식이 적절한 위치에 캡슐화되어 있다는 것이다.

로컬 버전의 셀렉터는 상태 트리의 일부분을 알고 있고 적절히 결합된다. 그러나 글로벌 트리에 대해서는 모른다.

글로벌 버전의 셀렉터는 지역 상태가 어디에 있는지 알고 있다. 그러나 해당 섹션의 구조나 모양에 대해서는 알지 못한다. 그에 대해서는 로컬 버전의 셀렉터에게 위임한다.

이 캡슐화는 하이브리드 및 위임 접근법 모두에 존재한다. 위임 접근의 장점은 로컬 셀렉터와 상태 자르기용 셀렉터를 반복하지 않아도 된다는 것이다. 사실, 클라이언트 코드는 이러한 구성이 일어난다는 사실조차 알 필요가 없다.

## 문재 해결. 맞지?

나는 위임 접근법이 좋은 해결책이라고 생각한다. 각 셀렉터의 추가 버전을 작성하는 비용이 들지만 캡슐화와 유연성은 가치가 있다.

상태 트리를 여러번 중첩된 레벨로 분리하면, 그 아래의 모든 셀렉터를 재정의해야하므로 끔찍한 작업이 된다. 어느 시점에서 이건 관리할 수 없게 된다.

추가적으로, Redux FAQ에서는 프로젝트 구조화에 대한 다른 방법들을 이야기하고 있다:

> - Rails-style: "actions", "constants", "reducers", "containers", "components"로 분리된 폴더를 두는 스타일
> - Domain-style: 피처나 도메인 별로 폴더를 두고 파일 타입에 따라 서브 폴더를 두는 스타일
> - Ducks: 도메인 스타일과 비슷하지만, 액션과 리듀서를 같은 파일에 명시적으로 함께 두는 스타일

Rails 스타일의 접근법을 사용하면 문제가 해결된다고 생각한다. Dan Abramov의 위임 접근법은 위에서 언급했듯이 좋은 방법이다.셀렉터를 리듀서 파일과 별개로 두는 방법을 선택하더라도 이 테크닉을 사용할 수 있다.

그러나 Domain이나 Ducks 접근법을 사용하기를 원한다면 이 테크닉은 순환 의존이라는 이슈를 발생시킨다.

다음 포스트에서 이 구조화에 대해서 더 많은 시간을 써 볼 것이다.
