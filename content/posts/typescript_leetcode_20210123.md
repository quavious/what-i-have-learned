+++
title = "타입스크립트 알고리즘 문제 해결 20210123"
date = "2021-01-23T22:43:01+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["Typescript", "Algorithm", "Problem", "Solving", "Data", "Structure", "Array", "Object", "Recursive", "Sort"]
keywords = ["Typescript", "Algorithm", "Problem", "Solving", "Data", "Structure", "Array", "Object", "Recursive", "Sort"]
description = ""
showFullContent = false
+++

# 타입스크립트 알고리즘 문제 해결

### 1290. Convert Binary Number in a Linked List to Integer

단방향 링크드리스트의 맨 앞 노드, head가 주어진다. 각 노드 내 값은 1 또는 0이다. 리스트 내 값들은 이진수 값을 표현하고 있다.

이진수 값을 구해서 10진수 값으로 변경

링크드리스트의 구조는 미리 정의되어있다.

```TS
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     val: number
 *     next: ListNode | null
 *     constructor(val?: number, next?: ListNode | null) {
 *         this.val = (val===undefined ? 0 : val)
 *         this.next = (next===undefined ? null : next)
 *     }
 * }
 */

function getDecimalValue(head: ListNode | null): number {
    if(!head) return 0;
    let cur = head;
    let str = ""
    while(!(cur == undefined || cur == null)) {
        str += cur.val.toString()
        cur = cur.next;
    }
    return parseInt(str, 2)
};
```

head가 null이면 값이 없다는 뜻이므로 0을 출력한다.

리스트를 순회할 커서를 하나 생성한 뒤 head로 초기화해준다. 실제로 head의 메모리 주소를 가지고 있어서 cur를 통해 리스트를 순회하는 것이 가능하다.

각 자리의 수를 쉽게 저장하기 위해 문자열을 생성한다. 반복문이 끝나면 parseInt를 통해 문자열을 정수로 변경할 수 있다.

cur을 계속 순회하여 cur이 참조하는 노드 내 값을 문자열에 toString으로 추가하고, cur.next로 cur이 다음 노드를 참조할 수 있게 메모리 주소를 변경해준다.

순회가 끝나서 cur가 undefined 또는 null이면 반복문이 끝난다.

계속 누적된 문자열을 정수로 변경한 뒤 반환하면 된다.

```
Input: head = [1,0,0,1,0,0,1,1,1,0,0,0,0,0,0]
Output: 18880
```

### 1365. How Many Numbers Are Smaller Than the Current Number

배열 nums가 주어지면 nums 내 요소마다 그 요소보다 작은 숫자의 수를 계산하여 배열로 반환하기

```TS
function smallerNumbersThanCurrent(nums: number[]): number[] {
    const sorted = [...nums].sort((a,b) => a-b)
    const obj = {}
    sorted.forEach((s, idx) => {
        if(obj[s] === undefined || obj[s] === null) {
            obj[s] = idx;
        }
    })
    return nums.map(el => obj[el])
};

const nums = [6,5,4,8]
```

nums을 오름차순으로 정렬한 sorted 배열을 생성한다. sort 메소드를 그대로 실행하면 원래 배열도 같이 정렬되므로 전개 연산자를 사용해서 배열을 새로 복사해준다.

그 다음 배열의 요소마다 값을 유동적으로 보관할 수 있게 객체 하나를 생성한다.

nums 배열을 정렬한 결과는 다음과 같다.

```ts
nums = [4, 5, 6, 8];
```

nums 내 배열의 값마다 인덱스의 값은 다음과 같다.

```ts
numsIndex = [0, 1, 2, 3];
```

4의 인덱스는 0, 5의 인덱스는 1, 6의 인덱스는 2, 8의 인덱스는 3이다. 이는 각 요소마다 각 요소보다 작은 요소의 개수와 같다.

똑같은 숫자가 있을 경우

```ts
nums = [5, 2, 5, 9];
numsIndex = [0, 1, 2, 3];
```

숫자 5가 배열 내에 2개 존재한다. 5보다 작은 숫자는 2 한 개 밖에 없으므로 두 개의 5 중에서 첫 번째 인덱스의 값, nums 배열의 경우에는 1을 지정해준다.

그 후 객체에는 배열 내 각 요소에 대하여 인덱스 값이 저장된다.

map 메소드를 이용하여 각 배열의 요소에 대한 객체의 값으로 이루어진 배열을 생성한다.

```ts
smallerNumbersThanCurrent([5, 2, 5, 9]);
// [0, 1, 1, 2]
```
