+++
title = "Prisma With Golang"
date = "2021-01-14T22:35:05+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["Prisma", "ORM", "Golang", "Database", "Relation", "Schema", "Echo", "Query"]
keywords = ["Prisma", "ORM", "Golang", "Database", "Relation", "Schema", "Echo", "Query"]
description = ""
showFullContent = false
+++

# Prisma

데이터베이스 내 데이터를 쉽게 처리할 수 있도록 도와주는 라이브러리이다. Javascript, Typescript에서 사용이 가능하고, Go에서도 Early Access 버전을 사용할 수 있다.

프로젝트 폴더 내 schema.prisma에 정의된 스키마 객체를 정의한 후 데이터베이스에 적용할 수 있다. 반대로 데이터베이스에 구성된 데이터 모델을 schema.prisma 파일에 가져와서 프로그래밍 언어에서 사용할 수도 있다.

프로그래밍 언어는 프리즈마 클라이언트를 통해 데이터베이스와 상호작용할 수 있다.

프리즈마 클라이언트가 실행되면 내부의 프리즈마 엔진과 연결된다. 프리즈마 엔진은 데이터베이스와 직접적으로 연결되어있고, Javascript, Typescript, Go 코드를 바이너리 데이터로 변환하여 데이터베이스에서 실행하도록 넘겨준다.

### Go에서 Prisma를 사용한 부분

웹 백엔드 API 서버에서 데이터 조작을 하는데 사용했다. 프리즈마는 타입 린트 기능이 제공되어서 코드를 작성할 때 코드 상의 오류가 발생하지 않도록 도와준다.

우선 새로운 Go 프로젝트를 생성한다.

```powershell
go mod init "원하는 패키지 이름"
```

Go 프로젝트에서는 다음의 명령어로 프리즈마를 설치할 수 있다.

```powershell
go get github.com/prisma/prisma-client-go
```

프리즈마가 정상적으로 설치되면 schema.prisma 파일을 생성해준다. 데이터베이스는 PostgreSQL을 사용했다.

```prisma
generator db {
  provider = "go run github.com/prisma/prisma-client-go"
}

datasource db {
  provider = "postgresql"
  url      = "postgresql://사용자계정:비밀번호@DB주소:포트/데이터베이스이름?parseTime=True"
}

model Post {
  id        Int      @id @default(autoincrement())
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  title     String
  published Boolean
  content   String?
  User      User?    @relation(fields: [userId], references: [id])
  userId    Int?
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  username  String   @unique
  password  String
  createdAt DateTime @default(now())
  verified  Boolean  @default(false)
  posts     Post[]
}
```

Post 테이블의 userId 컬럼은 User 테이블의 id 칼럼을 참조하여 User의 데이터에 접근할 수 있다.

자바스크립트나 타입스크립트는 prisma ... 형태로 명령을 실행했지만 Go에서는 go run github.com/prisma/prisma-client-go 키워드를 사용해야한다.

완성된 schema.prisma 파일을 데이터베이스에 마이그레이션 하려면 다음과 같은 명령어를 실행시키면 된다.

```powershell
go run github.com/prisma/prisma-client-go db push --preview-feature
```

데이터 손실의 우려가 있어 명령이 실행되지 않으면 --preview-feature --force를 사용해서 마이그레이션을 수행할 수 있다. 단 데이터베이스 내 일부분의 데이터가 사라질 수 있다.

그 다음 Go 코드와 상호작용할 프리즈마 클라이언트를 생성한다.

```powershell
go run github.com/prisma/prisma-client-go generate
```

이제 프리즈마 클라이언트를 사용해서 데이터베이스에 접근할 수 있다.

서버 내 클라이언트가 중복으로 생성되는 것을 방지하기 위해서 웹 서버 실행 시 최초 한 개의 클라이언트를 생성한 다음 데이터베이스와 연결이 필요하면 그 클라이언트만 가져와서 사용하도록 했다.

서버 프레임워크는 Echo를 사용했다.

```Go
// models/conn.go
var conn *db.PrismaClient = db.NewClient()
var err error = conn.Connect()

// DBPool gives one db connection pool.
func DBPool() (*db.PrismaClient, error) {
	return conn, err
}

// main.go
conn, err := model.DBPool()
if err != nil {
    log.Println(err)
    return
}
defer conn.Disconnect()
ctx := context.Background()

app := echo.New()
```

웹 서버가 종료되기 직전에 defer 키워드를 사용해서 클라이언트 연결을 종료하도록 했다.

Go에서 데이터베이스에서 데이터를 조작하려면 Context 객체가 필요하므로 이 때에도 웹 서버 실행 시 최초 1개만 생성해서 데이터베이스 연결이 필요할 때마다 매개변수로 넘겨주었다.

백엔드 API에서 GET 메소드를 사용해 데이터를 가져오는 코드이다. 파라미터 id는 분할된 데이터의 페이지를 의미한다.

```Go
app.GET("/api/post/:id", func(c echo.Context) error {
    id, err := strconv.Atoi(c.Param("id"))
    if err != nil || id <= 0 {
        log.Println(err)
        return c.String(http.StatusBadRequest, "Bad Request")
    }
    skip := (id - 1) * bunch
    posts, err := conn.Post.FindMany().Skip(skip).Take(bunch).With(db.Post.User.Fetch()).Exec(ctx)

    resp := []map[string]interface{}{}
    for idx := 0; idx < len(posts); idx++ {
        user, isOK := posts[idx].User()
        var username string
        if !isOK {
            username = "undefined"
        } else {
            username = user.Username
        }
        resp = append(resp, map[string]interface{}{
            "post":     posts[idx].InternalPost,
            "username": username,
        })

    }
    if err != nil {
        log.Println(err)
        return c.String(http.StatusBadRequest, "Bad Request")
    }
    return c.JSON(http.StatusOK, echo.Map{
        "status":   true,
        "len":      len(resp),
        "response": resp,
    })
})
```

```Go
posts, err := conn.Post.FindMany().Skip(skip).Take(bunch).With(db.Post.User.Fetch()).Exec(ctx)
```

코드에 보이지 않는 bunch는 한 페이지 당 가져올 레코드 개수이다.

- conn.Post.FindMany() 을 사용해서 전체 데이터를 가져오도록 한다.
- 그 다음 Skip 메소드로 몇 개의 데이터를 건너뛸지 정의한다. (페이지번호 - 1) \* 한 페이지 당 레코드 개수만큼 데이터를 건너뛴다.
- Take 메소드로 몇 개의 데이터를 받아올 지 정의한다.
- With 메소드를 사용하여, Post 레코드와 매핑된 User 레코드까지 전부 가져올 수 있다.
- Exec 메소드에 ctx, 컨텍스트 객체를 넘겨주어 데이터베이스 쿼리를 실행시킨다.

```Go
user, isOK := posts[idx].User()
var username string
if !isOK {
    username = "undefined"
} else {
    username = user.Username
}
resp = append(resp, map[string]interface{}{
    "post":     posts[idx].InternalPost,
    "username": username,
})
```

With 메소드를 통해 Post 데이터만 아니라 연결된 User 데이터까지 전부 가져올 수 있게 되었다. User 테이블에는 사용자의 비밀번호가 저장되어있어서 클라이언트에 그대로 넘겨줄 수 없다.

이 때 Post 모델에서 User 메소드를 사용하여 user 데이터만 따로 가져올 수 있다.
그리고 InternalPost 속성을 가져오도록 하는데, 이 속성을 사용하면 따로 연결된 데이터만 아닌 내부 데이터, Post 데이터만 가져올 수 있다.

레코드 한 개를 삭제하는 코드

```Go
app.DELETE("/api/post/id/:id", func(c echo.Context) error {
    payload := c.Get("user").(*jwt.Token)
    claims := payload.Claims.(*jwtCustomClaims)
    userID := claims.ID

    id := c.Param("id")
    postID, err := strconv.Atoi(id)
    if err != nil {
        log.Println(err)
        return c.String(http.StatusBadRequest, "Bad Request")
    }

    _, err = conn.Post.FindMany(
        db.Post.ID.Equals(postID),
        db.Post.UserID.Equals(userID),
    ).Delete().Exec(ctx)

    if err != nil {
        log.Println(err)
        return c.String(http.StatusBadRequest, "Bad Request")
    }
    return c.JSON(http.StatusOK, echo.Map{
        "status": true,
        "msg":    fmt.Sprintf("The post %d was removed.", postID),
    })
}, auth /* 사용자 인증 미들웨어 */ )
```

```Go
_, err = conn.Post.FindMany(
    db.Post.ID.Equals(postID),
    db.Post.UserID.Equals(userID),
).Delete().Exec(ctx)
```

각각의 Post 레코드는 User 테이블과 연결되는 userId 칼럼이 있다. userId는 User 레코드의 id를 의미한다.

Post를 작성한 사용자의 ID 값, userId와 삭제 요청을 한 사용자의 ID 값, userID가 일치하면 삭제 명령을 수행한다.

userID는 JWT 토큰을 복호화해서 얻을 수 있었다.

FindMany 메소드를 사용하면 매개변수에 주어진 조건에 맞는 레코드를 가져온다.

Post의 ID는 postID와 일치해야 하고, Post의 UserID은 userID와 일치하도록 한다.

두 속성 모두 일치하면 Delete 메소드를 실행하여 데이터를 삭제한다.

### 후기

Go에서의 Prisma가 아직 Early Access 단계이지만 개인 프로젝트를 수행하는데에는 문제가 없었다. Go ORM 중 제일 생산성이 좋다고 생각한다.
