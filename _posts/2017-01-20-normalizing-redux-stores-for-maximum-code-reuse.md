---
id: normalizing-redux-stores-for-maximum-code-reuse
title: 코드 재사용을 최대화하는 Redux 스토어 정규화
category: Redux
---
[원문보기](https://medium.com/@adamrackis/normalizing-redux-stores-for-maximum-code-reuse-ae6e3844ae95#.beyrhslyz)

# 코드 재사용을 최대화하는 Redux 스토어 정규화

Redux의 가장 큰 장점 중 하나는 애플리케이션 상태가 하나의 진리의 원천(single source of truth)을 포함한다는 것이다. 그 상태 데이터는 정규화된 구조로 저장된다. 따라서 최소한의 노력으로 비즈니스 로직을 공유하거나 확장할 수 있다. 단점은 Redux가 비일반적인 양의 보일러플레이트를 필요로 한다는 것이다. 그러나 이런 유연성이, 내 의견으로는, 당신으로 하여금 이 보일러플레이트를 구매하게 하는 주요 이점중에 하나다.

MobX와 같은 객체 지향 솔루션은 보일러플레이트를 줄이지만, 테스트가능하고, 응집되어 있으되, 지나치게 결합하지 않은 - 다른 말로는 객체 지향적인 - 애플리케이션 상태를 구성하는 당신 자신만의 방법을 요구한다. 이 선택지가 "더 좋은" 방법인 것은 아니다. 모든 프로젝트가 요구사항에 기초하여 절충해야만 하는 것이다. 강조하기 위해 추가하건대, 나는 MobX를 직장에서 프로페셔널하게 사용하고, 아주 좋아한다. OOP의 오버헤드가 Redux가 요구하는 것 보다 적을 수도 있고, 혹은 Knockout 같이 오래된 OOP 기반 프레임워크로 작성된 레거시 코드는 정규화된 Redux Store 보다는 MobX로 변환하는 것이 더 쉽다.

이 포스트는 Redux의 강점을 강조하는 유스케이스 하나를 보여준다. 일부러 찾으려고 한 것은 아니지만, 내 사이드 프로젝트 중 하나를 시작한 후로, Redux에서 특히 잘 작동하는 피처를 추가하고 있었다. 최근 내가 Redux에 제기된 비판을 본 것이 내가 이 글을 쓰는 동기가 되었다.

이 포스트는 Redux와 스토어를 정규화 하는 것에 익숙한 사람을 독자로 가정했다. 이전 포스트에서 이 문제에 대해 자세히 논의한 적이 있다. 아래의 샘플 코드는 메모이징 Redux 셀렉터 작성 툴인 reselect를 자유롭게 사용하고 있다.

## 유스케이스

이전과 마찬가지로 코드 샘플은 내 북-트랙킹 웹사이트에서 가져온 것이다. 나는 그것을 내 자신만을 위한 라이브러리로 사용하고 또한 그것에 새로운 것을 시도한다. 나는 그것을 작성함으로써 React를 배웠고, 결과적으로 큰 성공과 함께 팀을 바꿨기 때문에 시간을 잘 보냈다고 할 수 있다 :)

특히, 사용자는 계층적인 주제의 컬렉션을 저장할 수 있다: 예를 들면, 미국 혁명은 미국 역사 아래 두고, 이는 다시 역사 아래 놓을 수 있다. 계층 구조는 Mongo에서 구체적인 paths로 저장된다.

나는 간략하게 코드에서 주제들이 어떻게 저장되는지와, 정규화된 상태가 약간 미묘하게 다른 방식으로 재사용될 수 있는지 설명할 것이다.

다음과 같은 내용을 강조한다: MobX에서는 이를 많은 추가 작업없이 달성하기가 힘들다는 것이다. 요점은 내가 보여주고 싶은 것이 Redux가 어떻게 잘 동작하는지 보여주는 것 뿐 아니라, 내 의견에 따르면 더 나은 대안이 될 수 있다는 것이라는 것이다.

## 주제 저장하기

여기 내 주요 애플리케이션 레벨의 저장소에 대한 단순화 버전이 있다:

<script src="https://gist.github.com/arackaf/289d7ba664a6e5f54e01ab4d29cb5884.js"></script>

When the subjects come back, they’re flattened into a hash. Then, methods for shaping this hash back into a useful collection are exported, and used as needed by different parts of the application.

For example, the part of the application that lists books, allowing you to search for and select a subject to filter by, or add to a book looks in part like this

<script src="https://gist.github.com/arackaf/f10cb1e850c241d93428f84d660d4813.js"></script>

That same subjectHash is plucked from a different part of the application state, and then shaped with the exported `stackAndGetTopLevelSubjects` method, the results of which combined with the rest of that reducer’s state.


## Extending this code

More interestingly, there’s another part of the application dedicated to editing subjects. All hierarchical subjects are listed out, and the user can drag and drop them onto new parents. But, as a subject is hovering over a valid new parent, that parent will show the pending subject in its children, so the user can get a visual sanity check before completing the drop. (styles are a work in progress, as always)

![](https://cdn-images-1.medium.com/max/800/1*1arwYSRMEDRTPm6K0wbvvQ.png)

The reducer in question accepts actions indicating the drop is pending, and from there it’s fairly simple to tweak the subjects hash to create the new, temporary subject, and have it show up under its pending new parent.

<script src="https://gist.github.com/arackaf/606aa0cab08b68d942686cd93b6818d8.js"></script>

Basically the 5 lines under if(currentDropCandidateId){which creates a shallow copy of that same subjects hash, creates and adds a copy of the dragging subject, with the path to make it a child of the drop target, and that’s it.

And if there is no currentDropCandidateId, then the imported subjectsSelectoris called. Yes, this selector does just call the same stackAndGetTopLevelSubjectsinternally, and so I could just call that same method below the if and ditch the else to save 3 lines, but subjectsSelectoris separately memoized, so these calls never cause any re-computation.

Along those lines, it is obnoxious that every time there’s a drag, each subject has its children re-computed in stackAndGetTopLevelSubjects, even if they’ve obviously not changed, but this, too, can easily be improved with simple caching and an ES6 Map. It’s doubtful any of these perf optimizations make any difference in real life here; I only do them because I’m irrational and the inefficiencies annoy me, and because it’s good experience working with these tools.

## The benefits of normalization

Could this have been done with MobX? Of course. And it wouldn’t really be that hard. Presumably there’d be a main, subjects observable array — from there I figure there’d be a computed that would read both the main subjects array, and the current drop id. If no dropId, the main subjects array would be returned, simply enough. And, if the dropId were set, then the target subject would have to be cloned and replaced, with a copy of the dragged subject added to the target’s children array — all without violating any MobX invariants: observables can never be modified in a computed property definition. And, of course the target subject would have to be found, no matter how deep in the hierarchy it is, so either some sort of search to find it, or a computed lookup map defined off of the main subjects array. It’s doable, but I honestly think the Redux solution would be the more straightforward.

Or maybe there’s a simpler way I’m not seeing. Even if so, that still demonstrates my point: Redux forces you into a paradigm that’s fundamentally flexible and simple.

## Conclusion

There’s no silver bullet. Both approaches have pros and cons that should be understood. The Redux approach of storing normalized data, shaped as needed offers a unique flexibility, at the cost of lots of boilerplate, plus code that may be unintuitive to the uninitiated.

If that flexibility is not in high demand for your project, you may very well be better off with MobX. Also, the MobX ability to easily create cascading reactions lends itself well to some use cases, “spreadsheets” being a common example.

I suppose my main motivation for writing this is to try to combat the increasing criticisms I see Redux getting. Yes, it requires more code, but it also offers unique benefits.
