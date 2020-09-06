---
layout: post
title:  "ReScript의 여러 문법 알아보기 - 2, type"
date:   2020-09-02 17:53:57 +0900
categories: jekyll update
---

킹 갓 디바인 셀레스쳘 언어 ReScript의 문법을 알아보는 두번째 시간입니다. 
## Types 파라미터 (a.k.a 제너릭)

- 지난 시간은 주로 primitive type들, string, boolean, int, float, unit 을 알아보았는데요, 이번에는 조금 특별한 경우들을 알아보겠습니다. 다른 언어들의 '제너릭'과 비슷한 기능을 하는 타입입니다.

```javascript
    type coordinates <'a> = ('a, 'a, 'a)

    let a: coordinates<int> = (10, 20, 20)
    let b: coordinates<float> = (10.5, 20.5, 20.5)
```
- 위의 경우 'a 에 int가 들어가면 int 형식으로, float이 들어가면 float으로 치환되어 적용됩니다. 사실 ReScript의 타입 추론은 매우 강력해서, 'a 표현이 없이도 전부 잘 동작합니다.

```javascript
// `array<string>`으로 자동 추론됩니다.
let greetings = ["hello", "world", "how are you"]

// 아래의 경우는 에러가 납니다.
let greetings = ["hello", 12, "how are you"]

//   let greetings = ["hello", 12, "how are you"]
//   This has type:
//     int
//   But somewhere wanted:
//     string
```

- 언젠가는 자세히 다루려고 했는데, 자기 자신을 element로 가지는 재귀적 타입은 아래와 같이 나타납니다. 이를 이용해서 여러 자료구조나 알고리즘을 구현할..수 있겠죠?
```javascript
type rec person = {
    name: string,
    friends: array<person>
}
```

- 상호 재귀적 타입은 어떻게 될까요? 이 섹션을 보기전에 따로 시도해봤가 포기했는데, 바로 아래처럼 구현가능합니다.

```javascript
type rec student = {taughtBy: teacher}
and teacher = {students: array<student>}

//이렇게 사용되겠네요
let malfarty: teacher = {students:[]};
let harrykim = {
  taughtBy: malfarty
}
```

- ReScript는 강력하고 고집센 타입 시스템을 자랑하지만서도, 약간의 인간미(!)는 남겨두었습니다. 바로 "%identity"를 통한 타입 변환인데요, 아래의 예시를 살펴보겠습니다.
```javascript
external convertToFloat : int => float = "%identity"
let age = 10
let gpa = 2.1 +. convertToFloat(age) //12.1

external convertToString : int => string = "%identity"
let age = "hello, "
let gpa = 123
Js.log(age++convertToString(123)); //hello, 123
```
