---
layout: post
title: "Unboxed와 Union 타입"
date: 2020-11-16 17:53:57 +0900
categories: jekyll update
comments: true
---

ReScript의 variant가 하나의 필드를 가진 레코드로 된 인자를 갖고있다고 생각해 봅시다.

```javascript
type name = Name(string)
let name_student = Name("Joe")

type greeting = {message: string}
let greeting_hi = {message: "hello!"}
```

JavaScript로 변환된 결과를 확인 해 보면, 아래와 같습니다.

```javascript
var name_student = /* Name */ {
  _0: "Joe",
};

var greeting_hi = {
  message: "hello!",
};
```

위 예제의 greeting 같은 레코드와는 다르게, variant로 감싸진 레코드는 \_0과 같은 메타 데이터 필드가 있습니다. 하나의 필드를 가진 레코드를 하나만 가진 variant에 대해서, 값을 바로 가져올 순 없을까? 라는 질문에서 이번 [@unboxed] 가 나오게 되었습니다. 태그 이름에서 볼 수 있듯이, box를 제거해서 값만 노출시킬 수 있게 하는 역할을 합니다.

```javascript
@unboxed
type name = Name(string)
let name_student = Name("joe")

@unboxed
type greeting = {message: string}
let greeting_hi = {message: "hello!"}
```

이렇게 @unboxed 가 붙으면,

```javascript
var name_student = "Joe";
var greeting_hi = "hello!";
```

변수에 값들이 바로 바인딩 됩니다.

그럼 어떤 방법으로 @unboxed를 활용해 볼 수 있을까요?
간단히 얘기하면 '같은 구조의 레코드를 가진 variant를 구분할 수 있게 되는 것'에 이점이 있습니다. 아래의 coordinate 변환 예시를 보겠습니다.

```javascript
type coordinates = {x: float, y: float}
let toWorldCoordinates = (localCoordinates) => {
  {
    x: localCoordinates.x +. 10.,
    y: localCoordinates.x +. 20.,
  }
}

let playerLocalCoordinates = {x: 20.5, y: 30.5}
let renderDot = (coordinates) => {
  Js.log3("Pretend to draw at:", coordinates.x, coordinates.y)
}
renderDot(playerLocalCoordinates)
```

여기서 renderDot은 local이 아닌 globalCoordinate를 받아야 하는데, 지금의 renderDot은 아무 coordinate이나 받고 있는 문제가 있습니다. 여기서는 coordinate 레코드를 각각 localCoordinate, worldCoordinate variant의 인자로 받아 unboxed로 꺼내올 수 있습니다.

```javascript
type coordinates = {x: float, y: float}
@unboxed type localCoordinates = Local(coordinates)
@unboxed type worldCoordinates = World(coordinates)

let renderDot = (World(coordinates)) => {
  Js.log3("Pretend to draw at:", coordinates.x, coordinates.y)
}

let toWorldCoordinates = (Local(coordinates)) => {
  World({
    x: coordinates.x +. 10.,
    y: coordinates.x +. 20.,
  })
}

let playerLocalCoordinates = Local({x: 20.5, y: 30.5})
renderDot(playerLocalCoordinates->toWorldCoordinates)
```

참고로 ReasonML에서 [@unboxed]는 무조건 하나의 인자만 가질 수 있습니다.
위의 예시에서 rendoerDot은 globalCoordinate만 가질 수 있습니다. 만약 localCoordinate를 넣는다면,

```javascript
renderDot(playerLocalCoordinates) // ERROR!

  This has type: localCoordinates
  Somewhere wanted: worldCoordinates
```

위와 같은 에러가 나옵니다. 코스트 0의 JS 인터롭(interop)을 지원한다는 점에서 @unboxed 기능은 환영할 만 합니다.

## Union type

Union 타입은 여러 타입 중 하나가 될 수 있는 값의 타입입니다. JavaScript에서 버티컬 바 ( | )로 타입을 나누는 것처럼, ReScript에서도 @unboxed를 사용해서 비슷한 기능을 할 수 있습니다.

```javascript
@unboxed
type t =
  | Any('a): t;
  //컴파일 되지 않음. 이해도 되지 않음 Variant_Type: t 가 무슨 뜻일까요?
let a = (v: a) => Any(v);
let b = (v: b) => Any(v);
let c = (v: c) => Any(v);
```

@unboxed 기능으로 Any(a)는 런타임에서 a와 같은 값을 나타내게 됩니다. 여기서 유저로 하여금 a, b, c 타입만 타입 t로 변환이 가능하도록 하고 싶을 때는 어떻게 하면 좋을까요? 모듈 시스템의 도움을 받아 봅시다.

만약 a 타입의 값을 정확하게 알아야 할 경우는 어떨까요? 케바케긴 하지만, a,b,c의 타입의 런타임 인코딩에 공통부분이 있느냐에 따라 달라지게 됩니다. 원시타입에 대해선 Js.typeOf를 통해 number냐 string이냐를 구분해낼 수 있습니다.

TypeScript의 type guard 처럼, Union 타입을 구분할 때는 사용자의 지식에 의존해야 합니다. 하지만 그 사용자의 지식을 모듈화하여 적합성을 지역적으로 검사해 볼 수 있습니다. 아래의 number_or_string 예제를 봅시다.

```javascript
module Number_or_string: {
  type t;
  type case =
    | Number(float)
    | String(string);
  let number: float => t;
  let string: string => t;
  let classify: t => case;
} = {
  [@unboxed]
  type t =
    | Any('a): t;
  type case =
    | Number(float)
    | String(string);
  let number = (v: float) => Any(v);
  let string = (v: string) => Any(v);
  let classify = (Any(v): t): case =>
    if (Js.typeof(v) == "number") {
      Number(Obj.magic(v): float);
    } else {
      String(Obj.magic(v): string);
    };
};
```

**case라는 타입**은 Number를 가질수도, String을 가질 수도 있는 Union 타입입니다. 여기서 Variant의 값을 직접 확인하고 그에 따른 로직을 수행해주기 위해 Js.typeof 와 Obj.magic의 도움을 받습니다.

정리하자면, @unboxed 기능과 모듈 덕분에 union 타입을 algebraic data type으로 바꿀 수 있는 체계적인 방법이 생겼습니다. 이런 종류의 변환은 사용자의 사전 지식이 필요하기에 자세한 검토가 필요합니다. classify와 같은 함수가 필요치 않은 경우엔 이 변환은 완벽히 타입 세이프하게 이뤄질 수 있습니다.

## 백엔드에서 자주 발생하는 상황

백엔드 개발에서는 아래와 같은 상황이 자주 발생합니다.

```javascript
prisma.exchangeProduct
    ->Prisma.findMany({
        "orderBy": {
          "endPurchasingDate": "asc",
        },
        "where": {
============================================================
          "startPurchasingDate": {
              "gte" :
          }
############################################################
          "startPurchasingDate": {
              "lte" :
          }
-------------------------------------------------------------
          "endPurchasingDate": {
            "gte": Js.Date.make()->Js.Date.toISOString,
          },
        },
```

위의 경우처럼 어떤 경우에는 startPurchasingDate아래에 {"gte": something}을, 어떤 경우에는 {"lte": something}을 반환하고 싶은데, 이것을 하나의 함수로 표현하기가 어렵습니다. 가령 getCondition(time: bool)이라는 함수를 만들어서 switch 문으로 해결하려고 하면,

```javascript
switch(time){
    | => {"gte": something}
    | => {"lte": something} //ERROR! 리턴타입이 일치 하지 않음
}
```

오류가 나는 것이죠. 원시타입의 경우는 Js.typeof로 분류가 가능하지만, 그렇지 않을 경우엔 어떤 방법이 최선일까요? 고민은 계속됩니다.
