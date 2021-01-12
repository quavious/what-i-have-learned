+++
title = "자바스크립트의 비동기 프로그래밍"
date = "2021-01-12T23:24:08+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["Javascript", "Asynchronous", "setInterval", "setTimeout", "Promise", "Callback", "AsyncAwait"]
keywords = ["Javascript", "Asynchronous", "setInterval", "setTimeout", "Promise", "Callback", "AsyncAwait"]
description = ""
showFullContent = false
+++

# 자바스크립트 비동기 프로그래밍

웹 브라우저에 내장된 자바스크립트에서는 특정 시간이 지난 뒤에 함수를 실행하거나 함수를 주기적으로 실행시키는 함수를 사용할 수 있다.

이 함수들은 타이머 식별자를 반환한다. clearTimeout, clearInterval은 각각 timeout, interval 식별자를 받아서 타이머를 취소하는 함수이다.

```JS
// 자바스크립트에서는 초 단위가 ms이다.
setInterval(() => {
    console.log("저는 2초마다 출력됩니다.")
}, 2000)

setTimeout(() => {
    console.log("저는 2초 뒤에 출력됩니다.")
}, 2000)

const timer = setTimeout(() => {
    console.log("저는 출력이 안됩니다.")
}, 3000)

// 3초가 지나기 전에 해당 타이머 식별자는 취소된다.
clearTimeout(timer)
```

함수 매개변수로 받은 시간 정수를 꼭 지키지는 않는다.

```JS
(function(){
    const start = new Date()
    setTimeout(() => console.log(new Date() - start), 100)
})()
// 콘솔에는 115가 출력된다.
```

지연 시간을 0으로 주게 되어도 코드가 **순서대로** 작동하지 않는다.

```JS
setTimeout(() => console.log(1), 0);
console.log(2);

// 출력 순서는 2 1 이다.
```

# 브라우저의 콜스택, 태스크 큐

스택 형태의 저장소이다. 자바스크립트의 엔진은 함수 호출과 관련된 정보를 콜스택에서 관리한다.

콜스택에 저장되는 항목을 실행 맥락이라고 부르며, 함수 내부에서 사용되는 변수, 스코프 체인, this가 의미하는 객체를 저장한다.

브라우저가 자바스크립트 코드를 실행시킬 때 콜스택을 조작하는 과정

1. 전역 실행 맥락을 호출 스택에 추가한다.
1. 함수가 호출되면 함수의 실행 맥락을 생성해서 호출 스택에 추가한다.
1. 변수를 대입하는 동작이 발생하면 콜스택에 저장되어있는 변수를 변경한다.
1. 함수의 실행이 끝나면 결과값을 반환하고 호출 스택 최상단에서부터 실행 맥락을 제거해나간다.
1. 스크립트의 실행이 끝나면 전역 실행 맥락을 스택에서 제거한다.

웹 브라우저는 콜스택에 실행 맥락이 저장되어있는 동안 브라우저의 작동이 정지된다.
화면이 빠른 속도로 재 렌더링되기 때문에 우리가 평소에는 인지하지 못하지만, 코드가 오랜 시간 지속되면 브라우저의 작동이 멈추게 될 수 있다.

```JS
function suspend(val){
    const point = new Date()
    while((new Date() - point) < val ) {};
}
suspend(5000)
```

suspend 함수를 실행시키면 5초 동안 브라우저의 렌더링이 멈추는 것을 볼 수 있다.

setInterval이나 setTimeout와 같이 함수의 작동을 끝마치는데 오랜 시간이 걸리는 함수도 존재한다.

브라우저의 렌더링이 멈추는 것을 막기 위해 브라우저가 하는 일

1. 오랜 시간이 걸리는 함수는 자바스크립트 엔진에서 직접 처리하지 않고 브라우저에 맡긴다.
1. 함수가 끝나면 결과와 콜백 함수를 태스크 큐에 저장한다.
1. 브라우저는 콜스택이 비워질 때마다 태스크 큐에서 가장 늦게 등록된 작업을 꺼내와서 콜백을 실행시키는 과정을 반복한다.

이를 이벤트 루프라고 부른다.

태스크 큐에는 작업이 저장된 순서로 실행되며, 콜스택이 완전히 비워져 있는 상태여야 한다.

태스크 큐에 작업이 저장되어있어도 콜 스택이 비워져 있으면 브라우저는 웹 페이지를 재 렌더링할 수 있다.

코드 예시

```JS
setTimeout(() => console.log(1), 0);
console.log(2);
```

지연시간이 0으로 등록된 Timeout 함수는 콜백 함수를 태스크 큐에 저장한다.

자바스크립트 코드가 완전히 실행되어 콜스택이 비워지고 나서 태스크 큐에 저장된 함수가 실행된다.

이 때문에 콘솔에는 1이 나중에 출력되는 것이다.

브라우저 내 비동기 프로그래밍은 HTTP 통신과 같은 오래 걸리는 작업을 수행할 때 이루어진다.

예를 들어 HTTP 통신을 실행하는 도중에도 우리는 웹 애플리케이션을 이용할 수 있다.

이러한 비동기 프로그래밍 방식은 웹 애플리케이션의 지연 시간을 줄이고 사용자 경험을 최적화하는데 도움이 된다.

하지만 비동기 처리 코드를 많이 작성할수록 코드의 실행 순서를 파악하기 힘들고, 코드의 가독성이 떨어지는 문제가 있다.

이를 콜백 지옥이라고 부른다.

# 자바스크립트 비동기 프로그래밍 기법

### 콜백

다른 함수의 매개변수로 넘기는 함수를 의미한다. 콜백 함수를 태스크 큐에 저장함으로써 비동기 프로그래밍을 할 수 있다.

대표적으로 JQuery를 활용한 Ajax 통신이 있다.

```JS
function fetchAPI(){
    let data = null;
    $.get("API_주소", function(response){
        data = response;
    })
    return data;
}

console.log(fetchAPI())
```

get 메소드에 두 번째 매개변수로 콜백 함수를 넘겨준 모습이다. 통신이 끝나게 되면 콜백 함수가 실행되어 응답 객체를 data 변수에 전달하도록 했다.

실제로 콘솔에는 null이 출력되는 걸 볼 수 있다.

콜백 함수가 data를 변경하기 전에 이미 data는 반환되었기 때문이다.

콜백 함수를 잘못 사용하게 되면 코드의 가독성이 떨어지는 콜백 지옥이 발생한다.

매 연산마다 3초씩 멈추게 하고 싶은 경우

```JS
function callbackHell(data){
    setTimeout(() => {
        data = data * 2
        console.log(data)
        setTimeout(() => {
            data = data * 2
            console.log(data)
            setTimeout(() => {
                data = data * 2
                console.log(data)
                // ...
            }, 3000)
        }, 3000)
    }, 3000)
}
callbackHell(3)
// 3초 간격으로 6, 12, 24가 출력된다.
```

콜백 지옥을 해결하기 위해 Promise 패턴이 등장했다.

Promise는 언젠가 끝나는 작업의 결과값을 보관하는 객체이다. 객체가 선언되는 시점에서는 어떠한 결과값이 저장되는지 알 수 없다.

then 메소드로 콜백 함수를 등록해서 작업이 끝났을 때의 결과값을 가지고 추가적인 작업을 수행할 수 있다.

비동기 작업을 하는 Promise 객체

```JS
const pm = new Promise((res, rej) => {
    try {
        setTimeout(() => {
            console.log(2000)
            res("Hello Promise!")
        }, 2000)
    } catch(err) {
        console.error(err)
        rej("ERROR 발견")
    }
})
```

Promise 생성자는 콜백 함수를 매개변수로 받는다. 콜백 함수 내 매개변수 두 개가 있는데 각각 비동기 작업에서 성공, 또는 실패했을 시 값을 반환하도록 지정할 수 있다.

콜백 함수가 실행되는 2초 동안 pm은 아무런 값도 없다. 콜백 함수가 끝나게 되면 res, rej 함수를 통해 값이 저장된다.

콜백 함수가 정상적으로 종료되었다면 res, resolve 함수를 통해 pm에는 Hello Promise!가 저장된다.

그러나 실행 도중 오류가 발생하면 rej, reject 함수를 통해 ERROR 발견 메시지가 저장된다.

```JS
const pm = new Promise((res, rej) => {
    try {
        setTimeout(() => {
            console.log(2000)
            res("Hello Promise!")
        }, 2000)
    } catch(err) {
        console.error(err)
        rej("ERROR 발견")
    }
})
pm.then(str => console.log(str)).error((str) => console.error(err))
```

콜백 함수가 무사히 끝나서 res 함수를 통해 값이 저장되면 then 메소드를 통해 값을 가지고 추가적인 작업을 할 수 있다.

rej를 통해 오류를 의미하는 값이 저장되었다면 catch 메소드를 통해 추가적인 작업을 할 수 있다.

Promise 객체를 사용하는 대표적인 라이브러리는 HTTP 통신에 사용되는 axios이다. axios의 get, post 같은 통신을 수행하는 메소드는 Promise 객체를 반환한다.

then, catch를 통해 반환(return)하는 값도 Promise 객체이므로, 코드의 가독성을 떨어뜨리지 않고 Promise 패턴을 통한 비동기 작업을 계속해서 수행할 수 있다.

```JS
axios.get("API_주소")
.then(resp => {
    console.log(resp.data)
    return "성공"
}).then(str => console.log(str))
// 응답 객체
// 성공
```

3초마다 연산이 수행되는 함수를 Promise로 다시 작성한 모습

```JS
// 이 함수는 즉시 실행됩니다.
(function(){
    let data = 3
    new Promise((res, rej) => {
        setTimeout(() => {
            console.log(data * 2)
            res(data * 2)
        }, 3000)
    }).then(data => {
        setTimeout(() => {
            console.log(data * 2)
            return data * 2
        }, 3000)
    }).then(data => {
        setTimeout(() => {
            console.log(data * 2)
            return data * 2
        }, 3000)
    })
})()
```

똑같은 6, 12, 24를 출력하는 건 같지만 코드의 깊이(depth)가 줄어들었다.

ES2017에서 도입된 비동기 함수를 사용하면 Promise 패턴보다 더 가독성이 좋은 비동기 처리 코드를 작성할 수 있다.

함수를 선언할 때 async 키워드를 사용하여 함수를 비동기 함수로 만들 수 있다.

```JS
async function asyncFunc1(){

}

const asyncFunc2 = () => {

}
```

비동기 함수는 항상 Promise 객체를 반환한다. then, catch 메소드 전부 사용가능하다.

```JS
async function asyncFunc(){
    return new Promise((res, rej) => {
        setTimeout(() => res("결과값"), 3000)
    })
}

asyncFunc().then(str => console.log(str)) // 결과값
```

비동기 함수 내에서는 await 키워드를 사용할 수 있다. await 키워드 뒤에 오는 Promise 객체가 결과값을 받아서 변수로 반환할 때까지 이 비동기 함수의 작동은 중단된다.

비동기 함수이므로, 브라우저는 함수가 실행될 동안 다른 작업을 수행할 수 있다. 콜백 패턴을 사용하지 않고 비동기식 코드를 작성할 수 있도록 도와준다.

await 키워드는 for, if과 함께 사용할 수 있는 장점도 존재한다.

```JS
// async 키워드가 있으므로 브라우저는 멈추지 않는다.
async function asyncFunc(){
    // asyncFunc 함수 내 Promise 객체가 값을 받아서 전달할 때까지 asyncFunc는 작동을 멈춘다.
    const resp = await asyncFunc()
    console.log(resp) // 결과값
}

asyncFunc()
```
