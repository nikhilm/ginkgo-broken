Reproduceable test case for Ginkgo runner modifies some kind of global state.

Reproduced on 

    go version go1.7.1 darwin/amd64
    Darwin Nikhils-MBP 15.5.0 Darwin Kernel Version 15.5.0: Tue Apr 19 18:36:36 PDT 2016; root:xnu-3248.50.21~8/RELEASE_X86_64 x86_64
    Ginkgo Version 1.2.0

Environment:

    GOARCH="amd64"
    GOBIN=""
    GOEXE=""
    GOHOSTARCH="amd64"
    GOHOSTOS="darwin"
    GOOS="darwin"
    GOPATH="/Users/nikhil/iron/gocode"
    GORACE=""
    GOROOT="/usr/local/go"
    GOTOOLDIR="/usr/local/go/pkg/tool/darwin_amd64"
    CC="clang"
    GOGCCFLAGS="-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=/var/folders/3t/tx0j5bws5kv7m6s2rj2dtyvm0000gn/T/go-build959139225=/tmp/go-build -gno-record-gcc-switches -fno-common"
    CXX="clang++"
    CGO_ENABLED="1"

## What is the problem?

When a `package main` exports some fields/functions that the test package
imports to test, ginkgo works fine on the first run, but fails on subsequent
runs. No global state seems to be modified, nor are new files created, so
I cannot figure out what is the cause. After the command is run, `go test`
starts failing too.

## Steps to Reproduce

    go get github.com/nikhilm/ginkgo-broken
    cd $GOPATH/src/github.com/nikhilm/ginkgo-broken
    cd tests
    go test -v . # everything works!
    go test -v . # everything works!

These commands output:

    === RUN   TestYo
    YOYOYO
    --- PASS: TestYo (0.00s)
    PASS
    ok      github.com/nikhilm/ginkgo-broken/tests  0.006s

Let's continue:

    ginkgo -v . # everything works!

This works too:

    === RUN   TestYo
    YOYOYO
    --- PASS: TestYo (0.00s)
    PASS

    Ginkgo ran 1 suite in 593.024519ms
    Test Suite Passed

But it has broken something, because from this point on:

    go test -v . 

leads to:

    # github.com/nikhilm/ginkgo-broken/tests_test
    ./project_test.go:6: can't find import:
    "github.com/nikhilm/ginkgo-broken/project"
    FAIL    github.com/nikhilm/ginkgo-broken/tests [build failed]

Expected output: tests should always pass

Once the ginkgo command is run, this directory is broken "forever". I have not
yet been able to isolate what leads to the go runtime considering it "fixed".

Things I've found that work as a "fix":

* Touching `project/main.go`.
* Cloning repo again.

Things I've found that do not work as a "fix":

* Starting a new shell.
* Touching `test/project_test.go`
