+++
title = "자바스크립트 에러 처리"
date = "2021-01-18T23:32:28+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["Javascript", "NodeJS", "Typescript", "TryCatch", "Promise", "Callback", "Throw", "ERROR"]
keywords = ["Javascript", "NodeJS", "Typescript", "TryCatch", "Promise", "Callback", "Throw", "ERROR"]
description = ""
showFullContent = false
+++

# 자바스크립트 에러 처리

### 예외 처리

소스 코드가 동작하는 동안 예상치 못한 문제가 발생할 때 코드가 중단되지 않고 대처할 수 있도록 처리하는 것을 예외 처리라고 한다.

예외는 일반적인 프로그램 실행 흐름에 영향을 준다. 예기치 못한 예외가 발생했을 때 전체적인 코드를 중단시키지 않고 특정해 둔 시점에서부터 실행 흐름을 복구시킬 수 있다.

### 자바스크립트에서의 런타임 에러

1. 문법 에러 : 프로그래밍 언어의 문법에 맞지 않아 발생하는 에러

```JS
console.log(;
```

2. 런타임 에러 : 문법에 어긋나지는 않지만, 런타임에서 오류를 발생하는 에러

```JS
const var1 = 10;
console.log(var2)
console.log(var1)
```

var2 변수는 선언되지 않아서 undefined 오류가 발생한다.

3. 논리적 에러 : 문법에 어긋나지 않고, 런타임 오류도 발생하지 않지만 코드 작성자가 예상한 대로 동작하지 않는 현상

```JS
(async function(){
    const resp = await axios.get("API_URL")
    const data = resp.data;
    console.log(data);
})();
```

async function 내에서 axios을 사용해 API_URL 주소로부터 요청을 받고 응답 객체의 데이터를 콘솔에 출력하도록 했다.

resp 객체는 프로미스 객체인데, 프로미스 객체가 data 변수에 값을 반환해주기 전에 console이 이미 실행되어 콘솔에는 undefined가 뜬다.

### try catch finally

자바스크립트는 런타임 에러가 발생할 때 실행 흐름을 복구할 수 있도록 try catch finally 키워드를 제공한다. 코드 상의 에러가 발생하더라도 실행이 중단되지 않도록 할 수 있다.

```JS
try {
    //...
} catch(err) {
    console.error(err)
    //...
}
```

try catch의 작동 방식

1. 우선 try 블록 내부의 코드를 실행한다.

2. 오류가 발생하는 순간 try 블록 내 나머지 코드를 무시하고 catch 블록 내 코드가 실행되기 시작한다.

try 블록에서 catch 블록으로 실행 흐름을 복구한다.

catch 블록에서는 try 블록에서 발생한 에러의 정보를 담고 있는 객체를 매개변수로 받아 사용할 수 있다.

다음 예제에서는 createError 함수가 선언되지 않아서 오류가 발생한다.

```JS
try {
    console.log("Normal");
    createError();
    console.log("Not Logged");
} catch(err) {
    console.error(err);
    console.log(err.name, err.message);
}
```

브라우저 환경에서는 console.log 메소드와 console.error 메소드 실행 시의 콘솔 출력 결과가 다르다.

자바스크립트에서는 에러를 객체로 취급하며, 에러는 다음과 같은 속성을 가지고 있다.

- message : 에러 메시지
- name : 객체의 프로토타입에 들어있는 에러 이름
- stack : 에러가 발생한 지점
- constructor : 에러 이름과 같은 이름의 생성자

에러 객체를 인수로 받을 수 있는 점 때문에 발생한 에러의 타입에 따라서 코드를 다르게 작성할 수 있다.

```JS
try {
    createError();
} catch(err) {
    switch(err.constructor) {
        case SyntaxError:
            console.log("이 오류는 문법 에러");
            break;
        case ReferenceError:
            console.log("다른 에러");
            break;
    }
}
```

try 블록 내 오류 유무와 관계없이 코드의 작동이 끝난 다음 finally 블록 내에서 코드가 무조건 실행되도록 할 수 있다.

finally 블록은 catch 블록이 정의되어 있지 않아도 작동한다.

```JS
try {
    if(Math.random() < 0.5) {
        createError();
    } else {
        console.log("정상")
    }
} catch (err) {
    console.log(err.message)
} finally {
    console.log("END")
}
```

Math.random 메소드의 결과값에 따라 오류의 출력 여부가 결정된다.

오류 발생함과 관계없이 END 문자열이 출력된다.

### 커스텀 에러

throw 키워드를 사용해서 특정 조건일 때 에러를 발생시킬 수 있다.

throw는 에러 생성자와 함께 사용되어 오류를 발생시킨다.

Error 생성자에 매개변수로 입력된 문자열이 곧 에러 객체의 속성이 된다.

```JS
throw new Error("이건 오류가 아니지만 오류라고 볼 수 있습니다.")
```

### 비동기 코드 내 에러 처리

비동기식으로 작동하는 콜백함수의 내부에서 발생한 에러는 try 블록으로 잡아내기 힘들다.

```JS
try {
    setTimeout(() => {
        throw new Error("커스텀 에러")
    }, 1000)
} catch(err) {
    console.error(err)
}
```

1. try 블록 내 setTimeout이 발생하면 콜백 함수는 자바스크립트 엔진이 브라우저에게 맡겨둔 후 나머지 코드가 계속 실행된다.

2. 콜 스택에 저장된 try 블록이 지워지고 나면 태스크 큐에 적재된 콜백함수가 실행된다.

3. 콜백함수가 실행되어서 throw 구문이 실행될 때에는 이미 try catch 블록이 끝난 뒤이다.

따라서 비동기식에서 try catch를 작성할 때에는 콜백함수 내부에 작성해준다.

```JS
setTimeout(() => {
    try {
        throw new Error("에러")
    } catch(err) {
        console.error(err)
    }
}, 1000)
```

Promise 객체 내부에서 오류가 발생하면 rejected 상태가 된다.

객체의 then 메소드에서 첫 번째 매개변수로 넘겨준, resolve 콜백이 실행되지 않고, 두 번째 매개변수로 넘겨준, reject 콜백이 실행된다. reject 콜백에는 에러 객체가 매개변수로 주어진다.

Promise 객체 내에서는 throw 그리고 rej 함수로도 에러를 발생시킬 수 있다.

```JS
const prom = new Promise((res, rej) => {
    setTimeout(() => {
        const rand = Math.random()
        if (rand < 0.5) {
            res("RESOLVED")
        } else {
            rej("REJECTED")
            // throw new Error도 가능하다.
        }
    }, 1000)
})

prom.then(data => console.log(data), err => console.log(err))
```

rand 변수가 0.5 미만이면 resolve 콜백이 실행되어 RESOLVED 문자열이 출력된다.

반대로 rand 변수가 이상이면 reject 콜백이 실행되어 REJECTED 문자열이 출력된다.

reject 콜백 외에도 catch 메소드로 에러 처리를 수행할 수 있다.

```JS
prom.then(data => {
    console.log("RESOLVED")
}).catch(err => {
    console.log("REJECTED")
})
```

앞 코드와 똑같이 작동한다.

Promise와 다르게 비동기 함수에서는 await을 통해 rejected 상태의 Promise 객체를 try, catch, finally로 처리할 수 있다.

또는 catch 메소드로도 에러를 잡아낼 수 있다. 비동기 함수는 Promise 객체를 반환하기 때문이다.

```JS
async function func() {
    try {
        const resp = await axios.get("NON_URL")
    } catch(err) {
        console.error(err)
    }
}
```

NON_URL 주소로의 http GET 통신이 실패해서 오류가 발생하면 err, 에러 객체가 콘솔에 출력된다.

이 때 비동기 함수에 대해 await 키워드가 작성되어 있어야 한다.

```JS
async function func() {
    try {
        const resp = axios.get("NON_URL")
    } catch(err) {
        console.error(err)
    }
}
```

http GET 통신을 하는 동안 try catch 블록은 이미 실행 완료된 상태이다.

통신 과정에서 오류가 발생해도 에러 처리를 할 수 없다.
