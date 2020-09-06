---
layout: post
title:  "Applicative"
date:   2020-08-27 17:53:57 +0900
categories: jekyll update
comments: true
---

### Applicative 란 무엇일까
- 보통 Functor -> Applicative -> Monad 순으로 개념이 확장되는데, 포스팅의 순서는 Functor -> Monad -> Applicative 가 되었네요. 
간단하게 앞의 두 개념을 요약하자면 Functor는 **mappable한 타입**, Monad는 **function composition을 가능케 하는 두 성질 - pure 와 compose - 를 만족하는 타입** 이라고 할 수 있습니다. 본격적으로 Applicative에 대해 알아보겠습니다. 이번 포스팅은 [이 링크](https://medium.com/@tzehsiang/javascript-functor-applicative-monads-in-pictures-b567c6415221)를 주로 참고했습니다.

- Applicative 는 한마디로, **pure와 ap(apply) 의 속성을 만족하는 타입** 이라고 합니다. 모나드 관련 포스팅에서 pure의 의미에 대해 간락하게 살펴보았습니다. Wrapper라고 저는 이해했는데요, 수학적 언어로는 

```javascript
class (Functor f) => Applicative f where
    pure :: a -> f a
    (<*>) :: f (a -> b) -> f a -> f b
```
이렇게 표현이 가능하다네요. 수학적 해석은 차치하고, 우선 어떻게 쓰일 수 있는지 구체적 사례를 봐보겠습니다. 

```javascript
> [(*2), (+3)] <*> [1, 2, 3]
[2, 4, 6, 4, 5, 6]
```
- Applicative는 값 뿐만 아니라, 함수도 wrapping이 가능합니다. Wrapper에서 연산에 쓰이는 value(값), 연산을 정의한 함수를 꺼내서 연산 후, 결과를 붙여서 반환하게 합니다. 

- 전전 포스팅의 functor와 크게 다른 점은 서로 다른 함수들이 **제한없이 값들에게 적용될 수 있다는 점**입니다. 가령 array에 map을 적용할 때는 

```javascript
let array = [1,2,3,4];
let updatedArray = array.map(e=>e+3); // [4,5,6,7]
```
- 위와 같이 한 element당 하나의 오퍼레이션이 적용됩니다. Applicative는 (옳은 비유인지 모르겠습니다만) 
```javascript
(pseudo code) array<*>[addThree, multiplyByFour, subtract5] = [4,5,6,7,16,20,24,28,-1,0,1,2]
```
 와 같은 복수의 함수들이 한번에 적용될 수 있는 것이고, 결과값은 각각의 함수들이 매핑한 값을 이어 놓은 것이 되는 것이죠!

 - 그럼 이 친구는 어디에 쓸까요? [여기](https://stackoverflow.com/questions/6570779/why-should-i-use-applicative-functors-in-functional-programming) 에 따르면 **중간 결과값이 필요하지 않은 액션의 값을 순서대로 보여줄 때** 주로 쓰인다고 합니다. 가장 흔한 예로는 **파싱**이 있다네요. 시간이 여유로울 때 [이 링크](https://arunraghavan.net/2018/02/applicative-functors-for-fun-and-parsing/)를 분석해서 이번 포스팅을 보충해보려 합니다.




---

### Functor 요약
![functor1](https://miro.medium.com/max/1400/1*Xoa27kT2FJ-HBtwsZRFrHg.gif)
- functor의 일종인 배열이 파이프(함수)에 의해 잘 매핑되고 있습니다.

![functor2](https://miro.medium.com/max/1400/1*gVhVPPtYrf44OPLpXEdU4A.gif)
- 함수도 functor가 될 수 있습니다. 함수를 **함수가 반환하는 모든 output의 집합** 이라고 보면 가능합니다. 위의 그림에서와 같이 두 개 이상의 파이프가 합쳐져도 mappable 함을 알 수 있습니다. 


- Monad와 Applicative의 그림도 있는데,
![summary](https://miro.medium.com/max/1400/1*kXKjxS87eVC58Rj4KTkWVQ.gif)

뭔가 직관적으로 와닿지 않고 더 어려워진 느낌이라 일단 생략하겠습니다. 오늘 포스팅은 여기까지 하겠습니다.

