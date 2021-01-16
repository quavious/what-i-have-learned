+++
title = "자바스크립트의 This"
date = "2021-01-16T21:58:49+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["Javascript", "This", "Typescript", "Object", "Context", "Class", "Prototype", "NodeJS"]
keywords = ["Javascript", "This", "Typescript", "Object", "Context", "Class", "Prototype", "NodeJS"]
description = ""
showFullContent = false
+++

### Javascript의 This

this는 현재 실행 문맥을 의미한다고 한다. 타 언어에서는 class에서만 사용가능한 것과 다르게 일반 객체, 함수 등의 여러 곳에서 사용할 수 있다.

함수를 호출하여 실행 문맥이 선언될 때 this 또한 결정된다. 따라서 자바스크립트에서는 메소드 호출 방법에 따라서 this의 값이 달라지기도 한다.

전역 공간에서의 this는 그대로 전역 객체를 의미한다.

브라우저에서 전역 this는 window 객체를 의미한다. NodeJS 환경에서 전역 this는 global을 의미한다. window와 global을 모두 포함하는 globalThis라는 키워드도 존재한다.

```JS
console.log(this===window) //true 브라우저
console.log(this===global) //true NodeJS
console.log(this===globalThis) //true
```

전역 변수 또는 함수를 선언한다는 것은 전역 객체의 속성을 부여한다는 것과 같다.
객체를 선언하면 객체 자신이 상위 객체의 속성으로도 볼 수 있다.
특정한 객체를 실행 문맥의 어휘적 환경, LE로 부른다.

const, let으로 객체를 선언하면 전역 객체의 속성으로 인정되지 않는다. var로 선언함으로써 객체를 전역 객체의 속성으로 지정할 수 있다.

브라우저 내 전역 객체 선언

```JS
var func = function(){
    console.log("I am property of window object.")
}
console.log(func == this.func) // true
window.func() // I am property of window object.
```

NodeJS 내 전역 객체 선언

```JS
var func = function(){
    console.log("I am property of global object.")
}
console.log(func == this.func) // true
global.func() // I am property of global object.
```

자바스크립트에서 함수와 메소드를 구분하는 기준은 객체에 대하여 독립되어있는가의 여부이다.
객체 내에 할당된다고 해서 무조건 메소드가 아니라. 객체의 메소드로서 호출이 되어야 메소드로 간주할 수 있다.

다음과 같은 경우는 실행되는 객체가 전역 객체인 경우이다.

```JS
let func = function (x) {
    console.log(this, x)
}
func(1) // 매개변수와 global 객체가 뜬다.
```

실행되는 객체가 변경되면 this도 변경된다.

```JS
let obj = {
    method: func
}
obj.method(2) // obj 객체가 뜬다.
```

//함수 호출 시 무엇이 함수를 실행하였는지 판단하기가 어렵다. this가 지정되지 않으면 전역 객체를 의미한다.

```JS
const obj1 = {
    outer : function(){
        console.log(this)
        const innerfunc = function() {
            console.log(this)
        }
        innerfunc(); // 실행되는 객체가 지정되지 않았으므로 전역 객체가 뜹니다.
        const obj2 = {
            innerMethod: innerfunc
        };
        obj2.innerMethod();
    }
}
obj1.outer()
// outer 객체
// window 객체
// innerMethod 객체
```

첫번째 this는 obj1, 두번째 this는 Global, 세번째 this는 obj2.innerMethod가 뜬다.
똑같은 function을 함수로서 호출하느냐 메소드로서 호출하느냐에 따라 this에 바인딩되는 객체가 결정된다.

메소드를 호출하는 주체가 없을 때 주변 환경의 this를 상속받을 수 있게 하려면 변수를 활용하거나 화살표 함수를 사용해야 한다.

```JS
var obj3 = {
    outer: function(){
        console.log(this)
        var inner1 = function() {
            console.log(this)
        }
        inner1()

        var self = this
        var inner2 = function() {
            console.log(self)
        }
        inner2()
    }
}
obj3.outer()
```

inner2의 실행 객체가 지정되지는 않았다. 하지만 출력되면 outer 객체가 출력된다.
outer 함수 스코프에서의 this를 self라는 변수에 저장했기 때문이다.

```JS
const obj4 = {
    outer : function(){
        console.log(this)
        const inner = () => {
            console.log(this)
        }
        inner()
    }
};
obj4.outer()
```

inner 함수는 선언 시 화살표 함수로 선언되었다.
화살표 함수는 실행 컨텍스트가 생성될 때 상위 스코프의 this를 그대로 받아온다. 특정 객체를 "."으로 연결시켜주지 않아도 자동으로 상위 스코프에 바인딩된다.

inner 함수는 선언될 때 outer 스코프의 this를 그대로 상속받았다.

콜백 함수

콜백 함수도 함수에 포함되어 기본적으로 전역 객체를 참조하게 된다.

매개변수로 주어진 콜백 함수를 제어하는 함수가 this가 되는 대상을 지정해 줄 경우 콜백 함수는 그 대상에 바인딩된다.

```JS
setTimeout(function(){
    console.log(this)
}, 1000) // 1초 뒤 전역객체 출력

Array.from([1, 2, 3, 4, 5]).forEach(function(x){
    console.log(this, x) // 전역 객체와 배열 요소가 한 개 씩 출력됨
})
```

addEventListener 함수는 콜백 함수를 선언할 때 자신의 this를 상속하도록 설정되어 있다.

이벤트가 발생할 때 선언된 콜백 함수의 실행 객체는 이벤트가 발생한 객체가 된다.

```JS
const root = document.getElementById("root")
root.addEventListener("click", function() {
    console.log(this)
})
```

화살표 함수로 선언하면 선언될 때 전역 객체로 바인딩되므로 이벤트가 발생할 때에도 this는 전역 객체가 된다.

```JS
const root = document.getElementById("root")
root.addEventListener("click", () => {
    console.log(this)
})
```
