+++
title = "타입스크립트로 구현한 링크드리스트(Linked List)"
date = "2021-01-13T20:38:59+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["LinkedList", "Typescript", "DataStructure", "Algorithm"]
keywords = ["LinkedList", "Typescript", "DataStructure", "Algorithm"]
description = ""
showFullContent = false
+++

# 링크드리스트 (Linked List)

메모리 내 데이터가 순차적으로 입력되는 배열과 다르게, 링크드리스트는 각각의 노드에 데이터와 메모리 포인터가 있다.
노드 내 포인터는 다음 또는 이전 위치의 노드와 연결된다.

각각의 노드가 포인터로 연결되어있어서 삽입 및 삭제가 용이하다.

연결 리스트의 종류는 단일 연결 리스트, 이중 연결 리스트 등이 있다.

글에 첨부된 코드는 이중 연결 리스트이다. 노드 내 데이터가 하나 있으며, 이전 노드의 메모리 주소, 그리고 다음 노드의 메모리 주소가 있는 포인터 2개가 존재한다.

### 링크드리스트의 노드

```JS
class LinkedListNode<T> {
    val: T;
    next: LinkedListNode<T> | null;
    prev: LinkedListNode<T> | null;

    constructor(val: T) {
        this.val = val;
        this.next = null;
        this.prev = null;
    }
}

export default LinkedListNode
```

노드를 클래스로 선언해주었다. 노드에는 데이터를 담을 수 있는 변수 val이 있고, 이전 및 다음 위치의 노드의 메모리 주소를 보관하는 변수 prev, next가 있다. 타입(자바)스크립트에서는 객체는 참조 타입으로 취급되어, 변수를 객체로 선언했을 때 변수 자체에는 메모리 주소가 담기게 된다.

객체 변수를 콘솔로 출력했을 때에는 변수가 참조하는 데이터를 콘솔에 출력하는 것이라고 볼 수 있다.

최초 선언했을 때에는 값이 없기 때문에 null으로 선언했다.

```JS
interface List<T> {
    head: LinkedListNode<T>
    tail: LinkedListNode<T>
    size: number
}
```

리스트 인터페이스를 선언했다.

링크드리스트는 맨 앞 노드의 참조하는 head, 맨 뒤 노드를 참조하는 tail, 리스트의 길이를 나타내는 size가 있다.

```JS
class LinkedList<T> implements Iterable<T> {
    private list: List<T> | null;
    constructor(){
        this.list = null;
    }
}
```

링크드리스트가 순회 가능하도록 Iterable 인터페이스를 구현하는 클래스로 선언했다.
리스트 인터페이스를 구현하는 list 변수가 있다. list를 통해 head, tail 변수에 접근할 수 있다.

```JS
size(): number {
    return !this.list ? 0 : this.list.size;
}

isEmpty(): boolean {
    return !this.list
}
```

메소드 size는 클래스 내 리스트가 null이라면 리스트 내 노드가 없다는 의미이므로 0, 그렇지 않다면 리스트의 size 값을 반환한다.

메소드 isEmpty는 리스트의 길이가 null인지, 리스트 내 노드가 있는지 참, 거짓 값으로 반환해준다.

```JS
addFront(val: T): void {
    const newNode = new LinkedListNode(val)
    if(!this.list){
        this.list = {
            head: newNode,
            tail: newNode,
            size: 1
        }
    } else {
        this.list.head.prev = newNode;
        newNode.next = this.list.head;

        this.list.head = newNode
        this.list.size += 1;
    }
}
```

리스트 맨 앞에 노드를 추가하는 메소드이다. 매개변수 val을 받아서 새로운 노드를 생성한다.
리스트가 없으면 리스트의 head, tail 전부 생성된 노드를 참조하도록 한다.

리스트가 있으면 기존 head 노드의 prev가 새 노드를 참조하고, 새 노드의 next는 head를 참조하도록 한다.

그 다음 head가 참조하는 노드를 새 노드로 바꿔주고 size는 1 증가시킨다.

```JS
addBack(val: T): void {
    const newNode = new LinkedListNode(val)
    if(!this.list){
        this.list = {
            head: newNode,
            tail: newNode,
            size: 1
        }
    } else {
        this.list.tail.next = newNode;
        newNode.prev = this.list.tail;

        this.list.tail = newNode;
        this.list.size += 1;
    }
}
```

addBack 메소드는 매개변수로 받은 값으로 새로운 노드를 생성해서 리스트의 tail과 연결시켜준다.

tail 노드의 next가 새 노드를, 새 노드의 prev는 tail 노드를 참조하도록 한다.

addFront와 비슷하게 tail이 가리키는 노드를 새 노드로 바꿔주고 사이즈의 크기는 1 증가시킨다.

```JS
addAt(i: number, val: T): void {
    if(i == 0){
        this.addFront(val)
        return
    }
    if(i == this.size() - 1) {
        this.addBack(val)
    }
    if(i < 0 || i > this.size() || !this.list){
        throw new Error("Out of the bound")
    }
    let cur = this.list.head;

    for(let j = 0; j < i - 1; j++){
        cur = cur.next!
    }

    const newNode = new LinkedListNode(val)
    cur.next!.prev = newNode;
    newNode.next = cur.next;

    newNode.prev = cur;
    cur.next! = newNode;

    this.list.size += 1;
}
```

매개변수로 정해진 위치에 노드를 삽입하도록 할 수 있다.

링크드리스트는 타입스크립트의 iterable한 객체와 같이 인덱스 값이 0부터 시작한다.

리스트의 사이즈가 5이면, 전체 노드의 인덱스는 0, 1, 2, 3, 4
i가 0이면 맨 앞에 노드를 삽입하라는 의미이므로 addFront, i가 링크드리스트 길이 - 1 값과 같다면 addBack을 실행시킨다.

i 값이 음수이거나 마지막 인덱스보다 크거나, 아예 리스트가 없다면 오류가 발생하도록 했다.

먼저 맨 앞 노드를 참조하는 cur 변수를 생성해서 위치 i에 도착할 때까지 cur 변수가 참조하는 값을 그 다음 노드로 바꿔주었다.

**예시**

- 현재 노드 데이터가 A, B, C, D, E 이고, 3번째 순서에 Z를 넣고 싶다.
- 현재 3번째 순서에 위치한 노드는 C이다. C의 인덱스 값은 2이다.
- cur 변수는 for 반복문에서 참조하는 노드를 그 다음 노드로 바꿔주는 과정을 2회 반복한다.
- 3번째 순서에는 Z가 오게 된다.

삽입될 노드 A 뒤에는 cur 변수가 참조하는 노드의 **다음 노드** B, 앞에는 cur 변수가 참조하는 노드 C가 위치한다.

A의 next는 B를 가리키도록 하고 B의 prev는 A를 참조하도록 한다.
비슷하게 C의 next는 A를, A의 prev는 C를 참조하도록 한다.
마지막으로 size 값을 1 증가시켰다.

```JS
peekFront(): T {
    if(!this.list) {
        throw new Error("Empty")
    }
    return this.list.head.val;
}

peekBack(): T {
    if(!this.list) {
        throw new Error("Empty")
    }
    return this.list.tail.val;
}
```

각각 리스트의 맨 앞,뒤 노드의 값을 반환해준다. 리스트 내 노드가 없다면 오류가 발생한다.

```JS
get(i: number): T {
    if(i < 0 || i >= this.size() || !this.list) {
        throw new Error("Out of the bound")
    }

    let j = 0
    let cur = this.list.head
    while(j < i){
        cur = cur.next!;
        j += 1;
    }
    return cur.val;
}
```

인덱스 i에 있는 노드의 데이터를 가져온다. 노드를 참조하는 cur 변수를 생성해서 cur가 참조하는 노드를 다음 노드로 바꿔주는 과정을 i회 반복한다.

```TS
indexOf(val: T): number {
    if(!this.list) return -1;
    const equalsFunc = function(a: T, b: T) {
        return a === b
    }

    let i = 0;
    let cur = this.list.head;

    while(!equalsFunc(cur.val, val)) {
        i++;
        cur = cur.next!;
    }

    return i === this.list.size ? -1 : i;
}
```

리스트 내 매개변수로 주어진 값이 있는지 찾는다.
리스트를 순회하는 과정을 계속 반복함으로써 노드 내 데이터와 주어진 val이 일치하는지 파악한다.
일치하는 데이터를 발견하면 노드의 인덱스 값을 반환하고 그렇지 않으면 -1을 출력한다.

위 메소드에서 사용된 equalsFunc가 다음 메소드에서도 계속 사용된다고 가정한다.

```TS
contains(val: T): boolean {
    const index = this.indexOf(val)
    return index !== -1
}
```

리스트 내 매개변수로 주어진 값이 있는지 찾는다.
indexOf 메소드를 실행한 다음 반환된 인덱스 값에 따라 참 거짓이 정해진다.

```TS
removeFront(): T {
    if(!this.list) throw new Error("Empty")

    const val = this.list.head.val

    if(!this.list.head.next) {
        this.list = null;
    } else {
        this.list.head.next.prev = null;
        this.list.head = this.list.head.next;

        this.list.size -= 1
    }

    return val;
}
```

맨 앞 노드를 제거한다. 제거에 앞서 데이터는 val 변수에 저장해둔다.
맨 앞 노드의 next가 참조하는 노드가 없다면, 즉 노드의 갯수가 1이라면 list는 null이 된다.

그렇지 않다면 next가 참조하는 노드의 prev가 참조하는 노드를 null로 변경하고
리스트의 head가 참조하는 노드는 head의 next가 참조하는 노드로 바꾸어준다.

사이즈 값을 1 감소해준다.

```TS
removeBack(): T {
    if(!this.list) throw new Error("Empty")

    const val = this.list.tail.val

    if(!this.list.tail.prev) {
        this.list = null;
    } else {
        this.list.tail.prev.next = null;
        this.list.tail = this.list.tail.prev;

        this.list.size -= 1
    }

    return val;
}
```

removeFront와 작동 방식이 비슷하다. 리스트의 마지막 노드를 제거한다. 마지막 노드, tail의 prev가 참조하는 노드가 없다면 list가 null이 된다.

그렇지 않다면 tail 이전 노드의 next가 참조하는 노드를 null로 설정한다.
tail은 tail의 이전 노드를 참조하도록 했다.

그 후 size 값을 1 감소시켰다.

```TS
remove(val: T): T {
    const index = this.indexOf(val)

    if (index === -1) throw new Error("No value")
    return this.removeAt(index)
}
```

주어진 매개변수 값의 인덱스 위치를 찾는다. 인덱스 값이 -1이 아니라면 removeAt 메소드를 수행한다.

```TS
removeAt(i: number): T {
    if (i === 0) {
        return this.removeFront()
    }
    if (i === this.list!.size - 1) {
        return this.removeBack()
    }

    if (!this.list || i < 0 || i >= this.list.size) {
        throw new Error("Out of the bound")
    }

    let j = 0;
    let cur = this.list.head;

    while(j < i) {
        cur = cur.next!
        j += 1;
    }

    cur.next!.prev = cur.prev;
    cur.prev!.next = cur.next;

    this.list.size -= 1;

    const val = cur.val;
    return val
}
```

i가 0이거나 마지막 인덱스 값이라면 각각 removeFront, removeBack을 실행한다.
cur이 가리키는 노드를 cur의 다음 노드로 바꿔주는 과정을 i회 수행한다.

cur의 next가 가리키는 노드의 prev 값을 cur의 prev가 가리키는 노드로 바꿔주고,
cur의 prev가 가리키는 노드의 next 값을 cur의 next가 가리키는 노드로 바꿔준다.

```TS
clear(): void {
    this.list = null;
}
```

clear 메소드는 list를 null로 바꿔 리스트 내 존재하는 노드의 값을 지운다.

```Typescript
fromArray(A: T[]): LinkedList<T> {
    A.forEach(el => this.addBack(el))
    return this
}
```

주어진 배열에 따라서 계속적으로 addBack 메소드를 수행하여 리스트의 노드를 추가한다.

```TS
*[Symbol.iterator](): Iterator<T> {
    if(!this.list) return;

    for(let cur = this.list.head; cur != null; cur = cur.next!) {
        yield cur.val;
    }
}
```

타입스크립트에서 Iterable 객체면 객체 내 Symbol.iterator 속성에 존재하는 함수를 실행하여 Iterator 객체를 생성할 수 있어야 한다.

Iterator 객체를 쉽게 만들기 위해 Generator 문법을 사용한다.

리스트를 계속 순회하면서 cur가 참조하는 노드가 cur의 다음 노드로 바뀌는데 값이 바뀔 때마다 yield 키워드를 이용해 데이터를 순차적으로 전달하게 할 수 있다.
