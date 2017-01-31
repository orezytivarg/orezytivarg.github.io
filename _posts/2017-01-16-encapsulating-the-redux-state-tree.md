---
id: encapsulating-the-redux-state-tree
title: Redux 상태 트리 캡슐화하기
tags: [React, Redux]
---
[원문보기](http://randycoulman.com/blog/2016/09/13/encapsulating-the-redux-state-tree/)

# Redux 상태 트리 캡슐화하기

Redux 애플리케이션에서, 대량의 애플리케이션의 데이터는 중앙에 위치하는 "상태 트리"라는 스토어에 저장된다. 상태 트리의 모양과 구조는 애플리케이션의 쉬운 개발과 성능 향상에 큰 영향을 미친다. 시간이 지남에 따라 문제를 해결하기 위해 상태 트리를 리팩토링하는 것은 종종 중요해진다. 어떻게 리팩토링을 안전하게 할 수 있을까?

이전에도 Redux에 대해 몇 번 쓴적이 있다. 입문 혹은 주의환기를 위해서 그 [포스트들](http://randycoulman.com/blog/categories/redux/)을 가볍게 복기해보라.

데이터 구조 변경을 안전하게 만들기 위해서는 캡슐화가 가장 일반적으로 쓰이는 도구다. 캡슐화는 data-hiding 혹은 information-hiding이라고도 알려졌다. 캡슐화의 기본 개념은, 데이터 구조에 직접적으로 접근하기보다는 데이터 대신 제공하는 인터페이스에 접근하는 것이다.

이게 바로 우리가 이 포스트에서 알아볼 내용이다.

## 상태 트리는 어떻게 사용되는가?

상태 트리를 캡슐화하려면, 애플리케이션에서 상태 트리가 어떻게 쓰이고 있는지를 알아야 한다. 대부분의 데이터 구조에는 읽기와 쓰기라는 두 종류의 쓰임이 있다.

Redux 애플리케이션에서도 마찬가지다. Redux는 Flux 아키텍쳐의 구현이기 때문에 단방향 데이터 플로우라는 원칙을 따르기 때문이다. 그래서 상황을 더 쉽게 파악할 수 있다.

### 쓰기

우선 방정식의 쓰기 부분부터 시작해볼 것이다. Redux에서 가장 명확한 부분이기 때문이다.

상태 트리의 변경은 다양한 액션에 대한 리듀서의 응답에 의해 만들어진다. 변경은 직접적으로 만들어지지 않는다. 리듀서는 원하는 변화를 포함한 새로운 상태 트리를 리턴하고, Redux 스토어는 이 새로운 상테트리를 기억한다.

### 읽기

상태 트리 읽기는 대부분의 Redux 애플리케이션에서 분산되는 경향이 있다. 각각의 컨테이너 컴포넌트는 상태 트리의 일부분을 읽어서 컴포넌트의 props로 주입하는 기능을 하는 mapStateToProps 함수를 구현한다.

상태 트리의 불명확한 쓰임은 썽크 미들웨어로 동작하는 액션 생성자에서다. 썽크 액션들은 스토어의 getState 함수를 전달하며 액션을 생성할 때 이를 이용해 state에 접근할 수 있다.

상태 트리를 읽을지도 모르는 3번째 장소는 바로 리듀서 테스트들이다. 리듀서에 대한 테스트는 종종 리듀서가 액션을 올바르게 처리했는지를 알기 위해 상태 트리를 읽어야 할 필요가 있다.

## 그럼 이제부터 뭘 해야하는가?

이제 상태 트리가 어떻게 사용되는 지 알았으니, 뭘 하면 될까?

우선, 쓰기 방면에서는 이미 리듀서와 액션 생성자를 통해서 캡슐화 계층을 생성한다는 것을 알 수 있다. 우리의 애플리케이션은 상태 트리를 직접 다루지 않는다. 그보다는, 리듀서에 의해 처리되는 액션을 디스패치 하는 방법으로 상태 트리를 조작하게 한다. 더이상 캡슐화 할 내용이 없다.

읽기 방면에서는 상대적으로 쓰기 쪽 보다 덜 정의되어 있다. 읽기는 다양한 컨테이너 컴포넌트, 썽크 액션, 테스트 등지에 흩어져있기 때문에 상태 모양을 변경한 뒤 컴포넌트, 테스트, 썽크 액션 등에서 제대로 읽을 수 있도록 업데이트하기 힘들다.

드러난 모범 사례는 셀렉터라고 부르는 추상화 계층을 도입하는 것이다.

## 셀렉터

셀렉터는 상태(state) 및 추가적인 파라미터를 취하고 어떤 데이터를 반환하는 함수다.

모든 곳에서 state.todos를 사용하는 대신, 모든 todo를 리턴하는 allTodos(state)와 같은 셀렉터를 사용한다. 무의미한 것처럼 보일 수 있지만 상태 트리를 캡슐화하는 것은 중요하다.

일단 셀렉터를 사용하게 되면 나머지 애플리케이션 부분에서 todos가 상태 트리에 어떻게 저장되어 있는지 신경쓰지 않아도 되기 때문에, 필요할 때 훨씬 쉽게 리팩토링할 수 있게 된다.

셀렉터는 상태 트리에 있는 그대로 데이터를 리턴하거나, 혹은 어떤 연산을 수행한 다음 리턴할 수 있다.

Redux 문서에서는 셀렉터를 핵심 개념으로 다루지는 않치만 훌륭한 Reselect라는 라이브러리와 함께 파생 데이터를 계산하는 색션에서 이를 언급하고 있다.

그리고 Dan Abramov의 두 번째 Redux 비디오 시리즈에서, 특히 Redux:Colocating Selectors with Reducers 에서 철저히 다루어진다. 아직 비디오를 시청하지 않았다면, 강력 추천한다. Dan의 첫 번째 비디오 시리즈와 마찬가지로 매우 훌륭하다.

## 아직 더 남았나?

쓰기 부분에서 리듀서와 액션을 통한 캡슐화, 읽기 부분에서 셀렉터 처리를 통한 캡슐화, 이제 끝인가? 상태 트리를 안전하게 리팩토링할 수 있을까?

대부분의 파트에서는 그렇다, 끝난 것이다. 모든 코드가 액션과 셀렉터를 사용하고 결코 상태 트리에 직접 접근하지 않으면 우리는 상태 트리를 리팩토링해야할 때 무엇을 변경해야하는지 정확하게 알고 있다.

Dan Abramov는 심지어 캡슐화를 명확히 하기 위해서 셀렉터가 리듀서와 동일한 파일에 있어야 한다고 제안하고 있다. 나는 그렇게 하지는 않았지만, 어떤 이유인지는 확실히 이해한다.

하지만 테스트는? 테스트에서도 셀렉터를 사용하나?

## 테스팅

이전에 [Redux 애플리케이션 테스트하기](http://randycoulman.com/blog/2016/03/15/testing-redux-applications/)라는 글을 작성했다. 리듀서를 테스팅하는 엑션에서 Redux 문서에서 가져온 다음 테스트를 보았다.

Reducer Tests

```js
import { expect } from 'chai'
import reducer from 'reducers/todos'
import ActionTypes from 'constants/ActionTypes'

describe('todos reducer', () => {
  it('initializes the state', () => {
    expect(reducer(undefined, {})).toEqual([])
  })

  it('adds a todo', () => {
    expect(reducer([], addTodo('Run the tests'))).toEqual([
      {
        text: 'Run the tests',
        completed: false,
        id: 0
      }
    ])
  })
})
```

이 테스트에는 상태 트리의 모양을 알고 있는 부분이 몇 군데 있다. 상태 트리 모양과 결합된 모든 부분 중에서 가장 방어하기 쉬운 부분이다. 하지만 더 잘 할 수 있을까?

나는 리듀서 테스트 전략을 조금 발전시켜서, 현재 프로젝트에서는 완전히 상태 트리 모양과 테스트를 완전히 분리하는 실험을 하고 있다.

리듀서에 의해 생성된 초기 상태에 대한 테스트는 const를 정의하는 것에 의해 시작된다.

Initial State

```js
describe('todos reducer', () => {
  const initialState = reducer(undefined, {})

  # ...
})
```

만약 초기 상태에 대해 중요한 것이 있으면, 셀렉터를 이용해서 몇 가지 테스트를 작성할 것이다. 나는 상태 트리에 대한 테스트는 리듀서 구현 관점이 아니라, 상태 트리에 대한 클라이언트 관점에서 작성해야한다고 확신한다.

Testing the Initial State

```js
# ...

  describe('initial state', () => {
    it('starts with no todos', () => {
      expect(allTodos(initialState)).to.be.empty  
    })
  })

# ...
```

이 테스트는 todos가 저장되는 방법이나 초기 상태의 모양이 실제로 무엇인지에 대해서 전혀 언급하지 않는다. 단지 상태의 중요한 관측가능한 속성들을 테스트한다.

다양한 액션들을 핸들링하는 테스트를 작성하려면, 다음과 같이 작성할 것이다.

Testing an Action

```js
# ...

  describe('adding a todo', () => {
    const state = initialState
    const action = addTodo('TODO')
    const newState = reducer(state, action)
    const addedTodo = allTodos(newState)[0]

    it('includes the todo in the list', () => {
      expect(addedTodo).to.exist
    })

    it('remembers the name', () => {
      expect(addedTodo.name).to.eq('TODO')
    })

    it('assigns the next available id', () => {
      expect(addedTodo.id).to.eq(0)
    })

    it('marks the todo as initially incomplete', () => {
      expect(addedTodo.complete).to.be.false
    })
  })

# ...
```

내가 작성한 방식은 다소 장황하다; 하나의 테스트에서 추가된 todo의 모든 속성을 확인하는 것이 좋다. 나는 리듀서가 어떤 책임을 가지고 있는지 명확하게 하기 위한 의도를 전달하려는 목적으로 각 속성 테스트를 it 블럭으로 분리하는 경향이 있다.

나는 const state = ...; const action = ...; const newState = ...; 과 같은 패턴을 유지하는 경향도 있다. 테스트의 규칙적인 구조는 테스트를 읽기 쉽게 만든다.

만약 초기 상태가 아닌 다른 것에 대한 테스트를 작성한다면, 내가 필요한 상태를 얻기 위해 액션을 사용하는 실험을 하고 있다.

예를 들어, 이미 todo가 존재하는 todo list를 테스트하고 싶다면 다음과 같은 방법으로 시작할 것이다:

Starting With Other State

```js
# ...

  const state = reducer(initialState, addTodo('Pre-existing'))

# ...
```

시작 상태를 올바르게 하기 위해서 하나 이상의 액션이 필요하다면, 액션의 배열과 함께 reduce를 이용한다. 나는 Ramda를 이용한 버전의 reduce를 쓰고 있지만, 다른 선택지들도 있다.

Multiple Actions

```js
# ...
  const state = reduce(reducer, initialState, [
    addTodo('Already complete'),
    completeTodo(0),
    addTodo('Still outstanding')  
  ])
# ...
```

나는 지금까지 이 분리 개념이 정말 좋았지만, 이 상태 설정 스타일이 미래의 코드 독자들에게도 충분히 이해할만한 것인지는 배심원들에게 달려있다.

## 중첩된 리듀서는 어떤가?

지금까지 얘기한 모든 것들은 탑레벨 리듀서 하나만 가지고 있을 때만 효과적이다. 셀렉터와 리듀서는 잘 동작하고 테스팅에 대한 접근 방법도 매력적이다.

하지만 대부분의 Redux 애플리케이션에서는 combineReducers를 사용해서 탑레벨 리듀서를 서브 리듀서들로 나누고 있다. 각 서브 리듀서들은 상태 트리에서 잘라낸 작은 부분에 대해서 동작한다. 그러나 셀렉터들은 전체 상태 트리와 함께 작업해야 한다.

이런 리듀서와 셀렉터 사이의 비대칭성을 어떻게 처리해야하는가?

다음 주 게시물을 위해서 대답을 아껴두겠다.

## 결론

이미 우리가 갖고 이쓴 액션과 리듀서에다가 셀렉터를 추가하여 Redux 상태를 캡슐화하는 것을 강력 추천한다.

한단계 더 나아가서 액션과 셀렉터만 사용해서 리듀서 테스트를 작성하는 것은 아직까지는 괜찮은 접근방법으로 보인다. 리듀서와 셀렉터만 변경하여 상태 트리를 리팩토링할 수 있었기 때문이다. 특히 리듀서 테스트는 아무 것도 변경할 필요가 없었다는 것을 참고하라.
