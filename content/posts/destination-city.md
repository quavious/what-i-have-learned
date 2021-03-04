+++
title = "Leetcode 1436. Destination City"
date = "2021-03-05T01:54:23+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["Typescript", "Algorithm", "Leetcode", "Destination", "City", "String", "Object"]
keywords = ["Typescript", "Algorithm", "Leetcode", "Destination", "City", "String", "Object"]
description = ""
showFullContent = false
+++

# Destination City

출발지와 도착지가 명시된 경로 배열이 여러 개 주어진다. 만약 경로가 [A, B]라면 A에서 B로 향하는 것이다.

다른 지점으로 갈 수 없는 지역이 곧 종착지점이 된다.

처음엔 그래프로 해결하려고 했는데, 만약 D가 종착지점이라면 D에서 다른 지역으로 향하는 배열이 정의되지 않았을 것이라는 걸 알게되었다.

각 출발지와 도착지를 키:값 객체로 구성했다.

도착지를 순회하여 도착지를 또 다른 시작지점이 되는 경로가 있는지 파악한다.

경로가 없다면 그 지점이 바로 종착지점이 되는 것이다.

### 코드

```Typescript
interface obj {
    [key: string]: string;
}

function destCity(paths: string[][]): string {
    const routes: obj = {}
    for(let i = 0; i < paths.length; i++){
        routes[paths[i][0]] = paths[i][1]
    }
    let dest = ""
    Object.values(routes).forEach(el => {
        if(!routes[el]){
            dest = el;
        }
    })
    return dest;
};
```
