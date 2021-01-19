+++
title = "타입스크립트 그래프 순회와 힙 자료구조 구현"
date = "2021-01-19T21:51:44+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["Typescript", "Data", "Structures", "Heap", "Binary", "Max", "Graph", "BFS", "DFS", "Recursive", "Iterative", "Implementation"]
keywords = ["Typescript", "Data", "Structures", "Heap", "Binary", "Max", "Graph", "BFS", "DFS", "Recursive", "Iterative", "Implementation"]
description = ""
showFullContent = false
+++

# 타입스크립트 자료구조

### 그래프 구현

그래프는 각 요소의 정점과 정점을 잇는 간선으로 구성된 자료구조입니다.

방향 방식에 따라 단방향 그래프, 양방향 그래프로 분류할 수 있으며, 간선에 가중치를 붙일 수도 있습니다.

경로를 구성하였을 때 시작 정점과 마지막 정점이 같으면 순환(사이클) 그래프라고 부릅니다.

그래프의 정점(Vertex)과 간선(Edge)을 타입스크립트로 구현한 모습입니다. 정점을 연결하기 위해 인접 리스트를 사용했습니다.

각 노드에 연결되어있는 노드들이 순서 없이 나열되어 있습니다. 간선의 개수가 실행 시간에 영향을 미치므로 O(n)의 시간이 소요됩니다.

**정점과 링크 구현**

```TS
class Vertex {
    private name: string;
    private edges: string[];

    public getName(): string {
        return this.name
    }

    public setName(val: string){
        this.name = val;
    }

    public getEdges(): string[] {
        return this.edges;
    }

    public setEdges(val: string[]) {
        this.edges = val;
    }

    constructor(vertex: string){
        this.name = vertex;
        this.edges = [];
    }
}
```

하나의 정점을 클래스로 구현한 모습입니다. 각 정점의 이름이 문자열로, 정점에 연결된 다른 정점들이 문자열 배열로 저장됩니다.

get, set 메소드로 값을 읽거나 가져올 수 있습니다.

정점 인스턴스 생성 시 정점의 이름을 매개변수로 받습니다.

```TS
class Graph {
    private adjList: Vertex[];
    constructor(){
        this.adjList = [];
    }
```

그래프 클래스에는 우선 private 변수로 정점의 배열이 저장됩니다.

```TS
addVertex(newVertex: Vertex): void {
    if(this.adjList.find(e => e.getName() === newVertex.getName())) {
        return
    }
    this.adjList.push(newVertex);
    return;
}
```

그래프에 정점을 더하는 메소드입니다. 하나의 정점 객체를 받아서 같은 이름을 가진 정점이 없으면 인접 리스트에 추가합니다.

```TS
addAnEdge(vertex1: string, vertex2: string): void {
    this.adjList.find(el => el.getName() === vertex1)!.getEdges().push(vertex2)
    this.adjList.find(el => el.getName() === vertex2)!.getEdges().push(vertex1)
}
```

두 개의 정점을 매개변수로 받아서 간선을 추가하는 메소드입니다.
먼저 이름이 vertex1인 정점을 찾아서 간선 리스트에 vertex2를 추가합니다.
그리고 반대로 이름이 vertex2인 정점을 찾아서 간선 리스트에 vertex1을 추가합니다.

```TS
removeAnEdge(vertex1: string, vertex2: string): void {
    const newVertex1 = this.adjList.find(el => el.getName() === vertex1)!.getEdges().filter(el => el !== vertex2)
    this.adjList.find(el => el.getName() === vertex1)!.setEdges(newVertex1)

    const newVertex2 = this.adjList.find(el => el.getName() === vertex2)!.getEdges().filter(el => el !== vertex1)
    this.adjList.find(el => el.getName() === vertex1)!.setEdges(newVertex2)
}
```

두 개의 정점을 매개변수로 받아서 간선을 제거하는 메소드입니다.

먼저 이름이 vertex1인 정점을 찾아서 간선 리스트에 vertex2를 제거한 배열을 다시 간선 리스트에 새로 등록합니다.

반대로 이름이 vertex2인 정점을 찾아서 간선 리스트에 vertex1을 제거한 배열을 다시 vertex2 정점의 간선 리스트에 새로 등록합니다.

```TS
dfs(startVertexName: string): string[] {
    let result: string[] = [];
    let visited : any = {}
    let startVertex = this.adjList.find(v => v.getName() === startVertexName)

    let adjList = this.adjList;
    (function dfs(vertex: Vertex){
        if(!vertex){
            return;
        }
        result.push(vertex.getName())
        visited[vertex.getName()] = true;

        adjList.find(v => v.getName() === startVertexName)?.getEdges().forEach(neighbor => {
            if(!visited[neighbor]) {
                dfs(adjList.find(e => e.getName() === neighbor)!)
            }
        })
    })(startVertex!)

    return result
}
```

깊이 우선 탐색을 수행하는 메소드입니다. 그래프를 탐색할 때 하나의 방향으로 계속 탐색한 후 더 이상 탐색할 정점이 없으면 가장 가까운 갈림길로 돌아와서 다른 방향으로 탐색을 다시 수행합니다.

우선 결과값을 담는 result 문자열 배열을 우선 선언하고 각 정점이 탐색되었는지 판단하기 위해 visited 객체를 생성합니다. visited의 타입은 동적 타입 any라서 언제든지 속성의 추가 및 수정이 가능합니다.

매개변수로 전달한 이름에 해당하는 정점에서부터 탐색을 시작합니다. 시작에 앞서 인접 리스트를 가져옵니다.

그 후 dfs 즉시실행함수를 재귀적으로 실행시킵니다.

우선 정점의 이름을 result 배열에 추가한 다음, visited 객체 내 정점의 이름 속성 값을 true로 설정합니다. 이 뜻은 해당하는 정점이 탐색되었다는 뜻입니다.

정점의 간선 리스트에는 정점과 연결된 또 다른 정점들이 배열의 형태로 저장되어있습니다.

각 정점마다 이미 탐색이 되었는지 파악하고, 탐색이 되지 않았다면 dfs 함수에 정점의 이름을 매개변수로 넣어줌으로써 dfs 함수를 재귀적으로 실행시킵니다.

```TS
bfs(startVertexName: string): string[] {
    let result: string[] = [];
    let visited: any = {};
    let queue: string[] = [];
    queue.push(startVertexName)

    while(queue.length > 0) {
        let name = queue.shift()
        let currentVertex = this.adjList.find(e => e.getName() === name)
        if (!visited[currentVertex!.getName()]) {
            visited[currentVertex!.getName()] = true;
            result.push(currentVertex!.getName())

            currentVertex!.getEdges().forEach(neighbor => {
                if(!visited[neighbor]){
                    queue.push(neighbor);
                }
            })
        }
    }
    return result;
}
```

너비 우선 탐색입니다. 시작 정점에서 인접한 정점을 먼저 탐색합니다.

시작 정점으로부터 가까운 정점을 먼저 방문하고 멀리 떨어져 있는 정점을 나중에 방문하는 순회 방법입니다.

결과값을 담는 result 문자열 배열, 탐색했는지의 여부를 저장하는 visited 객체를 생성합니다.

그 후 queue라는 문자열 배열을 선언하는데, 이 문자열 배열은 탐색이 되지 않은 정점의 이름을 저장하고, 탐색이 완료된 정점의 이름을 제거합니다.

우선 매개변수로 주어진 정점의 이름을 queue에 추가합니다.

이름에 해당하는 정점이 아직 탐색되지 않았다면, 즉 visited 객체에서 정점 이름 속성 값이 false라면 탐색이 완료되었음을 의미하는 true로 바꾸고 result에 정점의 이름을 저장합니다.

현재 정점에 인접한 정점들을 아직 탐색이 되지 않았다면 하나씩 queue에 저장합니다.

이 과정을 queue의 길이가 0이 될 때가지, 즉 탐색이 되지 않은 정점이 없을 때까지 반복합니다.
