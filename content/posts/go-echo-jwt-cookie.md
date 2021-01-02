+++
title = "How to implement JWT Authentication Process in Golang Echo"
date = "2021-01-03T00:54:57+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["golang", "backend"]
keywords = ["echo", "golang", "jwt", "authentication", "cookie", "backend"]
description = "Implement Authentication with JWT and Cookies in Echo, Go Web Framework"
showFullContent = false
+++

### 개요

Go 웹 프레임워크 Echo, JWT, 쿠키를 이용하여 간이 인증 절차를 구현한다.

### 환경

- Windows 10
- Visual Studio Code
- Golang 1.15.x
- Echo
- Postman

### 코드

설명이 빈약하거나, 잘못된 점이 있을 수 있습니다. 실력이 부족한 점 양해바랍니다.

```Go
package main

import (
	"fmt"
	"log"
	"net/http"
	"time"

	"github.com/dgrijalva/jwt-go"
	"github.com/labstack/echo/v4"
	"github.com/labstack/echo/v4/middleware"
)

type loginForm struct {
	Email    string `json:"email"`
	Password string `json:"password"`
}

type jwtCustomClaims struct {
	Name  string `json:"name"`
	Admin bool   `json:"admin"`
	jwt.StandardClaims
}

func main() {
	e := echo.New()
	e.Use(middleware.Logger())
	e.Use(middleware.Recover())
	r := e.Group("/auth")
	config := middleware.JWTConfig{
		Claims:        &jwtCustomClaims{},
		SigningKey:    []byte("secret"),
		SigningMethod: "HS256",
		TokenLookup:   "cookie:x-token",
	}
	r.Use(middleware.JWTWithConfig(config))

	e.POST("/login", func(c echo.Context) error {
		user := new(loginForm)
		if err := c.Bind(user); err != nil {
			return c.JSON(http.StatusBadRequest, echo.Map{
				"status": false,
				"msg":    "Invalid Data",
			})
		}

        // 실제 백엔드 개발에서는 데이터베이스와 해쉬 함수를 같이 활용합니다.
        // 구현에 앞서 미리 설정해 둔 아이디와 비밀번호
        if user.Email != "이메일" || user.Password != "비밀번호" {
            return c.JSON(http.StatusBadRequest, echo.Map{
				"status": false,
				"msg":    "Invalid Data",
			})
        }

		claims := &jwtCustomClaims{
			user.Email,
			true,
			jwt.StandardClaims{
				ExpiresAt: time.Now().Add(time.Hour * 3).Unix(),
			},
        }

		token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)

		t, err := token.SignedString([]byte("secret"))
		if err != nil {
			return c.JSON(http.StatusBadRequest, echo.Map{
				"status": false,
				"msg":    "Server ERROR",
			})
		}

		cookie := new(http.Cookie)
		cookie.Name = "x-token"
		cookie.Value = t
		cookie.Expires = time.Now().Add(3 * time.Hour)
		c.SetCookie(cookie)
		return c.JSON(http.StatusOK, echo.Map{
			"status": true,
			"msg":    "Login Successful",
		})
	})
	r.GET("/posts", func(c echo.Context) error {
		user := c.Get("user").(*jwt.Token)
		claims := user.Claims.(*jwtCustomClaims)
		name := claims.Name
		return c.String(http.StatusOK, "Welcome "+name+"!")
	})
	e.Logger.Fatal(e.Start("localhost:5000"))
}
```

Group 메소드로 일종의 경로 그룹을 지정해줍니다.

http://localhost:5000/auth/... 주소에 접속하기 위해서는 JWT 토큰이 필요합니다.

JWT 설정

```Go
config := middleware.JWTConfig{
    Claims:        &jwtCustomClaims{},
    SigningKey:    []byte("secret"),
    SigningMethod: "HS256",
    TokenLookup:   "cookie:x-token",
}
```

TokenLookup을 통해 쿠키가 어디에 저장되어야 할지 명시해줍니다.

Go JWT 라이브러리에서 기본적으로 제공하는 Claims에 몇가지 속성을 추가하여 Custom Claim을 구성했습니다.

Custom Claim 구성

```Go
type jwtCustomClaims struct {
	Name  string `json:"name"`
	Admin bool   `json:"admin"`
	jwt.StandardClaims
}
```

JWT 기본 Claim에 Name, Admin 속성을 추가했습니다.

로그인 과정

```Go
type loginForm struct {
	Email    string `json:"email"`
	Password string `json:"password"`
}
...
e.POST("/login", func(c echo.Context) error {
    user := new(loginForm)
    if err := c.Bind(user); err != nil {
        return c.JSON(http.StatusBadRequest, echo.Map{
            "status": false,
            "msg":    "Invalid Data",
        })
    }
    if user.Email != "이메일" || user.Password != "비밀번호" {
        return c.JSON(http.StatusBadRequest, echo.Map{
            "status": false,
            "msg":    "Invalid Data",
        })
    }

    claims := &jwtCustomClaims{
        user.Email,
        true,
        jwt.StandardClaims{
            ExpiresAt: time.Now().Add(time.Hour * 3).Unix(),
        },
    }

    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)

    t, err := token.SignedString([]byte("secret"))
    if err != nil {
        return c.JSON(http.StatusBadRequest, echo.Map{
            "status": false,
            "msg":    "Server ERROR",
        })
    }

    cookie := new(http.Cookie)
    cookie.Name = "x-token"
    cookie.Value = t
    cookie.Expires = time.Now().Add(3 * time.Hour)
    c.SetCookie(cookie)
    return c.JSON(http.StatusOK, echo.Map{
        "status": true,
        "msg":    "Login Successful",
    })
})
```

사용자가 프론트엔드에서 JSON 형태의 정보를 가지고 POST 요청을 보냅니다.  
구조체 loginForm의 변수 user를 생성한 뒤 Bind 메소드를 사용하여 Body 데이터를 파싱할 수 있습니다.

Bind 하는데 문제가 생겼거나, 서버에서 설정해 둔 이메일과 비밀번호가 일치하지 않으면 오류 JSON 데이터를 클라이언트에게 전송해줍니다.

오류가 발생하지 않았다면 JWT Claim을 구성합니다. Name에 이메일, Admin에 true을 넣어주고 기본 Claim에는 3시간 후 토큰이 만료되도록 설정했습니다.
그리고 JWT 암호키를 사용해 암호화된 토큰 문자열을 발급합니다. 이 때에도 오류가 발생하면 오류 메시지를 클라이언트에 출력합니다.

JWT 토큰을 Cookie에 넣어주어야 합니다.
쿠키 이름은 x-token, 값을 생성된 토큰 값으로 설정했습니다. JWT 토큰과 똑같이 3시간 이후 종료되도록 설정했습니다.

SetCookie를 이용해서 쿠키를 HTTP 응답 헤더 메시지에 붙여넣을 수 있습니다.
헤더 메시지는 Set-Cookie : '우리가 생성한 쿠키'입니다.

이 모든 과정에 문제가 없었다면 Login Successful 메시지를 클라이언트에 보내줍니다.
이로써 로그인이 문제없이 수행되었음을 알 수 있습니다.

제한된 경로 접근

```Go
r.GET("/posts", func(c echo.Context) error {
    user := c.Get("user").(*jwt.Token)
    claims := user.Claims.(*jwtCustomClaims)
    name := claims.Name
    return c.String(http.StatusOK, "Welcome "+name+"!")
})
```

auth 그룹의 posts 경로로 접속합니다.

주소 : localhost:5000/auth/posts

서버에서는 컨텍스트에서 Get 메소드를 이용하여 user 값을 읽어냅니다.
앞서 JWT 설정에 TokenLookup에 cookie:x-token이 설정되어있으므로, x-token 쿠키 값을 읽게 될 것입니다.
(설명이 다소 추상적이라서 보다 자세한 과정은 덧붙이겠습니다.)

유효한 JWT 토큰을 파싱하게 되면 속성 Name을 읽어들입니다.
읽는데 성공하면 클라이언트에 Name 값을 전송할 수 있습니다.
