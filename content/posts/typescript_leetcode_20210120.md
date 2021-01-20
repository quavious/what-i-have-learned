+++
title = "타입스크립트 알고리즘 문제 해결 20210120"
date = "2021-01-20T23:26:46+09:00"
author = ""
authorTwitter = "Neon.Owl" #do not include @
cover = ""
tags = ["Typescript", "Algorithm", "Problem", "Solving", "Data", "Structure", "Array", "Object", "Recursive", "Sort"]
keywords = ["Typescript", "Algorithm", "Problem", "Solving", "Data", "Structure", "Array", "Object", "Recursive", "Sort"]
description = ""
showFullContent = false
+++

# 타입스크립트 알고리즘 문제 해결

### No. 136 Single Number

nums 배열을 받아서 중복되지 않은 숫자를 출력한다. 배열 내 중복되지 않는 숫자의 개수는 1개이다.

매개변수로 주어지지 않는 배열은 길이가 1 이상이다.

예시

```TS
const nums = [2,2,1]
// 1

const nums = [4,1,2,1,2]
// 4

const nums = [1]
// 1
```

답안 코드

```TS
function singleNumber(nums: number[]): number {
    return nums.reduce((acc=0, val) => acc ^ val)
};

singleNumber([1,1,2,2,3]) // 3
```

코드는 간단하지만 왜 이렇게 작성해야하는지 파악하는데 시간이 많이 걸렸다.

사용된 배열 객체의 메소드 reduce는 배열의 각 요소에 대해 콜백 함수, 리듀서 함수를 실행시켜 마지막에 하나의 결과값을 반환한다.

배열을 순회하면서 각 요소에 대해 콜백 함수가 실행되고 그 때마다 결과값이 누적된다.

위의 코드에서는 acc 변수와 val 변수의 XOR 결과값이 acc에 계속적으로 누적되고 배열의 순회가 끝나면 acc 변수가 반환된다.

XOR 연산자는 두 개의 값이 다르면 1, 같으면 0을 반환한다. 같은 숫자로 XOR 연산을 하면 결과는 0이 출력된다.

배열의 요소가 1,1,2,2,3 일 때, XOR 연산이 수행되는 과정

001(1) ^ 001(1) = 000

000(0) ^ 010(2) = 010

010(2) ^ 010(2) = 000

000(0) ^ 011(3) = 011

XOR은 교환법칙이 성립하므로, 배열 요소의 순서가 달라져도 결과는 같을 것이다.

### No. 896 Monotonic Array

주어진 배열의 순서가 오름차순이나 내림차순이면 true, 그렇지 않으면 false를 반복하는 문제이다.

자바스크립트, 타입스크립트에서는 배열이 참조 객체이므로, 배열을 선언한 변수에는 배열의 포인터 값이 저장된다. 변수만 비교 연산자로 비교하면 결국 배열의 포인터 값을 비교하는 것이다.

배열 객체 메소드 sort를 사용해서 정렬을 수행하면 정렬된 배열을 반환함과 동시에 원본 배열도 수정된다.

```TS
const A = [1,2,2,3]
const B = A.sort((a,b) => b - a)

console.log(A, B)
// A, B 모두 [3,2,2,1]이 출력된다.
```

다만 배열에 타입스크립트의 전개 연산자, Spread Operator를 사용하면 배열의 각 요소를 복사할 수 있다.

```TS
const A = [1,2,2,3]
const B = [...A].sort((a,b) => b - a)

console.log(A) // [1,2,2,3]
console.log(B) // [3,2,2,1]
```

...A를 통해 배열의 요소를 펼친 후, 다시 대괄호로 감싸서 새로운 배열을 생성했다.

답안 코드

```TS
function isMonotonic(A: number[]): boolean {
    const arr = JSON.stringify(A)
    const arr1 = [...A].sort((a,b) => a-b)
    const arr2 = [...A].sort((a,b) => b-a)
    return JSON.stringify(arr1) == arr || JSON.stringify(arr2) == arr
};

isMonotonic([1,3,2]) // false
isMonotonic([1,2,3,4,5]) // true
```

배열의 요소가 똑같아도 배열의 포인터 값이 다르므로 변수만으로는 배열의 요소가 같은지 비교하기 어렵다.

그래서 JSON 객체의 stringify 메소드를 이용해서 문자열의 형태로 직렬화한다.

타입스크립트에서 문자열은 원시 객체이므로, 값이 그대로 변수에 저장된다.

배열의 순서가 일정하게 저장되었다면 배열을 정렬했을 때에도 값이 똑같이 나올 것이다.

배열이 오름차순인지 내림차순인지 알 수 없으므로, 각각의 방식으로 배열을 정렬해 준 다음, 문자열로 직렬화하여 비교를 수행했다.

둘 중 하나라도 일치하면 true, 그렇지 않으면 배열의 순서가 일관적이지 않다는 의미이므로 false를 반환하도록 했다.
