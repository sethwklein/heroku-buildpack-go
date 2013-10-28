heroku-buildpack-go
===================

heroku-buildpack-go is a [custom buildpack][1] for [Heroku][2] that supports
[Go][3].

[1]: https://devcenter.heroku.com/articles/buildpacks#using-a-custom-buildpack
[2]: https://www.heroku.com/
[3]: http://golang.org/

History
-------

heroku-buildpack-go is a fork of kr's [heroku-buildpack-go][4]. The differences
are:

* `GOPATH` is the root of your project, `BUILD_DIR` in Heroku terms, not some
invisible directory created by the buildpack.
* `go install` is used instead of `go get`. You are expected to check your
dependencies into Git. This speeds deployment by reducing buildpack dependencies
making the cache smaller and by reducing network activity used for fetching
project dependencies. It also increases reliability and security by not having
your deployment involve fetching code from some random person's GitHub!

[4]: https://github.com/kr/heroku-buildpack-go

Usage
-----

```
$ mkdir heroku-buildpack-go-example
$ cd heroku-buildpack-go-example/
$ mkdir -p src/example.com/serve-heroku/
$ cat >src/example.com/serve-heroku/main.go <<'EOF'
package main

import (
	"net/http"
	"os"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("Hello, world!\n"))
	})
	panic(http.ListenAndServe(":" + os.Getenv("PORT"), nil))
}
EOF
$ echo 'serve-heroku' >.goinstall
$ echo 'web: serve-heroku' > Procfile
$ git init
Initialized empty Git repository in /Users/example/heroku-buildpack-go-example/.git/
$ git add .
$ git commit -m 'Add hello world'
...
$ heroku create
...
$ heroku config:add BUILDPACK_URL=https://github.com/sethwklein/heroku-buildpack-go.git
...
$ git push heroku master
...
$ curl -sS ...
```

Documentation
-------------

This buildpack will detect your repository as Go if it contains a `.goinstall`
file.

The .goinstall file contains import paths to be installed, separated by
whitespace. Example:

```
example.com/server
example.com/worker
```

The buildpack adds a `heroku` [build constraint][5], to enable heroku-specific
code. 

It also deletes the pkg directory after building to reduce slug size.

[5]: http://golang.org/pkg/go/build/#hdr-Build_Constraints

Bugs
----

As always, [sethwklein][6] tries to keep up with pull requests and issues!

[6]: https://github.com/sethwklein
