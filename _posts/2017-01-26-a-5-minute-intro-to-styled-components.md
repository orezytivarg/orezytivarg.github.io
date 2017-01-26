---
id: a-5-minute-intro-to-styled-component
title: Styled-Components 5분 소개
category: CSS
---
[원문보기](https://medium.freecodecamp.com/a-5-minute-intro-to-styled-components-41f40eb7cd55#.y3q8dwmf5)

![](https://cdn-images-1.medium.com/max/1000/1*DIFji4ZmJa4_H3EpbG2XAw.png)

CSS는 이상하다. 15분 내로 기본 내용을 익힐수 있다. 그러나 스타일을 조직하는 좋은 방법을 알아내기까지는 수 년이 걸릴 수 있다.

일부는 언어 그 자체의 단점 때문이다. CSS 그 자체로는 한계가 있다. 변수나 루프, 함수 등이 없다. 그와 동시에, 엘레멘트, 클래스, ID의, 혹은 그들의 어떻게 조합하더라도 스타일링할 수 있도록 허용하고 있다.

## 혼돈의 Style Sheets
당신 스스로 경험했듯이, 종종 혼돈을 만들어내는 조리법이 있다. SASS나 LESS같은 전처리기는 많은 유용한 피처들을 추가해주지만, 실제로 CSS의 난해함을 멈추기에는 부족하다.

BEM과 같은 방법론에 맡겨진 조직화 업무는 - 유용하지만 - 완전히 선택사항일 뿐이고, 언어나 툴 레벨에서 이를 강제할 수 없다.

## CSS의 새 물결
몇년 전부터 자바스크립트 기반의 툴을 사용하려는 새로운 물결이 CSS를 쓰는 방법을 바꿔버리는 식으로 이러한 문제를 발본색원하려고 노력하고 있다.

[Styled-Components](https://github.com/styled-components/styled-components)는 이러한 라이브러리 중 하나다. 그리고 혁신과 친숙함의 조합을 무기로 많은 이들의 관심과 주목을 받았다. 만약 React를 사용한다면(만약 아니라면 [내 자바스크립트 스터디 계획](https://medium.freecodecamp.com/a-study-plan-to-cure-javascript-fatigue-8ad3a54f2eb1)과 [React 소개글](https://medium.freecodecamp.com/the-5-things-you-need-to-know-to-understand-react-a1dbd5d114a3)을 확인하라), 이 새로운 CSS의 대체제를 살펴보는 것이 좋을 것이다.

나는 최근 [개인 사이트를 다시 디자인](http://sachagreif.com/)하기 위해서 이것을 사용해보았고, 그 과정에서 배운 몇 가지를 공유하기를 바랐다.

## 컴포넌트, 스타일링된

Styled-Components에 대해 이해해야만 할 주요 사항은 그 이름을 문자 그대로 대해야 한다는 것이다. 당신은 더이상 HTML 엘레멘트를 스타일링하거나, HTML 엘레멘트나 클래스를 기반으로하는 컴포넌트를 스타일링하지 않는다:

```
<h1 className="title">Hello World</h1>

h1.title{
  font-size: 1.5em;
  color: purple;
}
```

이렇게 하는 대신, 자신만의 캡슐화된 스타일을 지닌 styled compoents를 정의한다. 그 다음, 자유롭게 당신의 코드베이스에서 이를 사용한다.

```js
import styled from 'styled-components';

const Title = styled.h1`
  font-size: 1.5em;
  color: purple;
`;

<Title>Hello World</Title>
```

이것은 사소한 차이처럼 보일 수 있고, 사실 양쪽의 문법은 매우 유사하다. 그러나 중요한 차이점은 스타일이 그 컴포넌트의 일부라는 점이다.

다른 말로 하자면, 컴포넌트와 스타일 사이의 중간단계인 CSS 클래스를 제거하는 것이다.

Styled-Components의 공동 제작자인 Max Stoiber가 말하기를:

"Styled-Components의 기본 아이디어는 스타일과 컴포넌트간의 매핑을 제거함으로써 베스트 프랙티스를 강요하는 것이다"

## 복잡성 제거하기

처음에는 반-직관적인 것처럼 보일 것이다. HTML 엘레멘트(<font> 태그를 기억하는가?)를 직접 스타일링하는 것 대신 CSS를 사용한다는 것의 요점은 중간에 클래스 계층을 도입하여 마크업과 스타일을 분리하는 것이다.

하지만 이 분리로 인해 많은 복잡성 또한 생겨났고, CSS와 비교할 때, 자바스크립트같은 "진짜" 프로그래밍 언어가 이러한 복잡성을 처리할 수 있는 능력이 훨씬 뛰어나다는 주장이 있다.

## 클래스 대신 props

노-클래스의 철학을 유지하기위해, Styled-Components는 컴포넌트의 행동을 커스터마이징할 때 클래스 대신 props를 사용하게 한다. 따라서 다음과 같이 하는 대신에:

```
<h1 className="title primary">Hello World</h1> // will be blue

h1.title{
  font-size: 1.5em;
  color: purple;

  &.primary{
    color: blue;
  }
}
```

이렇게 작성한다:

```js
const Title = styled.h1`
  font-size: 1.5em;
  color: ${props => props.primary ? 'blue' : 'purple'};
`;

<Title primary>Hello World</Title> // will be blue
```

보다시피 Styled-Components를 사용하면 모든 CSS와 HTML을(그와 관련된 바깥쪽의 모든 구현 세부사항) 소유함으로써 당신의 React 컴포넌트를 말끔하게 정리해준다.

즉, Styled-Components의 CSS는 여전히 CSS다. 따라서 다음과 같은 (비록 약간 비관용적이지만) 코드도 완전히 유효하다:

```js
const Title = styled.h1`
  font-size: 1.5em;
  color: purple;

  &.primary{
    color: blue;
  }
`;

<Title className="primary">Hello World</Title> // will be blue
```

이것은 Styled-Components를 쉽게 도입할 수 있는 한 가지 기능이다: 의심스럽다면 언제든지 당신이 아는 상태로 돌아갈 수 있다!

## 주의사항

Styled-Components는 매우 어린 프로젝트고, 일부 기능은 아직 완전히 지원되지 않는다는 것도 중요하게 언급한다. 예를 들어 부모로부터 하위 컴포넌트에게 스타일을 지정하게 하려면 CSS 클래스에 의존해야 한다(적어도 버전2가 나오기 전까지는)

서버에서 CSS를 미리 렌더링하는 어떠한 "공식적인" 방법도 또한 존재하지 않는다. 스타일을 수동으로 주입함으로써 분명 가능하긴 하지만 말이다.

그리고 Styled-Components가 자체적으로 랜덤한 클래스 이름을 생성한다는 사실은 브라우저의 개발 도구를 사용하여 원래 스타일이 정의된 위치가 어디인지 찾는 것을 어렵게 만든다.

그러나 매우 고무적인 것은 Styled-Components 코어 팀이 이 모든 이슈를 인식하고, 하나씩 하나씩 열심히 수정하고 있다는 것이다. 버전 2가 곧 출시될 예정이고, 나는 굉장히 기대중이다!

## 자세히 알아보기

내 목표는 Styled-Components가 어떻게 작동하는지 자세히 설명하는 것이 아니라, 작은 것을 보여줌으로써 그것이 가치있는지 스스로 확인할 수 있게 하는 것이다.

만약 내가 호기심을 자극했다면, Styled-Components에 대해 더 알아볼 만한 장소가 몇 군데 있다:

 - Max Stoiber는 최근 Styled-Components에 대한 이유를 [Smashing Magazine](https://www.smashingmagazine.com/2017/01/styled-components-enforcing-best-practices-component-based-systems/)에 작성했다.
 - [Styled-Components 저장소](https://github.com/styled-components/styled-components)에서 자체적으로 광범위한 문서를 제공하고 있다.
 - [Jamie Dixon의 글](https://medium.com/@jamiedixon/styled-components-production-patterns-c22e24b1d896#.tfxr5bws2)에서 Styled-Components로 전환하는 것의 이점들에 대해 밝히고 있다
 - 실제로 어떻게 이 라이브러리가 구현된건지 알고 싶다면, Max의 [이 글](http://mxstbr.blog/2016/11/styled-components-magic-explained/)을 읽어보라.

그리고 더 나아가고 싶다면, Glamor에서 CSS의 새 물결에 대한 다른 이야기들을 확인할 수 있다!

셀프-프로모션 타임: 폼, 데이터 로드, 사용자 계정을 갖춘 풀스택 React & GraphQl 앱을 만드는 가장 쉬운 방법인 Nova를 돕기 위해 오픈 소스 컨트리뷰터를 찾고 있습니다! 우리는 아직 Styled-Components를 사용하고 있지는 않지만 당신이 이를 구현하는 첫 번째 사람이 될 수도 있습니다!
