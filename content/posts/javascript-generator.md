+++
title = "자바스크립트 이터레이터 그리고 제너레이터"
date = "2021-01-08T01:11:32+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["Javascript", "Symbol", "Iterator", "Iterable", "Generator", "반복자", "순회 가능한 객체"]
keywords = ["Javascript", "Symbol", "Iterator", "Iterable", "Generator", "반복자", "순회 가능한 객체"]
description = "자바스크립트 내 Iterable 프로토콜과 Iterator 프로토콜 관련한 설명, 그리고 Iterable 객체를 쉽게 만들 수 있는 Generator에 대해 다룹니다."
showFullContent = false
+++

### Iteration 프로토콜

Iterable 프로토콜과 Iterator 프로토콜로 나눌 수 있습니다. ES2015에서 추가된 이 프로토콜을 구현하기 위해선 대상이 되는 객체가 반복 가능한 객체여야합니다.

반복 가능한 객체는 Symbol.iterator 속성에서 **특별한** 메소드를 사용할 수 있습니다. 이 메소드를 호출하여 Iterator 객체를 반환할 수 있습니다. 이 객체는 Iterable 프로토콜을 만족한다고 부를 수도 있습니다.

참고로 Symbol은 자바스크립트 내 유일한 값을 띄는 객체입니다.

Iterator 객체는 next 메소드를 사용할 수 있는데 이 메소드는 객체의 현재 위치의 값과 순회의 종료 여부를 반환해줍니다.

```JS
let items = [1,2,3,4,5];
let iterItems = items[Symbol.iterator]();
while(true){
    let {value, done} = iterItems.next();
    console.log(value)
    if(done){
        break;
    }
}
```

```powershell
1, 2, 3, 4, 5, undefined
```

value는 차례대로 순회된 값, done은 객체 내 순회가 끝났는지 나타내는 boolean 값을 나타내줍니다..

예를 들어, 위에 있는 while 문도 조건문이 true로 무한히 반복되어야 하지만, done이 true가 되면 break가 실행되어 반복문이 종료됩니다.

for of 반복문으로 순회를 좀 더 쉽게 작성할 수 있다.

```JS
for(let itr of iterItems){
    console.log(itr)
}
```

```powershell
1, 2, 3, 4, 5
```

1, 2, 3, 4, 5가 순서대로 출력될 것입니다.

next 메소드를 호출하거나 for of 반복문을 사용하면 iterator 객체 내 요소가 순서대로 빠져나가게 됩니다.

```JS
const arr = [1,2,3,4,5,6];
const itr = arr[Symbol.iterator]()
for(let i of itr){
    console.log(i)
    console.log(...itr)
}
```

```powershell
1
2, 3, 4, 5, 6
```

console.log을 통해 첫번째 요소 i가 빠져나왔고, 그 다음 ...itr을 통해 나머지 요소가 전부 순회되어 빠져나왔습니다. 더 이상 순회할 것이 없는 반복문은 종료됩니다.

async function, 비동기 함수에서 for of loop을 실행하는 것은 전체 코드 실행에 있어 퍼포먼스 하락의 원인이 됩니다.
async 함수에서는 Promise all의 사용이 권장됩니다.

```JS
const items = [1,2,3,4,5]
const iterItems = items[Symbol.iterator]();
async function iter(){
    await Promise.all(iterItems)
    .then(value => console.log(value))
}
iter()
```

```powershell
[1, 2, 3, 4, 5]
```

순서대로 1, 2, 3, 4, 5가 출력되는 for loop과 next 메소드와는 다르게 [1, 2, 3, 4, 5], 즉 배열의 형태로 출력됩니다.

for of 반복문은 iterator 자료구조만 아니라 문자열, 배열과 같은 반복 가능한 모든 객체를 순회할 수 있습니다. 반복 가능한 모든 객체를 iterable 객체라 부른다.

해당 객체 역시 순회 가능한 객체이므로 Symbol.iterator 값의 메소드를 사용할 수 있고 iterable 프로토콜 또한 만족합니다.

iterable 객체는 배열 및 문자열, Map, Set 자료구조 등등이 포함됩니다. Symbol.iterator 메소드를 호출함으로써 이터레이터로 만들 수 있다.

또한 iterable 객체라면 for of 반복문, yield, 구조 분해 할당 등을 수행할 수 있고 for of 반복문으로도 출력할 수 있다.

```JS
let obj = {
    name: "Alice Kim",
    age : 24,
    career : "Postman", // Postman은 프로그램 뿐만 아니라 집배원이라는 뜻도 있습니다 :)
    address : "Seoul",
    money: 3000
}
let iterObj = obj[Symbol.iterator]();
for(let o of iterObj){
    console.log(o)
}
```

일반 객체(Object)는 순회가 불가능하여. 위와 같은 코드를 실행시키면 obj[ Symbol.iterator ] is not a function 오류가 발생합니다.

Object.keys(obj) 또는 Object.values(obj)를 호출하여 iterable 객체로 만들 수 있다.

```JS
let str = "HELLO*WORLD";
let iterStr = str[Symbol.iterator]();
for(let s of iterStr){
    console.log(s)
}
```

```powershell
H, E, L, L, O, *, W, O, R, L, D
```

### 제너레이터 Generator 함수

iterable 객체를 값으로 반환합니다. iterable 객체를 직접 작성할 수 있게 도와줍니다. 제너레이터 내에서는 순회 과정을 잠시 멈추거나 다시 시작하도록 코드를 작성할 수 있습니다.

반환된 iterable한 값은 제너레이터라고 부를 수 있습니다.

현재 객체의 상태를 관리할 수 있으며, 상태 관리 라이브러리 Redux-Saga에서도 Generator 함수를 사용합니다.

function\*() 구문으로 generator 함수를 생성할 수 있습니다.

generator 함수에서 반환된 제너레이터 객체는 iterable 이므로, Symbol.iterator 속성 또한 보유하고 있습니다.

```JS
function* myGenerator(){
    yield 5;
    yield 4;
    yield 3;
    yield 2;
    yield 1;
}
console.log(myGenerator)
```

```powershell
[GeneratorFunction: myGenerator]
```

yield 키워드는 값을 반환하는 역할을 하지만 여러 개의 yield 문이 있을 경우 한번에 출력하지 않고 순회하는 순서대로 출력해줍니다.

```JS
// 앞서 선언한 제너레이터를 사용합니다.
let gen = myGenerator()
for(let g of gen){
    console.log(g)
}

sampleGen = myGenerator()
console.log(sampleGen.next())
console.log(sampleGen.next())
console.log(sampleGen.next())
console.log(...sampleGen)
```

```powershell
1, 2, 3, 4, 5

value: 5, done: false
value: 4, done: false
value: 3, done: false
1, 2
```

yield 구문을 사용하여 타 제너레이터 함수에서 반환된 값을 다른 제너레이터 함수에 전달할 수 있습니다.

```JS
function* innerGen(){
    yield 1
    yield 2
    yield 3
}

function* outerGen(){
    yield* innerGen()
    yield* innerGen()
}

let instanceGen = outerGen()
for(let element of instanceGen){
    console.log(element)
}
```

```powershell
1, 2, 3, 1, 2, 3
```

generator 함수는 일반 함수를 포함합니다. 일반 함수에서 작성할 수 있는 코드는 generator 함수에서도 작성할 수 있습니다.

```JS
function* add(start = 1, param = 2, limit = 300){
    while(start < limit){
        yield start;
        start += param
        param += 2
    }
}
let sample = add()
console.log(...sample)
```

```powershell
// 1, 3, 7, 13, 21, 31, 43, 57, 73, 91, 111, 133, 157, 183, 211, 241, 273
```

점진적으로 요소가 증가하는 배열을 출력하는 예제입니다.

```JS
function* reducer(elements){
    let len = elements.length;
    let index = 1;
    while(index <= len){
        yield elements.slice(0, index)
        index += 1
    }
}
let elements = [1,2,3,4,5,6,7,8,9]
let reduced = reducer(elements)
console.log(...reduced)

```

```powershell
[ 1 ]
[ 1, 2 ]
[ 1, 2, 3 ]
[ 1, 2, 3, 4 ]
[ 1, 2, 3, 4, 5 ]
[ 1, 2, 3, 4, 5, 6 ]
[ 1, 2, 3, 4, 5, 6, 7 ]
[ 1, 2, 3, 4, 5, 6, 7, 8 ]
[ 1, 2, 3, 4, 5, 6, 7, 8, 9 ]
```

특이한 점은 제너레이터 함수로 반환한 값은 iterable한 객체인데, iterable 객체에서 Symbol.iterator 메소드를 사용했을 때 반환되는 iterator는 원본 iterable 객체와 똑같이 취급됩니다.

```JS
function* gen(){
    yield 1;
    yield 2
    yield 3
    yield 4
    yield 5
}

const foo = gen()
const bar = g[Symbol.iterator]()
console.log(foo === bar)
```

```powershell
true
```

그래서 어떤 사람들은 제너레이터의 iterable 객체는 다음의 로직으로 생성될 것이라고 추측하기도 합니다.

```JS
foo[Symbol.iterator] = () => this
```

마지막 부분은 공부를 좀 더 해서 보강하겠습니다.
