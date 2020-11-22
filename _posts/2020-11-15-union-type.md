---
layout: post
title: "Unboxed와 Union 타입"
date: 2020-11-16 17:53:57 +0900
categories: jekyll update
comments: true
---

오늘 다룰 주제는 @unboxed 기능과 Union타입, 그리고 실제 적용 사례입니다.
Variant가 하나의 필드를 가진 레코드로 된 인자를 갖고있다고 생각해 봅시다.

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

위 예제의 greeting 같은 레코드와는 다르게, variant로 감싸진 레코드는 \_0과 같은 메타 데이터 필드가 있습니다. 하나의 필드를 가진 레코드를 하나만 가진 variant에 대해서, 값을 바로 가져올 순 없을까? 라는 질문에서 이번 [@unboxed] 가 나오게 되었습니다. 태그 이름에서 볼 수 있듯이, 알맹이를 감싸고 있는 box를 제거해서 값만 노출시킬 수 있게 하는 역할을 합니다.

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
아래의 coordinate 변환 예시에선 coordinate 레코드가 주어지고, localCoordinate과 worldCoordinate 의 개념이 있습니다. RenderDot은 local을 world로 변환한 값을 인자로 받아야 합니다.

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

여기서 renderDot은 local이 아닌 worldCoordinate 받아야 하는데, 지금의 renderDot은 아무 coordinate이나 받고 있습니다. 여기서는 coordinate 레코드를 각각 localCoordinate, worldCoordinate variant의 인자로 받아 unboxed로 꺼내올 수 있습니다.

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

사실, 위의 예제는 **@unboxed가 없어도 잘 컴파일되고, 심지어 결괏값도 같습니다**! 유일한 차이는 컴파일된 Js파일의 형태인데요, 먼저 @unboxed가 있는 경우는,

```javascript
function toWorldCoordinates(coordinates) {
  return {
    x: coordinates.x + 10,
    y: coordinates.x + 20,
  };
}
```

없는 경우는 아래와 같습니다.

```javascript
function toWorldCoordinates(coordinates) {
  var coordinates$1 = coordinates._0;
  return /* World */ {
    _0: {
      x: coordinates$1.x + 10,
      y: coordinates$1.x + 20,
    },
  };
}
```

@unboxed를 활용한 variant와 arguemnt 구조분해를 통해 퍼포먼스의 영향없는 깨끗하고 variant의 래핑이 없는 Js 코드를 생성해 낸 것이죠!

## Union type

Union 타입은 여러 타입 중 하나가 될 수 있는 값의 타입입니다. JavaScript에서 버티컬 바 ( | )로 타입을 나누는 것처럼, ReScript에서도 @unboxed를 사용해서 비슷한 기능을 할 수 있습니다.

```javascript
@unboxed
type t =
  | Any('a): t;
let a = (v: a) => Any(v);
let b = (v: b) => Any(v);
let c = (v: c) => Any(v);
```

@unboxed 기능으로 Any(a)는 런타임에서 a와 같은 값을 나타내게 됩니다. 여기서 유저로 하여금 a, b, c 타입만 타입 t로 변환이 가능하도록 하고 싶을 때는 어떻게 하면 좋을까요? 모듈 시스템의 도움을 받아 봅시다.
만약 a 타입의 값을 정확하게 알아야 할 경우는 어떨까요? 케바케긴 하지만, a,b,c의 타입의 런타임 인코딩에 공통부분이 있느냐에 따라 달라지게 됩니다. 원시타입에 대해선 Js.typeOf를 통해 number냐 string이냐를 구분해낼 수 있습니다.

TypeScript의 type guard 처럼, Union 타입을 구분할 때는 사용자의 코드에 의존해야 합니다. 하지만 그 사용자의 지식을 모듈화하여 적합성을 지역적으로 검사해 볼 수 있습니다. 아래의 number_or_string 예제를 봅시다.

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

정리하자면, @unboxed 기능과 모듈 덕분에 union 타입을 algebraic data type으로 바꿀 수 있는 체계적인 방법이 생겼습니다. 이런 종류의 변환은 사용자의 사전 지식이 필요하기에 자세한 검토가 필요합니다. 사용자 지식은 결국 각 타입을 식별할 수 있는 classify와 같은 함수를 만드는데 필요한데, 이것이 필요치 않은 경우엔 각 변환은 완벽히 타입 세이프하게 이뤄질 수 있습니다.

## 현업 백엔드에서 자주 발생했던 상황

이제 이해를 바탕으로, 실제 사례를 봐보겠습니다.
백엔드 개발에서는 아래와 같은 상황이 자주 발생합니다.

```javascript
prisma.getItems
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

오류가 나는 것이죠. 원시타입의 경우는 Js.typeof로 분류가 가능하지만, 그렇지 않을 경우엔 어떤 방법이 최선일까요? 이 때 발견한 것이 Hongbo의 블로그 글입니다. (https://rescript-lang.org/blog/bucklescript-release-7-0-2) 예를 들면 이런 것이 가능합니다.

```javascript
[@unboxed]
type t =
  | Any ('a) : t;

let array = [|Any(3), Any("a")|];
```

```javascript
(컴파일 결과)
var array = [
  3, "a"
]
```

3, "a"와 같이 타입이 다른 데이터를 Any로 감싸면 한 배열안에 넣는것이 가능합니다. 컴파일러는 각각이 Any 같은 타입의 값이 다른 녀석들로 인식하기 때문이죠. 여기서 만약 @unboxed가 없었다면 Any안의 값을 불러오는 것이 불가능했을 겁니다. 실제로 @unboxed를 제거한 후의 컴파일 결과는 아래와 같습니다.

```javascript
(@unboxed가 없을 때)
var array = [
  /* Any */{
    _0: "hello"
  },
  /* Any */{
    _0: 3
  }
```

@unboxed를 이용하면 Variant안의 값을 바로 가져오기 때문에 서로 다른 타입의 데이터를 한 배열에 넣어 직접 사용할 수 있습니다. 언뜻 보면 직접 Js 코드를 삽입할 수 있는 [%raw]와 다를 것이 없어보이는데요, unboxed를 활용한 variant의 사용은 Any 안의 데이터 타입을 한정하여 원하는 타입만 받도록 강제할 수 있다는 장점이 있습니다. 이제 실제 코드를 바꿔보겠습니다.

```javascript
[@unboxed]
type t =
  | Any(Js.t({..})): t; //Js object로 인자를 제한합니다.
let any = a => Any(a);

prisma.getItems
    ->Prisma.findMany({
        "orderBy": {
          "endPurchasingDate": "asc",
        },
        "where": {
          "startPurchasingDate":
            if(condition_is_true){
              any({"lte": //something})
            }else{
              any({"gte": //something})
            }
          "endPurchasingDate": {
            "gte": Js.Date.make()->Js.Date.toISOString,
          },
        },
```

이렇게해서, 서로 다른 구조를 갖는 오브젝트를 반환하는데 성공했습니다! 여기선 굳이 Union type이 쓰이진 않았는데, 데이터를 모든 Js 객체로 한정했기 때문에 굳이 서로 다른 세부 타입으로 나눌 이유가 없었기 때문입니다. 이렇게해서 [%raw] 혹은 Obj.magic의 금기를 사용하지 않고 좀 더 괜찮은 코드가 탄생했습니다 :)
