---
layout: post
id: 6-great-uses-of-the-spread-operator
title: 스프레드 오퍼레이터의 훌륭한 사용법 6가지
tags: [Javascript]
---
[원문보기](https://davidwalsh.name/spread-operator?utm_source=javascriptweekly&utm_medium=email)

새로운 언어의 문법들부터 JSX같은 커스텀 파싱까지, 자바스크립트 작성을 엄청나게 동적으로 만들어준 ES6와 바벨같은 것들에 감사한다. 나는 스프레드 오퍼레이터에 팬이 되었다. 이 3개의 점은 아마도 당신의 자바스크립트 태스크를 완료하는 방법을 바꿔놓을 것이다. 이 글의 다음 부분에서 내가 최고로 좋아하는 스프레드 오퍼레이터의 사용법을 나열할 것이다!

## Apply없이 함수 호출하기

`Function.prototype.apply`를 호출할 때, 인자를 배열로 전달하기 때문에 인자들을 배열에 세팅해야 한다.

```js
function doStuff (x, y, z) { }
var args = [0, 1, 2];

// Call the function, passing args
doStuff.apply(null, args);
```

스프레드 오퍼레이터를 쓰면 전부 함쳐서 apply를 쓰는 것을 피할 수 있다. 단순히 함수 호출 시에 배열 앞에 스프레드 오퍼레이터를 붙이기만 하면 된다.

```js
doStuff(...args);
```

코드는 짧아지고, 말끔해지고, 쓸모없는 null을 넣어야할 필요도 없다!
The code is shorter, cleaner, and no need to use a useless null!

## 배열 합치기

항상 배열을 합치는 [다양한 방법들](https://davidwalsh.name/combining-js-arrays)이 있었다. 그러나 스프레드 오퍼레이터는 배열을 합치는 새로운 방법을 제공한다.

```js
arr1.push(...arr2) // arr2의 항목들을 뒤쪽에 추가한다
arr1.unshift(...arr2) // arr2의 항목들을 앞쪽에 추가한다
```

만약 두 배열을 함칠 때 아무 곳에서 요소를 추가하고 싶다면, 다음과 같이 하면 된다:

```js
var arr1 = ['two', 'three'];
var arr2 = ['one', ...arr1, 'four', 'five'];

// ["one", "two", "three", "four", "five"]
```

다른 메서드들 보다 더 짧은 문법에 위치 제어 기능까지 추가된다!

## 배열 복사하기

배열을 복사하는 것은 빈번한 작업이다. 이전에는 `Array.prototype.slice`를 사용하곤 했다. 그러나 새로운 스프레드 오퍼레이트를 쓰면 배열을 다음과 같이 복사할 수 있다.

```js
var arr = [1,2,3];
var arr2 = [...arr]; // like arr.slice()
arr2.push(4)
```

기억하라: 배열 안의 오브젝트는 여전히 레퍼런스다. 따라서 모든 것의 자체가 "카피"되는 것은 아니다.

## arguments나 NodeList를 배열로 변환하기

배열을 복사하는 것과 비슷하게 `Array.prototype.slice`를 이용해서 [NodeList](https://davidwalsh.name/nodelist-array)와 arguments 객체들을 진짜 배열로 변환했었다. 그러나 이제 스프레드 오퍼레이터로 작업을 완료할 수 있다.

```js
[...document.querySelectorAll('div')]
```

심지어 arguments를 함수 시그니처안에서 배열로 변환할 수도 있다:

```js
var myFn = function(...args) {
// ...
}
```

`Array.from`을 사용할 수도 있다는 것을 잊지 말자!

## Math 함수 사용하기

당연히 스프레드 오퍼레이터가 배열을 다른 arguments로 "펼치"기도 한다. 따라서 어떤 함수든 스프레드를 써서 arguments 수가 몇 개라도 받을 수 있는 함수들에 arguments를 전달할 수 있다.

```js
let numbers = [9, 4, 7, 1];
Math.min(...numbers); // 1
```

Math 오브젝트의 함수 세트들은 스프레드 오퍼레이터를 함수 인자로 사용하는 것에 대한 완벽한 예제다.

## 재미있는 디스트럭쳐링

디스트럭쳐링은 나의 많은 React 프로젝트에서 사용한 재미있는 연습이다. 뿐만아니라 Node.js 앱에서도 마찬가지다. 레스트 오퍼레이터과 디스트럭처링을 통해 정보를 추출해서 변수에 할당할 수 있다. 다음과 같이 말이다:

```js
let { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
console.log(x); // 1
console.log(y); // 2
console.log(z); // { a: 3, b: 4 }
```

나머지 프로퍼티들은 스프레드 오퍼레이터 뒤쪽의 변수에 할당된다!

ES6은 자바스크립트를 효율적으로 만들 뿐만 아니라 재미있게도 만든다. 모던 브라우저는 모두 ES6 문법을 지원하기 때문에 별로 이들을 사용해서 놀아본 적이 없다면 반드시 해보는 것이 좋을 것이다. 환경에 관계없이 더 실험하는 것을 선호한다면 내 포스트 중 [Getting Started with ES6](https://davidwalsh.name/es2015-babel)를 확인해보라. 어떤 경우든 스프레드 오퍼레이터는 아주 유용한 피처고, 당신은 반드시 이에 대해 알아야 한다!
