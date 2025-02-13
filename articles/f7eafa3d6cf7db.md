---
title: "GoでBunRouterを使ってAPIサーバーを立てる。"
emoji: "🐙"
type: "tech"
topics:
  - "go"
  - "tips"
published: true
published_at: "2023-09-06 14:03"
---

BunRouter という、シンプルな HTTP router のライブラリを見つけたので使ってみました。

# BunRouter とは

BunRouter は Echo や Gin といったフレームワークと同じように Web サーバーを構築する事ができるライブラリです。
僕は普段 Echo を使用しているのですが、BunRouter は Echo などとは、異なり最低限の機能しか提供していないですがシンプルで使いやすいライブラリです。
ちなみに、制作者さんの同じライブラリに ORM の Bun などもあります。

https://bunrouter.uptrace.dev/

# インストール

```bash
go get github.com/uptrace/bunrouter
```

# 使い方

```go:sample.go
package mai
import (
	"fmt"
	"log/slog"
	"net/http"

	"github.com/uptrace/bunrouter"
	"github.com/uptrace/bunrouter/extra/reqlog"
)

func main() {
	router := bunrouter.New(
		bunrouter.Use(reqlog.NewMiddleware()),
	)

	router.GET("/", func(w http.ResponseWriter, req bunrouter.Request) error {
		w.Write([]byte("Hello World\n"))
		return nil
	})

	router.WithGroup("/api", func(g *bunrouter.Group) {
		g.WithGroup("/user", func(g *bunrouter.Group) {
			g.GET("/id", func(w http.ResponseWriter, req bunrouter.Request) error {
				w.Write([]byte("ID : 1111\n"))
				return nil
			})
			g.GET("/name", func(w http.ResponseWriter, req bunrouter.Request) error {
				name := req.URL.Query().Get("name")
				w.Write([]byte(fmt.Sprintf("name is %s\n", name)))
				return nil
			})
		})

		g.GET("/list", func(w http.ResponseWriter, req bunrouter.Request) error {
			w.Write([]byte("API list\n1. /user/id\n2. /user/name parm: name\n"))
			return nil
		})
	})

	if err := http.ListenAndServe(":8080", router); err != nil {
		slog.Error(err.Error())
	}
}
```

BunRouter の基本手な使い方は Echo などの他のライブリと同じような感じです。
`BunRouter.New`で bunrouter の構造体を作成後、`GET()`や`POST()`などを呼び出すことでエンドポイントの作成を行うことができます。
また、`WithGroup()`を呼び出すことで、URL のグルーピングも可能です。上のサンプルコードだと`/user`・`/name`のエンドポイントが`/api`以下に作成されます。
今回のサンプルコードでは、簡単な文字列を送信しているだけですが JSON で送信することも可能なので API を作成するときにも困ることが少ないと思います。
それ以外にもパスパラーメータに対応していたり、各種ミドルウェアを使用することで認証を行ったりすることもできます。

詳しくは[公式ドキュメント](https://bunrouter.uptrace.dev/guide/golang-router.html)や[GitHub](https://github.com/uptrace/bunrouter)のサンプルコードを確認してみてください。

# 使ってみた感想

僕は、Echo を普段は使っていましたが Echo には使ったことがない機能もあったりしたので僕のような最低限の機能があればいいといった人には BunRouter はおすすめできるかもしれないです。
ある程度の部分を自分で管理することが求められますが、その分シンプルなライブラリで癖が少なめです。また、独自のミドルウェアを作成したりすることもできるらしいので今後はそういった部分も触っていきたいと思います。
(あと個人的に、BunRouter のデフォルトのロガーが Echo のロガーより見やすくていい)
