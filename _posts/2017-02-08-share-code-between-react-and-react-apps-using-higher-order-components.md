---
layout: post
id: share-code-between-react-and-react-apps-using-higher-order-components
title: Higher Order Components를 사용해서 React와 React App 간 코드 공유하기
tags: [React, React-Native]
---
[원문보기](https://hackernoon.com/code-reuse-using-higher-order-hoc-and-stateless-functional-components-in-react-and-react-native-6eeb503c665#.t3htnilwr)

개발자들은 Higher Order Components(HOC), Stateless Functional Components를 적용하고 있고 그럴만한 이유가 있다: 그것들이 개발자들의 열망 중 하나인 코드 재사용을 보다 쉽게 만들어주기 때문이다.

HOC와 Functional Stateless Components에 대한 많은 글들이 있다. 몇몇은 소개글이고, 몇몇은 깊이 파고드는 관점에서 서술된 것이다. 나는 기존 컴포넌트를 리팩토링해서 재사용가능한 엘레멘트를 만드는 관점에서 이 글을 쓴다.

당신은 아마도 코드 재사용이 과대 평가되었다고 생각할 것이다. 혹은 그게 너무 어렵다고 생각할 것이다. 특히 웹과 모바일 간에 코드를 공유하는 관점에서 말이다. 그러나 여기 몇 가지 고려해볼만한 장점이 있다:

 - UX 일관성. 디바이스 사이에서 혹은 애플리케이션 사이에서
 - 교차 보정 업그레이드하기: 사용되고 있는 모든 컴포넌트를 향상시키고 업데이트하는 일들
 - 라우팅과 인증 규칙들을 재사용하기
 - 라이브러리 전환(예를 들어, 애플리케이션 상태관리에 [MobX](https://mobx.js.org/)를 사용하고 있는 앱들을 [Redux](http://redux.js.org/)로 바꾸는 일)

나는 재사용을 달성하기 위한 HOC와 Functional Stateless Component에 중점을 둘 것이다. 당신은 [React](https://facebook.github.io/react/)와 [React Native](https://facebook.github.io/react-native/)에 대해 익숙해야만 한다. Alexis Mangin이 그들의 차이점에 대해 설명한 좋은 [글](https://medium.com/@alexmngn/from-reactjs-to-react-native-what-are-the-main-differences-between-both-d6e8e88ebf24#.rsas03guz)을 썼다.

> 이 글에는 많은 양의 세부 사항이 있다. 나는 컴포넌트 리팩토링에 대해 점차적인 접근 단계로 설명할 것이다. 만약 이러한(HOC같은) 아이디어에 친숙하다면, 시간을 절약하기 위해서, 혹은 인내하기 힘들다면, [The Payoff: Reusing the Components](https://hackernoon.com/code-reuse-using-higher-order-hoc-and-stateless-functional-components-in-react-and-react-native-6eeb503c665#5331)([최종 Github 저장소](https://github.com/csepulv/search-box))로 점프하라. 재사용된 컴포넌트들을 이용해서 추가적인 애플리케이션을 만드는 것이 얼마나 쉬운 일인지 알게될 것이다.

## Higher Order Components와 Stateless Functional Components란 무엇인가?

React 0.14에서 Stateless Functional Components가 도입되었다. 바로 컴포넌트를 렌더링하는 함수들이다. 문법이 더 간단하다. 클래스 정의나 컨스트럭터는 존재하지 않는다. 그 이름이 뜻하는 바와 같이 상태 관리도 없다(setState를 쓰지 않는다). 조금 뒤에서 튜터리얼의 후반부에 예제를 가지고 더 이야기하겠다.

Cory House가 좋은 [소개글](https://hackernoon.com/react-stateless-functional-components-nine-wins-you-might-have-overlooked-997b0d933dbc#.fxlvlf378)을 썼다.

Higher Order Components(HOC)는 새로운 컴포넌트를 만들어내는 함수다. 다른 컴포넌트(혹은 컴포넌트들)를 감싸고, 감싸진 컴포넌트를 캡슐화한다. 예를들면, 간단한 텍스트 박스를 상상해보자. 여기에 자동완성 기능을 추가하고 싶다. 그렇다면 HOC를 만들어서 이를 통해 텍스트박스를 감싸면 자동완성 텍스트 박스를 사용할 수 있다.

```js
const AutocompleteTextBox = makeAutocomplete(TextBox);
export AutocompleteTextBox;

//…later

import {AutoCompleteTextBox} from ‘./somefile’;
```

페이스북의 문서가 [여기](https://facebook.github.io/react/docs/higher-order-components.html) 있다. franleplat가 또한 상세한 내용의 [글](https://medium.com/@franleplant/react-higher-order-components-in-depth-cf9032ee6c3e#.d1838xba3)을 썼다.

우리는 앞으로 HOC와 Stateless Function Components를 몇몇 지점에서 사용할 것이다.

## 샘플 애플리케이션

우리는 매우 간단한 애플리케이션으로 시작할 것이다. 단순한 서치 박스다. 쿼리를 입력하면 결과 목록을 얻을 수 있다. 우리 경우에는 이름으로 색상을 찾을 것이다.

화면이 하나뿐안 애플리케이션이다. 컴포넌트 재사용에 집중하기 위해 라우트나 여러 화면으로 구성된 애플리케이션을 사용하지는 않을 것이다.

두 번째로, 우리는 애플리케이션 한 쌍을 추가할 것이다(React와 React Native). 컴포넌트를 추출해서 재사용할 애플리케이션들이다.

이 [Github 저장소 브랜치](https://github.com/csepulv/search-box/tree/baseline-apps)에 시작점의 애플리케이션이 있다(마지막 결과는 [여기](https://github.com/csepulv/search-box/)다). React(web), React Native(mobile)에 대한 전체 세부 사항을 [README](https://github.com/csepulv/search-box/blob/baseline-apps/README.md)에 적어두었다. 그러나 여기에서 개요를 설명한다:

 - [create-react-app](https://github.com/facebookincubator/create-react-app)을 통해 React 애플리케이션을 시작한다
 - React/web 애플리케이션에는 [Material UI](http://www.material-ui.com/#/)를 사용한다
 - [react-native init](https://facebook.github.io/react-native/docs/getting-started.html)을 통해 React Native 애플리케이션을 시작한다
 - 앱 상태 관리는 [MobX](https://mobx.js.org/)를 이용한다. (Michel Weststrate, Mobx의 창시자가 훌륭한 튜토리얼을 [이곳](https://medium.com/@mweststrate/interactive-introduction-to-mobx-and-reactjs-1760e448103c#.vqfntf8r1)과 [이곳](https://hackernoon.com/the-fundamental-principles-behind-mobx-7a725f71f3e8#.bg6q97rp9)에 두었다)

https://colors-search-box.firebaseapp.com/ 에서 웹 버전의 동작하는 데모를 확인할 수 있다. 각각의(web, 그리고 mobile) 스크린샷은 아래와 같다.

![](https://cdn-images-1.medium.com/max/800/1*_lHv4QLbnci7XVxXkCbYQw.png)
![](https://cdn-images-1.medium.com/max/800/1*Cj9-EDA6i12vmPWTLQicSw.gif)

## 재사용을 위한 리팩토링

### 코드 재사용은 관점에 대한 것이다

코드 재사용의 기본은 간단한다. 메서드(혹은 클래스나 컴포넌트)를 코드베이스에서 추출한 다음, 포함된 값들을 파라미터로 바꾼다. 그리고 그것들을 다른 코드베이스에서 사용한다. 그러나 재사용된 요소의 이점은 썩 많지 않을 때가 많으며 공유된 코드를 유지하는데 많은 비용을 소모해야할 수도 있다.

그러나 나는 다음 지침들을 적용함으로써 지속적인 재사용을 달성했다. [관심사 분리](https://en.wikipedia.org/wiki/Separation_of_concerns), [단일 책임 원칙](https://en.wikipedia.org/wiki/Single_responsibility_principle), 그리고 [복사본 제거](https://en.wikipedia.org/wiki/Duplicate_code).

Separation of Concerns(SoC)와 Single Responsibility Principle(SRP)는 동전의 양면이다. 주요 아이디어는 주어진 코드는 반드시 하나의 주요 목적을 가져야 한다는 것이다. 하나의 목적만 있다면, Separation of Concerns는 제품에 자연스럽게 반영된다. 하나의 목적을 가진 요소는 아마도 두 가지 책임을 가진 영역으로 섞이지 않을 것이다.

많은 IDE와 개발 도구들은 복사된 코드들을 병합하는 자동화 기능을 제공한다. 그러나 유사한 디자인 사이에서 복사본 제거는 더 어렵다. 당신은 코드 블록을 재정렬해야하는 복사본들을 직접 "봐야"만 한다.

이 아이디어를 적용하는 것은 퍼즐 조각을 움직여서 조각들이 만나는 곳, 조각들이 드러내는 패턴이 무엇인지 찾는 것과 동일하다.

복사본을 찾으러 떠나보자.

### 복사본 보기

웹과 모바일 애플리케이션은 두 메인 컴포넌트가 있다.
웹 애플리케이션에서는 `App.js`다

<script src="https://gist.github.com/csepulv/9c8e209df59b38697c13def860da846c.js"></script>

모바일 애플리케이션에서는 `SearchView.js`다

<script src="https://gist.github.com/csepulv/addc8fbcb6fad24def9f0ea9f5f81d74.js"></script>

구조 개요는 다음과 같다.

![](https://cdn-images-1.medium.com/max/800/1*BP-vOCbp_qodlz6WxVpUhA.png)
*거의 동일하지만 React와 React Native간 플랫폼 차이점이 있다*

두 컴포넌트는 유사한 구조를 지녔다. 이상적으로는 다음과 같이 생긴 컴포넌트를 공유할 수 있다.

![](https://cdn-images-1.medium.com/max/800/1*wWjnBWM-1XYVxqVqXtkNag.png)
*우리의 목표: 공통의 공유된 컴포넌트 세트*

유사-코드로는 다음과 같다.

<script src="https://gist.github.com/csepulv/e6c719e4b8fb09bb1dcedd7f8ec32cad.js"></script>

유감스럽게도 두 애플리케이션 사이에는 매우 적은 코드만이 공통적이다. React가 사용하는 컴포넌트(이 경우에는 Material UI)는 React Native가 사용하는 컴포넌트들과 다르다. 그러나 관심사 분리를 통해 개념적인 중복을 제거할 수 있다. 그리고 컴포넌트를 단일 책임을 지도록 리팩토링한다.

### 관심사 분리와 단일 책임

`App.js`와 `SearchView.js`는 모두 도메인 로직(우리의 애플리케이션 로직)과 플랫폼 구현, 라이브러리 통합이 섞여 있다. 이들을 고립시켜서 디자인을 향상시킬 수 있다.

 - UI 구현: `ListItem`, `ListView`를 분리하는 것 등
 - 상태 변경에 대한 UX: 결과를 보여주거나 업데이트하는 것으로부터 submitting을 분리 등
 - 컴포넌트들: search input, search results (list) 그리고 각각의 search result(list item) 등은 각각 컴포넌트로 분리해야만 한다

 마지막으로, 이러한 많은 변화들이 아무 것도 망치지 않는 것을 보장하기 위한 자동화된 테스트를 통해 리팩토링이 완료된다. 이러한 간단한 "smoke" 테스트를 추가할 것이다. 이 [Github 저장소/태그](https://github.com/csepulv/search-box/tree/smoke-tests) 에서 확인할 수 있다.

## Stateless Function Components 추출

쉽고 분명한 것부터 리팩토링하자. React는 컴포넌트에 대한 것이므로 컴포넌트들을 분리하자. 읽기 쉬운 Stateless Functional Components를 사용할 것이다.

SearchInput.js를 다음과 같이 생성할 수 있다:

<script src="https://gist.github.com/csepulv/4469f4582c6bfd35b6e2d325fba8adb3.js"></script>

React의 정수는 UI/View 프레임워크고, 위 컴포넌트에서 그 정수를 볼 수 있다.

오직 2개의 임포트된 엘레멘트만 있다: React (JSX를 위한 요구사항) 그리고 Material UI의 `TextField` - MobX도 없고 `MuiThemeProvider`도 없다. 색상 등등도 없다.

이벤트 핸들링은 핸들러(파라미터로 주어진)에게 위임된다. 단, Enter 키를 누르는 것을 제외되었다. 이것은 input box의 구현 고려사항이고, 이 컴포넌트에 캡슐화되야만 한다. (예를 들어, 다른 UI 위젯 라이브러리는 enter 키를 누르면 서브밋하는 기능이 포함되어 있을 수도 있다)

리팩토링을 이어가자. `SearchResults.js`를 생성할 수 있다:

<script src="https://gist.github.com/csepulv/bd88caf02804535053c5730198c29f09.js"></script>

`SearchInput.js`와 유사하다. 이 Stateless Functional Components는 단순하고 2개의 임포트만 가지고 있다. 관심사 분리(그리고 SRP)에 따라, 이 컴포넌트는 `ListItem`이라는 개별 검색 결과를 위한 파라미터를 전달받는다.

> `ListItem`을 감싸는 Higher Order Component를 생성할 수도 있다. 그러나 현재는 Stateless Functional Components를 사용할 것이다. 나중에 HOC를 사용할 것이다. (점차적으로, 우리는 `SearchResults.js`를 HOC로 리팩토링할 것이다.)

개별 검색 결과를 위해서, `ColorListItem.js`를 만들 것이다:

<script src="https://gist.github.com/csepulv/c109ae918c20aa13cc03a3d34f0cea12.js"></script>

이제 App.js를 리팩토링할 필요가 있다.

## Higher Order Components 추출

가독성을 위해서 `App.js`를 `SearchBox.js`로 이름을 변경하겠다. 이 컴포넌트의 리팩토링에는 몇 가지 선택지가 있다.

 1. `SearchBox`가 `ColorListItem`을 `SearchResults`로 전달하게 하기(prop으로)
 2. `index.js`가 `ColorListItem`을 `SearchBox`에 전달하고, `SearchResults`로 전달하게 하기
 3. `SearchBox`를 Higher Order Component(HOC)로 변환하기

(1) 방법은 다음과 같다:

<script src="https://gist.github.com/csepulv/cf0e0568a4e2923e4d4ebc1f877d8435.js"></script>

아무것도 잘못된 것이 없다. `SearchInput.js`와 `SearchResults.js`를 추출하는 논리적인 결론이다. 그러나 `SearchBox`에 `ColorListItem`이 바인딩되어서 관심사 분리를 위반한다. (`SearchResults`의 재사용도 제한한다.)

(2) 관심사들을 분리해서 고쳐보자

<script src="https://gist.github.com/csepulv/ab372ff93a048cf445275a1f15b22b5f.js"></script>

(재사용성을 명확히 하려고 `colors` prop을 `searchStore`로 이름을 변경했다.)

그러나 사용처를 보면 `ColorListItem`을 `index.js`에서 prop으로 전달하고 있음을 알 수 있다.

<script src="https://gist.github.com/csepulv/4912e2e1ff62a5b120f9fda0c320d3ce.js"></script>

다음 코드와 비교해보자:

<script src="https://gist.github.com/csepulv/836e4c992ef65c12fe9bcdcff6574fce.js"></script>

(3)의 경우, 즉 HOC를 사용했을 때의 `index.js`다. 사소한 차이지만 중요하다. `ColorSearchBox`는 `ColorListItem`을 포함하고 있으며, `ColorSearchBox`은 자신이 사용하고 있는 특정 search result 컴포넌트를 캡슐화한다.

(`searchStore`, `Colors`는 prop이다. 애플리케이션 내에서 하나의 인스턴스여야만 한다. 하지만 주어진 구성 요소의 인스턴스, 즉 `ColorSearchBox`가 여러 개 있을 수 있다.)

따라서, `SearchBox.js`를 HOC로 다음과 같이 만들 수 있다.

<script src="https://gist.github.com/csepulv/df7df5884136ce144ad29327fbdb17af.js"></script>

`SearchBox.js`가 이전 섹션([복사본 보기](https://hackernoon.com/code-reuse-using-higher-order-hoc-and-stateless-functional-components-in-react-and-react-native-6eeb503c665#ac93))의 유사코드와 닮아보인다는 것을 알아차릴 수 있다. 잠시 후에 더 정제해볼 것이다.

## React Native 컴포넌트 리팩토링

모바일 애플리케이션과 추출된 컴포넌트들을 이전 패턴에 따라 다음과 같이 리팩토링할 수 있다. `SearchInput`을 추출하는 것과 같은 모든 세부사항을 살펴보지는 않을 것이다. 그러나 이 사항들은 [README](https://github.com/csepulv/search-box/blob/refactorings/README.md)와 [Github 저장소 브랜치](https://github.com/csepulv/search-box/tree/refactorings)에 있다.

대신, 공통의 `SearchBox`를 리팩토링하는데 집중할 것이다. 이 컴포넌트는 web(React)와 mobile(React Native)에서 모두 사용할 것이다.

## Web과 Mobile 양쪽에서 공유하는 컴포넌트 추출하기

명확히 하기 위해서 `SearchInput.js`, `SearchResults.js`, `SearchBox.js`를 `WebSearchInput.js`, `WebSearchResult.js`, `WebSearchBox.js`로 개명했다.

`(Web)SearchBox.js`를 보자

<script src="https://gist.github.com/csepulv/df7df5884136ce144ad29327fbdb17af.js"></script>

2-10, 19, 20, 26, 27 번째 줄은 React에 특정된 것이다.

`MuiThemeProvider`는 Material UI components의 container고, 오직 Material UI에만 직접적인 의존성이 있다. 그러나 `SearchInput`과 `SearchResult`에도 묵시적인 의존성이 있다. 이러한 의존성을 `SearchFrame` 컴포넌트를 도입해서 분리시킬 수 있다. 이 컴포넌트는 `MuiThemeProvider`와 `SearchInput`과 `SearchResults`을 하위 컴포넌트로 갖고 캡슐화시킬 것이다. 그 다음엔 `SearchBox` HOC를 만들 수 있다. `SearchBox`는 `SearchFrame`, `SearchResults`, `SearchInput`을 사용할 것이다.

새로운 `SearchBox.js`는 다음과 같다.

<script src="https://gist.github.com/csepulv/ecd5f1ff4c4e4de34077cd5055cc0b11.js"></script>

[복사본 보기](https://hackernoon.com/code-reuse-using-higher-order-hoc-and-stateless-functional-components-in-react-and-react-native-6eeb503c665#ac93) 섹션의 유사코드와 비슷해보인다.

`WebSearchBox.js`의 내용을 바꿀 차례다

<script src="https://gist.github.com/csepulv/84f28432619b995ed233ad98bdca57ae.js"></script>

`WebSearchBox` (26번째 줄) 은 `SearchBox` HOC를 사용한 결과다.

> `children`은 특별한 React prop이다. 이 경우에는 `WebSearchFrame`이 `WebSearchInput`와 `WebSearchResults`을 포함하고 렌더링하도록 해준다. `SearchBox`에 의해 제공된 파라미터로 말이다. children prop에 대해 더 알고 싶다면 [이곳](https://facebook.github.io/react/docs/composition-vs-inheritance.html)을 확인하라.

또한 `WebSearchResults`를 HOC로 변경할 것이다. `ListItem`을 HOC 조합의 한 부분으로 캡슐화해야만 한다.

<script src="https://gist.github.com/csepulv/589842e3f552cd6f000ac9258d77cf40.js"></script>

이제 재사용가능한 컴포넌트 세트를 가지게 되었다. (여기 [Github 저장소와 브랜치](https://github.com/csepulv/search-box/tree/refactorings)가 있다. 주의, 몇몇 디렉토리는 명확성을 위해 이름을 바꿨다.)

## 결과: 컴포넌트 재사용

우리는 Github 저장소 검색 앱을 만들었다. (Github는 API key 없이 API를 사용하는 것을 허용한다. 이 튜토리얼에서 편리하게 쓰였듯이 말이다.)

초기설정과 같은 세부사항은 건너뛸 것이지만, 요약하자면 다음과 같다.

 - 웹 앱을 위해서는 `create-react-app`을 사용한다. 모바일 앱을 위해서는 `react-native init`을 사용한다
 - MobX, Material UI(웹앱용), [qs](https://www.npmjs.com/package/qs)(쿼리 스트링 인코딩) 등등을 추가한다. 더 자세한 내용은 `package.json`에 나와있다([웹](https://github.com/csepulv/search-box/blob/master/github-mobile/package.json), [모바일](https://github.com/csepulv/search-box/blob/master/github-mobile/package.json))

노력의 대부분은 새로운 검색 스토어를 작성하는 일이다. 이를 통해 컬러 대신에 Github 저장소들을 Git API를 통해 검색한다. 다음과 같이 `github.js`를 생성할 수 있다.

<script src="https://gist.github.com/csepulv/1d6df05e2ea0e71e967d00b347fd4870.js"></script>

(유닛 테스트는 [이곳](https://github.com/csepulv/search-box/blob/master/github-web/src/github.test.js)에 있다)

> 단순함을 위해 몇몇 공통 파일을 복사할 것이다. GitHub 저장소에서는 파일을 복사할 때 약간의 편리함을 위해 [webpack](https://webpack.github.io/)을 사용한다. 자바스크립트 프로젝트에서 파일/모듈을 공유하는 것은 보통 NPM이나 Bower를 이용한다. (돈을 [지불](https://www.npmjs.com/pricing)하면 [private module](https://docs.npmjs.com/private-modules/intro)을 등록할 수 있다) 혹은 [Git submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules)를 사용할 수도 있다. 비록 어설프더라도 말이다. 우리는 모듈 배포가 아니라 컴포넌트 재사용에 집중하고 있기 때문에 그저 파일을 복사하는 조금 우아하지 못한 짓을 하는 것이다.

나머지는 쉽다. `app.js`를 삭제하고(`App.test.js`도) `index.js`의 내용을 다음과 같이 바꾼다.

<script src="https://gist.github.com/csepulv/b9cd70776ac3a9fbef2eed98d7012a89.js"></script>

이제 `npm start`를 실행하면 다음 화면을 볼 수 있다.

![](https://cdn-images-1.medium.com/max/800/1*0G_2A4nHLILkixI36C71Rg.gif)

(https://github-repo-search-box.firebaseapp.com 에 가서 라이브 버전을 볼 수 있다)

## React Native:Github 모바일 앱

`github.js`와 `MobileSearch*.js`를 복사한 다음, `GitHubMobileSearchBox.js`를 생성한다.

<script src="https://gist.github.com/csepulv/c38bb9912d8c73ca5cf4651c75aeca64.js"></script>

그리고 `index.ios.js`의 내용을 다음과 같이 변경한다.

<script src="https://gist.github.com/csepulv/4d0eb2852a3fcf418c952f62e95b0a2a.js"></script>

두 개의 파일만으로 새로운 모바일 앱이 만들어진다. `react-native run-ios`

![](https://cdn-images-1.medium.com/max/800/1*Gmy46gNeM8AsOvXvbUdDYQ.gif)

리팩토링은 어려운 작업일 것이지만, 컴포넌트 재사용은 새로운 두 가지의 앱을 간단히 만들어낼 수 있다.

## 리뷰와 요약

우리 컴포넌트들에 대한 다이어그램을 살펴보자:

![](https://cdn-images-1.medium.com/max/800/1*FgeGQ8L4NcLstQeYc3cH0w.png)


리팩토링 결과는 훌륭하다. 새로운 앱에서는 특정 도메인 로직에 집중할 수 있게 해준다. 단지 GitHub API 클라이언트와 저장소 결과를 렌더링하는 법만 정의했을 뿐이다. 나머지는 "무료"로 제공된 것이다.

게다가, 비동기 문제를 다룰 필요가 없다. 예를 들자면 `github.js`에서의 비동기 `fetch` 호출에 대해서는 알지도 못한다. 이는 리팩토링 방식의 놀라운 이점 중 하나며 Stateless Functional Components를 활용한 방법이다. 프라미스와 비동기 프로그래밍은 오직 필요한 곳, `github.js`에서만 발생할 뿐이다.

이러한 기술들을 몇 번 정도 적용해보면 컴포넌트를 추출하고 재사용하는 것이 더 쉬워질 것이다. 아마도 코딩이 패턴화되면 새로운 뷰의 시작지점에서부터 재사용 가능한 컴포넌트를 작성하게될지도 모른다.

또한 [recompose](https://github.com/acdlite/recompose)와 같은 라이브러리를 살펴보면 HOC를 더 쉽게 작성할 수도 있다.

최종 [GitHub 저장소](https://github.com/csepulv/search-box/)를 살펴보고 당신만의 재사용가능한 컴포넌트에 대한 리팩토링 방법을 알려주세요.

다음: Microinterations and Animations in React. 이 게시물에 ♡를 눌러주시고 Medium혹은 twitter에서 저를 팔로잉 해주시기 바랍니다.
