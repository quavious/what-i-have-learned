+++
title = "LeetCode 349. Intersection of Two Arrays"
date = "2021-02-26T01:04:05+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["Javascript", "Typescript", "Leetcode", "Problem", "Solving", "Algorithm", "Object"]
keywords = ["Javascript", "Typescript", "Leetcode", "Problem", "Solving", "Algorithm", "Object"]
description = ""
showFullContent = false
+++

# Intersection of Two Arrays

### 문제 설명

두 배열이 주어질 때, 배열이 공통적으로 가지고 있는 요소만 따로 배열의 형태로 출력한다.

```
Input: nums1 = [1,2,2,1], nums2 = [2,2]
Output: [2]
```

```
Input: nums1 = [4,9,5], nums2 = [9,4,9,8,4]
Output: [9,4]
```

```Typescript
interface obj {
    [key: string]: number;
}

function intersection(nums1: number[], nums2: number[]): number[] {
    const arr1 = Array.from(new Set(nums1))
    const arr2 = Array.from(new Set(nums2))
    arr1.push(...arr2);

    const obj: obj = {};
    arr1.forEach(el => {
        if(!obj[el]) obj[el] = 1;
        else obj[el]++;
    })
    return Object.keys(obj).filter(e => obj[e] > 1).map(el => parseInt(el, 10))
};
```

타입스크립트로 객체를 사용할 때 괄호 표기법으로 속성에 접근하려고 하면 타입 오류가 발생한다.

```Typescript
const obj = {
    name: "Kim",
    age: 24,
    job: "FireFighter",
    address: "Seoul",
}
Object.keys(obj).forEach(e => obj[e]) // 에러 발생
```

객체를 접근할 때 동적인 변수를 사용해서 접근하면 any 타입으로 접근한다는 오류가 발생한다.

이 오류를 방지하기 위해서 타입스크립트의 인덱스 시그니쳐를 사용할 수 있다.

```Typescript
interface obj {
    [key: string]: number;
}
```

대괄호 내 key는 키 값의 자료형을 지정하기 위해 사용했다.

obj 인터페이스로 타입 지정된 객체에 문자열 속성을 정의하면 숫자 자료형을 반환한다고 명시해 줄 수 있다.

자바스크립트의 Set 자료형은 중복된 값이 없는 순회 가능한 자료구조이다.

new Set 생성자에 배열을 넘겨주고 다시 배열로 바꿔줌으로써 중복된 값을 제거할 수 있다.

그 다음 두 배열을 하나로 합쳐준다. 공통된 값은 합쳐진 배열에는 2개가 존재할 것이다.

배열을 순회하면서 배열 요소에 대해 객체 내 키 값이 정의되어있지 않다면 1로 초기화해주고, 존재한다면 1씩 증가한다.

예시

```Typescript
obj = {
    "4": 2,
    "9": 2,
    "1": 1,
    "5": 1,
}
```

그 다음 객체의 키만 추출한 배열에서 값이 2 이상인 키만 따로 배열에 보관한다.
