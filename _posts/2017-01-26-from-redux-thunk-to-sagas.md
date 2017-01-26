---
id: from-redux-thunk-to-sagas
title: redux-thunk에서 redux-saga로
category: CSS
---
[원문보기](https://medium.com/@deeepakampolu/from-redux-thunk-to-sagas-2896c0abc676#.ptlov4222)

redux를 사용해봤다면 HTTP 리퀘스트나 타이머 같은 비동기 조작을 사용하여 상태를 변경할 때 문제를 겪어봤을 것이다.

redux에 의해 제공되는 제약에 묶여있지 않다면, 우리는 Promise resolve나, 혹은 setTimeout 안의 mutate하는 함수를 통해 애플리케이션과 상태를 흩뜨릴 것이다. 하지만 우리는 mutable 상태와, 일관성없음과 버그들로 인해 불타버릴 것이라는 것을 더 잘 알고 있다.

우리 논의를 일반적으로 두기 위해서, 이미 상태를 반영하는 리듀서들을 만들어두었다고 가정하자. 이 방법으로, 액션과 미들웨어에 대해서 할 수 있을 것이다.

redux를 사용하여 서버에 비동기 요청을 수행하기 위한 여러가지 액션들을 정의한다:

 1. MAKE_ASYNC_REQUEST
 2. MAKE_ASYNC_REQUEST_PROGRESS
 3. MAKE_ASYNC_REQUEST_SUCCESS
 4. MAKE_ASYNC_REQUEST_FAILURE

비동기 작업을 시작해야할 필요가 있을 때, 즉 사용자 액션이나 애플리케이션 시작 시점에서, 첫 번째 액션이 디스패치된다. 두 번째 액션은 선택사항이며, 요청의 진행 상황(로딩 바를 말함)을 나타내기 위해 디스패치된다. 이런 요구사항이 없을 경우, 로딩 인디케이터를 화면에 나타내는 플래그를 애플리케이션 상태에 두는 것이 최고다.

세 번째 및 네 번째 액션 중 하나만 디스패치된다. 각각 성공 또는 실패를 나타내며, 그에 따른 애플리케이션 상태 변경과 다시 그에 따라 UI에 반영된다.

인기있는 추상화는 액션 대신 함수를 디스패치하는 것이다. 이 함수는 첫 번째 인수로 디스패치에 대한 엑세스 권한을 가지며 필요에 따라 여러 액션을 디스패치할 수도 있다.

<script src="https://gist.github.com/vamsiampolu/3ea1ec3a90fa1a3c6ac5b634665f6f8c.js"></script>
*thunk를 이용한 비동기 동작 수행*

우리는 redux-thunk라는 미들웨어를 redux에 사용하여 함수 디스패치를 다룬다. 여기서 주목해야할 한 가지는 flux 표준 액션으로 변환할 수 있는 적절한 미들웨어가 있다면, redux를 사용해서 모든 것을 디스패치할 수 있다는 것이다.

다룰 수 있는 적절한 redux 미들웨어가 있다면 Promises, Observables 혹은 심지어 Generators라도 디스패치할 수 있다. 비동기 작업을 기술할 때 thunks나 promises를 사용하는 것은 파악하기 어려울 수 있다.

또한, 그들 자신의 각 효과에 대해 파악하는 것이 제한된다. 어떤 경우에는 오프라인 store나, 웹 API, 혹은 기존 앱 상태의 조각에 대해 입력해야할 필요가 있을 수 있다.

간단히 말해, thunks는 좋은 시작점은 될 수 있겠지만, 확장에 쓰이기는 어렵다는 것이다. 더 적합한 접근 방법은 redux-saga 미들웨어를 사용해서 비동기 작업에 대한 워크플로우를 오케스트레이션하는 것이다.

saga는 한 프로세스를 몇 개의 더 작은 프로세스로 나눌 수 있도록 하는 오류 관리 패턴이다. 하위 프로세스 중 하나가 성공하거나 실패하면, 그 정보로 애플리케이션 상태를 업데이트한다.

sagas를 더 잘 이해하려면 다음 비디오를 참고하라:

<iframe width="560" height="315" src="https://www.youtube.com/embed/xDuwrtwYHu8" frameborder="0" allowfullscreen></iframe>

redux-saga는 또한 서로 통신하는 여러 프로세스에 의존하는 동시성 패턴인 Communicating Sequential Processes(CSP)에 의해 영감을 얻었다. 우리 예제의 경우, 주어진 순간에, 오직 두 개의 통신하는 프로세스만이 존재한다. 모든 프로세스들은 미들웨어와 직접 통신한다.

프로세스는 제네레이터로 표현된다. 제네레이터는 잠시 실행을 멈췄다가 다시 멈춘 부분부터 시작할 수 있는 특별한 종류의 함수다. 제네레이터는 기본적으로 꼭두각시이며 이터레이터가 그것을 제어한다. 문자열의 경우에는, 이터레이터 자체의 next와 throw 메서드에 의해 제어된다.

제네레이터는 단순하다. 이터레이터의 질문에 응답하고 이터레이터가 재응답하기 전까지 기다린다. 다음 질문에 도달하기 전까지 표현식들을 평가(evaluate)하고 그 값을 질문의 응답으로 돌려주는 것이다.

아래 예제에서, 제네레이터는 이터레이터에게 `3`을 알려주고 이터레이터는 에러를 제네레이터에게 던져서 응답한다. 또한 제네레이터 호출이 제네레이터 바디를 실행하지 않고 이터레이터만 생성하는 방법 대해 주목하라.

<script src="https://gist.github.com/vamsiampolu/3eb090828e80d5c360e3c97285d86df4.js"></script>

Kyle Simpson은 제네레이터에 대한 일련의 게시물을 작성했다. 제네레이터에 대해 더 알고싶다면 읽어보라:

[](https://davidwalsh.name/es6-generators)

그리고 당신이 물어보기 전에 내가 예제를 저곳으로부터 훔쳤다는 것을 알려준다. 미들웨어와 통신하기 위해 사가는 미들웨어에게 자신이 무엇을 하기를 원하는 지 알려준다. effect로 알려진 이것은 redux 액션과 평행성을 그린다(동일하다)

redux-saga는 몇 가지의 effects를 가지고 있다. 일부는 일반적이며, 일부는 redux에 특정된다. 그리고 다른 일부는 스레드와 같은 동시성을 가능케 한다. 다음은 몇 가지 인기있는 액션 생성자들이다:

|effect|explanation|
|:----:|:---------:|
|put|redux 스토어에 액션을 디스패치한다|
|select|셀렉터를 사용하여 기존 애플리케이션 상태의 일부를 얻어온다|
|call|다른 saga들이나 promise 등을 호출할 수 있다|
|take|액션이 디스패치되기를 기다린다|
|fork|서브 프로세스를 트리거한 뒤 완료를 기다리지 않고 이동한다|
|cancel|포크되었던 서브 프로세스를 취소한다|
|cancelled|현재 프로세스가 켄슬되었었는지 확인한다|
|delay|다음 구문으로 이동하기 전에 주어진 기간동안 대기한다, Promise를 리턴한다|

saga를 가지고 동일한 비동기 요청을 만들어보자:

<script src="https://gist.github.com/vamsiampolu/ccf619db7185ab808f3dfb54567f5bed.js"></script>
*보라, 콜백이 없다*

redux-saga로부터 임포트한 createSagaMiddleWare 함수를 호출해서 미들웨어를 구한다. 스토어를 만들 때 미들웨어를 등록하는 것 외에 추가적으로 애플리케이션의 메인 사가를 실행하는데도 쓰일 수 있다.

mainSaga를 사용해서 필요한 다른 모든 사가들을 시작할 수 있다. 이 사가들은 종종 워커 사가라고도 알려져있다. mainSaga는 sagaMiddleWare에 effect를 yield하여 info를 아규먼트로 사용해서 asyncRequestSaga를 call하라고 알려준다.

mainSaga 내에 여러 개의 사가가 있는 경우 call 대신 fork를 사용해야 한다. 왜냐하면 fork는 saga를 호출하고 다음 스템으로 이동하도록 디자인되어 있기 때문이다. call은 사가가 완료될 때까지 블록되기 때문이다.

asyncRequestSaga 내에서, 액션으로부터 몇몇 데이터를 받은 다음 call을 사용하여 사가 미들웨어에게 비동기 API를 호출하도록 알려준다. 약간의 복잡성을 추가기위해, 애플리케이션 상태가 필요하다고 가정하자. 우리는 사가 미들웨어에게 필요한 상태를 획득하는 함수를 알려줄 수 있다.

그 다음 `put`을 사용하여 디스패치해야하는 액션을 알려주자. 에러 핸들링 또한 쉽다. 그냥 try-catch를 사용하면 된다.

이 접근법을 좋아한다면, 아래의 몇몇 라이브러리들을 확인해야만 할 것이다. 경고하건대, 이 목록은 내 자바스크립트 윈도우 쇼퍼로서의 경험에 근거한 것이다.

[tj/co](https://github.com/tj/co)
[async-csp](https://github.com/dvlsg/async-csp)

만약 다른 언어들에도 관심이 있다면, 다음을 확인해봐도 된다:

[Goroutines](https://gobyexample.com/goroutines)
[core.async](http://www.core-async.info/tutorial)

이것들은 조금 부담 될 수도 있고 큰 영향을 주지 못할 수도 있지만 어쨌든 한번 확인해보라.
