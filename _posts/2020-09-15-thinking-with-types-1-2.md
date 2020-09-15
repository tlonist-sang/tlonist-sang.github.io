---
layout: post
title:  "Thinking with types - 1장-2"
date:   2020-09-15 17:53:57 +0900
categories: jekyll update
comments: true
---

## Curry-Howard Isomorphism과 문제풀이


[![img]({{ "/assets/curry-howard.png"|absolute_url}})]({{ "/assets/curry-howard.png"|absolute_url}})
- 위는 Curry-Howard isomorphism을 검색하니 나온 표입니다. ~~참~쉽죠?~~ 깔깔.

- Curry-Howard Isomorphism은 쉽게말해 모든 논리적 명제는 컴퓨터 프로그램과 동치이다- 라는 말인데요, 저번 포스팅에서 살펴보았듯이 덧셈은 Either로, 곱셈은 (a,b)로, 거듭제곱은 a->b의 형식으로 표기될 수 있기에 (반대도 역시 성립합니다) 가능하다고 볼 수 있습니다. 

- 흥미로운 식 하나를 보겠습니다.
> a^1 = a

- 이를 저번시간에 배운 타입형식으로 변환하면 ()->a = a 가 됩니다. (unit의 cardinality가 1임을 생각합시다) 즉, **어떤 한 값과 그 값을 산출해내는 프로그램은 근본적으로 같다** 를 나타냅니다. 함수형 프로그래밍의 data 중심 사고방식과, 각 값들의 immutability가 생각나는것은 왜 일까요.