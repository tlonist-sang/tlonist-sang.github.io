---
layout: post
title:  "ReScript의 JS interop"
date:   2020-08-31 17:53:57 +0900
categories: jekyll update
comments: true
---

오늘은 ReScript에서의 JavaScript interop에 대해서 알아보겠습니다. 대부분의 내용은 [공식 문서](https://rescript-lang.org/docs/manual/latest/bind-to-js-function)를 번역한 것입니다. 들어가기에 앞서, interop은 interoperation의 줄임말로, JavaScript interop 은 ReScript에서 **JavaScript를 사용할 수 있게** 해줍니다.

### JS Code를 그대로 사용하는 방법
- %%raw(``) 를 사용하면 JavaScript 코드를 **그대로** 사용하는 것이 가능합니다. 

```JavaScript
%%raw(`
// look ma, regular JavaScript!
var message = "hello";
function greet(m) {
  console.log(m)
}
`)
```

- 위의 코드는 아래와 동일합니다. 
```JavaScript
// look ma, regular JavaScript!
var message = "hello";
function greet(m) {
  console.log(m)
}
```
- %%raw(``) 가 최고로 날것의 JS 코드를 삽입하는것을 가능하게 했다면, 
- %raw(``)는 expression level의 JS코드를 넣을 수 있게 해줍니다. ReScript와 JavaScript가 공존하게 되는 것이죠.
- 디버거 삽입은 %debugger로 가능합니다. 

```JavaScript
let printOne = %raw(`
    function(){
        console.log('this is one')
    }`)
```

```JavaScript
let f = (x, y) => {
    %debugger //debugger 삽입
    x + y
}
```
- 이런방식의 interop은 ReScript를 안쓰는것과 같습니다. ReScript만의 파워풀한 타입 체킹과 패턴 매칭을 사용하기 위해서는 최대한 ReScript로만 코드를 작성하도록 해야합니다! 하지만 처음 ReScript를 도입하고자 할 때는 이런 방식으로 시작하여 점진적으로 코드를 리팩토링 해나가는 것도 좋은 방법이라고 생각합니다. 

### external 사용하기
external 은 JavaScript 값들을 사용하기 위한 기본적인 ReScript 기능입니다.
- let 바인딩과 비슷하지만, 다음과 같은 특징이 있어요.
    - = 오른편의 값은 값이 아니라 연결하고자 하는 JS 값의 이름입니다. 
    - 바인딩하게될 타입은 **반드시** 적어야 합니다. 
    - 파일이나 모듈의 최상위단 값들만 바인딩 가능합니다. 
- 예시를 통해 알아보겠습니다. 


```JavaScript
@bs.val 
external setTimeOut: (unit => unit, int) => float = "setTimeout"
```
- 위의 코드는 JavaScript 글로벌 함수인 setTimeout을 ReScript에 바인딩한 것입니다. SetTimeout은 함수(unit=>unit)와 interval 값 (int)를 받아 float을 리턴하는 함수기 때문에 (float을 리턴하는지는 처음 알았네요) 위와 같이 type을 직접적으로 명시해줬습니다. 아래처럼 타입을 직접적으로 명시하지 않는 방법도 존재합니다.
```JavaScript
@bs.val
external document: 'a = "document"
document["addEventListener"]("mouseup", _event => {
    Js.log("clicked!")
})
let loc = document["location"]
document["location"]["href"] = "rescript-rox"
```
- 하지만 위와같이 사용했을 경우, type 추측(inference)이 일어나기 때문에 사실상 문제없이 잘 돌아갑니다. ReScript를 처음 시작할때는 좋은 접근방법이 될 수 있지만, parameter와 return 값을 알 수 없기 때문에 결국 JavaScript를 쓰는것과 차이가 없게 됩니다. 

- @bs.val 과 @bs.scope : 글로벌 JS 값들과 바인딩합니다.
- @bs.module : JS로부터 imported/exported 된 값들을 바인딩합니다.
- @bs.send : JS method 들에 바인딩합니다. 
아래에서 하나씩 설명하겠습니다. 

### JS 객체에 바인딩하기 (@bs.module)
- JavaScript의 객체는 다음과 같은 사용성이 있는데요,

1. 다른 언어들의 'record'나 'struct' 로 사용 (각각 ReScript와 C에서)
2. hashMap으로 사용
3. class로서 사용
4. import/export되는 모듈로써 사용

- 이 중 1~3에 해당하는 부분이 @bs.module로 다뤄지게 됩니다. 
- 아래의 예시는 ReScript record로 바인딩된 JavaScript object의 예시입니다. 

```JavaScript
type person = {
    name: string,
    friends: array<string>
    age: int,
}

@bs.module("School")
external harry: person = "harry"
let harryName = harry.name
```

- 비슷하게 ReScript object로도 바인딩이 가능합니다. ReScript의 record와 object의 차이점은,
1. Record 는 immutable
2. Record 는 pattern-matching이 가능
3. 선언시 key 부분에 ""가 없으면 Record, 있으면 object
4. . 으로 key 값에 접근 가능하면 Record, ["key"] 로 접근해야 하면 object
정도가 있습니다. 

```JavaScript
type person = {
    "name": string,
    "friends": array<string>,
    "age":int,
}
@bs.module("School")
external harry: person = "harry"
let harryName = harry["name"]
```
- 참고로 @bs.scope 을 사용하면 객체 안의 nested된 다른 객체를 사용할 수 있습니다.
- 다음으로는 @bs get & set 을 사용해봅시다.

```JavaScript
type textarea = string
type intro = {
    description: textarea,
    name: string,
}
@bs.set 
external setName: (intro, string) => unit = "name"
@bs.get
external getName: intro => string = "intro"

let harry = {
    description: "Let me introduce myself",
    name: "ice cream"
}
setName(harry, "harry");
Js.log(harry); //{ description: 'Let me introduce myself', name: 'harry' }
```

- 위의 코드는 JS로는 아래와 같이 트랜스파일됩니다.
```JavaScript
var harry = {
  description: "Let me introduce myself",
  name: "ice cream"
};

harry.name = "harry";
console.log(harry);
exports.harry = harry;
```
- @bs.set을 사용할 경우 앞의 인자의 key로 접근이 되는 것을 볼 수 있습니다. 
<br/>

- Class인 JS object에 바인딩 하는 경우를 알아봅시다. 
```JavaScript
type t
@bs.new
external createDate: unit => t = "Date"
let date = createDate();
Js.log(date); //current date 가 출력됩니다.
```
- 위의 경우는, 이미 존재하는 JS 클래스인 "date"에 ReScript에서 임의로 정의한 createDate를 바인딩한 경우입니다. 생성자 역할을 해야 하기에 @bs.new가 사용되었고요. createDate는 얼마든지 원하는 다른 이름으로 부를 수 있습니다. 트랜스파일된 결과는 같습니다.

```JavaScript
type t
@bs.new
external anyNameIWant: unit => t = "Date" // createDate => anyNameIWant
let date = anyNameIWant();
Js.log(date);
```

<br/>
- class를 import하여 바로 생성자를 바인딩하고 싶다면 @bs.new 와 @bs.module을 함께 사용할 수 있습니다.

```JavaScript
//book.js
module.exports = class Book {
    constructor(input){
        this.name = input
    }
}

type b
@bs.new @bs.module
external book: string => t = "./book"  //open 을 하지 않았기에 여기선 ./로 경로표시를 해줍니다.
let mybook = book("The Lord of the ReScript")
Js.log(mybook) //Book { name: 'The Lord of the ReScript' }
```

- 위의 코드는 아래처럼 트랜스파일 됩니다.
```JavaScript
function book(prim) {
  return new Book(prim);
}
var mybook = new Book("The Lord of the ReScript");
console.log(mybook);
```

- 마지막으로 함수 바인딩에 대해서 살펴보겠습니다. 우선 보통 모듈에서 가져오는 함수들은 @bs.module로 바인딩이 가능합니다.
```javascript
@bs.module("path") external dirname: string => string = "dirname"
let root = dirname("/User/github") // returns "User"
```

- JS 객체에 붙어있는 함수들은 문법이 조금 다른데요, @bs.send를 사용합니다. 
여기서 유의할 점은, 객체의 함수를 부를 때는 객체 자체가 첫번째 인자로 온다는 것입니다.

```javascript
type document // abstract type for a document object
@bs.send external getElementById: (document, string) => Dom.element = "getElementById"
@bs.val external doc: document = "document"

let el = getElementById(doc, "myId") //doc 이 첫번째 인자로 옵니다.
```

```javascript
var el = document.getElementById("myId");
```

- 함수 관련해서는 여러 특별한 경우들이 있는데요, 자세한 내용은 내일 이어서 다루도록 하겠습니다.