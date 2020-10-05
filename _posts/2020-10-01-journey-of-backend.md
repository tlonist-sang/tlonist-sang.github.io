---
layout: post
title:  "Nexus, Graphql, Prisma와 함께한 백엔드 여정"
date:   2020-09-29 17:53:57 +0900
categories: jekyll update
comments: true
---

그린랩스의 개발팀의 선택 - 함수형 프로그래밍
그린랩스는 함수형 프로그래밍 언어인 ReasonML과 ReScript로 개발하고 있습니다. 둘은 서로 매우 가까운 친척관계로, OCaml이라는 함수형 언어에 문법적 설탕(Syntactic sugar)를 가미하여, BuckleScript를 통해 Node 생태계를 사용할 수 있도록 한 함수형 언어입니다. 

한가지 차이점이라면 ReasonML은 널리 쓰이고 있는 프론트 라이브러리인 React와 잘 결합하여 Reason-React라는 생태계를 공고히 갖춘 반면, 백엔드단의 지원은 두 언어 모두 미비한 부분이 많았습니다. 지금부터 함수형 언어를 프로덕션에 사용하고자하는 그린랩스의 원대한 출항이 어떻게 이뤄졌는지, 간략하게 살펴보겠습니다.

그린랩스 웹개발팀의 초기 백엔드 기술 스택은 Graphql, Prisma 그리고 Nexus 였습니다. Graphql과 Prisma는 익히 알려진 솔루션인 반면, Nexus는 비교적 최근에 등장한 프로젝트입니다. 코드 우선(code-first)의 [기치를 내걸고](https://www.prisma.io/blog/the-problems-of-schema-first-graphql-development-x1mn4cb0tyl3) resolver와 데이터 모델링 정의를 한곳에서 하며, 동적으로 graphql 스키마를 생성해주는 라이브러리입니다. [링크](https://nexusjs.org/docs/)

프론트가 Reason-React로 단번에 정해진 것과는 달리, 백엔드는 더 많은 개발진이 최근 활발히 작업하고 있는 ReScript로 가기로 결정했습니다. 막 개발팀이 생긴 초기에는, 새로운 함수형 친구 ReasonML과 ReScript를 익히기 위해 개발팀 전원이 [aoc](https://adventofcode.com/)등을 풀어가며 감을 익힘과 동시에 기술 스택에 대한 공부를 해 나갔습니다.

드디어 첫 과제가 나온 시점. 백엔드 개발에서는 넘어야 할 산이 있었으니 바로 타이핑(typing) 이었습니다. 내부에서 **퍼즐 맞추기 같다** 라는 이야기가 나올정도로, ReScript의 타입 시스템은 엄격하고 정교한 작업을 필요로 했습니다. 간략히 함수형 언어와 타입 시스템이 어떤 관계에 있는지 알아보겠습니다.

## 함수형 프로그래밍과 타입 시스템

[![img]({{ "/assets/orthogonal.png"|absolute_url}})]({{ "/assets/orthogonal.png"|absolute_url}})
<br/>
함수형 프로그래밍은 객체 지향 프로그래밍과 직교(orthogonal)하는 관계입니다. 세상을 모델링하는 방식에 있어 함수형은 부작용(side-effect)가 없는 코드를 작성하는 것에, 객체지향은 동작 추상화(behavioral abstraction) 초점을 맞추고 각기 발전해 왔습니다. 

강력한 정적 타입 시스템을 갖는 것이 함수형 언어의 필수조건은 아니지만, 부작용을 최소화하는 목적에 잘 부합합니다. 강력한 타입 시스템은, 
- 컴파일 시간에 많은 오류(스트링값을 곱하거나 뺀다거나 하는 등)를 잡아줄 수 있습니다. 
- 커리-하워드 정리에 의해 그 자체로 논리적 명제를 증명할 수 있습니다. 따라서 잘 짜여진 타입 시스템에서는 논리적으로 잘못된 프로그램이 작성될 수 없습니다(cannot go wrong). 

여기서 잠깐. 타입은 C나 자바같은 무수히 많은 언어들이 가진 특징인데, 함수형 언어에서만 특별할까요? 그럴수도, 그렇지 않을 수도 있습니다. 자바에서 String을 리턴하는 함수를 예로 들어보겠습니다. 

```java
public String hello(String input){
    String output;
    /*
        input을 사용하는 비지니스 로직
    */
    return output;
}
```
이 hello라는 함수가 리턴할 수 있는 값이 과연 엄격하게 String일까요? hello가 반환하는 값은 
- 스트링
- null
- 예외 (exception)
- 반환을 못함 (무한 루프등에 빠져)
등등의 의도하지 않은 경우가 많음을 알 수 있습니다. 하지만 정적 타입 시스템을 가진 함수형 언어에서는 스트링을 반환한다고 하면 문자 그대로 스트링을 반환함을 보장받을 수 있습니다. 타입 시스템은 개발자로 하여금 **사용하는 대상에 대한 명확한 이해와, 코드의 무결성**을 지원함으로써, 부작용없는 프로그램을 만드는데 도움을 주는 것이죠.


## 첫번째 시도 : 
첫번째 업무는 도매출하 신청을 받기위한 백엔드 구축이었습니다. 유저가 입력한 값을 받아 DB에 저장하는, 정말 기본중의 기본이었지만 ReScript로 작성하기 위해선 다음의 과정을 거쳐야 했습니다. 
1. GraphQL 서버 바인딩 - 서버 함수(new, start), schema와 context 정의
2. Prisma 바인딩 - client 생성, CRUD 함수, prisma object의 내부 값들 정의
3. Nexus 바인딩 - nexus 함수, 함수들의 인자, nexus플러그인
아래의 **바인딩**이란, 이미 존재하는 JavaScript 라이브러리에서 타입 없이 쓰이고 있던 모든 함수 및 객체들에 대해 각 함수의 인자와 반환값을 정의하고. 객체에 레코드 등의(record) 타입을 부여하는 작업을 뜻합니다.

예를 들어 nexus 바인딩을 위해선 아래와 같이 타입을 정의하는것이 먼저 필요합니다.
```javascript
type nexusSchema	
type mutationField	
type queryField	
type nexusObjectType	

type nexusSchemaPlugin	
type nexusSchemaPath = {	
  schema: string	
}	

type nexusObjectTypeOptions = {
  name: string
  definition: Prisma.prismaObject => unit
}
```
타입을 정의한 후에는 이 타입을 반환값 혹은 인자로 가지는 함수들에 대한 정의를 해줘야합니다.

```javascript
type nexusObjectTypeOptions = {	
  name: string	
  definition: Prisma.prismaObject => unit	
}	

@bs.module("@nexus/schema")	
external objectType: nexusObjectTypeOptions => nexusObjectType = "objectType"	

@bs.module("@nexus/schema")	
external mutationField: (~name: string, NexusResolver.mutationResolver<'argsType, 'prismaArgsType, 'instance>) => mutationField = "mutationField"	

@bs.module("@nexus/schema")	
external stringArg: argOpts => string = "stringArg"	
```
이렇게 **사용하고자하는 라이브러리의 모든 함수와 인자, 객체들을 바인딩**해야 했던 것이 극초기 백엔드의 모습이었습니다. 단순한 request라는 모델의 CRUD를 위해서 작성해야하는 코드의 양이 굉장히 많았고, 내부적으로 과연 이 방법이 지속가능한가에 대한 의문이 생겼습니다. 여러 회의를 통해 문제를 두 개로 정의했습니다.
첫째는 Graphql을 통해 요청되는 다양한 argument들에 대한 타이핑을 하나하나 해줘야 하는 것에 관한 것이었습니다.

```javascript
type User = {
  id: string,
  name: string,
  farm_id: int,
  address: Address,
  phone_number: string
}
```
Graphql에서는 이렇게 정의된 User에 대해 id만 물어볼수도, (id, name)을 물어볼 수도, (id, address, phone_number)을 물어볼 수도 있습니다. n개의 필드에 대해 2^n-1 개의 조합이 생길 수 있는데, 문제는 ReScript에선 이 조합을 하나하나 타입으로 만들어줘야 한다는 것이었습니다. 

```javascript
type user1{
  id: string
}
type user2{
  id: string,
  address: Address,
  phone_number: string
}
```
이런 식이었습니다. 물론 **Option으로** 값을 감싸서 **null 일 수 있음**을 표현할 수 있었지만, 그것도 나름의 문제가 있었습니다. 우선 null 일 수 있음을 용인하면 프론트단에서 오는 값이 정말 null로 보낸건지, 아니면 잘못된 요청이 온 것인지 구분할 수 없습니다. 또한 Option으로 표기된 값이 비어있을 경우 JavaScript에서는 undefined로 변환되는데, GraphQL이 의존하고 있던 라이브러리에서 객체안의 필드값이 undefined인 경우 오류를 뱉어내는 경우가 존재했습니다. 이를 핸들링하기 위해선 또 다른 전략이 필요한 상황이 된 것이죠.

두번째 문제는 prisma와 nexus단의 바인딩량이 너무 많다는 것이었습니다. 그에 앞서, 타이핑 안정성보다 [타입 안정성이 어떤 결과를 가져오는지](http://www.pl-enthusiast.net/2014/08/05/type-safety/)를 생각해보면, prisma와 nexus 단의 바인딩은 **라이브러리를 엄격하게 사용하는 것**과 관련되어 있었습니다. 즉, 바인딩을 통해 나는 넥서스(또는 프리즈마)를 요구하는대로 사용했다 - 라는 결론을 얻는 것인데, 이것이 우리가 추구하고자 하는 방향과 맞는가하는 의문이 있었습니다.

## 두번째 - 일단 타협:
빠른 대응이 필요한 시점이었기에, 우선은 첫번째에서 했던 시도를 중단하고 **JS로 돌아가 개발하기로** 일시적으로 합의하였니다. 

함수형 백엔드 개발에 대한 방향성은 그대로 가지며, 팀 내부적으로는 [Thinking with Types : type level programming in Haskell / Sandy Maguire 저], [Web development with ReasonML / J.David Eisenberg 저] 등을 열심히 읽으며 토론하며 돌파구로 GADT (Generalized Algebraic Data Type)를 염두해두고 공부해나갔습니다. 이 시기 회의를 통해 아래와 같은 결론을 얻었습니다.

### GraphQL
정말 타이핑이 필요한 부분은, 프론트엔드 -> 백엔드 로 넘어오는 GraphQL 인자입니다. 
```javascript
let createParam: NexusResolver.mutationResolver<NexusResolver.speciesQuantity, NexusResolver.userIdSpeciesQuantity,'b> = 
{
  _type: "request",
  args: {
    kind: stringArg(""),
    quantity: stringArg("")
  },
  resolve:  (_, args, { req, prisma }, _) => {
    // 이 부분에서 비즈니스 로직을 수행할 때 타입 세이프함이 중요합니다.

    prisma.request->create({
      data: {        
        kind: args.kind,
        quantity: args.quantity
      }
    })
  }
};
```
위의 예시를 보면, resolve 부분의 args에서 꺼낸 값들을, prisma의 인자로 넣기 전 args.kind, args.quantity 로 꺼내는 것을 볼 수 있습니다. 만약 prisma로 가기 전 어떤 로직을 수행해야 한다면, 서로의 타입을 고려한 연산이 이뤄져야 할 것입니다. (Int에 String을 더하거나, TimeStamp에 Float을 곱하는 등의 일이 없어야합니다)

이 부분이 백엔드 타이핑 중 가장 핵심적인 부분이라는 것에 동의한 후, 나머지 기술 스택에 대한 바인딩을 생각해보았습니다.

### prisma에 대해서
프론트단에서 들어오는 값들에 대한 Null 값 처리 및 타입 확인은 GraphQL에서 이뤄지고 있음을 확인했습니다. 여기서 Prisma Client가 쓰게 될 인자들에 대한 타이핑을 다시 하는 것이 의미있을까요? GraphQL에서 정의한 타입을 다시 정의하는 것밖에 되지 않는다고 판단해 Prisma의 인자들에 대해서는 Record가 아닌 Object를 사용하여 타입 체크를 하지 않기로 했습니다. ReScript의 Record와 Object에 대해선 [공식문서](https://rescript-lang.org/docs/manual/latest/object)를 참고하시면 좋을 것 같습니다.

### nexus에 대해서
그 다음 고민은 **과연 Nexus가 필요한가** 였습니다. Nexus에 대한 바인딩은 **넥서스를 얼마나 잘 사용하고 있는가**를 체크하는 행위로, 그걸 사용하는 입장에서 많은 시간과 노력을 들여가며 엄격히 다뤄야 할 필요를 느끼지 못했습니다. Nexus의 코드 우선(code-first) 개발에 대한 원론적인 회의감이 생겼습니다. 그래서 과김히 걷어내기로 결정했습니다. 

## 세번째 - 해결책 :

[![img]({{ "/assets/greenlabs-stack.png"|absolute_url}})]({{ "/assets/greenlabs-stack.png"|absolute_url}})

그렇게 해서 최종안이 탄생했습니다. Nexus 부분의 로직을 전부 걷어내고, Prisma에 대해서는 Record가 아닌 Object를 사용하기로 했습니다. 클라이언트에서 넘어오는 GraphQL 입력값(Input)에 대한 부분은 이전처럼 Type을 명시한 Record로 표기하기로 했습니다.

이렇게 결정한 후의 이점은 다음과 같습니다.
- 1. 의존하고 있는 라이브러리가 줄고, 바인딩해야 할 대상이 적어져 개발시간이 현저히 단축되었습니다.
- 2. 여전히 ReScript의 타입 시스템을 활용해 프론트엔드 단에서 넘어오는 데이터 조작 시 타입 안정성을 보장받을 수 있게 되었습니다.

[![img]({{ "/assets/rescript.png"|absolute_url}})]({{ "/assets/rescript.png"|absolute_url}})

이렇게 해서 함수형 언어, ReasonML과 ReScript를 사용한 그린랩스의 백엔드 개발은 첫 발을 내딛게 되었습니다. 곳곳에 혁신이 필요한 많은 다양한 업무들이 기다리고 있는데요, 언어가 발전하고 우리의 이해가 깊어지면서 앞으로는 더욱 새롭고 풍부한 함수형 개발이 있을 예정입니다!

