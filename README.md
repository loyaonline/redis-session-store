# Redis store for [Session](https://github.com/loyaonline/session)

## Quick Start

### Download and install

```bash
go get -v github.com/loyaonline/redis-session-store
```

### Create file `server.go`

```go
package main

import (
	"context"
	"fmt"
	"net/http"

	"github.com/loyaonline/redis-session-store"
	"github.com/loyaonline/session"
)

func main() {
	session.InitManager(
		session.SetStore(redis.NewRedisStore(&redis.Options{
			Addr: "127.0.0.1:6379",
			DB:   15,
		})),
	)

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		store, err := session.Start(context.Background(), w, r)
		if err != nil {
			fmt.Fprint(w, err)
			return
		}

		store.Set("foo", "bar")
		err = store.Save()
		if err != nil {
			fmt.Fprint(w, err)
			return
		}

		http.Redirect(w, r, "/foo", 302)
	})

	http.HandleFunc("/foo", func(w http.ResponseWriter, r *http.Request) {
		store, err := session.Start(context.Background(), w, r)
		if err != nil {
			fmt.Fprint(w, err)
			return
		}

		foo, ok := store.Get("foo")
		if ok {
			fmt.Fprintf(w, "foo:%s", foo)
			return
		}
		fmt.Fprint(w, "does not exist")
	})

	http.ListenAndServe(":8080", nil)
}
```
