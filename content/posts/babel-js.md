+++
title = "Babel and Webpack으로 자바스크립트 번들링하기"
date = "2021-03-13T00:05:07+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["NodeJS", "Webpack", "Browser", "Babel", "Preset", "Yarn", "Build"]
keywords = ["NodeJS", "Webpack", "Browser", "Babel", "Preset", "Yarn", "Build"]
description = ""
showFullContent = false
+++

# 바벨JS와 웹팩으로 자바스크립트 파일 번들링

현재 대부분의 브라우저는 ES6 모듈을 지원하고 있다. 스크립트 태그를 다음과 같이 작성함으로써 ES6 모듈을 불러올 수 있다.

```HTML
<script type="module" src="src/app.js"></script>
```

이 때 자바스크립트 경로는 파일 디렉토리 내 경로가 아닌, 서버 내 경로를 명시해주어야 한다.

웹 브라우저는 파일 시스템에 접근할 수 없기 때문에, 파일 시스템 내 모듈을 불러올 경우 보안 정책에 의해 차단된다.

간이 백엔드 웹 서버를 구축해 줌으로써 자바스크립트 모듈을 정상적으로 불러올 수 있다.

```
📂 public
    📝 index.html
    📝 app.js
📝 server.js
```

```JS
// server.js
const express = require("express");
const server = express();
server.use(express.static("public"))
server.get("/", (req, res) => res.send("index.html"))
server.listen(3000, null);
```

public 폴더에서 HTML, JS와 같은 정적 파일을 제공하도록 할 수 있다.

이 때 app.js의 경로는 ./public/app.js가 아닌 ./app.js가 된다.

## 서버 없이 모듈을 불러오도록 하고 싶다.

바벨을 통해 ES6 코드를 하위 버전으로 바꿀 수 있다.

```
yarn add @babel/cli @babel/core @babel/polyfill @babel/preset-env @babel/plugin-proposal-class-properties
```

@babel/polyfill을 설치하는 이유는 ES5에 존재하지 않는 ES6 문법까지 같이 컴파일하기 위함이다.

@babel/plugin-proposal-class-properties은 ES6 클래스까지 컴파일 할 수 있도록 한다.

babel.config.json 파일을 생성해서 프리셋을 명시한다.

프리셋은 바벨 컴파일하는데 사용되는 일종의 플러그인이다.

```JSON
{
    "presets": ["@babel/preset-env"]
}
```

package.json에 build 명령어를 지정할 수 있다.

```JSON
{
    "scripts": {
        "build": "babel src -d dist"
    }
}
```

src에 있는 자바스크립트 파일을 컴파일해서 그 결과를 dist 폴더에 놓겠다는 의미이다.

파일 구조는 다음과 같다.

```
📂 node_modules
📂 src
    📝 app.js
📂 dist
    📝 app.js
📝 package.json
📝 babel.config.json
📝 yarn.lock
📝 index.html
```

다음과 같은 자바스크립트 파일을 작성한다.

Promise 객체를 이용해서 1초 뒤에 1000을 출력할 수 있게 해주는 코드이다.

```JS
import "@babel/polyfill";

(async() => {
    await new Promise((res, rej) => {
        try {
            setTimeout(() => {
                console.log(1000)
                res()
            }, 1000)
        } catch (err) {
            console.error(err)
            rej()
        }
    })
})();
```

코드 맨 상위에 @babel/polyfill을 불러온다.

yarn build를 실행해서 컴파일한 뒤 node ./dist/app.js로 컴파일된 코드를 실행할 수 있다.

### dist 폴더에 있는 자바스크립트 파일을 HTML 파일에서 불러오고 싶다.

index.html을 다음과 같이 작성한다.

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <script src="dist/app.js"></script>
</body>
</html>
```

HTML 파일을 실행해서 콘솔을 보면 다음과 같은 오류가 발생한다.

```
Uncaught ReferenceError: require is not defined at app.js...
```

노드JS 환경에서 모듈을 불러올 수 있는 CommonJS 모듈 시스템의 require 문을 웹 브라우저가 이해하지 못하기 때문이다.

이를 극복하기 위해 웹팩(Webpack)을 사용할 수 있다.

웹팩은 의존 관계에 있는 모듈들을 하나의 자바스크립트 파일로 번들링하는 모듈 번들러이다.

여러 개의 연결된 모듈이 하나의 파일로 번들링되므로 번들 로더가 필요하지 않으며, HTML 파일에서 여러 개의 자바스크립트 파일을 불러오지 않아도 된다.

웹팩, 그리고 웹팩에서 바벨을 불러올 수 있도록 babel-loader를 설치한다.

```
yarn add webpack webpack-cli babel-loader
```

그 다음 webpack.config.js 파일을 생성해서 기초적인 웹팩 설정을 해준다.

```JS
const path = require("path")

module.exports = {
    entry: "./src/app.js",
    output: {
        path: path.resolve(__dirname, "dist"),
        filename: "bundle.js"
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                include: [
                    path.resolve(__dirname, "src")
                ],
                exclude : /node_modules/,
                use: {
                    loader: "babel-loader",
                    options: {
                        presets: ['@babel/preset-env'],
                        plugins: ["@babel/plugin-proposal-class-properties"]
                    }
                }
            }
        ]
    },
    devtool: "source-map",
    mode: "development"
}
```

**각 속성에 대해서는 나중에 다시 작성하겠습니다.**

자바스크립트 파일의 엔트리는 app.js가 된다. 엔트리는 웹팩이 번들링 수행하는데 있어 시작점이 된다.

웹팩으로 번들링한 결과는 dist/bundle.js 파일에 나타날 것이다.

include로 번들링할 파일을 따로 지정할 수 있다. src 폴더에 있는 자바스크립트 파일이 번들링된다.

use 속성에서 바벨 프리셋과 플러그인을 불러올 수 있다.

그 다음 package.json에서 build 속성을 다음과 같이 바꿔준다.

```JSON
{
    "scripts": {
        "build" : "webpack -w"
    }
}
```

yarn build로 웹팩 번들링을 실행시킨다.

index.html 파일에서 dist/bundle.js를 불러오면 번들링된 자바스크립트 파일이 잘 실행되는 것을 볼 수 있다.

**사념**

간단한 ES6 자바스크립트 파일을 바벨로 컴파일하면 다음과 같은 결과물이 나온다.

```JS
"use strict";

require("@babel/polyfill");

function asyncGeneratorStep(gen, resolve, reject, _next, _throw, key, arg) { try { var info = gen[key](arg); var value = info.value; } catch (error) { reject(error); return; } if (info.done) { resolve(value); } else { Promise.resolve(value).then(_next, _throw); } }

function _asyncToGenerator(fn) { return function () { var self = this, args = arguments; return new Promise(function (resolve, reject) { var gen = fn.apply(self, args); function _next(value) { asyncGeneratorStep(gen, resolve, reject, _next, _throw, "next", value); } function _throw(err) { asyncGeneratorStep(gen, resolve, reject, _next, _throw, "throw", err); } _next(undefined); }); }; }

_asyncToGenerator( /*#__PURE__*/regeneratorRuntime.mark(function _callee() {
  return regeneratorRuntime.wrap(function _callee$(_context) {
    while (1) {
      switch (_context.prev = _context.next) {
        case 0:
          _context.next = 2;
          return new Promise(function (res, rej) {
            try {
              setTimeout(function () {
                console.log(1000);
                res();
              }, 1000);
            } catch (err) {
              console.error(err);
              rej();
            }
          });

        case 2:
        case "end":
          return _context.stop();
      }
    }
  }, _callee);
}))();
```

타입스크립트 환경에서 tsc 명령어를 실행하면 타입스크립트 파일을 원하는 버전의 자바스크립트로 컴파일할 수 있다.

똑같은 코드를 tsc로 컴파일한 결과물

```JS
"use strict";
var __awaiter = (this && this.__awaiter) || function (thisArg, _arguments, P, generator) {
    function adopt(value) { return value instanceof P ? value : new P(function (resolve) { resolve(value); }); }
    return new (P || (P = Promise))(function (resolve, reject) {
        function fulfilled(value) { try { step(generator.next(value)); } catch (e) { reject(e); } }
        function rejected(value) { try { step(generator["throw"](value)); } catch (e) { reject(e); } }
        function step(result) { result.done ? resolve(result.value) : adopt(result.value).then(fulfilled, rejected); }
        step((generator = generator.apply(thisArg, _arguments || [])).next());
    });
};
var __generator = (this && this.__generator) || function (thisArg, body) {
    var _ = { label: 0, sent: function() { if (t[0] & 1) throw t[1]; return t[1]; }, trys: [], ops: [] }, f, y, t, g;
    return g = { next: verb(0), "throw": verb(1), "return": verb(2) }, typeof Symbol === "function" && (g[Symbol.iterator] = function() { return this; }), g;
    function verb(n) { return function (v) { return step([n, v]); }; }
    function step(op) {
        if (f) throw new TypeError("Generator is already executing.");
        while (_) try {
            if (f = 1, y && (t = op[0] & 2 ? y["return"] : op[0] ? y["throw"] || ((t = y["return"]) && t.call(y), 0) : y.next) && !(t = t.call(y, op[1])).done) return t;
            if (y = 0, t) op = [op[0] & 2, t.value];
            switch (op[0]) {
                case 0: case 1: t = op; break;
                case 4: _.label++; return { value: op[1], done: false };
                case 5: _.label++; y = op[1]; op = [0]; continue;
                case 7: op = _.ops.pop(); _.trys.pop(); continue;
                default:
                    if (!(t = _.trys, t = t.length > 0 && t[t.length - 1]) && (op[0] === 6 || op[0] === 2)) { _ = 0; continue; }
                    if (op[0] === 3 && (!t || (op[1] > t[0] && op[1] < t[3]))) { _.label = op[1]; break; }
                    if (op[0] === 6 && _.label < t[1]) { _.label = t[1]; t = op; break; }
                    if (t && _.label < t[2]) { _.label = t[2]; _.ops.push(op); break; }
                    if (t[2]) _.ops.pop();
                    _.trys.pop(); continue;
            }
            op = body.call(thisArg, _);
        } catch (e) { op = [6, e]; y = 0; } finally { f = t = 0; }
        if (op[0] & 5) throw op[1]; return { value: op[0] ? op[1] : void 0, done: true };
    }
};
(function () { return __awaiter(void 0, void 0, void 0, function () {
    return __generator(this, function (_a) {
        switch (_a.label) {
            case 0: return [4 /*yield*/, new Promise(function (res, rej) {
                    try {
                        setTimeout(function () {
                            console.log(1000);
                            res(null);
                        }, 1000);
                    }
                    catch (err) {
                        console.error(err);
                        rej();
                    }
                })];
            case 1:
                _a.sent();
                return [2 /*return*/];
        }
    });
}); })();
```

바벨과 타입스크립트 컴파일 방식을 기회가 된다면 공부하고 싶다.
