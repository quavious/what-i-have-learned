+++
title = "재사용 가능한 Axios 인스턴스 생성 및 사용하기"
date = "2021-01-06T23:58:06+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["Axios", "Ajax", "Typescript", "Http", "Request", "Crawling", "Scraping", "Web"]
keywords = ["Axios", "Ajax", "Typescript", "Http", "Request", "Crawling", "Scraping", "Web"]
description = "노드JS 환경에서 axios 인스턴스를 재사용하는 방법입니다."
showFullContent = false
+++

### 문제 상황

클라이언트가 백엔드 API 서버에서 데이터를 받아오거나, 웹 스크래핑을 수행하기 위해선 HTTP 통신을 해야 합니다.
자바스크립트에서는 HTTP 통신을 요청하고 응답을 받는데 다양한 메소드를 사용할 수 있습니다.

- XMLHttpRequest
- Jquery의 Ajax(Asynchronous Javascript And XML)
- 브라우저 내장 Fetch API, 그러나 노드JS에서도 사용 가능
- 노드JS Request 모듈
- Axios

Axios를 사용함으로써 다양한 이점을 챙길 수 있습니다.

- HTTP 응답 데이터가 JSON 형태로 쉽게 변환됩니다.
- Promise 기반이라 비동기 처리를 쉽게 수행할 수 있습니다.
- XSRF 공격을 클라이언트 기반에서 방어할 수 있습니다.
- 구형 브라우저를 지원합니다.
- Promise 기반입니다.

그동안 노드JS 기반에서 웹 크롤링을 수행하거나 API를 호출할 때 Axios 라이브러리를 요긴하게 사용했습니다.
하지만 많은 횟수의 요청을 수행할 때면 가끔씩 몇 개의 예상하지 못한 오류가 발생하는 것을 확인할 수 있었습니다.

제일 많이 마주친 오류는 Socket Hang Up ECONNRESET 였습니다.

이 오류는 서버에서 연결을 강제로 종료하였을 때 나타나는 오류임을 확인했습니다.

오류의 자세한 원인은 아직 파악하지 못했지만 구글에 검색한 결과 axios 인스턴스가 계속적으로 생성돼서 병목현상이 발생할 수 있었다는 것을 알게 되었습니다.

한 개의 axios 인스턴스를 만들어서 그 인스턴스만 HTTP 요청을 할 수 있도록 코드를 수정해보라는 조언을 찾을 수 있었습니다.

Axios 인스턴스를 하나 생성할 때 인스턴스의 설정값을 미리 설정해 줄 수 있는데 설정값 중 keepAlive를 true로 작성해주어야 합니다.

코드 예시

```Typescript
import axios from 'axios'
// agentkeepalive 모듈을 사용합니다.
// 노드JS 내장 모듈은 요청 대상의 IP 주소가 변경되었을 때 소켓이 서버를 찾지 못해 연결이 종료되는 문제가 있습니다.
import * as Agent from 'agentkeepalive'

const httpAgent = new Agent({
    maxSockets: 100,
    maxFreeSockets: 10,
    // timeout을 설정해서 얼마동안 연결을 유지할지 지정해줍니다.
    timeout: 60000,
    freeSocketTimeout: 30000,
})

// Axios 인스턴스 생성. 저는 이 인스턴스를 계속 사용합니다.
const httpClient = axios.create({
    httpAgent: httpAgent,
    headers: {
        // 요청 헤더 값
    }
})
// HTTP 요청은 노드JS에서는 비동기적으로 작동합니다.
async function func(apiURL: string){
    const resp = await httpClient.get(apiURL)
    const data = await resp.data;
    // 이후 작업 진행
}
```

async 키워드로 func 함수가 비동기적으로 작동한다는 것을 명시해 줄 수 있습니다.

이 func 함수 내부에서는 await 키워드를 입력하여 비동기 함수, 메소드가 작업을 완료할 때까지 기다려 줄 수 있습니다.

함수 내에서 await httpClient.get 메소드를 마주치면 get 메소드의 작동이 멈추게 됩니다.

get 메소드는 프로미스 객체를 반환하는데, 프로미스 객체가 비동기 작업의 결과값을 받으면 다시 메소드가 실행되어 resp 변수에 값을 넘겨줍니다.

이렇게 하여 불필요한 Axios 인스턴스를 생성하지 않고 하나의 인스턴스로 여러 개의 HTTP 요청을 보낼 수 있습니다.

실제로 사용한 결과 아직 오류는 발생하지 않았지만 앞으로 계속 지켜봐야 할 것 같습니다.
