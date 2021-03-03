+++
title = "Leetcode 897. Increasing Order Search Tree"
date = "2021-03-03T23:57:02+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["Typescript", "NodeJS", "Leetcode", "Increasing", "Order", "Search", "Tree", "Graph", "Binary"]
keywords = ["Typescript", "NodeJS", "Leetcode", "Increasing", "Order", "Search", "Tree", "Graph", "Binary"]
description = ""
showFullContent = false
+++

# Increasing Order Search Tree

문제 주소 : <https://leetcode.com/problems/increasing-order-search-tree/>

주어진 이진 트리의 요소를 오름차순으로 재배열한다.

```Typescript
/**
 * 이진 트리의 정의
 * class TreeNode {
 *     val: number
 *     left: TreeNode | null
 *     right: TreeNode | null
 *     constructor(val?: number, left?: TreeNode | null, right?: TreeNode | null) {
 *         this.val = (val===undefined ? 0 : val)
 *         this.left = (left===undefined ? null : left)
 *         this.right = (right===undefined ? null : right)
 *     }
 * }
 */

function increasingBST(root: TreeNode | null): TreeNode | null {
    const arr: number[]=[];

    (function func(node: TreeNode | null){
        if(!node) return;

        func(node.left)
        arr.push(node.val);
        func(node.right)
    })(root);

    const tree = new TreeNode(arr[0])
    let cur = tree;
    for(let i = 1; i < arr.length; i++){
        const child = new TreeNode(arr[i])
        cur.right = child;
        cur = cur.right;
    }
    return tree;
};
```

트리의 요소만을 보관할 수 있는 배열을 하나 생성했다.

트리의 모든 요소를 순회하면서 배열에 요소를 하나씩 보관한 후 새로운 트리의 루트 노드를 생성해서 오른쪽 노드에 계속 이어붙인다.

이진 트리는 부모 노드의 자식 노드로 각각 왼쪽 노드와 오른쪽 노드가 있다.

왼쪽 노드는 부모 노드의 값보다 작고 오른쪽 노드는 큰 값을 가진다.

트리를 순회할 때는 IIFE를 사용하여 재귀적으로 순회한다.

트리의 부모 노드의 값을 배열에 넣기 전, 왼쪽 자식 노드의 값을 배열에 저장하도록 했다.

왼쪽 자식 노드가 null이라면 실행되는 func 함수가 리턴되어 부모 노드의 값을 저장할 것이다.
