---
id: performance-tools
title: 퍼포먼스 측정 도구들
category: React
---

[원문보기](https://facebook.github.io/react/docs/perf.html)

**임포트 하기**

```javascript
import Perf from 'react-addons-perf' // ES6
var Perf = require('react-addons-perf') // ES5 with npm
var Perf = React.addons.Perf; // ES5 with react-with-addons.js
```

## 개요

React는 그냥 두어도 적당히 빨라보인다. 그러나, 애플리케이션의 1 온스의 성능을 쥐어 짜야하는 상황에서는 React의 diff 알고리즘에 최적화 힌트를 추가할 수 있는 [shouldComponentUpdate()](https://facebook.github.io/react/docs/react-component.html#shouldcomponentupdate) 훅을 제공한다.

일반적으로 앱의 전반적인 성능을 살펴볼 수는 있지만, 추가적으로 제공하는 프로파일링 도구 `Perf`를 이용하면 어디를 후킹하여 최적화할 필요가 있는지 알 수 있다.

[Benchling Engineering Team](http://benchling.engineering)에서 작성한 다음 두 기사에서 심도있는 프로파일링 툴 사용법을 소개하고 있다.

 - ["Performance Engineering with React"](http://benchling.engineering/performance-engineering-with-react/)
 - ["A Deep Dive into React Perf Debugging"](http://benchling.engineering/deep-dive-react-perf-debugging/)

### 개발용 빌드 vs 제품용 빌드(Development vs. Production Builds)

만약 React 애플리케이션의 퍼포먼스를 본다거나 혹은 벤치마킹 중이라면 제품용 빌드[minified production build](https://facebook.github.io/react/downloads.html)를 사용하여 테스트 중인지 확인해야 한다.
개발용 빌드를 사용하고 있다면 개발에 도움이 되는 추가적인 경고들을 포함하고 있기 때문에 성능 저하의 원인이 되기 때문이다.

하지만, 이 페이지에서 소개하고 있는 perf 도구는 개발용 빌드에서만 동작합니다.  따라서 프로파일러는 앱 중에서 _상대적으로_ 문제가 되는 부분만을 표시한다.

### Perf 사용하기

`Perf` 오브젝트는 개발 모드에서만 동작한다. 앱의 제품용 빌드에 포함해서는 안된다.

#### 측정 방법

 - [`start()`](#start)
 - [`stop()`](#stop)
 - [`getLastMeasurements()`](#getlastmeasurements)

#### 결과 출력

다음 메서드들은 [`Perf.getLastMeasurements()`](#getlastmeasurements)의 리턴된 측정값을 사용하여 결과를 보기 좋게 한다.

 - [`printInclusive()`](#printinclusive)
 - [`printExclusive()`](#printexclusive)
 - [`printWasted()`](#printwasted)
 - [`printOperations()`](#printoperations)
 - [`printDOM()`](#printdom)

* * *

## 레퍼런스

### `start()`
### `stop()`

```javascript
Perf.start()
// ...
Perf.stop()
```

측정을 시작하고 멈춘다. 그 사이에 일어나는 React의 조작들이 기록되고 분석된다. 시간을 별로 들이지 않은 조작들은 무시된다.

측정을 멈추고 나서, [`Perf.getLastMeasurements()`](#getlastmeasurements)를 호출하여 측정값을 구할 필요가 있다.

* * *

### `getLastMeasurements()`

```javascript
Perf.getLastMeasurements()
```

바로 이전의 start-stop 세션에서의 측정치를 기술한 불투명한(opaque) 구조를 가진 데이터를 얻는다. 이 데이터를 저장면 [`Perf`](#printing-results)의 다른 출력 메서드에 전달할 수 있다.

> 유의사항
>
> 이 부분이 공개 API에 적용되는 부분이 되면 문서에 업데이트 할 것이므로, 반환값의 정확한 형식에 의존하지 말아라

* * *

### `printInclusive()`

```javascript
Perf.printInclusive(measurements)
```

전체 시간을 출력한다. 만약 아무런 아규먼트도 전달되지 않는다면 기본적으로 모든 측정값을 출력한다. 이 메서드는 다음 그림과 같이 멋진 형식의 테이블로 콘솔에 출력한다:

![](https://facebook.github.io/react/img/docs/perf-inclusive.png)

* * *

### `printExclusive()`

```javascript
Perf.printExclusive(measurements)
```

"Exclusive"는 컴포넌트를 마운트하는데 걸린 시간을 포함하지 않는다. 즉, props를 처리하고 `componentWillMount`, `componentDidMount` 등에 걸린 시간은 제외한다.

![](https://facebook.github.io/react/img/docs/perf-exclusive.png)

* * *

### `printWasted()`

```javascript
Perf.printWasted(measurements)
```

**프로파일러에서 가장 중요한 부분**.

"Wasted"는 아무것도 실제로 렌더링하지 않았을 때 소비된 시간이다. 즉, 렌더링 결과가 그대로여서 아무런 DOM도 터치하지 않은 시간이다.

![](https://facebook.github.io/react/img/docs/perf-wasted.png)

* * *

### `printOperations()`

```javascript
Perf.printOperations(measurements)
```

기본 DOM 조작들을 출력한다. 즉, "set innerHTML" 혹은 "remove"을 뜻한다.

![](https://facebook.github.io/react/img/docs/perf-dom.png)

* * *

### `printDOM()`

```javascript
Perf.printDOM(measurements)
```

제거될 예정이고, deprecation 경고를 출력한다. [`printOperations()`](#printoperations)로 변경되었다.
