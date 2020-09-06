


### 자료구조를 위한 Type 정의
- Tree, Linked list 등을 직접 구현할 때는 객체 안 객체 형식을 주로 사용하게 됩니다. 한 객체에서 그 다음 객체로 가야 할 경우 node.NEXT_NODE의 형식으로 사용되며 대게 Parent, Leaf, Child, Left, Right 등의 이름을 갖습니다. ReasonML에서는 어떻게 할 수 있는지 알아보겠습니다.

- 간단한 그래프 구조를 만들어 보고, 그것들을 DFS로 순회해보겠습니다. 
```javascript

type node = {
    name: string,
    number: int,
    next: Set.String.t,
}

```


```javascript
    module rec A: {
    type t = {
        name: string,
        children: ASet.t,
    };
    let compare: (t, t) => int;
    } = {
    type t = {
        name: string,
        children: ASet.t,
    };
    let compare = (t1, t2) => compare(t1.name, t2.name);
    }
    and ASet: Set.S with type elt = A.t = Set.Make(A);
```

##### Applicatives
- Applicative 란, **wrapped function과 wrapped value**의 


##### Monads
- Monad 는 Wrapped Mappable을 반환하는 Wrapped function 입니다.


Functor, Applicative, Monad 는 **추상적 개념입니다.** 
**“Curse of Monads: When you finally understand it, you’ll lose the ability to explain it to others,”** 라는 무시무시한 말도 있네요. 지금처럼 적당~히 알아듣는척 하고 있다가 나중에 아! 하는 순간이 오길 기대해봅니다. 