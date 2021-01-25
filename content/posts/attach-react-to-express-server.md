+++
title = "리액트 프로젝트와 익스프레스 서버 연동"
date = "2021-01-25T23:52:15+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["React", "Express", "CORS", "JSON", "Build", "Client", "Front-End", "Back-End", "NodeJS"]
keywords = ["React", "Express", "CORS", "JSON", "Build", "Client", "Front-End", "Back-End", "NodeJS"]
description = ""
showFullContent = false
+++

# 리액트와 익스프레스 연결

프론트엔드로 리액트, 백엔드로 익스프레스를 사용한 노드JS 웹 어플리케이션을 만들고 싶다.

우선 react-express 폴더를 하나 생성해주고 폴더 내 client, server 폴더를 만들어주었다. 각각 리액트와 익스프레스가 설치될 것이다.

### 리액트 코드 작성

우선 client 폴더로 이동한 후 create-react-app으로 리액트 프로젝트를 설치한다.

```
yarn create react-app .
```

.을 찍으면 별도의 폴더가 생성되지 않고 현재 위치한 폴더에 리액트 프로젝트가 그대로 설치된다.

```
📂 client
 📂 node_modules
 📂 public
 📂 src
  🧾 App.js
  🧾 App.css
  🧾 index.js
 🧾 package.json
 🧾 yarn.lock
```

App.js에 리액트 자바스크립트 코드를 작성한다. React Hooks를 사용한 함수형 컴포넌트로 구성했다.

data를 서버로부터 받아서 이름, 직업, 나이, 사는 곳을 출력한다.

```js
import "./App.css";
import { useEffect, useState } from "react";
import axios from "axios";

function App() {
  const [data, setData] = useState(null);
  useEffect(() => {
    (async () => {
      try {
        const { data } = await axios.get("http://localhost:5000/api/index");
        setData(data);
      } catch (err) {
        console.error(err);
      }
    })();
  }, []);
  return (
    <div className="App">
      {(() => {
        if (!data) {
          return <p>No Data</p>;
        } else {
          return (
            <>
              <h3>I am {data.name}</h3>
              <h4>{data.age} years old.</h4>
              <p>I am working {data.job} every day.</p>
              <p>Living in {data.address}</p>
            </>
          );
        }
      })()}
    </div>
  );
}

export default App;
```

useEffect에 의존성 배열을 빈 배열로 넣어줌으로써 컴포넌트 최초 렌더링시에만 데이터를 서버로부터 받아오게 했다.

데이터가 무사히 수신되면 !data가 false가 되어 여러가지 정보가 화면에 렌더링된다.

리액트 포트는 3000번이다.

JSX 코드 내 if문을 사용하고 싶어서 괄호 안에 IIFE 문을 작성했다.

### 익스프레스 서버 백엔드 API 코드 작성

위치를 client 폴더에서 server 폴더로 바꾸어 Express 백엔드 API 코드를 작성했다.

폴더 구조는 다음과 같다.

```
📂 server
 📂 node_modules
 📂 public
 🧾 server.js
 🧾 package.json
 🧾 yarn.lock
```

server.js 파일에 코드를 작성한다.

```js
const express = require("express");
const app = express();

app.use(express.json());

app.get("/api/index", (req, res) =>
  res.json({
    name: "Kim",
    job: "Developer",
    age: 22,
    address: "Seoul",
  })
);

app.listen(5000, () => console.log("Server on 5000"));
```

서버 포트는 5000번이다. localhost:5000/api/index 주소로 REST GET 요청을 보내면 서버에서는 name, job, age, address로 이루어진 객체 데이터를 다시 프론트엔드로 전달할 것이다.

서버와 클라이언트 모두 실행한 후 3000번 포트의 로컬호스트 주소로 접속해서 데이터를 받아오는지 확인한다.

```
서버 - server 폴더에서 node server.js
클라이언트 - client 폴더에서 yarn start
```

데이터가 화면에 출력되지 않는다. 개발자 도구를 켜서 콘솔 화면을 보면 다음과 같은 에러창이 뜬다.

```
Access to XMLHttpRequest at 'http://localhost:5000/api/index' from origin 'http://localhost:3000' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

CORS 정책에 의해 막혀있다는 내용이다.

CORS 정책에 앞서 동일 출처 정책, Same-Origin Policy의 정의는 다음과 같다.

- MDN 문서 내 동일 출처 정책의 정의

**어떤 출처에서 불러온 문서나 스크립트가 다른 출처에서 가져온 리소스와 상호작용하는 것을 제한하는 중요한 보안 방식이다. 동일 출처 정책은 잠재적으로 해로울 수 있는 문서를 분리함으로써 공격받을 수 있는 경로를 줄여준다.**

즉 웹 어플리케이션 내 스크립트는 프로토콜, 호스트 이름, 포트를 공유하는 주소로만 AJAX 요청을 보낼 수 있다.

```
a.com에 존재하는 JS 파일에서 a.com/api/user 등의 주소로는 도메인, 호스트 이름이 같으므로 HTTP 요청을 보낼 수 있다.
하지만 a.com에 존재하는 JS 파일에서 b.com으로 접근하는 건 허용되지 않는다.
```

클라이언트는 3000, 서버는 5000 포트가 서로 다르므로 Axios 요청이 정상적으로 이루어지지 않는다.

이를 위해서는 서버에서 CORS, 교차 출처 리소스 공유 설정을 바꾸어서, 도메인 또는 포트가 다르더라도 HTTP 요청을 보낼 수 있도록 해주어야한다.

server 폴더에서 cors 라이브러리를 설치한다.

```
yarn add cors
```

그 다음 server.js 파일 내 코드를 변경해준다.

```JS
const express = require("express")
const cors = require("cors")
const app = express()

app.use(express.json())
app.use(cors({
    origin: "http://localhost:3000",
    optionsSuccessStatus: 200,
    methods: ['GET', 'POST'],
}))


app.get("/api/index", (req, res) => res.json({
    "name": "Kim",
    "job" : "Developer",
    "age": 22,
    "address": "Seoul"
}))

app.listen(5000, () => console.log("Server on 5000"))
```

app.use에는 미들웨어를 설정할 수 있다. 클라이언트에서 요청을 보내면 서버 미들웨어를 먼저 경유한 후 서버 내 주소로 접속할 수 있다.

cors 설정에서 서버로의 접속이 허가되는 주소를 설정하기 위해 origin을 http://localhost:3000으로 설정해준다.

그리고 REST Method 설정에서는 GET, POST를 추가하여 3000번 포트에서 보낸 요청 중 GET, POST 요청만 서버에서 받도록 한다.

서버 코드를 수정한 후 다시 웹 애플리케이션을 실행시키고 3000번 포트로 접속하면 포트 5000번의 서버로부터 데이터가 잘 수신된 것을 확인할 수 있다.

리액트 프로젝트를 빌드해서 백엔드 API 서버에 붙이는 방법도 있다.

우선 리액트 프로젝트를 빌드한다.

```
yarn build
```

빌드 과정을 수행함으로써 리액트의 소스 코드를 정적인 HTML, JS, CSS 파일로 변환해준다. 이렇게 생성된 파일은 리액트가 설치되어있지 않아도 접속할 수 있다.

하지만 index.html 파일을 더블 클릭해서 접속하면 아무것도 뜨지 않는다.

정상적인 접속을 위해선 serve 라이브러리를 설치 후 serve 명령어를 실행하여 정적 서버를 단독으로 구동하거나, 백엔드 API에 정적 파일의 형태로 붙여야 한다.

그래서 빌드된 파일이 담겨진 build를 server 폴더에 복사 붙여넣기 후 백엔드 API 코드를 다음과 같이 수정한다.

```js
const express = require("express");
const cors = require("cors");
const app = express();

app.use(express.json());
app.use(express.static("build"));
app.use(
  cors({
    origin: "http://localhost:3000",
    optionsSuccessStatus: 200,
    methods: ["GET", "POST"],
  })
);

app.get("/api/index", (req, res) =>
  res.json({
    name: "Kim",
    job: "Developer",
    age: 22,
    address: "Seoul",
  })
);

app.listen(5000, () => console.log("Server on 5000"));
```

미들웨어를 하나 더 추가해준다. express.static을 작성하여 build 폴더가 서버 내 정적 파일을 담고 있다는 것을 명시할 수 있다.

server.js 파일을 실행시켜서 5000번 포트의 로컬호스트로 접속하면 3000번 포트에 접속했을 때의 화면을 그대로 볼 수 있다.

### 참고

리액트에서는 모듈을 불러올 때 import 문을 사용하지만, 익스프레스에서는 require 문으로 불러올 수 있다.

리액트를 create-react-app으로 설치하면 webpack, babel 등의 리액트를 보조할 자바스크립트 라이브러리들이 같이 설치된다.

이 중에서 babel 라이브러리가 import 문과 같은 ES6 이상의 문법이 사용된 코드를 웹 브라우저가 인식할 수 있도록 트랜스파일링해준다.

express에서도 import 문 등의 ES6+ 코드를 작성하려면 babel을 직접 설정해주거나, 타입스크립트를 사용한 후 자바스크립트로 컴파일하는 방법을 사용할 수 있다.
