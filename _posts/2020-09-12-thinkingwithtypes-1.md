---
layout: post
title:  "Thinking with types - 1장"
date:   2020-09-14 17:53:57 +0900
categories: jekyll update
comments: true
---

Thinking with Types를 읽기 시작했습니다. 챕터 1부터 시작합니다.
## Chaper 1. The Algebra behind types

- 이 장의 목표는 Cardinality와 Isomorphism의 정의에 대한 이해, 그리고 연산을 타입으로 표시하는 방법에 대한 설명입니다. 뒤에는 간단한 수식들을 Haskell의 type을 가지고 정의해볼겁니다.

### 1.1 Isomorphism(동형)과 Cardinalities(집합원의 갯수)

- 우선 Cardinality는 '집합원의 갯수'라고 할 수 있습니다. **한 타입에서 가능한 경우의 수**라고 이해하면 깔끔할 것 같습니다. 예를 들어,
    - data Void 의 Cardinality는 0
    - data () = () (unit입니다) 는 1
    - data Bool = False | True 로 2 입니다.

- 두 개의 Cardinality가 같은 타입은 항상 Isomorphic(동형) 이라고 합니다. 한마디로 서로 1대1 대응이 가능하다는 뜻인데요, 두 타입에 대한 to와 from이 아래처럼 정의될 수 있을 때,
to :: s->t
from :: t->s
isomorphic 하다고 하며 s~=t라고도 적습니다. 

- 두개의 Isomorphic 한 타입이 n의 Cardinality를 갖는다고 할 때, 가능한 i**somorphism의 조합은 n!**이 됩니다. 간단하게 예를 들어 두 개의 타입 n과 l이 
type n = 1 | 2 | 3 | 4
type l = a | b | c | d
- 라고 했을 때, 타입 l을 unique하게 나열하는 방법의 수는 4!이며, 여기에 위치값인 1,2,3,4 가 대응된다고 생각하면 n!이 isomorphism의 경우의 수 임을 알 수 있습니다. 

### 1.2 Sum, product and exponential type

- Type으로 sum과 product, 그리고 exponential 을 나타내보겠습니다.
[Maybe a] = 1 + [a]
[Either a b] = [a] + [b]
[(a,b)] = [a] * [b]

- [Maybe a] = 1 + [a] 는 nothing or a입니다. 따라서 nothing을 나타내는 상태 1에 a의 상태갯수인 [a]가 더해져 1+a
- [Either a b] = [a] + [b] 는 a 혹은 b 이므로 두 개의 상태가 합쳐진 [a] + [b]가 됩니다.
- [(a,b)] = [a] * [b] 에서 (a,b)가 조합가능한 쌍을 나타낼 때, (a,b)의 가지수는 a*b가 되므로 이것이 성립합니다.

- 그렇다면 exponential은 어떻게 표현될까요?
[a->b] = [b]^[a] 
- 이유는 모든 a의 값에 대해서 b의 가능성이 있기 때문입니다. 예를 들어 a가 (가위, 바위, 보)고 b가 bool(이긴다, 진다)이면, 각각 가위를 냈을 때 이기고 지는 경우, 바위를 냈을 때, 보를 냈을 때의 경우의 수가 있기 때문에 2*2*2 = 8이 가능한 모든 결과입니다.

[![img]({{ "/assets/curry-howard.png"|absolute_url}})]({{ "/assets/curry-howard.png"|absolute_url}})
- 위는 Curry-Howard isomorphism을 검색하니 나온 표입니다. ~~참~쉽죠?~~ 깔깔.

- Curry-Howard Isomorphism은 쉽게말해 모든 논리적 명제는 컴퓨터 프로그램과 동치이다- 라는 말인데요, 저번 포스팅에서 살펴보았듯이 덧셈은 Either로, 곱셈은 (a,b)로, 거듭제곱은 a->b의 형식으로 표기될 수 있기에 (반대도 역시 성립합니다) 가능하다고 볼 수 있습니다. 

- 흥미로운 식 하나를 보겠습니다.
> a^1 = a

- 이를 저번시간에 배운 타입형식으로 변환하면 ()->a = a 가 됩니다. (unit의 cardinality가 1임을 생각합시다) 즉, **어떤 한 값과 그 값을 산출해내는 프로그램은 근본적으로 같다** 를 나타냅니다. 함수형 프로그래밍의 data 중심 사고방식과, 각 값들의 immutability가 생각나는것은 왜 일까요.

