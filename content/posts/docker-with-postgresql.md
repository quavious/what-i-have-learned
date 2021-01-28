+++
title = "Docker로 Go 서버에 PostgreSQL 데이터베이스 연결하기"
date = "2021-01-29T00:37:52+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["PostgreSQL", "Entgo", "Docker", "Compose", "Volume", "Dockerfile", "Network", "Port", "Echo"]
keywords = ["PostgreSQL", "Entgo", "Docker", "Compose", "Volume", "Dockerfile", "Network", "Port", "Echo"]
description = ""
showFullContent = false
+++

# Go 서버 컨테이너와 PostgreSQL 데이터베이스 컨테이너 연결

Docker로 올린 Go 서버 컨테이너에 PostgreSQL 데이터베이스 컨테이너를 생성해서 연결한다.

Docker Compose를 활용하여 같은 네트워크에 있는 여러 컨테이너를 한번에 띄울 수 있다.

Go 소스 코드 상에서 데이터베이스를 조작하기 위해 entgo ORM 라이브러리를 사용했다. entgo를 사용하면 Go에서 쉽게 데이터베이스 스키마 정의, 데이터 조작을 할 수 있다.

- Go 프로젝트에서 entgo 설치하는 방법

```powershell
go get github.com/facebook/ent/cmd/ent
```

지난 번에 만들었던 golang-docker-tutorial을 그대로 사용한다.

사용자 정보를 데이터베이스에 저장하고 싶다. User 모델을 생성하는 명령어는 다음과 같다

```
ent init User
```

이렇게 되면 프로젝트 폴더 내 ent/schema 폴더에 user.go 파일이 생성되어있다. 여기서 User 스키마의 정의를 할 수 있다.

```Go
// ./ent/schema/user.go
package schema

import (
	"github.com/facebook/ent"
	"github.com/facebook/ent/schema/field"
)

// User holds the schema definition for the User entity.
type User struct {
	ent.Schema
}

// Fields of the User.
func (User) Fields() []ent.Field {
	return []ent.Field{
		field.Int("id").Unique(),
		field.Int("age").Positive(),
		field.String("name").NotEmpty(),
		field.String("job").NotEmpty(),
		field.String("address").NotEmpty(),
		field.String("introduce"),
	}
}

// Edges of the User.
func (User) Edges() []ent.Edge {
	return nil
}
```

스키마의 칼럼만 추가해주었다. 추가된 칼럼은 id, age, name, job, address, introduce이다.

각각 id, 나이, 이름, 직업, 주소, 인사말을 담고 있다.

필드에 메소드를 호출함으로써 이 칼럼 값의 조건을 정해줄 수 있다.

- Unique : 유일한 값
- Positive : 양의 정수 값
- NotEmpty : 문자열의 길이가 1 이상

그 다음 실제로 코드 상에서 User 스키마를 사용할 수 있도록 다음과 같은 명령어를 입력한다.

```powershell
go generate ./ent
```

이렇게 되면 다음과 같은 파일들이 생겨난다.

```
ent
├── client.go
├── config.go
├── context.go
├── ent.go
├── migrate
│   ├── migrate.go
│   └── schema.go
├── predicate
│   └── predicate.go
├── schema
│   └── user.go
├── tx.go
├── user
│   ├── user.go
│   └── where.go
├── user.go
├── user_create.go
├── user_delete.go
├── user_query.go
└── user_update.go
```

그 다음 서버 소스 코드에서 데이터베이스 연결을 해 주고, 데이터베이스 연결이 성공하면 데이터 생성 및 열람을 할 수 있도록 라우팅을 정의해주었다.

앞서 .env 파일 내 환경변수 값을 불러오기 위해서 godotenv 패키지도 설치한다. .env 파일에는 데이터베이스 이름, 비밀번호가 담겨져 있다.

godotenv.Load 메소드를 사용하여 환경변수 값을 불러오고 os.Getenv로 값을 사용할 수 있다.

main.go 소스 코드

```
go get github.com/joho/godotenv
```

```.env
DB_NAME=NAME
DB_PASSWORD=PASSWORD
```

```Go
package main

import (
	"context"
	"log"
	"net/http"
	"os"

	"github.com/joho/godotenv"
	"github.com/labstack/echo/v4"
	"github.com/labstack/echo/v4/middleware"
	_ "github.com/lib/pq"
	"github.com/quavious/golang-docker-tutorial/ent"
	"github.com/quavious/golang-docker-tutorial/model"
)

func main() {
	godotenv.Load()
	dbPassword := os.Getenv("DB_PASSWORD")
	dbName := os.Getenv("DB_NAME")
	conn, err := ent.Open("postgres", "host=db port=5432 user=postgres dbname="+dbName+" password="+dbPassword+" sslmode=disable")
	ctx := context.Background()
	if err != nil {
		log.Println(err)
		return
	}
	defer conn.Close()

	if err := conn.Schema.Create(ctx); err != nil {
		log.Println(err)
		return
	}
	log.Println("DB Schema was created")

	app := echo.New()
	app.Use(middleware.Logger())
	app.Use(middleware.Recover())

	app.POST("/user", func(c echo.Context) error {
		u := new(model.User)
		if err := c.Bind(u); err != nil {
			log.Println(err)
			return c.String(http.StatusBadRequest, "Server ERROR")
		}
		_, err := conn.User.Create().SetName(u.Name).SetJob(u.Job).SetAddress(u.Address).SetIntroduce(u.Introduce).SetAge(u.Age).Save(ctx)
		if err != nil {
			log.Println(err)
			return c.String(http.StatusBadRequest, "Server ERROR")
		}
		return c.String(http.StatusOK, "User Created!")
	})

	app.GET("/user", func(c echo.Context) error {
		users, err := conn.User.Query().All(ctx)
		if err != nil {
			log.Println(err)
			return c.String(http.StatusBadRequest, "Server ERROR")
		}
		return c.JSON(http.StatusOK, echo.Map{
			"status": true,
			"users":  users,
		})
	})

	app.GET("/", func(c echo.Context) error {
		return c.String(http.StatusOK, "Hello World")
	})
	app.Logger.Fatal(app.Start(":5000"))
}
```

데이터베이스 연결 및 스키마 생성

```Go
conn, err := ent.Open("postgres", "host=db port=5432 user=postgres dbname="+dbName+" password="+dbPassword+" sslmode=disable")
ctx := context.Background()
if err != nil {
    log.Println(err)
    return
}
defer conn.Close()

if err := conn.Schema.Create(ctx); err != nil {
    log.Println(err)
    return
}
log.Println("DB Schema was created")
```

Host가 localhost, IP 주소가 아닌 db로 되어있다. 이유는 Docker Compose yml 파일에서 추후에 확인할 수 있다.

데이터베이스 연결에 성공하면 Schema.Create 메소드로 ./ent/schema 에서 정의했던 스키마를 데이터베이스에 적용할 수 있다.

데이터 연산에 사용할 컨텍스트, ctx도 하나 선언해준다.

로컬 환경에서 데이터베이스를 연결할 때 보안문제 때문에 연결이 되지 않는 문제가 있어서 sslmode를 disable로 설정했다.

HTTP Post 요청을 받았을 때 데이터를 구조체에 바인딩하기 위해 model 폴더에 User 구조체를 선언해주었다.

```Go
package model

// User has several information of one user.
type User struct {
	Name      string `json:"name"`
	Age       int    `json:"age"`
	Job       string `json:"job"`
	Address   string `json:"address"`
	Introduce string `json:"introduce"`
}
```

/user 주소로 POST 요청을 보내면 user 데이터를 하나 생성한다. /user 주소에 접속하면 데이터베이스에 저장된 user 데이터를 볼 수 있다.

```Go
app.POST("/user", func(c echo.Context) error {
    u := new(model.User)
    if err := c.Bind(u); err != nil {
        log.Println(err)
        return c.String(http.StatusBadRequest, "Server ERROR")
    }
    _, err := conn.User.Create().SetName(u.Name).SetJob(u.Job).SetAddress(u.Address).SetIntroduce(u.Introduce).SetAge(u.Age).Save(ctx)
    if err != nil {
        log.Println(err)
        return c.String(http.StatusBadRequest, "Server ERROR")
    }
    return c.String(http.StatusOK, "User Created!")
})

app.GET("/user", func(c echo.Context) error {
    users, err := conn.User.Query().All(ctx)
    if err != nil {
        log.Println(err)
        return c.String(http.StatusBadRequest, "Server ERROR")
    }
    return c.JSON(http.StatusOK, echo.Map{
        "status": true,
        "users":  users,
    })
})
```

docker-compose.yml에서 데이터베이스 컨테이너까지 만들기 위해 데이터베이스 설정을 해준다.

```yml
version: "3"
services:
  api:
    build:
      context: .
    working_dir: /go/src/app
    ports:
      - "5000:5000"
    depends_on:
      - db
    links:
      - db
    restart: unless-stopped
    environment:
      - DB_NAME=<NAME>
      - DB_PASSWORD=<PASSWORD>
    env_file:
      - .env
    networks: ["golang-docker-network"]
  db:
    image: postgres
    # volumes:
    #   - pgdata:/var/lib/postgresql/data
    #   - pgconf:/etc/postgresql
    #   - pglog:/var/log/postgresql
    ports:
      - "5001:5432"
    restart: unless-stopped
    environment:
      - POSTGRES_DB=<NAME>
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=<PASSWORD>
      - POSTGRES_INITDB_ARGS=--encoding=UTF-8
    networks: ["golang-docker-network"]

# volumes:
#   pgdata:
#     driver: local
#   pgconf:
#     driver: local
#   pglog:
#     driver: local

networks: { golang-docker-network: {} }
```

하단에 있는 networks에 golang-docker-network를 따로 선언해주고 api, db 컨테이너 내 networks 속성에 golang-docker-network를 사용하도록 했다.

네트워크를 따로 설정하지 않으면 Docker의 기본 네트워크를 사용하는데 불필요한 컨테이너까지 연결되어, 서버와 데이터베이스가 정상적으로 연결되지 못하는 문제점이 있었다.

db 컨테이너 설정

```yml
db:
  image: postgres
  # volumes:
  #   - pgdata:/var/lib/postgresql/data
  #   - pgconf:/etc/postgresql
  #   - pglog:/var/log/postgresql
  ports:
    - "5001:5432"
  restart: unless-stopped
  environment:
    - POSTGRES_DB=<NAME>
    - POSTGRES_USER=postgres
    - POSTGRES_PASSWORD=<PASSWORD>
    - POSTGRES_INITDB_ARGS=--encoding=UTF-8
  networks: ["golang-docker-network"]
# volumes:
#   pgdata:
#     driver: local
#   pgconf:
#     driver: local
#   pglog:
#     driver: local
```

ports 속성에는 5001:5432 포트가 매핑되어있는데, 네트워크 외부에서 접속하려면 5001, 네트워크 내 컨테이너끼리 연결하기 위해서는 5432 포트를 사용해야한다.

로컬 PC 내 데이터베이스의 포트와 겹치는 것을 방지하기 위함이다.

주석 처리된 부분은 볼륨이다. 볼륨은 데이터베이스 내 데이터를 영구적으로 저장하기 위해 사용한다.

연습하는 과정에서 docker-compose up & down 명령어를 계속 사용하게 되는데 기존 볼륨이 남아있으면 yml 파일을 수정해도 적용이 안 되는 경우가 있었다.

테스트 할 때는 주석처리해주고, 실제 배포할 때에는 주석 처리를 해제하는 방법을 사용할 수 있다.

사용자 권한 때문에 리눅스 내 데이터베이스 데이터에 접근이 되지 않는 문제가 있었다.

각 데이터를 저장하는 볼륨의 driver를 local로 설정해서 권한 문제를 해결했다.

environment 속성에서 데이터베이스의 이름, 사용자, 비밀번호 등을 설정할 수 있다.

restart 속성에서 언제 컨테이너가 재시작할지 설정할 수 있다.

컨테이너 이름이 db로 되어있다.

```yml
db:
  # ...
```

Go 서버 컨테이너에서 데이터베이스 컨테이너에 접근하기 위해서는 DB HOST 값을 디비 컨테이너 값으로 설정해야 한다.

그래서

```Go
conn, err := ent.Open("postgres", "host=db port=5432 user=postgres dbname="+dbName+" password="+dbPassword+" sslmode=disable")
```

host 값이 db로 설정되었다.

api 컨테이너가 db 컨테이너가 완전히 켜진 후에 작동하도록 바꿔준다.

```yml
api:
  build:
    context: .
  working_dir: /go/src/app
  ports:
    - "5000:5000"
  depends_on:
    - db
  links:
    - db
  restart: unless-stopped
  environment:
    - DB_NAME=<NAME>
    - DB_PASSWORD=<PASSWORD>
  env_file:
    - .env
  networks: ["golang-docker-network"]
```

networks 속성에 같은 네트워크 값을 설정해주고 depends_on, links에 db 컨테이너 이름을 작성한다. db 컨테이너가 먼저 실행되면 api 컨테이너가 그 이후에 작동되도록 하기 위함이다.

설정이 끝났으면 docker images ls, docker volume ls 명령어로 중복된 값의 이미지나 볼륨이 있는지 확인한다. 있다면 rm 명령어로 지워준다.

```
docker rmi <IMAGE_NAME>
docker volume rm <VOLUME_NAME>
```

그 다음 docker-compose up으로 yml 파일에 정의한 컨테이너를 모두 실행한다.

docker-machine ls 명령어로 컨테이너의 IP 주소를 확인하여 5000 포트로 접속할 수 있다.

/user에 POST 요청을 보내서 값을 저장하거나 GET 요청으로 user 데이터를 볼 수 있다.

POST 요청

```JSON
{
    "name": "Kim",
    "age": 23,
    "job": "Developer",
    "address" : "Seoul",
    "introduce": "I am fool stack developer 😥"
}
```

GET 요청

```JSON
{
    "status": true,
    "users" : {
        "name": "Kim",
        "age": 23,
        "job": "Developer",
        "address" : "Seoul",
        "introduce": "I am fool stack developer 😥"
    }
}
```
