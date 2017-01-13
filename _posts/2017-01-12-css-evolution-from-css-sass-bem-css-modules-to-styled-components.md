---
id: css-evolution-from-css-sass-bem-css-modules-to-styled-components
title: CSS의 진화 - CSS, SASS, BEM, CSS Modules로부터 Styled Components로
category: CSS
---
![](https://cdn-images-1.medium.com/max/1000/1*yBxZo9LNEjRaL7eKUBqRSA.png)

[원문보기](https://m.alphasights.com/css-evolution-from-css-sass-bem-css-modules-to-styled-components-d4c1da3a659b#.mcz0jzhui)

# CSS의 진화: CSS, SASS, BEM, CSS Modules로부터 Styled Components로

Since the beginnings of the Internet we’ve always had the need to style our websites, CSS has been around forever and has evolved at its own pace over the years, this article will take you through it.

To begin with we need to be on the same page of what CSS is, I think we can all agree that css is used for describing the presentation of a document written in a markup language.

It’s no news that CSS has evolved along the way and has become more powerful nowadays, but it’s widely known that additional tooling needs to be used in order to make css somehow work for teams.

## CSS in the wild-west

In the 90’s we use to focus on creating “fancy” interfaces, the wow factor was the most important thing, inline styles were the thing back then, we just didn’t care if things looked different, in the end Webpages were like cute toys we could threw some gifs, marquees and other horrible (at the time impressive) elements over and expect to catch your visitors’ attention.

https://twitter.com/ivantodorov/status/324420248201736195

After that, we started creating dynamic sites, but css remained a consistent mess, every developer had his/her way of doing css. Some of us struggled with specificity, causing visual regressions when introducing new code, we relied on using !important to set in stone our strong will to make a UI element look in a certain way. But we soon realized that:

![](https://cdn-images-1.medium.com/max/800/1*vePubKIIK_96qGEgKo5G4Q.jpeg)

All those practices became more evident and bigger problems as soon as projects grew in size, complexity, and team members. So not having a consistent pattern to do styling became one of the biggest blockers for experienced and inexperienced developers who struggled to find a right way to do things in CSS. In the end there was no right or wrong thing to do, we just cared to make the thing look ok.

![](https://cdn-images-1.medium.com/max/800/1*3IDlXD210gSmF5jYko1RtQ.gif)

## SASS to the rescue

SASS transformed CSS into a decent programming language in the form of a preprocessing engine that implemented nesting, variables, mixins, extends and logic into stylesheets, so you could better organize your css files and have at least some ways of deconstructing your css chunks in smaller files, which was a great thing back then.

It essentially takes scss code, preprocesses it and outputs the compiled versions of the file in a global css bundle. Great right? but not so much I’d say, After a while it became apparent that unless there were strategies and best practices in place, SASS only caused more troubles than it alleviated.

Suddenly we became unaware of what the preprocessor was doing under the hood and relied on lazily nesting to conquer the specificity battle but causing compiled stylesheets to go nuts in sizes.

Until BEM came along….

## BEM and component based thinking

When BEM came along it was a breath of fresh air that made us think more about reusability and components. It basically brought semanticity to a new level, it let us make sure that className is unique thus reducing the risk of specificity clash by using a simple Block Element Modifier convention. Look at the following example:

<script src="https://gist.github.com/carlosepp/01ebb1e7fb6cc7aacf96896de718cf4d.js"></script>

If you analyze a bit the markup you can see immediately the Block Element Modifier convention in play here.

You can see that we have two very explicit blocks in the code: .scenery and .sky, each one of them have their own blocks. Sky is the only one that has modifiers as it could be foggy, daytime or dusk, those are different characteristics that could be applied to the same element.

Let’s take a look at the companion css with some pseudo code that will let us analyze it better:

<script src="https://gist.github.com/carlosepp/090f44bce2a1344a6eeaece7762c10a2.js"></script>

If you want to have an in-depth understanding of how BEM works, I recommend you take a look at this article , written by my colleague and friend Andrei Popa.

BEM is good in the sense that you made sure that components were unique #reusabilityFtw. With this kind of thinking some apparent patterns became more evident as we started migrating our old stylesheets into this new convention.

But, another set of problems came along:

- Classname selection became a tedious task
- Markup became bloated with all those long class names
- You needed to explicitly extend every ui component whenever you wanted to reuse
- Markup became unnecessarily semantic

CSS Modules and local scope

Some of the problems that neither SASS or BEM fixed was that in the language logic there is no concept of true encapsulation, thus relying on the developer to choose unique class names. A process that felt could be solved by tools rather by conventions.

And this is exactly what CSS modules did, it relied on creating a dynamic class names for each locally defined style, making sure no visual regressions are caused by injecting new css properties, all styles were properly encapsulated.

CSS-Modules quickly gained popularity in the React ecosystem and now it’s common to see many react projects using it, it has it’s pros and cons but over all it proves to be a good paradigm to use.

But… CSS Modules by itself doesn’t solve the core problems of CSS, it only shows you a way of localizing style definitions: a clever way of automating BEM so you don’t need to think about chosing a class name ever again (or at least think about it less).

But it does not alleviate the need for a good and predictable style architecture that is easy to extend reuse and control with the least amount of effort.

This is how local css looks like:

<script src="https://gist.github.com/carlosepp/89edc1dd4bc87f129cf62233672e1536.js"></script>

You can see that it’s just css, the main difference is that all classNames prepended with :local will generate a unique class name that looks something like this:

`.app-components-button-__root — 3vvFf {}`

You can configure the generated ident with the localIdentName query parameter. Example: `css-loader?localIdentName=[path][name]---[local]---[hash:base64:5]` for easier debugging.

That’s the simple principle behind Local CSS Modules. If you can see, local modules became a way to automate the BEM notation by generating a unique className that was sure it wouldn’t clash with other’s even if they used the same name. Quite convenient.

## Styled Components to blend css in JS (fully)

Styled-components는 순수한 시각적 primitives다; 실제 html 태그에 매핑될 수 있으며 자식 컴포넌트들을 styled-component로 감싸는 일을 한다.

다음 코드로 더 설명하는 것이 나을 것이다:

<script src="https://gist.github.com/carlosepp/f3b342565e1a41cf4488a626e696e26e.js"></script>

보기만해도 styled component는 굉장히 이해하기 쉬울 것이다. 템플릿 리터럴 표기법을 사용하여 css 프로퍼티를 정의하고 있다는 것은 core styled-compoents 팀이 es6와 css의 모든 파워를 혼합한 것 처럼 보인다.

Styled-components는 기능적, 상태적인 컴포넌트에서 UI를 완전히 분리하고 재사용할 수 있는 매우 단순한 패턴을 제공한다. React Native 혹은 브라우저의 HTML 양쪽 태그 모두를 억세스하는 api를 만드는 것이다.

이것이 Styled Component에 props(혹은 modifiers)를 전달하는 방법이다:

<script src="https://gist.github.com/carlosepp/b1bbf0b9ab60807c9e8f74902a55a97e.js"></script>

props가 갑자기 각 컴포넌트가 받는 modifier가 되고, css 몇 라인의 출력으로 처리될 수 있음을 알 수 있다. 산뜻하지 않은가?

따라서 스타일을 처리하는데 JS의 모든 기능을 사용하면서 일관성있고 재사용 가능한 상태를 유지한 채로 빠르게 움직일 수 있게 한다.

## 모든 사람이 재사용하는 Core UI

CSS 모듈이나 Styled Components나 그 자체로는 완벽한 솔루션이 아니라는 것은 꽤 빠르게 명백해졌고, 작동과 확장을 목적으로 하는 몇 가지 패턴이 필요하다. 패턴은 컴포넌트가 무엇인지 정의하는 것과, 로직으로부터 완전히 분리하는 것, 스타일을 지정하는 것 뿐이다.
It quickly became apparent that CSS Modules nor Styled Components by themselves was not the perfect solution, it needed some kind of pattern in order for it to work and scale. The pattern emerged by defining what a component is and separating it fully from logic, creating core components which sole purpose is to style and nothing more.

CSS 모듈을 사용한 컴포넌트의 구현 예제다:

<script src="https://gist.github.com/carlosepp/b8342e24955c1ceb5be36cd3c72ac135.js"></script>

보다시피 팬시한 뭔가가 있지는 않고, 그저 props를 받아서 하위 컴포넌트에 매핑하는 컴포넌트일 뿐이다. 다시 말하자면, props를 children에 전달하는 포장용 컴포넌트일 뿐이다.

다음과 같은 방법으로 컴포넌트를 사용할 수 있다.

<script src="https://gist.github.com/carlosepp/e35be27d5e00d7de939bc4774b260c90.js"></script>

styled-components를 사용하여 구현한 비슷한 예제를 살펴보자:

<script src="https://gist.github.com/carlosepp/d6c7cebd374b88609c140bfdfe270498.js"></script>

이 패턴이 흥미로운 점은 컴포넌트가 dumb이고 상위 컴포넌트에 매핑된 css 정의의 wrapper일 뿐이라는 것이다. 이 일을 하는 것에는 한 가지 이점이 있다:

기본 UI api를 정의하게 해주므로 이를 스왑함으로 모든 UI가 애플리케이션 전체에 걸쳐 일관성있게 유지되도록 해준다.

이 방법은 디자인 처리와 구현 처리를 완전히 분리시켜서 원한다면 각 처리를 병렬적으로 시작할 수 있다. 피쳐 구현에 포커싱한 개발자와 UI를 갈고닦는 개발자에 대해서 서로 완전히 관심사를 분리할 수 있게 한다.

지금까지는 아주 훌륭한 해결책인것 같다. 내부적으로 우리는 이 문제를 두고 토론을 시작했으며 좋은 아이디어라고 생각했다. 그리고 이 패턴과 함께 또다른 유용한 패턴을 식별하기 시작했다:

## Prop receivers

이 패턴은 어떤 컴포넌트로 전달된 props를 리스닝하는 기능을 하므로, 어떤 컴포넌트에든 이 기능을 사용하기 쉽고, 어떤 주어진 컴포넌트라도 재사용성과 가용성을 확장할 수 있는 성배로 만들어준다. modifier를 상속하는 방법으로 생각할수도 있다. 다음 예제가 바로 내가 말하는 바다:

<script src="https://gist.github.com/carlosepp/bcb7d275546ab344805a032e9d659bbc.js"></script>

이 방법을 통해 특정 컴포넌트를 위한 각 border를 하드코드할 필요가 없음을 확신할 수 있다. 엄청나게 시간을 절약할 수 있다.

## Placeholder / Mixin like functionality

styled component에서는 JS의 풀파워를 사용할 수 있다. 즉 prop receivers 뿐만 아니라 서로 다른 컴포넌트 간의 코드 공유같은 방법으로. 여기 예제가 있다:

<script src="https://gist.github.com/carlosepp/7704cce24edba0520eb6b36d894f04ae.js"></script>

## Layout Components

우리는 애플리케이션 작업을 할때 처음으로 필요한 것이 UI elements를 레이아웃 하는 것임을 알아냈다. 이를 처리하는 과정에서 레이아웃 목적으로 사용하는 몇몇 컴포넌트를 식별해냈다.

이 컴포넌트들은 종종 구조를 세팅하는 힘든 시간을 보내는 몇몇 개발자들에게 매우 유용함이 증명되었다(css 포지셔닝 테크닉보다 충분히 익숙하지는 않다). 여기 그런 컴포넌트들의 예제가 있다:

<script src="https://gist.github.com/carlosepp/2e898cc74dcd51f9e4da2602566cf41f.js"></script>

width와 height를 props로 받고 horizontal prop도 받아서 스크롤바를 하단에 출력하는 <ScrollView /> 컴포넌트를 볼 수 있다.

## Helper components

헬퍼 컴포넌트는 엄청난 재사용을 허용하여 우리의 삶을 간단하게 만든다. 우리의 모든 공통 패턴을 저장한 장소다.

지금까지 꽤 유용했던 헬퍼 몇몇은 다음과 같다:

<script src="https://gist.github.com/carlosepp/112987c144fd6e301ca35940a675ae2e.js"></script>

## Theme

테마를 지원하는 것은 애플리케이션 전체에 걸친 1개의 source of truth of values를 가지게 할 것이다. 이는 컬러 팔렛트나 일반적인 룩앤필과 같은 애플리케이션에서 공통적으로 재사용되는 값들을 저장하는데 유용함이 증명되었다.

<script src="https://gist.github.com/carlosepp/891d2f20f65f16bfb7c17a0e0afd4e16.js"></script>

장점

- 풀파워의 JS가 우리 손에 들어왔다. 이는 컴포넌트 UI와 풀 커뮤니케이션을 의미한다.
- className을 통한 컴포넌트와 스타일의 매핑을 제거한다.(매핑은 속에서 완료된다)
- 지금까지 중 최고의 개발 경험을 제공한다. 클래스네임과 컴포넌트를 매핑을 생각하는데 들이는 엄청난 양의 시간을 줄인다.

단점

- 야생에서 테스트된 적이 없다.
- React를 위해 개발되었다.
- 아직은 정말 미성숙하다.
- classNames를 사용하거나 arai-labels를 위한 테스팅이 필요하다.

## 결론

SASS, BEM, CSS 모듈, Styled Components 어떤 기술을 사용하더라도 다른 개발자들이 시스템의 새로운 파트를 가져오는 것이나 시스템을 깨뜨리는 것에 대해 너무 많이 생각하지 않아도 당신의 코드 베이스에 기여할 수 있도록 잘 정의된 스타일링 아키텍쳐를 대체할 수는 없다.

이 접근 방법은 애플리케이션을 적절하게 확장시키는데 주요하다. plain CSS 혹은 BEM을 사용하더라도 중요한 차이점은 각 구현에 필요한 작업량과 LOC다. 전반적으로 보면 styled-components는 거의 모든 React 프로젝트에 잘 맞고, 아직 야생에서 태스트되지는 않았지만 충분히 유망한 스타일링 방법이다.

의견이나 의견, 조언 등이 있으시면 아래에 의견을 남기고 트위터 @perezpriego7을 통해 연락 바란다.

Refact, Ember, Rails, Elixir, GraphQL을 사용하여 멋진 프로젝트에서 작업하고 싶다면 AlphaSights에서 채용 중이다. 채용 직군을 확인하려면 https://engineering.alphasights.com/#positions 를 방문하라.
