---
layout: post
id: tao-of-react
title: 리액트의 도(Tao of React)
tags: [React]
---

## 리액트의 도(Tao of React)

[원문보기](https://alexkondov.com/tao-of-react/)

저는 2016년 부터 리액트를 가지고 작업을 해왔지만 여전히 어플리케이션 구조나 설계에 대한 하나의 모범 사례는 없는 것 같습니다.

마이크로 레벨의 모범 사례는 있었지만, 대부분의 팀들은 자신들만의 구조나 설계를 가지고 있었습니다.

물론 단 하나의 모범사례를 모든 비즈니스와 애플리케이션에 적용할 수는 없습니다. 하지만 생산적인 코드 베이스를 만들어 내기 위해 따라야할 몇 가지 일반적인 룰이 있습니다.

소프트웨어 아키텍쳐와 설계의 목적은 생산성 향상과 유연함을 유지하는 것입니다. 개발자들은 핵심 부분을 재작성하지 않고도 효율적으로 수정 작업을 할 필요가 있습니다.

이 글은 제가 리액트로 작업을 하면서 효율적임이 증명된 원칙과 룰의 모음입니다.

컴포넌트와 애플리케이션 구조, 테스트, 스타일링, 상태 관리와 데이터 가져오기에 대한 좋은 사례들의 개요를 잡았습니다. 몇몇 예제들은 지나치게 단순화 했지만 구현에 중점을 두는 것이 아니라 원칙에 중점을 두는 것이기 때문에 양해 바랍니다.

모든 것은 의견일 뿐 절대적인 것이 아닙니다. 소프트웨어를 작성하는 방법은 하나 이상입니다.

## 컴포넌트

### Functional Component를 우선하세요.

Functional Component는 더 간단한 문법을 가지고 있습니다. 라이프사이클 메서드, 생성자나 판박이(boilerplate) 코드도 없습니다. 가독성을 잃지 않고도 같은 로직을 더 적은 문자수로 표현할 수 있습니다.

에러 바운더리 컴포넌트 구현을 제외하면 Functional Component를 우선시해야 합니다. 당신의 머릿속에서 고려해야 할 멘탈 모델의 크기가 훨씬 줄어듭니다.

```javascript
// 👎 클래스 컴포넌트는 장황합니다
class Counter extends React.Component {
  state = {
    counter: 0,
  }

  constructor(props) {
    super(props)
    this.handleClick = this.handleClick.bind(this)
  }

  handleClick() {
    this.setState({ counter: this.state.counter + 1 })
  }

  render() {
    return (
      <div>
        <p>counter: {this.state.counter}</p>
        <button onClick={this.handleClick}>Increment</button>
      </div>
    )
  }
}

// 👍 함수형 컴포넌트는 읽기 쉽고, 유지 보수하기도 쉽습니다
function Counter() {
  const [counter, setCounter] = useState(0)

  handleClick = () => setCounter(counter + 1)

  return (
    <div>
      <p>counter: {counter}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  )
}
```

### 일관성 있는 컴포넌트 작성

컴포넌트들에 대해 동일한 스타일을 고수하세요. 헬퍼 함수들을 같은 위치에 두고, 같은 명명 패턴을 따르고, 같은 방법으로 export 하세요.

다른 방법들보다 더 이득이 되는 방법이란 것은 존재하지 않습니다.

파일 맨 아래쪽에서 익스포트하는 방법을 사용하건, 컴포넌트를 선언과 동시에 바로 익스포트하는 방법을 사용하건 상관없습니다. 다만 하나를 골라서 일관성 있게 적용하세요.

### 컴포넌트 이름짓기

항상 컴포넌트에 이름을 지으세요. React Dev Tools를 사용할 때나, 에러 스택 트레이스를 읽을 때 큰 도움이 됩니다.

또한 컴포넌트에 이름이 있다면 컴포넌트 개발 중에도 해당 컴포넌트가 어디에 있는지 찾기 쉬워집니다.

```javascript
// 👎 이렇게 하지 마세요
export default () => <form>...</form>

// 👍 함수에 이름을 붙이세요
export default function Form() {
  return <form>...</form>
}
```

### 헬퍼 함수들을 조직하기

클로저를 유지할 필요 없는 헬퍼 함수들을 컴포넌트 바깥으로 이동하세요. 이상적인 이동 장소는 컴포넌트 정의 이전 위치 입니다. 위에서부터 아래로 쭉 읽어올 수 있게 됩니다.

컴포넌트 안의 노이즈를 감소시키고, 딱 필요한 것들만 컴포넌트 안에 존재할 수 있도록 합니다.

```javascript
// 👎 클로저를 유지할 필요없는 함수를 굳이 컴포넌트 안쪽에 넣지 마세요
function Component({ date }) {
  function parseDate(rawDate) {
    ...
  }

  return <div>Date is {parseDate(date)}</div>
}

// 👍 컴포넌트 정의 위쪽에 헬퍼 함수를 두세요
function parseDate(date) {
  ...
}

function Component({ date }) {
  return <div>Date is {parseDate(date)}</div>
}
```

컴포넌트 정의 내에 최소한의 헬퍼 함수들만 유지합니다. 가능한한 많은 것들을 바깥 쪽으로 이동시키고 스테이트를 아규먼트로 전달합니다.

입력에만 의존하는 순수 함수로 로직을 구성하면 버그를 추적하고 기능을 확장하기가 쉬워집니다.

```javascript
// 👎 헬퍼 함수들은 컴포넌트의 스테이트를 읽을 필요가 없습니다
export default function Component() {
  const [value, setValue] = useState('')

  function isValid() {
    // ...
  }

  return (
    <>
      <input
        value={value}
        onChange={e => setValue(e.target.value)}
        onBlur={validateInput}
      />
      <button
        onClick={() => {
          if (isValid) {
            // ...
          }
        }}
      >
        Submit
      </button>
    </>
  )
}

// 👍 헬퍼 함수들을 추출해서 필요로 하는 값만 전달하세요
function isValid(value) {
  // ...
}

export default function Component() {
  const [value, setValue] = useState('')

  return (
    <>
      <input
        value={value}
        onChange={e => setValue(e.target.value)}
        onBlur={validateInput}
      />
      <button
        onClick={() => {
          if (isValid(value)) {
            // ...
          }
        }}
      >
        Submit
      </button>
    </>
  )
}
```

### 마크업을 하드코드하지 않기

네비게이션, 필터, 리스트를 위해 마크업을 하드코드하지 마세요. 설정 객체(configuration object)와 루프를 사용하세요

이것은 마크업과 아이템의 변경을 한 곳에서 다룰 수 있게 해줍니다.

```javascript
// 👎 하드코드된 마크업은 관리하기 어렵습니다
function Filters({ onFilterClick }) {
  return (
    <>
      <p>Book Genres</p>
      <ul>
        <li>
          <div onClick={() => onFilterClick('fiction')}>Fiction</div>
        </li>
        <li>
          <div onClick={() => onFilterClick('classics')}>
            Classics
          </div>
        </li>
        <li>
          <div onClick={() => onFilterClick('fantasy')}>Fantasy</div>
        </li>
        <li>
          <div onClick={() => onFilterClick('romance')}>Romance</div>
        </li>
      </ul>
    </>
  )
}

// 👍 루프와 설정 객체를 사용하세요
const GENRES = [
  {
    identifier: 'fiction',
    name: Fiction,
  },
  {
    identifier: 'classics',
    name: Classics,
  },
  {
    identifier: 'fantasy',
    name: Fantasy,
  },
  {
    identifier: 'romance',
    name: Romance,
  },
]

function Filters({ onFilterClick }) {
  return (
    <>
      <p>Book Genres</p>
      <ul>
        {GENRES.map(genre => (
          <li>
            <div onClick={() => onFilterClick(genre.identifier)}>
              {genre.name}
            </div>
          </li>
        ))}
      </ul>
    </>
  )
}
```

### 컴포넌트 길이

리액트 컴포넌트는 그저 props를 입력으로 받아 마크업을 돌려주는 함수일 뿐입니다. 그러므로 리액트 컴포넌트들도 함수와 동일한 소프트웨어 설계 원칙을 준수합니다.

보통 함수 하나가 너무 많은 일들을 하게되면, 로직을 다른 함수로 추출하고 그 함수를 호출하도록 합니다. 컴포넌트도 마찬가지입니다. 만약 컴포넌트가 너무 많은 기능을 가지고 있다면, 더 작은 컴포넌트를 만들고 그것을 호출하도록 합니다.

마크업 부분이 복잡하다면, 즉 루프와 조건 분기를 요구한다면, 더 작은 컴포넌트로 추출하세요.

데이터 전달과 커뮤니케이션은 props와 콜백에 의존하세요. 코드 줄 수는 객관적인 측정 수치가 아닙니다. 코드 줄 수 대신 추상화나 책임에 대해 생각하세요.

### JSX에 주석 쓰기

더 명확히 밝혀야할 것이 있다면, 코드 블록을 열고 추가 정보를 제공하세요. 마크업도 로직의 일부이므로 더 명확히 밝힐 사항이 있다면 코드 블록을 열어 주석으로 추가 정보를 제공하면 됩니다.

```javascript
function Component(props) {
  return (
    <>
      {/* If the user is subscribed we don't want to show them any ads */}
      {user.subscribed ? null : <SubscriptionPlans />}
    </>
  )
}
```

### Error Boundary 사용하기

한 컴포넌트에서 에러가 발생했더라도, 전체 UI가 망가져서는 안됩니다. 전체 페이지가 망가졌음을 나타내야할 치명적인 에러가 드물게 발생하기는 합니다만, 대부분의 경우에서는 스크린에서 특정 엘리먼트만 숨기는게 나을 겁니다.

데이터를 다루는 함수 안에는 많은 try/catch 구문이 존재할 수도 있습니다. 에러 바운더리를 탑레벨에만 두지 마세요. 스크린의 각 엘레멘트를 에러 바운더리로 감싸서 분리시키면 연쇄적인 오류를 방지할 수 있습니다.

```javascript
function Component() {
  return (
    <Layout>
      <ErrorBoundary>
        <CardWidget />
      </ErrorBoundary>

      <ErrorBoundary>
        <FiltersWidget />
      </ErrorBoundary>

      <div>
        <ErrorBoundary>
          <ProductList />
        </ErrorBoundary>
      </div>
    </Layout>
  )
}
```

### Props 비구조화하기

대부분의 리액트 컴포넌트들은 함수일 뿐입니다. props를 받아서, 마크업을 리턴합니다. 일반적인 함수들은 전달인자를 그대로 전달받습니다. 리액트 컴포넌트도 같은 원칙을 적용하는 것이 이치에 맞습니다. props를 여기저기서 반복적으로 사용할 필요가 없다는 이야기입니다.

props를 비구조화해서 사용하지 않는 이유는 외부로부터 비롯된 것과 내부 상태를 구분하기 위해서입니다. 하지만 일반 함수는 전달인자와 내부 변수를 구별하지 않습니다. 불필요한 패턴을 만들어내지 마세요.

```javascript
// 👎 컴포넌트 내에서 props를 반복해서 사용하지 마세요
function Input(props) {
  return <input value={props.value} onChange={props.onChange} />
}

// 👍 비구조화하여 직접 사용하세요
function Component({ value, onChange }) {
  const [state, setState] = useState('')

  return <div>...</div>
}
```

### Props의 갯수

컴포넌트가 얼마나 많은 props를 받아야만 하는가 하는 질문은 다분히 주관적입니다. props의 갯수는 컴포넌트가 얼마나 많은 일을 하느냐와 연관되어 있습니다. 더 많은 props를 전달한다면 컴포넌트가 더 많은 책임을 갖고 있다는 것입니다.

높은 props 갯수는 컴포넌트가 너무 많은 일을 한다는 신호입니다.

저는 만약 5개 이상의 props가 있다면, 컴포넌트를 쪼개는 것을 고려합니다. 어떤 경우에는 그냥 데이터가 많은 것일 수도 있습니다. 예로 들면 인풋 필드 같은 경우에는 props를 많이 가질 수 있습니다. 그러나 그 외 경우에는 추출해야할 무언가가 있다는 신호입니다.

NOTE: 컴포넌트가 더 많은 props를 취한다면, rerender될 이유가 더 많다는 것입니다.

### 원시타입들 대신 오브젝트를 전달하기

props의 숫자를을 제한하는 한가지 방법은 원시타입들 대신에 오브젝트를 전달하는 것입니다. username, emaill, settings를 하나씩 하나씩 전달하는 것보다, 그룹으로 묶어서 전달하는 것이 낫습니다. 이 방법은 또한 user가 추가적인 필드를 가지게 될 때, 필요로 하는 변경의 양을 줄일 수 있습니다.



타입스크립트를 사용한다면 더 쉽게 만들 수 있습니다.

```
// 👎 서로 연관된 값을 하나씩 전달하지 마세요
<UserProfile
  bio={user.bio}
  name={user.name}
  email={user.email}
  subscription={user.subscription}
/>

// 👍 대신 하나로 묶은 오브젝트를 사용하세요
<UserProfile user={user} />
```

### 조건부 렌더링

어떤 상황에서 숏 서킷 연산자를 사용해 조건부 렌더링을 하는 것은 역효과를 내거나, 의도치 않은 `0` 을 UI에 노출하게 됩니다. 이를 피하기 위해서 기본적으로 삼항 연산자를 사용하세요. 유일한 단점은 삼항연산자가 더 장황하다는 것입니다.

숏 서킷 연산자를 사용하는 것은 코드의 양을 줄여줍니다. 코드의 양을 줄이는 것은 항상 훌륭한 일입니다. 삼항 연산자는 더 장황합니다. 하지만 별로 잘못될 일이 없죠. 거기에다가 조건문을 변경 해야 하는  경우에 코드 변경량이 더 적습니다.

```
// 👎 숏 서킷 연산자를 사용하는것을 피하도록 노력해 보세요
function Component() {
  const count = 0

  return <div>{count && <h1>Messages: {count}</h1>}</div>
}

// 👍 대신 삼항 연산자를 쓰세요
function Component() {
  const count = 0

  return <div>{count ? <h1>Messages: {count}</h1> : null}</div>
}
```

### 중첩된 삼항 연산자 피하기

첫 번째 레벨의 삼항 연산자 이후의 삼항 연산자는 정말 읽기 힘듭니다. 시간과 공간의 낭비를 줄일 수 있다고 하더라도 말이죠. 당신의 의도를 명확하고 확실하게 나타내는 것이 더 낫습니다.

```
// 👎 JSX 내 중첩된 삼항 연산자는 읽기 어렵습니다
isSubscribed ? (
  <ArticleRecommendations />
) : isRegistered ? (
  <SubscribeCallToAction />
) : (
  <RegisterCallToAction />
)

// 👍 내부 컴포넌트를 하나 만들어서 로직을 명확히 드러내세요
function CallToActionWidget({ subscribed, registered }) {
  if (subscribed) {
    return <ArticleRecommendations />
  }

  if (registered) {
    return <SubscribeCallToAction />
  }

  return <RegisterCallToAction />
}

function Component() {
  return (
    <CallToActionWidget
      subscribed={subscribed}
      registered={registered}
    />
  )
}
```

### 리스트들은 별도 컴포넌트로 분리하기

`map` 함수를 통해서 아이템 목록을 반복하는 것은 일상다반사입니다. 그러나 많은 마크업을 가지고 있는 컴포넌트에서 추가적인 들여쓰기(indentation)와 `map` 문법은 가독성에 전혀 도움이 되지 않습니다.



만약 여러분이 엘레멘트들을 map으로 반복해야할 때라면, 별도의 목록 컴포넌트로 추출하세요. 비록 원래 컴포넌트 내에 마크업이 많지 않더라도 말입니다. 부모 컴포넌트는 세부 사항에 대해 알 필요가 별로 없습니다. 그냥 목록을 표시하기만 하면 됩니다.



컴포넌트의 주요 책임이 마크업을 반복하고 표시하는 것일 경우에만 마크업에 루프를 두어도 됩니다. 컴포넌트 당 하나의 맵핑만을 두도록 노력해보세요, 하지만 마크업이 길거나 복잡한 경우엔 무조건 리스트 컴포넌트로 추출하세요.

```
// 👎 나머지 마크업과 루프를 함께 작성하지 마세요
function Component({ topic, page, articles, onNextPage }) {
  return (
    <div>
      <h1>{topic}</h1>
      {articles.map(article => (
        <div>
          <h3>{article.title}</h3>
          <p>{article.teaser}</p>
          <img src={article.image} />
        </div>
      ))}
      <div>You are on page {page}</div>
      <button onClick={onNextPage}>Next</button>
    </div>
  )
}

// 👍 리스트를 하나의 컴포넌트로 추출하세요
function Component({ topic, page, articles, onNextPage }) {
  return (
    <div>
      <h1>{topic}</h1>
      <ArticlesList articles={articles} />
      <div>You are on page {page}</div>
      <button onClick={onNextPage}>Next</button>
    </div>
  )
}
```

### 비구조화 시 props에 기본값 할당하기

props의 기본값을 지정하는 한가지 방법은 컴포넌트에  `defaultProps` 프로퍼티를 덧붙이는 것입니다. 이는 컴포넌트 함수와 그 전달인자의 값이 같은 위치에 있지 않게 된다는 뜻입니다.



props를 비구조화할 때, 직접 기본값을 지정하는 것을 우선시하세요. 이 방법을 이용하면 코드를 읽을 때, 불필요하게 코드의 윗부분 아랫부분을 왔다갔다 할 필요 없이, 전달인자가 정의된 곳에서 기본값을 읽을 수 있게 만들기 때문에 코드가 읽기 쉬워집니다.

```
// 👎 함수 바깥에 props의 기본값을 정의하지 마세요
function Component({ title, tags, subscribed }) {
  return <div>...</div>
}

Component.defaultProps = {
  title: '',
  tags: [],
  subscribed: false,
}

// 👍 전달인자 목록에 기본값을 함께 정의하세요
function Component({ title = '', tags = [], subscribed = false }) {
  return <div>...</div>
}
```

### 중첩된 렌더 함수 피하기

컴포넌트나 로직에서 마크업을 추출해야할 필요가 있을 때, 같은 컴포넌트 내에 추출한 마크업을 함께 두지 마세요. 컴포넌트는 그저 함수일 뿐이고, 쉽게 부모 컴포넌트 내부에 중첩된 함수로 정의해 둘 수 있습니다.

하지만 그렇게 하면 중첩된 함수가 부모 컴포넌트의 상태와 데이터에 접근할 수 있게 됩니다. 이것이 코드를 더욱 읽기 어렵게 만듭니다. 이 모든 컴포넌트 사이에서 이 함수가 과연 무슨 일을 하는것일까? 하는 의문을 들게 만드는 것이죠

부모 내부에 컴포넌트를 정의하지 말고 따로 컴포넌트로 분리하세요. 클로저를 사용하기 보다는 이름을 붙이고 props를 전달하세요.

```
// 👎 렌더 함수 내에 중첩된 렌더 함수를 두지 마세요
function Component() {
  function renderHeader() {
    return <header>...</header>
  }
  return <div>{renderHeader()}</div>
}

// 👍 별도의 컴포넌트로 추출하세요
import Header from '@modules/common/components/Header'

function Component() {
  return (
    <div>
      <Header />
    </div>
  )
}
```



## 상태 관리

### 리듀서 사용하기

종종 상태(State) 변화를 관리하는 더 강력한 방안이 필요할 때가 있습니다. 그럴 때 외부 라이브러리로 눈을 돌리기 전에 먼저 `useReducer` 로 시작해보세요. `useReducer`는 복잡한 상태 관리를 다루는 훌륭한 메커니즘이면서, 서드파티 의존성을 요구하지도 않습니다.

리액트의 Context, 타입 스크립트, useReducer의 조합은 정말 강력합니다. 하지만 불행히도, 널리 쓰이진 않고 있습니다. 사람들은 아직도 서드파티 라이브러리에 손을 대고 있습니다.

만약 조각조각 분리된 여러 상태가 필요해진 상황이라면, 상태를 리듀서로 옮기는 것을 고려해보세요.

```javascript
// 👎 조각조각 분리된 상태가 너무 많습니다. 이렇게 사용하지 마세요
const TYPES = {
  SMALL: 'small',
  MEDIUM: 'medium',
  LARGE: 'large'
}

function Component() {
  const [isOpen, setIsOpen] = useState(false)
  const [type, setType] = useState(TYPES.LARGE)
  const [phone, setPhone] = useState('')
  const [email, setEmail] = useState('')
  const [error, setError] = useSatte(null)

  return (
    ...
  )
}

// 👍 대신 리듀서로 통합하세요
const TYPES = {
  SMALL: 'small',
  MEDIUM: 'medium',
  LARGE: 'large'
}

const initialState = {
  isOpen: false,
  type: TYPES.LARGE,
  phone: '',
  email: '',
  error: null
}

const reducer = (state, action) => {
  switch (action.type) {
    ...
    default:
      return state
  }
}

function Component() {
  const [state, dispatch] = useReducer(reducer, initialState)

  return (
    ...
  )
}
```

### HOC와 Render props보다 Hooks를 우선시하기

종종 기존 컴포넌트의 기능을 강화(enhance)하거나, 기존 컴포넌트에 외부 상태에 접근할 수 있는 능력을 주고 싶은 경우가 있습니다. 보통 세 가지 방법이 사용됩니다. 바로 higher order components(HOCs), render props, hooks 입니다.

Hooks는 이러한 컴포넌트의 능력을 조합하는 데 가장 효과적인 방안임이 증명되었습니다. 철학적인 관점에서, 컴포넌트는 다른 함수들을 사용하는 함수입니다. Hooks를 사용하면 서로 충돌하는 일 없이 외부로부터 기능을 가져와서 덧붙일 수 있게 해줍니다. hooks의 갯수가 상당히 늘어나더라도, 각각의 값들이 어디로부터 온건지 쉽게 알 수 있습니다.

HOCs를 이용하는 경우 props로 값들이 전달됩니다. 이 값들은 부모 컴포넌트로부터 온 것인지, 아니면 HOCs로부터 전달된 것인지 알기 어려워집니다. 또한, 여러 props를 체이닝하는 것은 에러의 원인으로 알려져 있습니다.

Render props는 깊은 들여쓰기와 좋지 못한 가독성을 야기합니다. 마크업 트리에 여러 컴포넌트를 render props로 중첩시키는 것은 보기에도 좋지 않습니다. 또한 마크업 자체는 값들만을 노출하므로 로직을 따로 작성하거나, 아래로 전달해야 합니다.

hooks를 사용하면 추적하기 쉽고, JSX를 방해하지 않는 간단한 값들만으로 작업할 수 있습니다.

```
// 👎 render props 사용을 피하세요
function Component() {
  return (
    <>
      <Header />
      <Form>
        {({ values, setValue }) => (
          <input
            value={values.name}
            onChange={e => setValue('name', e.target.value)}
          />
          <input
            value={values.password}
            onChange={e => setValue('password', e.target.value)}
          />
        )}
      </Form>
      <Footer />
    </>
  )
}

// 👍 단순함과 가독성을 위한 훅을 사용하세요
function Component() {
  const [values, setValue] = useForm()

  return (
    <>
      <Header />
      <input
        value={values.name}
        onChange={e => setValue('name', e.target.value)}
      />
      <input
        value={values.password}
        onChange={e => setValue('password', e.target.value)}
      />
    )}
      <Footer />
    </>
  )
}
```

### 데이터 가져오기용 라이브러리 사용하기

종종 상태 안에서 관리하려는 데이터를 API로부터 가져오는 경우가 있다. 메모리에 데이터를 유지할 필요가 있으며, 다양한 곳에서 접근하고 업데이트할 수 있어야한다.

[React Query](https://react-query.tanstack.com/) 같은 모던 데이터 페칭 라이브러리는 외부 데이터를 관리하기에 충분한 메커니즘을 제공합니다. 캐시할 수도 있고, 캐시를 무효화 한다음 새로 받아올 수도 있습니다. 받아오기만 하는게 아니라 데이터를 보내거나, 아니면 리프레시로 인해 추가적인 데이터 조각을 가져오는 데에도 사용할 수 있습니다. 

[Apollo](https://www.apollographql.com/docs/react/) 같은 GraphQL 클라이언트를 사용하면 더 쉽습니다. 클라이언트 상태 개념이 내장되어 있기 때문입니다.

### 상태 관리 라이브러리

아마 대부분의 경우에 상태 관리 라이브러리는 필요없을 겁니다. 아주 복잡한 상태를 관리하는 커다란 애플리케이션에만 필요하기 때문입니다. 이 주제에 대한 다양한 가이드가 있으므로, 살펴볼 만한 두 라이브러리만 언급하고 넘어가겠습니다.

[Recoil](https://recoiljs.org/) 과 [Redux](https://redux.js.org/) 입니다.

## 컴포넌트 멘탈 모델

### Container와 Presentational

현재 우세한 사고방식은 컴포넌트를 두 그룹으로 나누는 것입니다. 바로 presentational 컴포넌트와 container 컴포넌트로 말이죠. 각각 스마트 컴포넌트(컨테이너)와 바보 컴포넌트(프레젠테이셔널)라고도 불립니다.

이 아이디어는 어떤 컴포넌트는 어떠한 상태나 기능을 같고 있지 않는다는 것입니다. 프레젠테이션 컴포넌트는 그저 부모로부터 props를 전달받아 호출되기만 합니다. 컨테이너 컴포넌트는 비즈니스 로직을 담고 있으며, 데이터를 페칭하고 상태를 관리합니다.

이 멘탈 모델은 백엔드 애플리케이션의 MVC 구조입니다. 어디에서나 작동하기에 충분히 일반적이며 잘못될 일이 별로 없습니다.

그러나, 현대적인 UI 애플리케이션에서는 이 패턴만으로는 부족합니다. 몇 가지의 컴포넌트가 모든 논리를 가지고 있으면, 부풀어 오르기 마련입니다. 해당 컴포넌트들은 너무나 많은 책임을 가지고 있어서, 관리하기가 어려워집니다.

### Stateless & Stateful

컴포넌트를 상태가 있는 컴포넌트와 상태가 없는 컴포넌트로 나누는 것입니다. 위에서 언급한 Container & Presentation 멘탈 모델은 몇몇 소수의 컴포넌트들로만 커다란 규모의 복잡성을 다루어야만 한다는 것을 의미합니다. 그렇게 하는 대신, 우리는 앱 전체에 이 복잡성을 쪼개어 퍼트려야 합니다.

데이터는 사용되는 위치와 가까운 곳에 있어야 합니다. GraphQL client를 사용할 때, 데이터를 보여주는 컴포넌트에서 해당 데이터를 가져옵니다. 탑 레벨의 컴포넌트가 아니어도요. **Container** 에 대해서는 생각하지 마세요. 대신 **Responsibilities** 에 대해서 생각하세요. 논리적으로 가장 적합한 위치에 상태를 두는 것에 대해 고려하세요.

예를 들어, `<Form />` 컴포넌트는 해당 form 이 필요한 데이터를 가져야만 합니다. `<Input />` 컴포넌트는 값들과 변경을 알릴 콜백을 전달해야 합니다. `<Button />` 컴포넌트는 눌러졌을 때 폼에 통지하도록 하고, 폼은 일어난 일에 대해 핸들링해야 합니다.

폼의 유효성 검증은 누가 해야 할까요? 인풋 필드의 책임일까요? 이것은 당신의 애플리케이션 내 컴포넌트가 비즈니스 로직을 인식하게됨을 의미합니다. 폼에 에러가 발생하면 어떻게 통지해야 할까요? 어떻게 에러 상태가 갱신되야 할까요? 폼이 그것을 알아야 할까요? 에러가 있지만 서브밋을 시도한다면 무슨 일이 일어날까요?

이와 같은 질문에 직면했다면 책임이 뒤섞이고 있음을 인식해야 합니다. 위 경우에는 인풋 필드는 stateless로 남겨두고 폼으로부터 에러 메시지를 전달받도록 하는 것이 더 나을 것입니다.

## 애플리케이션 구조

### 라우트나 모듈을 기준으로 그룹화하기

컨테이너 컴포넌트와 프레젠테이션 컴포넌트로 그룹을 만드는 것은 애플리케이션 구조를 알아보기 어렵게 합니다. 컴포넌트가 어디에 속하는지 이해하게 하려는 것은 친숙함을 필요로 합니다.

모든 컴포넌트가 전부 동등하지는 않아서, 글로벌하게 사용되는 몇몇 컴포넌트와 애플리케이션의 특정 부분에서만 사용되는 컴포넌트로 그룹화하는 구조가 있습니다. 이 구조는 가장 작은 규모의 프로젝트에서는 잘 동작합니다. 그러나 이 구조에서는 점점 컴포넌트가 늘어나게 되면서 관리하기가 어려워지는 단점이 있습니다.

```
// 👎 기술적인 세부사항을 기준으로 그룹화하지 마세요
├── containers
|   ├── Dashboard.jsx
|   ├── Details.jsx
├── components
|   ├── Table.jsx
|   ├── Form.jsx
|   ├── Button.jsx
|   ├── Input.jsx
|   ├── Sidebar.jsx
|   ├── ItemCard.jsx

// 👍 모듈/도메인 기준으로 그룹화하세요
├── modules
|   ├── common
|   |   ├── components
|   |   |   ├── Button.jsx
|   |   |   ├── Input.jsx
|   ├── dashboard
|   |   ├── components
|   |   |   ├── Table.jsx
|   |   |   ├── Sidebar.jsx
|   ├── details
|   |   ├── components
|   |   |   ├── Form.jsx
|   |   |   ├── ItemCard.jsx
```

라우트나 모듈 별로 그룹화하는 것으로 시작하세요. 애플리케이션이 변화하고 성장하는 것을 지원하는 구조입니다. 요점은 애플리케이션이 성장하면서 특정 아키텍쳐 부분이 너무 커져버리게 하지 않는 것입니다. 컨테이너 컴포넌트와 프레젠테이션 컴포넌트 기반으로 그룹화할 때 자주 그런 일이 일어납니다.

모듈 기반의 아키텍쳐는 확장하기 쉽습니다. 별도의 복잡성을 증가시키지 않고 그저 탑 레벨의 피쳐 모듈 하나를 추가하기만 하면 됩니다.

컨테이너 컴포넌트 프레젠테이션 컴포넌트 기반의 구조는 틀리지 않았습니다면 너무 일반적입니다. 코드를 읽는 독자에게 그저 리액트 컴포넌트로 구성되어있다는 것 밖에 알려주지 않습니다.

### 공통 모듈 작성하기

버튼이나 인풋, 카드 같은 컴포넌트들은 모든 곳에서 사용됩니다. 모듈 기반의 애플리케이션 구조를 가져간다고 하더라도, 이러한 컴포넌트들을 따로 추출해두는 것이 좋습니다.

그렇게 한다면 스토리북을 사용하지 않더라도 어떤 공통 컴포넌트가 있는지 확인할 수 있어서, 중복을 피할 수 있습니다. 당신도 아마 당신 팀의 모든 팀원들이 자기들만의 버전의 버튼 컴포넌트를 가지는 것을 원하지는 않을 것입니다. 불행히도 잘못 구성된 프로젝트 때문에 자주 일어나는 일입니다.

### 절대 경로 사용하기

구조 변경을 쉽게 만들어두는 것이 프로젝트 구조화의 기본입니다. 절대 경로는 컴포넌트를 옮겨야할 필요가 있을 때 더 적게 바꿀 수 있음을 의미합니다. 또한 임포트한 모듈을 어디로부터 가져온 것인지 쉽게 알 수 있게 됩니다.

```
// 👎 상대경로를 사용하지 마세요
import Input from '../../../modules/common/components/Input'

// 👍 절대적인것은 변하지 않습니다
import Input from '@modules/common/components/Input'
```

보통 저는 `@` 프리픽스를 사용하지만 `~`를 사용하는 경우도 보았습니다.

### 외부 컴포넌트를 한 번 감싸기

너무 많은 서드파티 컴포넌트를 직접 임포트하는 것을 하지 않도록 시도해보세요. 서드파티 라이브러리들을 한번 감싼 어댑터를 만들어서 필요한 경우에 API를 수정할 수 있도록 하세요. 또한 다른 서드 파티 라이브러리로 교체할 경우 어댑터 한 곳만 수정하게 되는 이득이 있습니다.

Semantic UI같은 컴포넌트나 그 밖의 유틸리티 컴포넌트도 마찬가지 입니다. 가장 간단한 방법은 임포트한 컴포넌트를 바로 다시 익스포트하는 모듈을 만드는 것입니다.

감싼 컴포넌트를 사용하는 컴포넌트는 감싼 컴포넌트가 어떤 라이브러리를 사용하고 있는지 알 필요가 없습니다. 그냥 감싼 컴포넌트가 노출하는 것만 알면 됩니다.

```
// 👎 직접적으로 임포트하지 마세요
import { Button } from 'semantic-ui-react'
import DatePicker from 'react-datepicker'

// 👍 컴포넌트를 감싸서 노출시킨 다음 내부 모듈로 레퍼런스하게 하세요
import { Button, DatePicker } from '@modules/common/components'
```

### 컴포넌트를 자체 폴더로 만들어주기

저는 리액트 애플리케이션을 작성할 때 각각의 모듈 폴더마다 컴포넌트 폴더를 만들어 둡니다. 컴포넌트를 만들 때 가장 먼저 하는 일이 폴더를 만드는 일입니다. 스타일이나 테스트같이 추가적인 파일들이 필요하다면 그 폴더에 넣습니다.

일반적인 사례는 `index.js` 파일로 리액트 컴포넌트를 익스포트하는 것입니다. 그렇게 하면 `import From from 'components/UserForm/UserForm'` 과 같이  import 경로가 반복되는 일이 없게 됩니다. 그리고 또 하나. 여러 index.js 파일이 열려 있을 때 혼동되지 않도록 컴포넌트 이름으로 파일을 두어야 합니다.

```
// 👎 여러 컴포넌트 파일들을 함께 두지 마세요
├── components
    ├── Header.jsx
    ├── Header.scss
    ├── Header.test.jsx
    ├── Footer.jsx
    ├── Footer.scss
    ├── Footer.test.jsx

// 👍 컴포넌트만의 폴더를 만들어서 옮기세요
├── components
    ├── Header
        ├── index.js
        ├── Header.jsx
        ├── Header.scss
        ├── Header.test.jsx
    ├── Footer
        ├── index.js
        ├── Footer.jsx
        ├── Footer.scss
        ├── Footer.test.jsx
```

## 성능

### 섣부르게 최적화하지 않기

어떠한 종류의 최적화라도, 하기 전에 꼭 최적화의 이유를 확실히 해야 합니다. 애플리케이션에 영향이 없는 한 모범 사례를 맹목적으로 따르는 것은 노력을 낭비하는 것입니다.

예. 어떤 특정한 것들에 대한 인식은 중요합니다. 하지만 읽기 쉽고 유지 보수하기 쉬운 컴포넌트를 구축하는 것을 성능보다 우선 순위를 높게 두어야 합니다. 잘 작성된 코드는 향상시키기도 편합니다.

만약 애플리케이션에서 퍼포먼스 문제가 발생했다면, 측정한 다음에 문제의 원인을 파악해야 합니다. 번들 사이즈가 엄청 큰 상황에서 리렌더링 회수를 줄이는 것은 거의 의미가 없습니다.

성능 문제의 원인을 파악한 다음에는 영향이 큰 순서대로 수정하세요.

## 번들 사이즈 살펴보기

브라우저로 전달되는 자바스크립트의 양은 성능에서 가장 중요한 요소입니다. 당신의 애플리케이션이 엄청나게 빠를 수도 있지요. 하지만 4MB의 자바스크립트가 로드되기 전까지는 아무도 그 사실을 알지 못할 겁니다.

자바스크립트를 하나의 번들로 만들어서 전달하지 마세요. 애플리케이션을 라우트 레벨 혹은 더 낮은 레벨로 쪼개세요. 가능한한 적은 양의 자바스크립트를 전송하도록 만드세요.

백그라운드에서 나머지 번들을 로드하거나 사용자가 의도를 보일때 해당 번들을 로드하세요. 만약 PDF를 다운로드하는 기능의 버튼이 있다면, 해당 버튼에 마우스 커서가 호버되기 전까지 PDF 관련 라이브러리를 로드하지 않을 수도 있습니다.

### 리렌더링 - 콜백, 배열 그리고 객체

불필요한 리렌더링의 양을 줄이는 것은 좋은 시도입니다. 하지만 불필요한 리렌더를 줄이는 것이 애플리케이션에 가장 큰 영향을 끼치는 것은 아니라는 것을 염두에 두세요.

가장 일반적인 충고는 인라인 콜백 함수를 props로 전달하는 것을 피하라는 것입니다. 인라인 콜백 함수 하나를 사용하면 각각의 렌더링마다 새로운 함수가 만들어지는 것을 뜻하고, 이는 하위의 렌더링을 유발합니다. 지금 제가 제시하는 접근 방식 때문에, 저는 콜백으로 인한 성능 문제를 겪은 적이 없습니다.

실제로 성능 문제를 겪고 있다면 클로저가 원인인지 확인하고 제거해보세요. 하지만 그로 인해 코드 가독성을 떨어뜨리거나, 더 장황하게 만들지는 마세요.

props로 배열이나 객체를 인라인으로 전달하는 것도 같은 종류의 문제를 유발합니다. 레퍼런스가 매번 갱신되므로 리렌더링이 유발됩니다. 만약 고정된 배열을 전달하는 경우에는 컴포넌트 정의의 바깥쪽에 해당 배열을 상수로 정의해두고 그 레퍼런스를 전달하도록 만들어보세요.

## 테스팅

### 스냅샷 테스트에 의존하지 않기

제가 리액트로 작업하기 시작했던 2016년부터 지금까지 스냅샷 테스트가 컴포넌트 문제를 발견한 상황은 단 하나뿐입니다. 전달인자 없이 `new Date()` 를 호출해서 항상 기본값이 현재 시간으로 설정되었던 문제입니다.

이 외에도 스냅샷은 컴포넌트가 변경될 때 빌드가 실패하게되는 원인이 될 뿐입니다. 보통의 작업 흐름은 컴포넌트를 변경한 다음 스냅샷 테스트가 실패한 것을 확인하고, 스냅샷을 업데이트 한 뒤 계속 진행하는 것입니다.

오해하지 마세요. 스냅샷 테스트는 좋은 정밀 테스트(Sanity Test) 입니다. 하지만 컴포넌트 레벨의 테스트를 대체하기에는 좋지 않습니다. 저는 이저 더이상 스냅샷 테스팅을 하지 않습니다.

### 올바르게 렌더링되는지 테스트하기

여러분이 테스트할때 주로 테스트해야하는 것은 컴포넌트가 기대한대로 동작하는지 검사하는 것이여야 합니다. default props로 올바르게 렌더링 되는지, 전달된 props로 렌더링 되는지를 테스트하게 하세요.

주어진 인풋(props)에 따라 올바른 결과(JSX)를 반환해야 합니다. 스크린 상의 필요한 모든 것을 검사하세요.

### 상태와 이벤트를 검사하기

내부 상태를 가진 컴포넌트(stateful component)는 대부분 이벤트에 대한 응답을 통해 내부 상태가 변경됩니다. 이벤트를 시뮬레이션해서 컴포넌트가 맞게 응답하는지 확인하세요.

핸들러 함수가 호출되었고, 올바른 인수가 전달되었는지 확인하세요. 그 결과로 내부적인 상태가 올바르게 설정되었는지 확인하세요.

### 엣지 케이스 테스트

기본적인 테스트를 끝냈다면, 몇몇 엣지 케이스 다루는 테스트 케이스를 추가하세요.

빈 배열을 전달해서 확인없이 인덱스를 접근하는 일이 없는지를 검사하고, API 호출시 에러를 던지고 컴포넌트가 그것을 핸들링하는지 검사한다는 의미입니다.

### 통합 테스트 작성하기

통합 테스트는 전체 페이지나 커다란 컴포넌트를 검사하는 것을 의미합니다. 추상가 잘 동작하는지 테스트합니다. 통합 테스트는 애플리케이션이 예상대로 잘 동작하는지 확인하는 가장 자신감을 가질 수 있는 방법입니다.

컴포넌트 하나하나가 잘 동작하고, 단위 테스트가 잘 통과하더라도, 이들 컴포넌트를 통합하는 것에는 문제가 있을 수 있습니다.

## 스타일링

### CSS-in-JS 사용하기

많은 사람들이 동의하지 않을 만한 논란이 있는 의견입니다만, 컴포넌트에 대한 모든 것을 자바스크립트로 표현할 수 있기 때문에 Styled Components 혹은 Emotion을 사용하고 싶습니다. 유지보수할 파일이 하나 줄어들고 CSS 컨벤션에 대해 생각할 필요가 없어집니다.

리액트의 논리적인 단위는 컴포넌트이므로 관심사 분리 측면에서 관련된 모든 것을 소유해야만 합니다.

> NOTE: SCSS, CSS modules, Taliwind 같은 스타일링이 잘못된 옵션이란 것은 아닙니다. 그저 제가 추천하는 방법이 CSS-in-JS라는 것입니다.

### 스타일과 컴포넌트를 함께 유지하기

CSS-in-JS가 적용된 컴포넌트에서는 하나의 파일에 스타일과 함께 있는 게 일반적입니다. 이상적으로는 컴포넌트와 그 컴포넌트가 사용하는 것들이 한 파일에 있는 것입니다.

하지만 파일이 너무 길어지게 된다면 스타일만을 위한 파일을 만들어서 컴포넌트 파일 옆에 둘 수 있습니다. 이러한 패턴을 Spectrum 같은 오픈 소스 프로젝트에서 본 적이 있습니다.

## 데이터 가져오기

### 데이터 가져오기용 라이브러리 사용하기

리액트는 API로부터 데이터를 가져오거나 데이터를 업데이트하는 방법에 대해 의견을 제시하지 않습니다. 각 팀들은 보통 API 통신을 위한 자신들만의 비동기 함수의 구현을 서비스에 포함합니다.

이러한 방식은 로딩 상태를 관리하고 http 오류를 자체적으로 처리해야 함을 의미하고, 이는 많은 판박이 코드(boilerplate code)를 가진 장황한 코드를 유발합니다.

이러한 방법 대신 [React Query](https://react-query.tanstack.com/) 나 [SWR](https://swr.vercel.app/) 과 같은 라이브러리를 사용하세요. 컴포넌트 라이프사이클의 일부분에 딱 맞는 서버의 통신 방법을 제공합니다. 바로 훅이죠.

이러한 라이브러리는 자체적으로 로딩이나 에러 상태를 관리하고, 캐시를 만들어줍니다. 우리는 그냥 로딩 상태 에러 상태, 캐시를 다루기만 하면 되죠. 또한 데이터 처리를 위한 상태 관리 라이브러리의 사용을 없애줍니다.

























