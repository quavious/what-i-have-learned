+++
title = "Go Echo 서버 Docker Compose를 이용하여 실행 및 접속 확인"
date = "2021-01-28T01:09:41+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["Golang", "Echo", "Docker", "Dockerfile", "Compose", "yml"]
keywords = ["Golang", "Echo", "Docker", "Dockerfile", "Compose", "yml"]
description = ""
showFullContent = false
+++

# Dockerfile와 Docker-Compose를 사용하여 Go 서버 배포하기

Docker를 활용하여 Go 웹 어플리케이션을 실행시킨다.

우선 Windows 10에서 Docker Toolbox를 이용하여 우선 Docker 환경을 구성한다.

Docker 이미지에 Ubuntu를 먼저 설치한 후 그 이미지에 Go 서버 파일을 올린다. Go 파일을 컴파일하여 생성된 하나의 바이너리 파일을 실행시켜 사용자의 접속을 받는다.

### Go 서버 생성하기

파일 구조는 다음과 같다.

```
📂 golang-docker-tutorial
 📂 model
  🧾 post.go
 🧾 main.go
 🧾 Dockerfile
 🧾 docker-compose.yml
```

go mod init으로 패키지를 선언한다. Echo 프레임워크를 사용했다.

```powershell
go mod init "github.com/<github_username>/<repository_name>"
go get "github.com/labstack/echo/v4"
```

Go 서버 코드

```Go
// model/post.go
package model

import (
	"encoding/json"
	"log"
	"net/http"
)

// Post has one post data from jsonplaceholder
type Post struct {
	UserID int    `json:"userId"`
	ID     int    `json:"id"`
	Title  string `json:"title"`
	Body   string `json:"body"`
}

// FetchPost takes mock data from parameter
func (p *Post) FetchPost(id string) error {
	apiURL := "https://jsonplaceholder.typicode.com/posts/" + id
	resp, err := http.Get(apiURL)
	if err != nil {
		log.Println(err)
		return err
	}
	defer resp.Body.Close()

	json.NewDecoder(resp.Body).Decode(p)
	return nil
}
```

```Go
// main.go
package main

import (
	"log"
	"net/http"

	"github.com/labstack/echo/v4"
	"github.com/labstack/echo/v4/middleware"
	"github.com/quavious/golang-docker-tutorial/model"
)

func main() {
	app := echo.New()

	app.Use(middleware.Logger())
	app.Use(middleware.Recover())

	app.GET("/posts/id/:id", func(c echo.Context) error {
		param := c.Param("id")
		posts := new(model.Post)
		err := posts.FetchPost(param)
		if err != nil {
			return c.String(http.StatusBadRequest, "Server ERROR")
		}
		log.Println(param)

		return c.JSON(http.StatusOK, echo.Map{
			"status": true,
			"posts":  posts,
		})
	})

	app.GET("/", func(c echo.Context) error {
		return c.String(http.StatusOK, "Hello World")
	})
	app.Logger.Fatal(app.Start(":5000"))
}
```

model 폴더에는 Post 구조체가 선언되어있다. Post 구조체에 FetchAPI 메소드로 jsonplaceholder API에 요청해서 받은 데이터를 보관한다.

"/" 주소에 접근하면 Hello World 문자열을 받을 수 있다.

"/post/id/:id" 주소에 접근하면 :id 값에 따라 jsonplaceholder API를 호출해서 JSON 형태로 화면에 출력한다.

go run main.go로 서버를 실행시키면 로컬호스트 5000번 포트에서 접속이 잘 된다.

이제 Dockerfile을 생성한다.

```Dockerfile
FROM ubuntu:latest
FROM golang as builder
RUN apt update && apt full-upgrade
RUN mkdir /go/src/app
ADD . /go/src/app
WORKDIR /go/src/app
RUN go install github.com/quavious/golang-docker-tutorial
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .
EXPOSE 5000
CMD [ "/go/src/app/main" ]
```

파일 내 명령어 의미

1. 우선 Docker 이미지에 Ubuntu OS 이미지를 받아서 설치한다. :latest 태그는 최신 버전을 사용하겠다는 의미이다.
2. 같은 이미지의 Go 이미지를 받아서 설치한다. Go 파일을 컴파일하기 위해 as builder를 더 작성해야한다.
3. apt update apt full-upgrade로 우분투 OS를 업데이트한다.
4. Go 이미지를 내 이미지에 올리면 파일 구조는 다음과 같다.

```
📂 go
 📂 bin
 📂 src
```

5. 소스 코드 파일을 담을 /go/src/app 디렉토리를 생성한다.
6. ADD 명령어로 로컬 PC 내 현재 위치에 있는 모든 파일을 도커 이미지에 복사한다.
7. 현재 실행되는 디렉토리를 /go/src/app 디렉토리로 설정한다.
8. go install로 go mod 파일에 명시된 Go 패키지를 설치한다.
9. go 소스 코드를 컴파일한다. C 바이너리가 설치되어있지 않으므로 CGO_ENABLED를 0으로 설정하여 cgo의 실행을 막는다. go build -o main .로 go 모든 코드를 컴파일하여 main 파일을 생성한다.
10. 5000번 포트를 연다
11. main 파일을 실행하여 서버를 연다.

Dockerfile 만으로 도커 이미지를 실행할 수 있지만 Docker Compose로 Docker 이미지 실행 시 여러가지 환경변수를 설정해 줄 수 있다.

```docker-compose.yml
version: "3"
services:
  api:
    build:
      context: .
    working_dir: /go/src/app
    ports:
      - "5000:5000"
    volumes:
      - ./volumes
    restart: always
    environment:
```

1. services에 실행하려는 컨테이너의 이름을 api로 명세한다.
2. 서비스에 build로 무슨 이미지를 실행할지, 또는 타 폴더 내 도커 파일의 위치를 지정해 줄 수 있다.
3. working_dir로 현재 작업 디렉토리를 설정한다.
4. ports로 도커 내부 포트와 외부의 포트를 연결한다. 내부 및 외부 포트를 5000번으로 설정했다. 도커 내 컨테이너 간 서로 연결하려면 내부 포트를 사용하고, 외부에서 도커 컨테이너로 접속하려면 외부 포트로 접속해야한다.
5. volumes로 도커 컨테이너의 데이터를 저장할 수 있다.
6. environment로 도커 컨테이너를 실행할 때 환경 변수를 지정할 수 있다.
7. restarts로 도커 컨테이너가 특정 조건일 때 다시 시작하도록 할 수 있다.

다음과 같은 명령어로 docker-compose.yml 내 선언된 컨테이너를 실행할 수 있다. 다른 명령어 또한 사용할 수 있도록 데몬(daemon) 컨테이너로 실행한다.

```powershell
docker-compose up -d
```

도커 컨테이너를 종료할 때의 명령어는 다음과 같다.

```
docker-compose down
```

docker-compose로 실행한 컨테이너와 컨테이너 간 네트워크를 종료할 수 있다.

이렇게 실행한 컨테이너 내 서버는 웹 브라우저에서 localhost로 접속이 되지 않는다.

docker-machine 명령어로 현재 도커 컨테이너의 ip를 파악한다.

```powershell
docker-machine ls
```

```
NAME      ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER      ERRORS
default   *        virtualbox   Running   tcp://<DOCKER_URL>:2376           v19.03.12
```

DOCKER_URL에 생성한 컨테이너의 IP를 볼 수 있다.

도커 외부 포트를 5000번으로 개방했으므로 URL에 있는 IP 주소에 5000번 포트로 접속할 수 있다.

http://DOCKER_URL:5000으로 접속하면 Hello World!가 뜬다.

http://DOCKER_URL:5000/posts/id/1로 접속하면 다음과 같은 JSON 데이터가 뜬다.

```JSON
{
    "status": true,
    "posts": {
        "userId": 1,
        "id": 1,
        "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
        "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"
    }
}
```
