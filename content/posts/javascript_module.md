+++
title = "자바스크립트 모듈"
date = "2021-01-23T00:22:54+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["Javascript", "Module", "mjs", "Export", "Default", "Typescript", "CommonJS", "AMD", "UMD"]
keywords = ["Javascript", "Module", "mjs", "Export", "Default", "Typescript", "CommonJS", "AMD", "UMD"]
description = ""
showFullContent = false
+++

# 자바스크립트 모듈

모듈은 애플리케이션을 구성하는 요소로서 여러 번에 걸쳐 사용할 수 있는 장점이 있다. 일반적으로 파일 단위로 분리되어 있다.

애플리케이션은 코드 내 모듈을 명시함으로써 필요한 코드를 불러올 수 있다. 코드의 단위, 또는 쓰임새를 명확하게 분리할 수 있어서 코드의 유지보수를 용이하게 하고 효율성 또한 높일 수 있다.

필요한 코드만을 불러올 수 있어서, 메모리의 사용량이 감소하는 효과도 있다.

Javascript 모듈은 ES2015부터 지원하기 시작했다.

구형 브라우저는 모듈 문법을 지원하지 않아서 Webpack, Browserify, Gulp와 같은, 서드파티 번들러를 통해 변환 과정을 거쳐, 브라우저가 사용할 수 있는 JS 파일로 만들어준다.

NodeJS에서는 CommonJS를 통해 require 문으로 모듈을 불러올 수 있고,

module.exports = "모듈" 혹은 module.exports.theModule = "모듈" 로 모듈을 타 코드에서 불러오도록 할 수 있다.

```JS
// require
const {theModule} = require("./app");

// export
module.exports.theModule = moduleSample;
```

혹은 자바스크립트 파일에 다음과 같이 명시함으로써 브라우저에서 사용할 수 있다. 이 때는 확장자가 js가 아닌 mjs이다. 웹 상에서는 확장자를 따로 구분하고 있지는 않다.

```JS
<script src="moduleSample.mjs" type="module"></script>

import {theLibrary} from "./lib.mjs";
```

한 개의 모듈을 여러 번 불러와도 단 한 번 실행된다.

모듈은 기본적으로 strict 모드이다. JS 파일로 구성되어 있으므로, HTML 주석은 사용할 수 없다.

블록(렉시컬) 스코프를 따르고 있어서, 전역 변수를 선언하더라도 모듈 스코프 내에서 선언된다. window 객체에서 접근하는 것은 불가능하다.

```js
const m = window.oneModule; // ERROR
```

NodeJS 환경에서는 CommonJS 모듈의 규칙을 따르고 있다.
import 문을 사용하면 정상적으로 실행이 되지 않아, Babel을 통해 트랜스파일링을 거쳐 주어야 한다.

const module... 및 import module... 와 같은 정적 import 문은 모든 모듈을 불러오고 나서 코드가 실행된다.
반대로 유저의 특정한 동작에 모듈이 작동되도록 할 수 있다.
전체 모듈 로드 시간을 줄일 수 있다.

```html
<script type="module">
  (async () => {
    const foo = "./lib.mjs";
    const { func1, func2 } = await import(foo);
    await func2();
    func1();
    const { func3 } = require("./the-module.js");
    await func3();
    console.log("Module loaded dynamically");
  })();
</script>
```

버튼만 누를 때 oneModule 모듈을 불러오는 예제 코드

```js
const button = document.getElementById("btn"); // 버튼
button.addEventListener("click", function () {
  const oneModule = require("./one-module.js");
  oneModule.func4();
  console.log("Module loaded dynamically...");
});
```

모듈 스코프에서 정의된 변수은 export를 통해 타 코드에서 불러오도록 할 수 있다.
변수의 값이 아닌 변수의 이름을 불러오는 것이 특징이다.

```js
const bar = "variable";
export { bar };
```

변수를 다른 파일에서 사용할 수 있도록 한다. 변수의 이름을 내보냈으므로 named export라고 한다. import 또는 require 문을 통해 가져올 수 있다.

변수 뿐만 아니라 함수, 객체 또한 export 가능하다.

```js
import { bar } from "./two-module.js";
const obj = {
  name: "Kim",
  career: "Doctor",
};
export { obj };

// 분리된 파일
import { obj } from "./three-module.js";
```

NodeJS 내장 라이브러리 또는 npm에서 받아온 라이브러리가 아닌 직접 생성한 모듈은 경로 지정을 해주어야 한다.

```JS
const fs = require("fs")
```

선언과 동시에 export 할 수 있다. export default로 단 하나의 모듈만을 export 할 수 있다. 이렇게 선언된 모듈은 중괄호 없이 불러올 수 있다.

이 경우에도 구조분해할당이 적용된다.

```js
import Person from "./person.js";
import React, { Component } from "react";
```

이렇게 모듈을 불러오면 Component 객체 사용 시 React.Component 대신 Component만 적어도 된다.

모듈을 불러올 때 이름을 따로 지정해 줄 수 있다.

```js
const var1 = 400;
export { var1 };

import { var1 as Var } from "./four-module.js";
```

var1으로 export된 변수를 Var로 불러왔다.

```js
const var2 = 400;
export { var2 as Var };

// 분리된 파일
import { Var } from "./fivemodule.js";
```

또는 Var로 export된 변수를 불러와서 var2의 값을 사용할 수 있다.

import export는 모듈을 내보내거나 사용하도록 명시해주는 것 뿐이다.
현재 웹 브라우저는 ES6 모듈을 완전히 지원하고 있지 않다.

script 태그에 type="text/module"을 명세하거나
Webpack, Browserify 등의 라이브러리를 사용해야 한다.

React도 create-react-app으로 실행해서 리액트 프로젝트를 생성 후 열어보았을 때
Webpack 라이브러리가 설치되어 있는 것을 확인할 수 있다.
