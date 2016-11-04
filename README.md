## Lint - run linters from Go

Lint makes it easy to run linters from Go code. This allows lint checks to be part of a regular `go build` + `go test` workflow. False positives are easily ignored and linters are automatically integrated into CI pipelines without any extra effort. For detailed documentation check the [project website](https://www.timeferret.com/lint).

### Quick Start

Download using
```
go get -t github.com/surullabs/lint
```
Run the default linters by adding a new test at the top level of your repository
```
import (
    "testing"
    "github.com/surullabs/lint"
)

func TestLint(t *testing.T) {
    if err := lint.Default.Check("./..."); err != nil {
        t.Fatal("lint failures: %v", err)
    }
}
```

Ignore some errors you aren't interested in
```
func TestLint(t *testing.T) {
    // Run default linters and ignore lint errors from auto-generated files
    filtered := lint.Skip(lint.Default, lint.RegexpMatch(`_string\.go`, `\.pb\.go`))
    
    if err := filtered.Check("./..."); err != nil {
        t.Fatal("lint failures: %v", err)
    }
}
```

### How it works

`lint` runs linters using the excellent `os/exec` package. It searches all Go binary directories for the needed binaries and when they don't exist it downloads the needed binaries using `go get` before running them. Errors are split by newline and can be skipped as needed.

### Default linters

  - `gofmt` - Run `gofmt -d` and report any differences as errors.
  - `govet` - `govet -shadow`
  - `golint` - [https://github.com/golang/lint](https://github.com/golang/lint)
  - `gosimple` - [https://github.com/dominikh/go-simple](https://github.com/dominikh/go-simple)
  - `gostaticcheck` - [https://github.com/dominikh/go-staticcheck](https://github.com/dominikh/go-staticcheck)
  
### Why `lint`?

There are a number of excellent linters available for Go and Lint makes it easy to run them from tests. While building our mobile calendar app [TimeFerret](https://www.timeferret.com), (which is built primarily in Go), including scripts that run linters as part of every repository grew tiresome very soon. Using `lint` to create tests that ran on each commit made the codebase much more stable, since any unneeded false positives could be easily skipped. The main advantages of using `lint` over running tools manually is:

  - Skip false positives explicitly in your tests - This makes it easy to run only needed checks.
  - Enforce linter usage with no overhead - No special build scripts are needed to install linters on each developer machine as they are automatically downloaded.
  - Simple CI integration - Since linters are run as part of tests, there are no extra steps needed to integrate them into your CI pipeline.

### Adding a custom linter

Adding a new linter is made dead simple by the `github.com/surullabs/lint/checkers` package. The entire source for the `golint` integration is

```
import "github.com/surullabs/lint/checkers"

type Check {
}

func (Check) Check(pkgs ...string) error {
    return checkers.Lint("golint", "github.com/golang/lint/golint", pkgs)
}
```

The `github.com/surullabs/lint/testutil` package contains utilities for testing custom linters.

### License

Lint is available under the Apache License. See the LICENSE file for details.

### Contributing

Pull requests are always welcome! Please ensure any changes you send have an accompanying test case.