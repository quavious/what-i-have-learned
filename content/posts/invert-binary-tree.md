+++
title = "Invert Binary Tree"
date = "2021-02-22T23:00:55+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["Tree", "Invert", "Binary", "Recursive", "Search", "Typescript", "IIFE", "Graph"]
keywords = ["Tree", "Invert", "Binary", "Recursive", "Search", "Typescript", "IIFE", "Graph"]
description = ""
showFullContent = false
+++

# Invert Binary Tree

이진 트리를 좌우 반전시키는 문제이다.

**작성한 코드**

```Typescript
/**
 * 이진 트리 노드를 정의한 클래스이다.
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

function invertTree(root: TreeNode | null): TreeNode | null {
    if(!root) return null;
    (function traverse(node: TreeNode | null){
        if(!node) return;
        traverse(node.left);
        traverse(node.right);
        let temp = node.left
        node.left = node.right
        node.right = temp

        return node
    })(root);
    return root
};
```

이진 트리 전체를 좌우 반전해주면 된다.

노드의 구조가 다음과 같다고 한다면

```
     A
   /   \
  B     C
 / \   / \
D   E F   G
```

트리의 가장자리 노드의 자식 노드를 서로 바꿔준 다음, 그 가장자리 노드의 상위 노드로 올라가서 똑같은 작업을 수행하도록 했다.

타입스크립트에서 객체는 참조 타입이기 때문에 변수를 서로 교체하는 것은 객체의 주소를 교체하는 것이다.

A의 자식 노드, B C가 서로 교체되면 B는 C의 메모리 주소를 가지고 있기 때문에 자식 노드 F, G를, C는 자식 노드 D, E를 가지게 된다.

B, C 노드의 자식 노드를 각각 교체한 다음 B, C의 노드의 위치를 교체했다.
