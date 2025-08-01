# Limbo driver for Go's `database/sql` library

**NOTE:** this is currently __heavily__ W.I.P and is not yet in a usable state.

This driver uses the awesome [purego](https://github.com/ebitengine/purego) library to call C (in this case Rust with C ABI) functions from Go without the use of `CGO`.

## Embedded Library Support

This driver includes an embedded library feature that allows you to distribute a single binary without requiring users to set environment variables. The library for your platform is automatically embedded, extracted at runtime, and loaded dynamically.

### Building from Source

To build with embedded library support, follow these steps:

```bash
# Clone the repository
git clone https://github.com/tursodatabase/turso

# Navigate to the Go bindings directory
cd limbo/bindings/go

# Build the library (defaults to release build)
./build_lib.sh

# Alternatively, for faster builds during development:
./build_lib.sh debug
```

### Build Options:

* Release Build (default): ./build_lib.sh or ./build_lib.sh release

    - Optimized for performance and smaller binary size
    - Takes longer to compile and requires more system resources
    - Recommended for production use

* Debug Build: ./build_lib.sh debug

    - Faster compilation times with less resource usage
    - Larger binary size and slower runtime performance
    - Recommended during development or if release build fails

If the embedded library cannot be found or extracted, the driver will fall back to the traditional method of finding the library in the system paths.

## To use: (_UNSTABLE_ testing or development purposes only)

### Option 1: Using the embedded library (recommended)

Build the driver with the embedded library as described above, then simply import and use. No environment variables needed!

### Option 2: Manual library setup

#### Linux | MacOS

_All commands listed are relative to the bindings/go directory in the limbo repository_

```
cargo build --package limbo-go

# Your LD_LIBRARY_PATH environment variable must include limbo's `target/debug` directory

export LD_LIBRARY_PATH="/path/to/limbo/target/debug:$LD_LIBRARY_PATH"

```

#### Windows

```
cargo build --package limbo-go

# You must add limbo's `target/debug` directory to your PATH
# or you could built + copy the .dll to a location in your PATH
# or just the CWD of your go module

cp path\to\limbo\target\debug\lib_limbo_go.dll .

go test

```
**Temporarily** you may have to clone the limbo repository and run:

`go mod edit -replace github.com/tursodatabase/turso=/path/to/limbo/bindings/go`

```go
import (
    "fmt"
    "database/sql"
    _"github.com/tursodatabase/turso"
)

func main() {
	conn, err := sql.Open("sqlite3", ":memory:")
	if err != nil {
        fmt.Printf("Error: %v\n", err)
        os.Exit(1)
	}
    sql := "CREATE table go_limbo (foo INTEGER, bar TEXT)"
    _ = conn.Exec(sql)

    sql = "INSERT INTO go_limbo (foo, bar) values (?, ?)"
    stmt, _ := conn.Prepare(sql)
    defer stmt.Close()
    _  = stmt.Exec(42, "limbo")
    rows, _ := conn.Query("SELECT * from go_limbo")
    defer rows.Close()
    for rows.Next() {
        var a int
        var b string
		_ = rows.Scan(&a, &b)
        fmt.Printf("%d, %s", a, b)
    }
}
```

## Implementation Notes

The embedded library feature was inspired by projects like [go-embed-python](https://github.com/kluctl/go-embed-python), which uses a similar approach for embedding and distributing native libraries with Go applications.
