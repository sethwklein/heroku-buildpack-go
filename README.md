heroku-buildpack-go-simple
==========================

[heroku-buildpack-go-simple][m] is a [custom buildpack][1] for [Heroku][2] that
supports [Go][3].

[m]: https://github.com/sethwklein/heroku-buildpack-go-simple
[1]: https://devcenter.heroku.com/articles/buildpacks#using-a-custom-buildpack
[2]: https://www.heroku.com/
[3]: http://golang.org/

History
-------

heroku-buildpack-go-simple is a fork of [kr][k]'s [heroku-buildpack-go][4]. The
differences are:

* `GOPATH` is the root of your project (`BUILD_DIR` in Heroku terms) not some
mysterious directory created by the buildpack.
* `go install` is used instead of `go get`. This means no Virtualenv, Mercurial,
or Bazaar are installed. You are expected to check your
dependencies into Git. This speeds deployment by reducing buildpack dependencies
(making the cache smaller) and by reducing network activity used for fetching
project dependencies. It also increases reliability and security by not having
your deployment involve fetching code from some random person's GitHub!

[4]: https://github.com/kr/heroku-buildpack-go
[k]: https://github.com/kr

Usage
-----

```bash
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
$ echo 'example.com/serve-heroku' >.goinstall
$ echo 'web: serve-heroku' > Procfile
$ git init
Initialized empty Git repository in /Users/example/heroku-buildpack-go-example/.git/
$ git add .
$ git commit -m 'Add hello world'
[master (root-commit) 7293c81] Add hello world
 3 files changed, 15 insertions(+)
 create mode 100644 .goinstall
 create mode 100644 Procfile
 create mode 100644 src/example.com/serve-heroku/main.go
$ heroku create
Creating morning-wildwood-4996... done, stack is cedar
http://morning-wildwood-4996.herokuapp.com/ | git@heroku.com:morning-wildwood-4996.git
Git remote heroku added
$ heroku config:add BUILDPACK_URL=https://github.com/sethwklein/heroku-buildpack-go-simple.git
Setting config vars and restarting morning-wildwood-4996... done, v3
BUILDPACK_URL: https://github.com/sethwklein/heroku-buildpack-go-simple.git
$ git push heroku master
Counting objects: 8, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (8/8), 672 bytes | 0 bytes/s, done.
Total 8 (delta 0), reused 0 (delta 0)

-----> Fetching custom git buildpack... done
-----> GoSimple app detected
-----> Installing Go 1.2... done
-----> Running: go install -tags heroku example.com/serve-heroku
-----> Discovering process types
       Procfile declares types -> web

-----> Compiled slug size: 1.2MB
-----> Launching... done, v4
       http://morning-wildwood-4996.herokuapp.com deployed to Heroku

To git@heroku.com:morning-wildwood-4996.git
 * [new branch]      master -> master
$ curl -sS http://morning-wildwood-4996.herokuapp.com/
Hello, world!
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

It also deletes the `pkg` directory after building to reduce slug size.

[5]: http://golang.org/pkg/go/build/#hdr-Build_Constraints

Bugs
----

As always, [sethwklein][6] tries to keep up with pull requests and issues!

[6]: https://github.com/sethwklein
