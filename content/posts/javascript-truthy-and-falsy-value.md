+++
title = "자바스크립트의 참, 거짓 같은 값"
date = "2021-03-08T19:55:01+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["Javascript", "Truthy", "Falsy", "Boolean", "Object", "Array", "NaN", "null", "undefined", "0"]
keywords = ["Javascript", "Truthy", "Falsy", "Boolean", "Object", "Array", "NaN", "null", "undefined", "0"]
description = ""
showFullContent = false
+++

# 자바스크립트의 참, 거짓 같은 값 - Truthy and False Values

참, 거짓 같은 값은 자바스크립트 내에서 각각 true, false로 평가되는 값이다.

다음과 같은 값들은 조건문에서 true로 간주된다.

- 불리언 true
- 객체, 비어있는 객체 및 배열도 포함된다.
- 숫자, BigInt, 무한대 자료형도 포함된다.
- 길이 1 이상의 문자열
- 날짜

```Javascript
if (true)
if ({})
if ([])
if (42)
if ("0")
if ("false")
if (new Date())
if (-42)
if (12n)
if (3.14)
if (-3.14)
if (Infinity)
if (-Infinity)
```

다음과 같은 값들은 조건문에서 false로 간주된다.

- 불리언 false
- null
- undefined
- NaN - Not a Number, 숫자 자료형이지만 숫자가 아니다.
- 숫자 0, BigInt 0n
- 길이 0의 문자열

```Javascript
if (false)
if (null)
if (undefined)
if (0)
if (-0)
if (0n)
if (NaN)
if ("")
```

## NaN에 대하여

NaN은 값이 숫자 자료형이지만 숫자가 아닐 때 발생한다.

전역 객체의 속성으로 지정되어있어서 브라우저 콘솔 창에 this.NaN을 입력하면 NaN이 출력된다.

NaN을 반환하는 연산은 다음과 같다.

- 숫자를 받아야 하는 함수 매개변수로 다른 자료형을 넘겨줌
- 수학 계산식이 허수가 나올 때
- 피연산자가 NaN
- 정의할 수 없는 계산식
- 문자열을 포함하면서 덧셈이 아닌 계산식

```Javascript
parseInt(""), parseInt(undefined)
Math.sqrt(-1)
0 * Infinity
NaN * 7
"문자열" * 100
```

- 문자열과 숫자를 더하면 문자열이 나온다.

```JS
"100" + 100
// => "100100"
```

- 뺄셈은 NaN이 나올 수 있지만 NaN만 나오는 것은 아니다.

```JS
"1000" - 100
// => 900
"string" - 100
// => NaN
```

## NaN 판별

NaN은 모든 비교 연산자로 비교를 수행했을 때 전부 false가 출력된다.

NaN과 NaN을 비교해도 같지 않다고 출력된다.

```JS
console.log(NaN === NaN)
// => false
```

isNaN 함수와 Number.isNaN 메소드로 값이 NaN인지 아닌지 확인할 수 있다.

Number.isNaN은 매개변수로 받는 값이 무조건 NaN일 때 true를 반환한다.

함수 isNaN은 숫자로 변환했을 때 NaN이 되는 자료형도 포함된다.

기본적으로 NaN과 NaN은 서로 같지 않기 때문에 일부 배열 메소드는 NaN을 찾을 수 없다.

```JS
[NaN, NaN].includes(NaN)
// => true
[NaN, NaN].indexOf(NaN)
// => -1
[NaN, NaN].findIndex(n => Number.isNaN(n))
// => 0
[NaN, NaN].findIndex(n => isNaN(n))
// => 0
```
