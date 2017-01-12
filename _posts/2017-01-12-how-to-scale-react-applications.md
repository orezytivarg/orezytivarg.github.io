---
id: how-to-scale-react-applications
title: React 애플리케이션을 확장하는 방법
category: Redux
---
[원문보기](https://www.smashingmagazine.com/2016/09/how-to-scale-react-applications/)

우리는 최근 몇 달 간의 작업 끝에, 가장 인기있는 React 스타터 킷 중 하나인 React Boilerplate의 버전3를 릴리즈했다. 팀은 수백명의 개발자가 있는 팀은 어떻게 웹 애플리케이션을 만들고 확장하는지에 대해서 이야기했고, 우리가 그 과정에서 배운 것을 공유하고자 한다.

우리는 작업 초기에 "그저 또다른 보일러플레이트"가 되는 것을 원치 않았다. 우리는 제품 개발을 시작하는 개발자에게 확장성 있는 최고의 개발 토대를 제공하려고 했다.

전통적으로, 소프트웨어 규모의 확장은 대부분 서버사이드 시스템과 관련있었다. 그리고 점점 많은 사용자가 당신의 애플리케이션을 사용할 것이므로, 클러스터에 더 많은 서버를 클러스터에 추가할 수 있는지, 데이터베이스를 여러 서버로 분할할 수 있는지 등을 확인해야할 필요가 있었다.

최근에는 리치 웹 에플리케이션 때문에 프론트엔드 쪽에서도 역시 규모의 확장이 더 중요한 주제가 되었다. 복잡한 앱의 프론트엔드는 더 많은 수의 사용자들과 개발자들과 파트들을 처리할 수 있어야 한다. 이러한 3가지 카테고리의 확장(사용자, 개발자, 파트)을 고려해야 한다. 그렇지 않으면 문제가 발생할 것이다.

# 컨테이너들과 컴포넌트들

큰 애플리케이션의 명확한 첫 번째 개선점은 상태가 있는 컴포넌트("컨테이너")와 상태가 없는 컴포넌트("컴포넌트") 간의 차이다. 컨테이너는 데이터를 관리하거나 상태와 연결되어 있고 일반적으로 그들과 연관된 스타일링을 가지고 있지 않다. 한편, 컴포넌트는 그들과 연관된 스타일링을 가지고 있으며 어떤 데이터나 상태 관리에 대한 책임을 가지고 있지 않다. 처음에는 이것이 혼동되었다. 기본적으로 일이 어떻게 작동하는지 책임을 지고 구성 요소는 일이 어떻게 보이는지에 대해 책임을 진다.

이와 같이 컴포넌트를 분리하면 재사용 가능한 컴포넌트와 데이터를 관리하는 중간 계층을 명확하게 구분할 수 있다. 따라서, 데이터 구조가 엉망이 될 염려 없이 컴포넌트를 수정할 수 있고, 스타일링이 엉망이 될 염려 없이 컨테이너를 수정할 수 있다. 그리하여 애플리케이션의 동작을 추론하는 것이 훨씬 더 쉬워지고 명확하게 된다!

# 구조

전통적으로, 개발자들은 React 애플리케이션을 타입에 따라 구조화했다. 이것이 뜻하는 바는 애플리케이션이 actions/, components/, containers/ 등과 같은 폴더를 가지고 있다는 뜻이다.

NavBar라고 불리는 네비게이션 바 컨테이너를 생각해보자. NavBar는 몇몇 연관된 상태를 가지고 있고 네비게이션 바를 열고 닫는 toggleNav과 같은 액션을 가지고 있을 것이다. 이것이 파일을 타입별로 그룹지어 구조화하는 방법이다.

```
react-app-by-type
		├── css
		├── actions
		│   └── NavBarActions.js
		├── containers
		│   └── NavBar.jsx
		├── constants
		│   └── NavBarConstants.js
		├── components
		│   └── App.jsx
		└── reducers
		    └── NavBarReducer.js
```

이 예제에서는 잘 동작하겠지만, 수백 혹은 잠재적으로 수천 개의 컴포넌트가 있을 수 있는데, 이런 경우에는 개발이 매우 힘들어진다. 기능을 추가하려면 수천 개의 파일이 있는 수십 개의 서로 다른 폴더에서 올바른 파일을 검색해야 한다. 이렇게 되면 쉽게 지칠 뿐더러 코드 베이스에 대한 확신도 줄어들 것이다.

Github의 이슈 트래커에서 오랜 논의 끝에 다양한 구조를 시도한 결과, 우리는 훨씬 더 나은 해결책을 발견했다고 생각한다:

애플리케이션의 파일을 유형별로 그룹화하는 대신 기능별로 그룹화하라! 즉, 한 기능(예: 네비게이션 바)과 관련된 모든 파일을 같은 폴더에 넣는다.

NavBar 예제에서 폴더 구조가 어떤지 확인해보자:

```
react-app-by-feature
		├── css
		├── containers
		│    └── NavBar
		│        ├── NavBar.jsx
		│        ├── actions.js
		│        ├── constants.js
		│        └── reducer.js
		└── components
		    └── App.jsx
```

이 애플리케이션의 개발자는 무언가를 작업하기 위해 하나의 폴더로 이동해야 한다. 그리고 새로운 기능을 추가하기 위해서 이 폴더 아래 단 하나의 폴더만 만들어야 한다. 찾기 및 바꾸기 기능으로 이름 바꾸기는 쉽고, 수백명의 개발자가 충돌없이 한 번에 동일한 애플리케이션에서 작업할 수 있다!

내가 처음으로 이렇게 작성하는 방법에 대해 알았을 때 생각하기를 "왜 그렇게 했었을까? 다른 방법으로도 잘 동작하네!" 나는 내 오픈 마인드에 자부심을 느끼고, 작은 프로젝트에서 이 구조화를 시도해봤다. 15분 후에 감동받았다. 내 코드 베이스에 대한 자신감은 막대했고, 그 위에서 일하는 것은 매우 쉬웠다.

이것이 반드시 redux 액션과 리듀서를 해당 컴포넌트에서만 사용할 수 있다는 것을 의미하는 것은 아니라는 것에 주의하라. 다른 컴포넌트에서도 얼마든지 임포트하여 사용될 수 있다!

이런 식으로 일하는 동안 두 가지 질문이 내 머리 속에서 튀어나왔다. "어떻게 스타일링을 처리할 수 있을까?" 그리고 "어떻게 원격 데이터를 가져올 수 있을까?" 이 질문들을 각각 다뤄보겠다.

# 스타일링

아키텍쳐를 위한 결정 외에도, 컴포넌트 기반 아키텍쳐에서 CSS를 사용하는 것은 언어 자체의 두 가지 특정 속성 때문에 어렵습니다: 글로벌 네임 충돌과 상속

## 유니크 클래스 네임

커다란 어플리케이션 내 어딘가의 CSS를 상상해보자:

```css
.header { /* … */ }
.title {
	background-color: yellow;
}
```

즉시 문제가 있음을 알 수 있다: title은 매우 일반적인 이름이다. 다른 개발자(혹은 시간이 흐른 후의 같은 개발자)가 다음과 같은 코드를 작성할 수 있다.

```css
.footer { /* … */ }
.title {
	border-color: blue;
}
```

이름 충돌이 발생하여 갑자기 파란색 테두리와 노란색 배경의 title이 도처에 나타나며, 모든 것을 엉망으로 만든 이 선언을 찾기 우해 수천 개의 파일을 파고들어야 할 것이다.

고맙게도, 소수의 똑똑한 개발자들이 있어서 CSS 모듈이라는 이름의 이 문제에 대한 해결책을 제시했다. 그들의 접근 방식의 핵심은 폴더에 있는 컴포넌트에 스타일을 함께 배치하는 것이다.

```
	react-app-with-css-modules
		├── containers
		└── components
		     └── Button
		         ├── Button.jsx
		         └── styles.css
```

특정한 명명 규칙을 염려할 필요가 없다는 것을 제외하면 CSS는 완전히 동일해 보인다. 그리고 매우 일반적인 이름을 지정할 수 있다:

```css
.button {
	/* … */
}
```

그러고나서 CSS 파일을 컴포넌트에 넣거나 가져오고 JSX 태그에 styles.button을 className으로 지정한다:

```js
/* Button.jsx */
var styles = require('./styles.css');

<div className={styles.button}></div>
```

브라우저에서 DOM을 살펴보면 <div class="MyApp__button__1co1k"></div>와 같이 표시된다! CSS 모듈은 애플리케이션 이름을 추가하고 클래스 내용의 짧은 해시 뒤에다 추가하여 클래스 이름을 "고유하게" 관리한다. 즉, 클래스가 겹칠 가능성은 거의 없다. 혹여 중복되는 경우라도 상관없다.(해시 - 즉 스타일 내용이 동일하므로)

# 각 컴포넌트를 위한 속성 리셋

CSS에서 특정 속성은 여러 노드에 걸쳐 상속된다. 예를 들어, 부모 노드에 line-height가 설정되어 있고 하위에 지정된 것이 없으면 자동으로 부모 노드와 동일한 line-height가 적용된다.

컴포넌트 기반 아키텍쳐에서는 이런 것을 원하지 않는다. 다음과 같은 스타일을 가진 Header와 Footer 컴포넌트를 상상해보자:

```css
.header {
	line-height: 1.5em;
	/* … */
}

.footer {
	line-height: 1;
	/* … */
}
```

우리가 이 두 컴포넌트 내부에 Button을 렌더링하자고 가정하자. 갑자기 헤더와 푸터에서 다른 모양의 버튼이 보이게 된다! line-height 뿐만 아니라 CSS 속성들도 상속된다. 애플리케이션에서 이러한 버그를 추적하고 제거하는 것은 굉장히 어려울 것이ㅏ.

프론트엔드 세상에서, 스타일 시트를 리셋해서 브라우저간에 스타일을 표준화 하는 것은 아주 일반적이다. Reset Css, Normalize.css sanitize.css등의 옵션이 인기있다! 만약 이런 개념을 가지고 모든 컴포넌트를 리셋하면 어떨까?

이를 auto-reset이라고 부르고, PostCSS용 플러그인이 존재한다! 만약 PostCss Auto Reset을 PostCSS 플러그인으로 추가하면, 각 컴포넌트를 로컬 리셋으로 감싸고, 모든 상속 가능한 특성을 그들의 기본값으로 설정하여 상속을 오버라이드한다.

# Data-Fetching

이 아키텍쳐와 관련된 두 번째 문제점은 원격 데이터를 가져오는 것이다. 컴포넌트와 그 작업을 함께 배치하는 것은 대부분의 작업에 적합하지만 원격 데이터를 가져오는 것은 본질적으로 단일 컴포넌트에 묶여있지 않은 전역 작업이다!

대부분의 개발자는 Redux로 원격 데이터를 가져올 때 Redux Thunk를 사용한다. 일반적인 thunked 액션은 다음과 같다:

```js
/* actions.js */

function fetchData() {
	return function thunk(dispatch) {
		// Load something asynchronously.
		fetch('https://someurl.com/somendpoint', function callback(data) {
			// Add the data to the store.
			dispatch(dataLoaded(data));
		});
	}
}
```

이것은 원격 데이터를 가져올 수 있는 훌륭한 방법이지만 두 가지의 문제점이 있다: 함수를 테스트하는 것이 매우 어렵고, 개념적으로는 액션에서 원격 데이터를 가져오게 하는 것이 옳은 것처럼 보인다는 것이다.

Redux의 큰 장점은 테스트하기 쉬운 순수한 액션 생성자다. 액션에서 썽크를 반환하게 하면 우리는 액션을 2번 호출하고 dispatch 함수를 목킹하는 작업 등을 해야한다.

최근의 새로운 접근방식이 React 세계에 폭풍을 불러왔다: redux-saga. redux-saga는 Esnext의 generator를 사용하여 비동기 코드를 동기식으로 보이게 만들고 매우 쉽게 테스트할 수 있게 한다. saga 뒤에 숨겨진 모든 멘탈 모델은 애플리케이션의 나머지 부분을 괴롭히지 않고도 모든 비동기 작업을 처리하는 별도의 스레드와 같이 만들어준다!

예제로 설명하겠다:

```js
/* sagas.js */

import { call, take, put } from 'redux-saga/effects';

// The asterisk behind the function keyword tells us that this is a generator.
function* fetchData() {
	// The yield keyword means that we'll wait until the (asynchronous) function
	// after it completes.
	// In this case, we wait until the FETCH_DATA action happens.
	yield take(FETCH_DATA);
	// We then fetch the data from the server, again waiting for it with yield
	// before continuing.
	var data = yield call(fetch, 'https://someurl.com/someendpoint');
	// When the data has finished loading, we dispatch the dataLoaded action.
	put(dataLoaded(data));
}
```

이상한 모양의 코드에 겁먹지 마라: 비동기식 흐름을 처리하는 훌륭한 방법이다!
콜백 헬을 피할 수 있고, 거의 소설처럼 쉽게 읽히는 코드, 게다가 테스트하기도 쉽다. 자, 스스로에게 물어보라, 왜 테스트하기가 쉬울까? 이유는 완료되지 않은 상태에서도 "effect"들을 노출하는 redux-saga의 능력 때문이다.

파일의 맨 위에서 임포트한 effects는 redux 코드와 상호작용할 수 있도록 도와주는 핸들러다.

>put() 우리 saga로부터 액션을 디스패치한다.

>take() 앱에서 액션이 일어날때까지 일시 중지한다.

>select() redux의 state의 일부분을 가져온다.(mapStateToProps 같은 역할)

>call() 첫 번째 인수를 나머지와 함께 전달하여 함수를 호출한다.

왜 이러한 effects들이 효과적인가? 테스트 예제를 한번 보자:


```js
/* sagas.test.js */

var sagaGenerator = fetchData();

describe('fetchData saga', function() {
	// Test that our saga starts when an action is dispatched,
	// without having to simulate that the dispatch actually happened!
	it('should wait for the FETCH_DATA action', function() {
		expect(sagaGenerator.next()).to.equal(take(FETCH_DATA));
	});

	// Test that our saga calls fetch with a specific URL,
	// without having to mock fetch or use the API or be connected to a network!
	it('should fetch the data from the server', function() {
		expect(sagaGenerator.next()).to.equal(call(fetch, 'https://someurl.com/someendpoint'));
	});

	// Test that our saga dispatches an action,
	// without having to have the main application running!
	it('should dispatch the dataLoaded action when the data has loaded', function() {
		expect(sagaGenerator.next()).to.equal(put(dataLoaded()));
	});
});
```

Exnext의 generator는 generator.next()가 호출될 때까지 yield 키워드를 지나가지 않는다. generator는 다음 yield 키워드를 만나기 전까지 함수를 실행한다! redux-saga effects를 사용함으로 테스트를 위해 네트워크에 의존하지 않아도 되고 무언가를 모킹할 필요도 없다. 아주 쉽게 비동기 작업을 테스트를 해볼 수 있다.

한편, 우리는 테스트 파일들도 역시 테스팅중인 파일들과 함께 배치한다. 왜 다른 폴더에 넣어야 하는가? 이렇게 하면 컴포넌트와 관련된 모든 파일이 실제로 테스트할 때도 같은 폴더에 있게된다!

redux-saga의 이점이 여기서 끝난다고 생각한다면, 큰 실수다! 사실, 원격 데이터 가져오기를 쉽게 하고, 아름답고 테스트하기 쉬운 코드는 redux-saga의 가장 작은 이점일 뿐이다!

## redux-saga를 모르타르로 사용하기

컴포넌트들은 이제 분리되었다. 다른 스타일이나 로직에 신경쓰지 않는다; 자신의 사업에만 관심을 갖고있다 -- 음, 거의 대부분은.

Clock과 Timer 컴포넌트를 상상해보자. 시계의 버튼을 누르면, 타이머가 시작된다; 그리고 타이머의 정지 버튼을 누르면 시계에 시간을 표시하려고 한다.

일반적으로는 다음과 같이 했을 것이다:

```js
/* Clock.jsx */

import { startTimer } from '../Timer/actions';

class Clock extends React.Component {
	render() {
		return (
			/* … */
			<button onClick={this.props.dispatch(startTimer())} />
			/* … */
		);
	}
}

/* Timer.jsx */

import { showTime } from '../Clock/actions';

class Timer extends React.Component {
	render() {
		return (
			/* … */
			<button onClick={this.props.dispatch(showTime(currentTime))} />
			/* … */
		);
	}
}
```

갑자기 이 컴포넌트들을 분리하여 사용할 수 없게되고, 재사용은 거의 불가능해진다!

이렇게 하는 대신, redux-saga를 분리된 컴포넌트 사이의 "모르타르"로 사용할 수 있다. 다른 액션을 리스닝함으로써, 애플리케이션에 따라 다양한 방식으로 대응할 수 있다. 즉 컴포넌트가 이제 완전히 재사용 가능해진다는 뜻이다.

컴포넌트를 수정해보자:

```js
/* Clock.jsx */

import { startButtonClicked } from '../Clock/actions';

class Clock extends React.Component {
	/* … */
	<button onClick={this.props.dispatch(startButtonClicked())} />
	/* … */
}

/* Timer.jsx */

import { stopButtonClicked } from '../Timer/actions';

class Timer extends React.Component {
	/* … */
	<button onClick={this.props.dispatch(stopButtonClicked(currentTime))} />
	/* … */
}
```

각 컴포넌트가 자체적인 액션만 import하고 관심을 같는 것을 유의하라!
자, 이제 사가를 사용해서 분리된 컴포넌트를 하나로 뭉쳐보자:

```js
/* sagas.js */

import { call, take, put, select } from 'redux-saga/effects';

import { showTime } from '../Clock/actions';
import { START_BUTTON_CLICKED } from '../Clock/constants';
import { startTimer } from '../Timer/actions';
import { STOP_BUTTON_CLICKED } from '../Timer/constants';

function* clockAndTimer() {
	// Wait for the startButtonClicked action of the Clock
	// to be dispatched.
	yield take(START_BUTTON_CLICKED);
	// When that happens, start the timer.
	put(startTimer());
	// Then, wait for the stopButtonClick action of the Timer
	// to be dispatched.
	yield take(STOP_BUTTON_CLICKED);
	// Get the current time of the timer from the global state.
	var currentTime = select(function (state) { return state.timer.currentTime });
	// And show the time on the clock.
	put(showTime(currentTime));
}
```

아름답다.

# 요약

기억해야할 주요 사항은 다음과 같다:

> 컨테이너와 컴포넌트를 구별하라.
> 파일을 피처 단위로 구조화하라.
> CSS 모듈과 PostCSS Auto Reset을 사용하라.
> redux-saga를 사용하라 - 다음 목적으로:
    비동기 흐름을 테스트하고 읽기 쉽게하고, 분리된 컴포넌트를 합쳐라.
