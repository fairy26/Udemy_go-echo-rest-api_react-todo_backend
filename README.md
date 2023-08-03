# 環境構築

## インストール

```bash
$ brew install --cask pgadmin4
$ brew install --cask postman
$ brew install go

$ go version
go version go1.20.6 darwin/arm64
```

## 環境構築

```bash
$ cd ~/Documents && mkdir go-rest-api && cd go-rest-api
$ git init
$ go mod init go-rest-api

$ touch docker-compose.yml
```

-   [GitHub レポジトリ](https://github.com/GomaGoma676/echo-rest-api) からコピペ

-   Docker のアプリを立ち上げる

```bash
$ docker compose up -d

$ docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS                    NAMES
303868da336f   postgres:15.1-alpine   "docker-entrypoint.s…"   34 seconds ago   Up 33 seconds   0.0.0.0:5434->5432/tcp   go-rest-api-dev-postgres-1
```

```bash
touch .env && code .env
```

-   GitHub から `.env.example` をコピペ

-   `SECRET={}` は jwt 生成に使うパスワード（任意）

```bash
$ mkdir model && touch model/user.go model/task.go
$ mkdir db && touch db/db.go
$ mkdir migrate && touch migrate/migrate.go
```

-   実装において、各パッケージは import に書いたら `go mod tidy` でインストール

```bash
# migration
$ GO_ENV=dev go run ./migrate/migrate.go
Connected
Successfully Migrated
```

-   `GO_ENV`の値は `.env` を読み込む前なので、コマンドで渡す必要がある

-   pgAdmin で出来ているか確認

    -   Add New Server
        -   General
            -   name: "DB" (なんでもいい)
        -   Connection: 以下を `.env` で設定している値に
            -   Host name/address
            -   Port
            -   Username
            -   Password
            -   Save password?: true
    -   DB > Databases > Schemas > Tables
        -   それぞれを右クリック > View/Edit Data > All Rows
        -   カラムが正しいことを確認する

-   User に関する機能を実装する

```bash
$ mkdir repository && touch repository/user_repository.go
$ mkdir usecase && touch usecase/user_usecase.go
$ mkdir controller && touch controller/user_controller.go
$ mkdir router && touch router/router.go
$ touch main.go
```

```bash
# 起動
$ GO_ENV=dev go run ./main.go
```

-   postman で挙動確認
    -   `+` で API を叩くタブを追加
        -   POST `http://localhost:8080/signup`
            -   Body > raw > JSON
                ```JSON
                {
                    "email": "fairy1@test.com",
                    "password": "fairy4216"
                }
                ```
            -   SEND
            -   pgAdmin の users にレコードが追加されていることを確認する
    -   同じ request body で `/login` `/logout` も叩く
        -   Cookie が設定（消去）されていることを確認する

```bash
$ touch repository/task_repository.go
$ touch usecase/task_usecase.go
$ touch controller/task_controller.go

$ GO_ENV=dev go run ./main.go
```

-   postman で挙動確認
    -   GET `http://localhost:8080/tasks/`
        -   401 Unauthorized
    -   POST `http://localhost:8080/login`
    -   GET `http://localhost:8080/tasks/`
    -   POST `http://localhost:8080/tasks/`
        ```JSON
        {
            "title": "hello"
        }
        ```
        -   2 回
    -   GET `http://localhost:8080/tasks/2`
    -   PUT `http://localhost:8080/tasks/2`
        ```JSON
        {
            "title": "updated"
        }
        ```
    -   DELETE `http://localhost:8080/tasks/2`
    -   POST `http://localhost:8080/logout`
    -   DELETE `http://localhost:8080/tasks/1`
        -   401 Unauthorized

```bash
$ mkdir validator && touch validator/user_validator.go validator/task_validator.go
```

-   postman で挙動確認

    -   login 周りは validation が効いていることを確認

-   CORS/CSRF の middleware を追加
-   postman で挙動確認
    -   GET `/csrf` で token を取得
    -   POST `/login`, POST `/tasks` の際に以下を Header に追加
        ```JSON
        {
            "X-CSRF-TOKEN": "WVdqEsL5wqa1BnhkvGMdSUbTx9Xg9QXD" // e.g.
        }
        ```
