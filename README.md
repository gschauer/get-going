# Get Go-ing

## Useful Libraries & Frameworks
Please find below an opinionated list of de-facto standard libraries.
Needless to say, there are plenty alternatives for logging, HTTP, etc.
Please refer to https://github.com/avelino/awesome-go for a comprehensive list of awesome Go projects.
However, the projects listed here have been chosen deliberately because they provide more features or are significantly faster than comparable projects.

### Command Line
* [AlecAivazis/survey]: library for building interactive prompts
* [jessevdk/go-flags]: command line option parser with support for long flags, environment variables, etc.
* [kballard/go-shellquote]: shell-like word splitting/joining
* [spf13/cobra]: a commander for modern Go CLI interactions

### Configuration
* [spf13/viper]: configuration library; supports environment variables, ini, Java properties, json, toml, yaml, etc.

### Dependency Injection
* [google/wire]: Google's compile-time dependency injection framework

### File Management
* [spf13/afero]: filesystem abstraction; supports different provides: memory, native, readonly, HTTP, SFTP, etc.

### Logging
* [rs/zerolog]: zero allocation JSON logger

### Misc
* [abc-inc/browser]: opens URLs or files in the most appropriate browser or assigned application
* [abc-inc/goava]: Google's Guava ported to Go; provides case format, char matcher, stopwatch, escapers, etc.
* [dustin/go-humanize]: formatters for units to human friendly sizes
* [mitchellh/mapstructure]: decoding generic map values into native Go structures and vice versa.

### Networking
* [abc-inc/terminus]: IP subnet address calculator
* [c-robinson/iplib]: working with IP addresses and networks

### Resource Embedding
* [GeertJohan/go.rice]: makes working with resources such as html, js, images, templates, etc very easy
* [shurcooL/vfsgen]: generates Go code that statically implements a FileSystem; similar to go.rice but less options

### System
* [shirou/gopsutil]: cross-platform library for process and system monitoring

### Testing
* [google/go-cmp]: comparing Go values in tests (deep equality)
* [jonboulle/clockwork]: a fake clock for simulating time
* [stretchr/testify]: toolkit with common assertions and mocks that plays nicely with the standard library

### UUID
* [google/uuid]: UUIDs based on RFC 4122 and DCE 1.1

### Web
* [julienschmidt/httprouter]: high performance HTTP request router on top of net/http


## Best Practices

### Code Style
* Avoid the `else` clause. Reduce the amount of indentation as much as possible.
* Comments should be proper sentences, and the first line should start with the name of the struct, function, etc.
* `New<Something>()` should return pointers to avoid re-allocation.
* Prefer exporting variables over getters/setter.
If really needed, omit the `Get` prefix e.g., use `Customer()` for reading and `SetCustomer()` for writing.
* Use short expressive package names - no `util`, prefer `db` over `database`.
* Use short descriptive variable names - prefer `count` and `cust` over `counter` and `customer` (`c` would be ambiguous).
* Variable names should be shorter if they have limited scope and longer if used globally.

### Do's & Don'ts
* Do not `panic()` unless you really have to - `log.Fatal()` is a good alternative.

### Enums
* Enums are ordinary integers but can be enriched with additional methods.
The idiomatic way of adding a string representation is the following:
    ```go
    type State int // declare a new type
    
    const (
    	New State = iota // use iota (reverse abbreviation of 'ASCII to int')
    	Progress
    	Done
    )

    // String returns the State string representation.
    func (s State) String() string {
    	return []string{"New", "Progress", "Done"}[s]
    }
    ```

### HTTP server
* Typically, the HTTP server starts in sub milliseconds.
However, if it is necessary to log the exact time when the server become healthy, the following snippet can be used:
    ```go
    url := "http://" + addr.String() + "/health"
    if _, err = http.Get(url); err == nil {
        log.Fatal().Msgf("Cannot bind %s", addr)
    }

    done := make(chan bool)
    log.Info().Msg("Starting server...")
    go func() { log.Err(server.ListenAndServe()).Send() }()

    log.Debug().Msgf("Checking health on %s", url)
    for i := 1; i <= 100; i++ {
        res, err := http.Get(url)
        if err == nil {
            if res.StatusCode != http.StatusOK {
                break
            }

            log.Info().Msgf("Server started at %s after %d µs", server.Addr, (time.Now().Sub(start).Microseconds()))
            <-done
        }
        time.Sleep(time.Duration(i) * time.Millisecond)
    }

    log.Fatal().Msg("Server failed to start")
    ```

### Language
* Slices and arrays are reference types and modification of the value of a slice of a slice of a ... modifies the backing array.

### Modules & Dependency Management
* `go get -t -v` should be used to download required dependencies.
* `go mod download` downloads all dependencies listed in `go.mod`, regardless whether they are required or not.
* `go mod tidy` cleans up `go.mod` and `go.sum` after adding new dependencies.

### Releasing
* Use the `goimports` tool, which applies `gofmt` styles and sorts imports.

### Static Linking
* Under some circumstances e.g., when using `net` or the `user` package, the linker incorporates libraries from the host.
In order to force static linking, the easiest way is to set `CGO_ENABLED=0`.

### Testing
* Avoid `init()` functions, because they make testing more complicated.
* Put test code into the same directory, but use `package <pkg>_test`.

## Go Commands & Compiler Flags
### Testing and Code Coverage
```shell script
go test -coverprofile "${TEMP_DIR}/reports/coverage.out" ./... && \
  go tool cover -html "${TEMP_DIR}/reports/coverage.out" -o "${TEMP_DIR}/reports/coverage.html"
```

### Linting
#### golangci-lint
This is the de-facto standard tool containing 48 Linters.
It is available as Docker container as well as binary, which can be launched as follows: 

```shell script
golangci-lint run ./...
```

Check https://golangci-lint.run/ for installation instructions and configuration options.

#### GitHub SuperLinter
GitHub provides Super-Linter, which can be downloaded and executed everywhere.
It contains dozens of different linters for many programming languages, including Go.
If you don't mind pulling a 4 GB Docker image, you can do the following:

```shell script
docker run --rm \
  -e FILTER_REGEX_EXCLUDE="" \ # if required
  -e RUN_LOCAL=true \
  -v "${PWD}:/tmp/lint" \
  github/super-linter
```

Further information can be found at https://github.com/github/super-linter.

### Optimization
In general, debug information can be stripped from binaries to minimize their size.
Moreover, Go compiles absolute paths into the binary, which can be stripped using `-trimpath`. 

```shell script
export CGO_ENABLED=0
go build -ldflags "-s -w" -o <binary> -trimpath ./...
```

### Compile-Time Constants
If you want to introduce a constant such as build version, Git tag or something similar, you can use a linker flag.
The `-X` flag takes an argument in the form `package.ExportedVariable=value` (and the flag can be repeated multiple times) e.g.:

```shell script
go build -ldflags "-X 'main.Version=v1.2'" ./...
``` 

### Windows
GUI applications on Windows launch a console.
If this is not desirable, it can be suppressed using the following flag:

```shell script
go build -ldflags "-H=windowsgui" ./...
```

### Makefile
Here is a template Makefile to copy for convenience:

```makefile
BUILD_DIR=bin
TEMP_DIR=tmp
export CGO_ENABLED=0

all: clean build

build: test
	mkdir -p "${BUILD_DIR}"
	go build -o "${BUILD_DIR}/" ./...

check:
	golangci-lint run ./...

clean:
	rm -fr "${BUILD_DIR}" "${TEMP_DIR}"

test:
	mkdir -p tmp/reports && \
	go test -coverprofile "${TEMP_DIR}/reports/coverage.out" ./...
	go tool cover -html "${TEMP_DIR}/reports/coverage.out" -o "${TEMP_DIR}/reports/coverage.html"

.PHONY: all build check clean test
```


[abc-inc/browser]: https://github.com/abc-inc/browser
[abc-inc/goava]: https://github.com/abc-inc/goava
[abc-inc/terminus]: https://github.com/abc-inc/terminus
[AlecAivazis/survey]: https://github.com/AlecAivazis/survey
[c-robinson/iplib]: https://github.com/c-robinson/iplib
[dustin/go-humanize]: https://github.com/dustin/go-humanize
[GeertJohan/go.rice]: https://github.com/GeertJohan/go.rice
[google/go-cmp]: https://github.com/google/go-cmp
[google/uuid]: https://github.com/google/uuid
[google/wire]: https://github.com/google/wire
[jessevdk/go-flags]: https://github.com/jessevdk/go-flags
[jonboulle/clockwork]: https://github.com/jonboulle/clockwork
[julienschmidt/httprouter]: https://github.com/julienschmidt/httprouter
[kballard/go-shellquote]: https://github.com/kballard/go-shellquote
[mitchellh/mapstructure]: https://github.com/mitchellh/mapstructure
[rs/zerolog]: https://github.com/rs/zerolog
[shirou/gopsutil]: https://github.com/shirou/gopsutil
[shurcooL/vfsgen]: https://github.com/shurcooL/vfsgen
[spf13/afero]: https://github.com/spf13/afero
[spf13/cobra]: https://github.com/spf13/cobra
[spf13/viper]: https://github.com/spf13/viper
[stretchr/testify]: https://github.com/stretchr/testify
