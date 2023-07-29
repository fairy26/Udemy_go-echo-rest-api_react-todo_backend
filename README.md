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
