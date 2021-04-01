+++
title = "프로그래머스 문제 오픈채팅방"
date = "2021-04-01T23:57:08+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["Javascript", "Algorithm", "Programmers", "Problem", "Solving", "NodeJS"]
keywords = ["Javascript", "Algorithm", "Programmers", "Problem", "Solving", "NodeJS"]
description = ""
showFullContent = false
+++

# 카카오톡 BLIND RECRUITMENT 문제 오픈채팅방

오픈채팅방 문제에 관한 설명은 다음과 같다.

[문제링크](https://programmers.co.kr/learn/courses/30/lessons/42888?language=javascript)

A가 채팅방에 접속하면 "A님이 들어왔습니다." 라는 메시지가 남는다.

채팅방에 접속한 사람이 나가면 "A님이 나갔습니다." 라는 메시지가 남는다.

A가 채팅방에 나간 후 다시 접속할 때 B라는 닉네임을 가지고 다시 접속하면 기존에 남은 A라는 닉네임도 B로 변경되어 출력된다.

A로 접속되어있는 사람이 닉네임을 C로 변경하면 A가 작성한 메시지는 C가 작성한 메시지라고 나온다.

채팅방을 나간 사람이 닉네임을 바꾸는 경우는 없다.

가상의 닉네임을 사용하고, 닉네임의 중복을 허용하기 때문에 유저를 실질적으로 구별할 수 있는 uid라는 값이 따로 주어진다.

Record를 한번 순회할 때, 키를 uid, 값을 닉네임으로 하는 자바스크립트 객체를 생성한다.

사용자가 입장(Enter)할 때, 객체 속성이 정의되어있지 않으면 키 값을 추가하고, 유저가 닉네임을 변경(Change)할 때 값(닉네임)을 바꾼다.

닉네임 변경에 따라서 uid 키에 대한 최종 닉네임 값이 달라진다.

그 후 Record를 다시 순회하여 각각의 사용자의 동작에 따라 최종 닉네임과 연결시켜 출력한다.

action이 사용자의 행동, uid가 유저 아이디, name이 유저 닉네임이다.

입력값은 문자열의 배열로 주어진다.

```JS
["Enter uid1234 Muzi", "Enter uid4567 Prodo","Leave uid1234","Enter uid1234 Prodo","Change uid4567 Ryan"]
```

```JS
["Prodo님이 들어왔습니다.", "Ryan님이 들어왔습니다.", "Prodo님이 나갔습니다.", "Prodo님이 들어왔습니다."]
```

```JS
function solution(record) {
    const obj = {};
    const result = [];
    for(let i = 0; i < record.length; i++){
        let [action, uid, name] = record[i].split(" ")
        if(action === "Change" && obj[uid]) {
            obj[uid] = name;
        } else if(!obj[uid]) {
            obj[uid] = name;
        } else if(action === "Enter" && obj[uid]) {
            obj[uid] = name;
        }
    }
    for(let i = 0; i < record.length; i++){
        let [action, uid, name] = record[i].split(" ")
        if(action === "Enter") {
            result.push(obj[uid] + "님이 들어왔습니다.")
        } else if(action === "Leave") {
            result.push(obj[uid] + "님이 나갔습니다.")
        }
    }
    return result;
}
```

사용자가 채팅방에서 퇴장하고 나서 닉네임을 바꾸는 입력은 잘못된 입력이라 설명되었다.

따라서 그 상황에 대해서의 코드는 따로 작성하지 않았다.
