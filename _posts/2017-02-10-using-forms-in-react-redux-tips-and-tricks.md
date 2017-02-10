---
layout: post
id: using-forms-in-react-redux-tips-and-tricks
title: React-Redux에서 폼 사용하기 - 팁과 트릭들
tags: [React, Redux, Redux-Form]
---
[원문보기](https://hackernoon.com/using-forms-in-react-redux-tips-and-tricks-48ad9c7522f6#.rjayg450p)

예전에 React와 Redux로 옮겨가면서 배운 몇 가지 교훈에 관한 [포스트](https://medium.com/@royisch/6-lessons-learned-from-going-to-production-with-react-redux-19257f6724f6#.xu43yqlko)를 작성했었다.

그 뒤로 꽤 긴 시간이 지났지만 아직도 React와 그 친구들을 사용하고 있고, 더 공유해보고 싶은 것이 있다. 무엇이 가장 큰 고통이며 어떻게 그것을 이겨낼 것인가 하는 것 말이다. 그것은 바로 forms를 구현하는 것이다.

많은 웹 애플리케이션들은 폼 기반이다. 당신의 플랫폼을 사용자에게 제공하려는 목적으로 사용자로 하여금 폼을 채우고 폼 컴포넌트를 조작하게끔 한다. 곧, 폼 컴포넌트를 React에서는 어떻게 사용하고 작성해야 하는지에 대한 의문이 생긴다. 그리고 React와 Redux가 어떻게 함께 동작하는지도 말이다. 예를 들어 다음과 같은 의문들이다.

 - \# 컴포넌트는 어떻게 동작해야 하는가?
 - \# 데이터를 어디에 유지해야 하는가?
 - \# 유효성 검사는 어떻게 수행하는가?
 - \# 폼의 상태와 그 변화는 어떻게 탐지하는가?

[Bizzabo](https://www.bizzabo.com/)에서는 꽤 많은 페이지들이 폼 기반이다. 이 페이지들은 꽤 오래전부터 해왔던 것이기 때문에 Backbone.js를 사용했던 동안 어떤 도전들이 있었고 어떤 가이드가 필요한지 알고 있었다. 그러나 React는 또다른 경기장이였고, 사용자가 수월하게 폼의 내용을 채우게 하기 위해서는  Backbone을 사용한 폼이나 React를 사용한 폼이나 차이점을 느끼지 못해도록 해야 한다. 그러나 한편으로는 이 모든 물음들에 대한 대답을 위해 폼을 사용하는 방법을 완전히 새롭게 설계했다.

이 물음들에 대답하기에 앞서, 결과부터 이야기하자면 우리는 [redux-form](http://redux-form.com/)을 사용했다. [Erik Rasmussen](https://github.com/erikras)에 의해 만들어진, redux 위에서 동작하는 폼 상태 관리 라이브러리로, 매우 잘 관리되고 있다.

## \# 컴포넌트가 어떻게 동작해야 하는가?

시스템 내의 모든 폼 컴포넌트는 사용자 인터페이스 동작을 가지고 있어야만 한다. 예를 들어 input을 보자면, 많은 상태를 가지고 있다: 유효한지, 터치되었는지, 포커스되었는지, 더티상태인지 등등. 당신은 이러한 모든 상태가 시각적으로 표현되기를 원할 것이다. 에러나 경고를 표시하는 등 말이다. 우리는 Redux-form의 솔루션을 선택했다. 이 솔루션은 컴포넌트의 상태에 대한 메타 정보를 제공한다. 과연 redux form 컴포넌트를 어떻게 사용할까? <Field /> 컴포넌트는 어떻게 각각 개별적인 input과 어떻게 Redux 스토어를 연결시킬까?(다음 주제에서 이를 연마할 것이다)

```js
<Field name={keyName} component={MyCustomInput} {...this.props} />
```

이를 활성화하기 위해서 해야할 것은 단 한가지다. 위 구문처럼 컴포넌트를 `<Field />` 컴포넌트로 name과 함께 감싸는 것이다. 그러면 당신이 원하는 대로 컴포넌트는 UI 상태들을 표시할 수 있게 된다. redux form의 `<Field />`를 생성하는 여러 방법들이 있으니 이 [문서](http://redux-form.com/6.2.1/docs)를 참조하라.

따라서 이제 UI의 각 상태를 다음과 같이 추가할 수 있다.

```js
const {meta: {error, touched, dirty}} = this.props;
<div>
     <input
       disabled={isDisabled}
       type={type || "text"}
       placeholder={placeholder || 'Enter a value'}
       name={name}
     />
</div>
{error && <label className="error">{error}</label>}
{touched && <label className="touched">{touched}</label>}
{dirty && <label className="dirty">{dirty}</label>}
```

위에서 보듯이 `<Field />`를 사용하면 공짜로 UI 상태를 얻을 수 있다. 그리고 상태에 따라 알맞은 알림을 표시할 수 있게 된다.

## \# 데이터를 어디에 유지해야 하는가?

주요 질문 중 하나는 데이터를 어디에 유지해야 하는 가이고 이에 대해 대답할 필요가 있겠다. 그 대답을 위해서 먼저 걱정거리들을 찾아보자:
모든 폼 요소들은 초기값을 가지고 있어야만 하고, 폼 중 하나는 "new"라는 상태를 가질 때 비워져야 하거나, "edit"라는 상태를 위한 값을 가져야할 수도 있다. blur 혹은 keyup과 같은 어떠한 변화(혹은 관심을 가져야할 만한 어떠한 이벤트)에서도 값이 변경되어야 한다. 그래서, 어디에 둔단 말인가?

Redux-form의 솔루션은 Redux reducer다. 이 reducer들은 redux-form의 액션 디스패치를 듣고서는 Redux 내의 폼 상태를 관리한다. 폼들은 "form" key name 내에 마운트된다. 그리고 모든 폼은 고유한 이름을 가지고 있어서 이를 form[name]을 통해 접근할 수 있다.

아래의 코드를 보면 쉽게 이해할 수 있다.

```js
import { createStore, combineReducers } from 'redux'
import { reducer as formReducer } from 'redux-form'

const reducers = {
  event,
  account,
  user,
  form: formReducer     
}
const reducer = combineReducers(reducers)
const store = createStore(reducer)
```

redux-form은 초기값과 현재값을 위한 셀렉터를 제공한다. 이것을 사용하는 것이 훌륭한 이유는 그것이 바로 redux 자체라는 것이다. 특정 커스텀 액션을 듣고서 데이터를 그에 맞게 조작하는 것이다. redux-form만의 어떤 액션이 아니고 당신이 만든 특정 액션에 대해서도 말이다. [그저 플러그인 함수만 사용하면 된다.](http://redux-form.com/6.2.1/docs/api/ReducerPlugin.md/)

지금까지는 컴포넌트의 현재 상태를 표시하는 것에 대해 이야기하고, 데이터를 관리하는 위치에 대해서 이야기했다.

그러나 데이터 자체를 다루는 것은 어떤가?

## \# 유효성 검사는 어떻게 수행하는가?

모든 폼들에서 가장 주요한 도전과제는 유효성 검사다. 사용자 입력을 제출할 때(혹은 그에 준하는 어떤 브라우저 이벤트든 간에) 폼 컴포넌트와 그 값이 업데이트 하기 충분하지 않을 때 값에 대한 유효성 검사가 필요해진다. 대답해야할 질문의 이슈는 다음과 같다: 폼 컴포넌트의 값을 상태에 두고 유효성 검사가 통과하면 스토어로 전달해야하나? 아니면 스토어에 있는 모든 데이터에 대해 유효성 검사를 수행해야 하나?

Redux form은 모두에 대한 대답을 한다. 유효성 검사는 라이브러리의 에코시스템 중 한 부분이다. redux form을 이용해 생성한 폼에 validation method를 전달할 수 있다. 혹은 특정한 `<Field />`에 대해서 (우리만의 특정한 버전으로부터)유효성 검사를 수행할 수도 있다. 1가지 제약 사항만이 존재한다. 바로 error 오브젝트의 구조다. 다음과 같다. `{ field1: <string>, field2: <string> }`. 3rd 파티 라이브러리를 사용하거나 직접 validation 메서드를 구현할 수 있다. 우리는 후자를 택해서 [validator.js](https://github.com/skaterdav85/validatorjs)를 사용한다. 이 라이브러리는 거대한 양의 유효성검사를 커버하고, 특별한 검사를 위한 플러그인 기능을 지원한다. 그리고 가장 중요한 것은 redux form이 요구하는 error 오브젝트를 반환한다.

```js
const validate = (data, props) => {

    const rules = {
        'ga-id': ['regex:/^UA-\\d{4,10}-\\d{1,4}$/'],
        'input': 'max:50 | required'
    };

    const messages = {
        'max.input': 'This is my max value text',
        'required.input': 'This is my required text'
        'regex.ga-id': 'My Google Analytics id format text'
    };

    const validator = new Validator(data, rules, messages);
    validator.passes();
    return validator.errors.all();
};
```
*validator.js를 사용한 유효성 검사*

위 예제를 통해 유효성 검사를 살펴보자. 몇몇 검사를 각각 필드(예. input)에 제공할 수 있고, 각각 에러에 대한 텍스트를 제공할 수 있다. 그리고 검사를 한 번 수행한 다음 error 오브젝트를 반환하면 된다.

## \# 폼의 상태와 그 변화는 어떻게 탐지하는가?

우리의 마지막으로 풀어야할 큰 항목은 폼의 현재 상태에 대한 탐지와 변화에 대한 추적이다. 예를 들어보자. 우리가 원하는 것은 원래 값으로부터 변했는지 여부다. 사용자가 다른 곳으로 이동하기 전에 저장하지 않은 변경사항이 있는지 알려주기 위한 것이다. Redux Form은 현재 폼 상태에 대한 완전한 시야를 제공한다. dirty, pristine 등등과 심지어 사용자에 의한 touched 여부까지 말이다(해당 요소에 포커싱을 준 적이 있는지). 그리고 redux 액션을 통해 상태 변화를 유발하는 것을 허용하는 외부 API도 제공한다. 예를들어 폼을 제출할 때나, 혹은 폼을 리셋할 때나 redux 액션을 사용해서 실제 제출 버튼을 누를 필요가 없다면 실제 제출 버튼 없이도 이를 수행할 수 있다.

## 결론

폼은 언제나 까다롭다. 모든 폼들은 어떠한 요구사항이 있고 가끔은 회피하기 위한 솔루션이 필요할때도 있다. 하지만 대부분의 시나리오는 redux-form에 의해 커버할 수 있다. 그리고 그게 안된다면 당신만의 솔루션을 redux form으로 마이그레이션할 수 있다. 우리에게는 훌륭하게 동작하고 있으므로, 당신에게도 그렇게 되었으면 좋겠다.

[Bizzabo](https://www.bizzabo.com/about#career)에서 함께 일합시다! 합류하세요!
