+++
title = "Javascript Design Pattern - 자바스크립트 디자인 패턴"
date = "2021-01-05T00:41:10+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["Javascript", "DesignPattern", "Theory", "Prototype", "Class"]
keywords = ["Javascript", "DesignPattern", "Theory", "Prototype", "Class"]
description = "자바스크립트 디자인 패턴에 대해서 배운 내용 정리"
showFullContent = false
+++

# Javascript Design Pattern

Java, C-Sharp과 같은 많은 객체지향 프로그래밍 언어는 클래스(Class) 개념을 사용하고 있다.

자바스크립트는 프로토타입 패턴을 따른다. 객체가 생성자의 역할을 하는데, 프로토타입 패턴을 사용하여 기존의 객체를 복사하여 새로운 객체를 생성할 수 있다.

ES6부터 자바스크립트에서도 유사 클래스 문법을 지원하기 시작했다.

```Javascript
// func 변수는 이제 함수가 아닌 생성자이다.
const func = function(name){
    this.name = name
}
func.prototype = {
    introduce: function(){
        console.log(`I am ${this.name}`)
    }
}

const fn = new func("Kim")
fn.introduce() // => I am Kim
```

유튜브 동영상 강좌에서 보고 배운 것을 복습 차원에서 정리했습니다.

각 패턴에 대한 설명은 추후에 보강하겠습니다.

### 팩토리(Factory) 패턴

구체적인 객체의 이름을 감추기 위해 사용한다. 사용자로 하여금 어떤 하위 계층의 객체를 생성할지 결정하게 만들 수 있다.

```JS
// Developer 생성자
function Developer(name){
    this.name = name;
    this.type = "Developer";
}

// Tester 생성자
function Tester(name) {
    this.name = name;
    this.type = "Tester"
}

// 직원 객체를 생성하는 Factory 생성자
// 매개변수 name은 this.name을 결정하고, type은 어떤 객체가 생성될지 결정한다.
function EmployeeFactory() {
    this.create = (name, type) => {
        switch(type){
            case 1:
                return new Developer(name);
            case 2:
                return new Tester(name);
            default:
                break;
        }
    }
}

function say() {
    console.log("Hi, I am " + this.name + " I am a " + this.type)
}

const employeeFactory = new EmployeeFactory()
const employees = [];
employees.push(employeeFactory.create("Lee", 1))
employees.push(employeeFactory.create("John", 2))
employees.forEach(employee => {
    // say 함수(메소드)의 실행되는 객체를 바인딩하기 위해 bind 메소드 사용
    // say 함수 내 this는 employee 객체를 의미하게 된다.
    say.bind(employee)()
})
// => Hi, I am Lee I am a Developer.
// => Hi, I am John I am a Tester.
```

### 싱글톤(Singleton) 패턴

생성자가 무수히 많이 실행되어도 결국 반환되는 객체는 모두 같다.

```JS
// 이 생성자는 Process 객체를 무한히 만들 수 있다.
function Process(state) {
    this.state = state;
}

// 즉시실행함수표현식(IIFE)을 이용하므로, 괄호 내 함수는 한 번만 실행된다.
// 리턴되는 getProcessManager 메소드는 같은 객체를 리턴한다.
const Singleton = (function(){
    function ProcessManager() {
        this.numProcesses = 0;
    }
    let pManager;

    function setProcessManager(){
        pManager = new ProcessManager()
        return pManager
    }
    return {
        // pManager 객체가 null이라면 setProcessManager 메소드를 실행해서 객체를 생성한다.
        // 그렇지 않다면 이전에 생성된 객체를 리턴한다.
        getProcessManager: () => {
            if(!pManager) {
                pManager = setProcessManager()
            }
            return pManager
        }
    }
})()

const processManager = Singleton.getProcessManager()
const processManager2 = Singleton.getProcessManager()

console.log(processManager === processManager2) // => true
```

### 반복자(Iterator) 패턴

순서가 있는 객체, 예를 들면 배열 등을 편하게 탐색할 수 있게 해주는 패턴이다.

```JS
// 다양한 자료형의 값이 저장된 배열
const items = [1, "String", false, 1.24, {name: "lee", age: 24}];
function Iterator(items) {
    this.items = items;
    this.index = 0
}

Iterator.prototype = {
    // index 값이 배열의 길이보다 클 경우 더 이상 순회할 수 있는 값이 없다.
    hasNext: function(){
        return this.index < this.items.length;
    },
    // next 메소드를 한번 실행할 때마다 해당 index의 값을 반환하고, index의 값은 1 증가한다.
    next: function(){
        return this.items[this.index++]
    }
}

const iter = new Iterator(items)

// 조건문이 false가 될 때까지 계속적으로 순회한다.
while(iter.hasNext()){
    console.log(iter.next())
}
console.log(iter.hasNext()) // => false
```

### 전략(Strategy) 패턴

상황에 따라서 사용되는 객체를 편하게 교체할 수 있다.

```JS
function Fedex() {
    this.calculate = package => {
        // fedex calculations
        return 2.45
    }
}

function UPS(){
    this.calculate = package => {
        // ups calculations
        return 1.56
    }
}

function USPS(){
    this.calculate = package => {
        // usps calculations
        return 4.5
    }
}

const fedex = new Fedex()
const ups = new UPS()
const usps = new USPS()

const package = {
    from : "Lee",
    to: "John",
    weight: 4.6
}

function Shipping() {
    this.company;
    this.setStrategy = (company) => {
        this.company = company
    }
    this.calculate = (package) => {
        return this.company.calculate(package)
    }
}

// 같은 매개변수를 주었지만 콘솔에 출력되는 객체에 따라 달라진다..
const shipping = new Shipping()

// fedex 객체가 사용된다.
shipping.setStrategy(fedex)
console.log(shipping.calculate(package)) // => 2.45

// 전략이 바뀌어 ups 객체가 사용된다.
shipping.setStrategy(ups) // Change Strategy
console.log(shipping.calculate(package)) // => 1.56

// 전략이 또 바뀌어 usps 객체가 사용된다.
shipping.setStrategy(usps) // Change Strategy
console.log(shipping.calculate(package)) // => 4.5
```

### 관찰자(Observer) 패턴

관찰자 패턴은 객체의 상태 변화를 관찰하는 관찰자들의 목록을 객체(배열)에 등록한다.

해당 객체는 선언된 메소드를 통해 객체의 상태 변화가 있을 때마다 각각의 관찰자에게 통지하도록 지시한다.

```JS
// 해당 객체
function Subject(){
    this.observers = [] // Array of Observer functions
}

Subject.prototype = {
    // 관찰자를 등록한다.
    subscribe: function(fn){
        this.observers.push(fn)
    },
    // 관찰자를 제외한다. filter 고차함수 사용
    unsubscribe: function(fnToRemove){
        this.observers = this.observers.filter(fn => fn !== fnToRemove)
    },
    // 관찰자들에게 무언가를 통지하도록 지시한다.
    fire: function(){
        this.observers.forEach(fn => {
            fn()
        })
    }
}

const subject = new Subject()

function observerOne() {
    console.log("Observer 1 is firing!")
}

function observerTwo() {
    console.log("Observer 2 is firing!")
}

function observerThree() {
    console.log("Observer 3 is firing!")
}

subject.subscribe(observerOne)
subject.subscribe(observerTwo)

subject.fire()
// => Observer 1 is firing!
// => Observer 2 is firing!

subject.unsubscribe(observerTwo)
subject.fire()
// => Observer 1 is firing!
```

### 중재자(Mediator) 패턴

각각의 객체의 상황 및 동작을 하나의 중재자가 관리한다. 채팅 애플리케이션에 주로 사용된다.

```JS
// 멤버 객체이다. chatroom 프로퍼티는 지금은 null 값이지만 추후에 채팅방 객체로 변경된다.
function Member(name){
    this.name = name;
    this.chatroom = null; // After becomes Chatroom
}

Member.prototype = {
    send: function(message, toMember){
        // 채팅방 객체의 send 메소드가 사용된다
        // 채팅방 객체의 send 메소드를 사용하여 자기 자신의 객체(this)에서 지정된 객체로 메시지를 보낸다.
        this.chatroom.send(message, this, toMember);
    },
    receive: function(message, fromMember){
        // 수신자는 송신자에서 자기 자신(this)에게 보낸 메시지를 읽을 수 있다.
        console.log(`${fromMember.name} to ${this.name}: ${message}`)
    }
}

function Chatroom(){
    this.members = {};
}

Chatroom.prototype = {
    addMember: function(member){
        // 채팅방 객체의 접속자들 프로퍼티에서는 addMember 메소드를 통해
        // 사용자 이름 : 사용자의 형태로 등록된다.
        // 이 때 사용자 객체의 채팅방 프로퍼티는 채팅방 자기 자신(this)로 변경된다.
        this.members[member.name] = member;
        member.chatroom = this;
    },
    send: function(message, fromMember, toMember) {
        // 채팅방 객체의 send 메소드가 수신자 객체의 receive 메소드를 실행시켜 수신자가 메시지를 읽을 수 있게 한다.
        toMember.receive(message, fromMember)
    }
}

const chat = new Chatroom()
const bob = new Member("bob")
chat.addMember(bob)
const john = new Member("john")
chat.addMember(john)
const kim = new Member("kim")
chat.addMember(kim)

bob.send("Hello World", john) // => bob to john : Hello World!
john.send("Welcome JS World!", kim) // john to kim : Welcome JS World!
```

### 프록시(Proxy) 패턴

웹 서비스에 주로 사용되는 패턴이다.

웹 API의 병목현상을 막기 위하여 프록시 객체를 앞에 배치한다.

프록시 객체가 가지고 있는 값이 본 객체와 다르면 값을 업데이트하고, 그렇지 않으면 프록시 객체에서 값을 리턴한다.

```JS
// External API Services
function CryptocurrencyAPI(){
    this.getValue = function(coin){
        console.log("Calling External API...")
        switch(coin){
            case "Bitcoin":
                return "$8400";
            case "Litecoin":
                return "$50";
            case "Etherum":
                return "$175";
            default:
                break
        }
    }
}

function CryptocurrencyProxy(){
    // Proxy 객체에서 API 객체를 사용할 수 있다.
    this.api = new CryptocurrencyAPI()
    // 값을 보관할 cache 객체이다.
    this.cache = {}

    this.getValue = function(coin){
        // cache 내 coin의 이름에 대한 값이 없다면 API로부터 값을 가져온다.
        // 그렇지 않으면 캐시 객체에서 값을 가져다 리턴해준다.
        if(!this.cache[coin]){
            this.cache[coin] = this.api.getValue(coin)
        }
        return this.cache[coin]
    }
}

const proxy = new CryptocurrencyProxy()
console.log(proxy.getValue("Bitcoin")) // => "Calling External API..." & $8400
console.log(proxy.getValue("Litecoin")) // => "Calling External API..." & $50
console.log(proxy.getValue("Etherum")) // => "Calling External API..." & $175

// 캐시에 모든 값이 저장되었다.
console.log(proxy.getValue("Bitcoin")) // => $8400
console.log(proxy.getValue("Litecoin")) // => $50
```

### 래핑(Wrapper) 패턴

먼저 선언된 객체에 더 추가적인 기능을 제공하기 위해 사용한다.

객체를 매개변수로 하여 새로운 객체를 앞서 선언된 객체에 대입한다.

```JS
function Add(x,y){
    return x+y
}

const Logger = function(func){
    return function(x, y){
        console.log(`The arguments are ${x} and ${y}.`)
        func(x, y)
        return
    }
}

// Add 함수는 주어진 변수의 합 값 반환에서 변수 출력까지 수행할 수 있다.
Add = Logger(Add) // Overwrite
console.log(Add(2, 3))
```

### 프로토타입 체이닝

프로토타입 체인 패턴에서는 앞서 선언된 객체를 참조하여 객체의 속성에 접근할 수 있다. 그 객체 자체 속성과 객체의 프로토타입 객체와 속성, 그 프로토타입의 프로토타입 객체와 속성을 계속 탐색한다.

프로토타입 체인의 마지막에 이를 때까지 속성을 탐색할 수 있다.

상속된 함수가 실행 될 때, this는 상속된 오브젝트를 나타낸다.

```JS
// Prototype Chaining

// 문자열 - 정수 - 문자열 - 문자열 - 배열 - 문자열 - 숫자
"String".indexOf("S").toString().trim().split("").join(':').length // And more

function MoneyDispenser(amount){
    this.amount = amount;
}

MoneyDispenser.prototype = {
    compute: function(billDenomination) {
        // 매개변수로 전달된 billDenomination으로 this.amount을 나눈다.

        const numberOfBills = Math.floor(this.amount / billDenomination)
        // $32 and $10 denomination = 3
        this.amount -= numberOfBills * billDenomination
        console.log(`Dispensing ${numberOfBills} ${billDenomination} bills, Amount left : ${this.amount}`)
        // Can Call the method compute again

        // compute 메소드는 자기 자신을 반환한다.
        // this.amount 값은 변형되었다.
        return this
    }
}

const dispenser = new MoneyDispenser(328)
dispenser.compute(100).compute(50).compute(20).compute(10).compute(5).compute(1)
// => Dispensing 3 100 bills, Amount left : 28
// => Dispensing 0 50 bills, Amount left : 28
// => Dispensing 1 20 bills, Amount left : 8
// => Dispensing 0 10 bills, Amount left : 8
// => Dispensing 1 5 bills, Amount left : 3
// => Dispensing 3 1 bills, Amount left : 0
```

### 방문자 패턴

객체 내 값에 접근하기 위해 방문자 함수를 사용한다. 객체 구조와 메소드를 분리하는 패턴이다.

객체에 직접 접근하는 것을 막을 수 있고 객체 내 연산을 추가하는 것도 가능하다.

```JS
function Employee(name, salary){
    this.name = name;
    this.salary = salary;
}

Employee.prototype = {
    getSalary: function(){
        return this.salary
    },
    setSalary: function($){
        this.salary = $;
    },
    accept: function(visitorFunction){
        visitorFunction(this)
    }
}

const lee = new Employee("lee", 1000)
console.log(lee.getSalary()) // 1000

// Employee 객체를 매개변수로 받아서 객체의 메소드를 실행한다.
// 객체 내 프로퍼티의 값을 안전하게 변경할 수 있다.
function ExtraSalary(employee){
    employee.setSalary(employee.getSalary() * 2)
}
lee.accept(ExtraSalary)
console.log(lee.getSalary()) // 2000
```

기술블로그 글이 처음이라 설명이 많이 빈약합니다. 개념적인 내용을 다시 공부하여 추후에 보강하도록 하겠습니다.

고맙습니다.
