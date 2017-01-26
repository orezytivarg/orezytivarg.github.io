---
id: styled-components-production-patterns
title: Styled Components 제품 패턴
category: CSS
---
[원문보기](https://medium.com/@jamiedixon/styled-components-production-patterns-c22e24b1d896#.972759r99)

Styled-Components는 React와 React Native를 위한 라이브러리로 자바스크립트와 CSS를 혼합하여 컴포넌트 레벨의 스타일링을 사용할 수 있게 해준다.

Styled-Components에 익숙하지 않다면 [웹사이트](https://styled-components.com/)를 방문해서 살펴보라.

기본적인 Styled-Components의 컴포넌트 모양은 다음과 같다:

![](https://cdn-images-1.medium.com/max/800/1*DIFji4ZmJa4_H3EpbG2XAw.png)

지난 몇 주간 두 개의 프로젝트에서 Styled-Components를 사용했으며, 발견된 몇 가지 이점과 패턴을 적어보고자 한다.

## 압축된 스타일

Styled-Components는 스타일 값으로 함수를 전달할 수 있기 때문에, 기존 CSS처럼 HTML 엘리먼트에 새 클래스를 추가하는 것이 아니라 prop 값을 기반으로 해서 스타일을 바꿀 수 있다.

결과적으로 코드량이 줄어든다. 처음에 CSS를 Styled-Components로 전환했을 때, 아주 드라마틱한 향상을 볼 수 있었다.

![원본 CSS](https://cdn-images-1.medium.com/max/800/1*NCEuP2Xo2j5VwlK8JRYoUQ.png)

![Styled-Components로 변환](https://cdn-images-1.medium.com/max/800/1*G2GsxjVgtRnYwK31OGUSQA.png)

## Clearer JSX

당신이 나처럼 JSX를 `<div>`와 `<span>`으로 흐트려뜨리는 사람이라면, 아마도 Styled-Components가 기본적으로 더 의미에 맞는 컴포넌트 계층 구조를 구성하게 한다는 사실을 알 수 있을 것이다.

![클래스 훅을 통해 스타일링한 원본 JSX](https://cdn-images-1.medium.com/max/800/1*Y1FPLYaimV5SACmtWlfylw.png)

![className이 사용되지 않은 Styled-Components로 변환된 JSX! 태그의 의미를 보라]

난 이미 당신의 JSX가 두 번째 예제처럼 보일 것이라는 것을 확신한다 :P. 그게 아니라면, Styled-Components는 성공으로 가는 기본 통로로 당신을 이끌어 당신에게 큰 도움을 줄 수 있을것이다.

## 스타일 합성

이것은 Styled-Components에서 내가 가장 좋아하는 피처다. 한번 Styled-Components를 생성하고 나면 새로운 Styled-Components에 쉽게 합성할 수 있다.

왜냐면 Styled-Components에게 DOM 엘레멘트 뿐만 아니라 컴포넌트도 전달할 수 있기 때문이다.

![Message를 가지고 합성된 두 새로운 컴포넌트, Success와 Danger](https://cdn-images-1.medium.com/max/800/1*qi1X679F99NqLW02he-Xig.png)

## Prop 필터링

React 15.2.0부터 DOM 엘레멘트가 알지 못하는 prop을 전달하면 경고가 발생한다. <div foo="foo">와 같이 전달하면 "foo"가 알지못하는 속성이라는 경고 메시지가 나온다.

때로는 컴포넌트가 가능한 모든 DOM 속성을 허용하고 내부의 DOM 엘레멘트로 이를 전달해야하는 경우가 있다. 모든 DOM 속성을 수동으로 지정하여 이 작업을 수행하기는 어렵다. 따라서 이러한 컴포넌트는 props를 다음과 같이 펼쳐버린다: `<div {...props}>`

우리는 앞서 언급한 알지못하는 prop 경고를 피하기 위해 유효한 DOM 속성이 아닌 prop을 필터링하기 시작했다. 흥미롭게도 Styled-Components는 이미 내부적으로 이 작업을 수행하고 있는 잘 관리되는 라이브러리라서 이러한 필터링에 대한 항목을 업데이트할 필요가 없었다.

![`<span>`과 같은 DOM 엘레멘트에 유효한 DOM 속성을 필터링하는 함수를 가지고 있었다](https://cdn-images-1.medium.com/max/800/1*mRMB1rv5yNVL-9TTKiBJWg.png)

![이런 필터링으로부터 자유로워졌다! 심지어 아무런 스타일도 필요하지 않은 경우에도!](https://cdn-images-1.medium.com/max/800/1*RbwbxXorxdm3utC3LRpTxA.png)

이들은 우리가 경험한 패턴이나 이점의 일부에 지나지 않는다. 우리가 계속 사용한다면 시간이 갈 수록 더 많은 것들이 나타나리라 확신한다.

Styled-Components를 만든 Max Stoiber와 Glen Maddern에게 큰 감사를 표한다.
