# Golang Code Style Guide


## Formatting

Good code formatting is essential for maintainability. It makes the code easier to read and understand, which helps in spotting errors and following the code flow.


### Imports

The following import groups must be organized as follows:

* standard library;
* external libraries;
* inDrive libraries;
* inDrive project repository.

**Bad examples**:

```go
import (
    "fmt"
    "os"
    "github.com/go-playground/validator/v10"
    "github.com/go-redis/redis_rate/v9"
    "github.com/inDriver/inDriverGO/server/core"
    "github.com/inDriver/inDriverGO/server/db"
    "github.com/inDriver/inDriverGO/server/db/plugin"
)
```

**Good examples**:

```go
import (
    "fmt"
    "os"
  
    "github.com/go-playground/validator/v10"
    "github.com/go-redis/redis_rate/v9"

    "github.com/inDriver/lib-go/v2/logger"
	
    "github.com/inDriver/inDriverGO/server/core"
    "github.com/inDriver/inDriverGO/server/db"
    "github.com/inDriver/inDriverGO/server/db/plugin"
)
```


### Code Formatting Tools


#### Imports Formatting Tools

Code must be formatted with `gci` tool.

Use this [gci](https://github.com/daixiang0/gci) repo.

`gci write -s standard -s default -s 'prefix(github.com/inDriver)' -s localmodule --skip-generated .`

To install: `go install github.com/daixiang0/gci@latest`.


#### Code Formatting Tools 

Code must be formatted with `gofmt`.

You may use the `gofumpt` tool. `gofumpt` is a stricter version of `gofmt`, the standard Go formatter. It applies additional changes that can help simplify code further.

Use this [repo](https://github.com/mvdan/gofumpt).

To install:

* install terminal: `go install mvdan.cc/gofumpt@latest`;
* install on IDE: [link](https://github.com/mvdan/gofumpt#installation).


### Line Length

Lines of code must not exceed 120 characters. Adhering to this limit makes the code easier to read, especially in side-by-side views in code review tools.

However, there may be exceptions where a longer line is more readable or necessary. In such cases, carefully weigh the trade-offs before deciding to exceed the recommended line length.

```go
// bad example
func getPaymentMethods(cityID int64, paymentMethods []neworder.PaymentMethod, cityPaymentMethods []neworder.CityPaymentMethod) ([]neworder.PaymentMethod, error) {
	...
}

// good example
// if all input params fits 120 chars we should put them in one line
func getPaymentMethods(
	cityID int64, paymentMethods []neworder.PaymentMethod, cityPaymentMethods []neworder.CityPaymentMethod, 
) ([]neworder.PaymentMethod, error) {
...
}

// good example
// if input params don't fit 120 chars we use new line for each param
func getPaymentMethods(
    cityID int64, 
	paymentMethods []neworder.PaymentMethod, 
	cityPaymentMethods []neworder.CityPaymentMethod,
) ([]neworder.PaymentMethod, error) {
...
}
```


### Blank Lines

Use blank lines to group related chunks of code together. For example, include a blank line between the import statement group and the rest of the code or between functions in a file.

The strategic use of blank lines can significantly enhance code readability. Here are some guidelines:

* there must be one blank line between the import statement group and the rest of the code;
* there must be one blank line between functions to separate them visually;
* within a function, you may use blank lines to group related chunks of code;
* there must not be a blank line between the function name and the first line of code within the function;
* avoid multiple consecutive blank lines, as they don’t improve readability and may unnecessarily increase code length.

Here is an example of good blank line usage:

```go
import (
	"fmt"
	"log"
)

func CalculateSum(a int, b int) (int, error) {
	if a < 0 || b < 0 {
		return 0, errors.New("Inputs must be non-negative integers")
	}
	
	sum := a + b
	
	return sum, nil
}

func main() {
	fmt.Println(CalculateSum(5, 3))
}
```

In this example, blank lines are used to separate import statements, functions, and logically related groups of statements within a function.

With this revision, the code adheres to your guidelines better and provides a clear example of the rules in action.


## Naming Conventions

Naming conventions in Go are straightforward, but they play a crucial role in making your code readable and maintainable.


### Packages

The package name is crucial as it acts as an accessor for the package contents and is frequently used in code. Therefore, the package name:

* must be lowercase, without underscores or mixedCaps.
* should be short, concise, and clearly indicate what the package does.
* must be the base name of its source directory. For example, the package in `src/encoding/base64` is imported as `"encoding/base64"` but is named `base64`, not `encoding_base64` or `encodingBase64`. An exception is versioned code, where the directory might be named `v9`, and the package name would be `redis`.
* should not be plural. For example, use `net/url`, not `net/urls`.
* should not use names like "common," "util," "shared," or "lib," as these are uninformative.

You should not use the `import .` notation.

Avoid redundancy in exported names within a package since they are always referenced with the package name. For instance, the buffered reader type in the `bufio` package is named `Reader`, not `BufReader`, because it is accessed as `bufio.Reader`.

**Bad examples**:

```go
import "src/encoding/encoding_base64"  // underscores and repetition of 'encoding'
import "utils"  // too generic, does not provide context
import "src/MyPackage"  // mixedCaps in package name
import "a_very_long_and_descriptive_package_name"  // too long, uses underscores
import "src/cool_stuff"  // uses underscores, too generic
import "vague"  // too vague, doesn't give context
import "github.com/inDriver/new-order/internal/operations/bid_commands" // uses underscores
```

**Good examples**:

```go
import "bytes"  // short, descriptive
import "encoding/base64"  // descriptive, follows directory structure
import "net/http"  // clear and follows directory structure
import "os"  // short and descriptive
import "database/sql"  // follows directory structure, clear what it provides
import "github.com/user/project/router"  // third-party package, follows directory structure
```

For a more detailed guide, refer to the [Effective Go section on Package Names.](https://go.dev/doc/effective_go#package-names)


#### Handling Package Collisions

In cases of package name collisions, you can use an alias for the package at the import point to disambiguate between packages with the same name. Import aliasing must be used if the package name does not match the last element of the import path.

When using an alias, it:

* must be clear and provide context for the package’s usage within the file;
* should not be overly verbose or cryptic.

```go
import (
    orderDTO "github.com/myorg/order/dto"
    rideDTO "github.com/myorg/ride/dto"
)
```

In the above example, `orderDTO` and `rideDTO` are aliases for the `dto` packages under `orders` and `rides`, respectively. These aliases make it clear which DTOs are being referred to in the following code.

Remember, the goal is to write code that is easy to read, understand, and maintain. Always consider these aspects when choosing an alias.

>**NOTE**
>
>The use of package aliases should be an exception, not the norm. It’s generally preferable to choose unique package names to avoid needing aliases. Aliases should only be used when a package name collision is unavoidable.


### Files

File names are important for quickly understanding the structure of the codebase. Therefore, the file name:

* must be lowercase and avoid mixedCaps;
* should be short and concise;
* should clearly reflect the contents or purpose of the file.

Words in file names must be separated with an underscore (`_`).

**Bad examples**:

```go
myCoolFile.go   // Uses mixedCaps
VeryLongAndDescriptiveFileName.go  // Too long
generic.go   // Too generic, doesn't provide context
```

**Good examples**:

```go
main.go   // Short and clear
http_handler.go  // Clear about its purpose, uses underscore because it's a convention for filenames with multiple words
user_test.go   // Clearly a test file, uses underscore as it's a convention for test files
```


### File Size

Code should be grouped in files by functionality or types.

While there’s no strict limit on file size, it’s recommended to keep Go files under 200 lines of code. This isn’t a hard rule but a guideline to maintain readability and ease of navigation within the codebase.

If a file significantly exceeds this size, consider whether the code can be better modularized. Could some functions or types be moved to other files or even packages? Is the file adhering to the Single Responsibility Principle?

Here are a few reasons why a 200-line limit is often suggested:

here are a few reasons why a 200-line limit is often suggested:

* **readability**: smaller files are easier to read, often allowing you to view the entire file without scrolling and making it easier to grasp the file’s purpose and functionality at a glance;
* **simplicity**: smaller files tend to contain fewer functions or classes, which helps keep complexity down, making the code easier to understand and maintain;
* **single responsibility principle**: in line with the single responsibility principle, smaller files are more likely to focus on a single task or feature, aiding in understanding and reusability;
* **easier navigation**: in larger codebases, it’s quicker and easier to navigate through a set of smaller files with distinct purposes, as opposed to fewer large, complex files;
* **merge conflicts**: smaller files can reduce the frequency and complexity of merge conflicts when multiple developers work on the same codebase.

it’s important to note that 200 lines isn’t a strict number and may vary depending on team preferences. the key is to aim for smaller, manageable files whenever possible. these guidelines are recommendations rather than hard rules and can be adapted based on the situation.


### Variables, Constants, and Types


#### General Guidelines

When naming variables, constants, and types, aim for clarity and readability. The name should make the purpose and usage of the entity clear. Here are some guidelines:

* name must be in MixedCaps or camelCase;
* variable names must be short and concise. The shorter the name of a variable, the more clearly its purpose is defined. This is especially true for local variables with limited scope;
* variable names should not contain redundant or unnecessary words. For example, avoid names like `theProduct` or `myProduct`; instead, use `product`;
* variable names should not be single letters or abbreviations unless they are common in the programming community. For example, `i` for an iteration variable or `err` for an error are acceptable;
* if a single-letter variable name is used, it should be meaningful and conventional. For example, `r` for a reader, `w` for a writer, `i` for an index, `v` for a value in a loop, etc.;
* variables with a larger scope must have more descriptive names.

Avoid using underscores or all caps for constants, which is a convention in some other languages but not in Go.

**Bad examples**:

```go
var my_var int  // uses underscore
const Myconstant int = 10  // lowercase type for public constant name containing multiple words
type mytype int  // lowercase type for name containing multiple words
```

**Good examples**:

```go
var myVar int  // camelCase for variables
const MyConstant int = 10  // CamelCase public constant name containing multiple words
type myType int  // CamelCase type name

for i, v := range mySlice {
    // i and v are meaningful here.
}
```


#### Boolean Variables

* boolean variables must be named so that their true or false meaning is clear;
* consider prefixing boolean variables with `is`, `has`, `can`, `should`, etc.;
* avoid negative naming, like `isNotReady` or `noCache`.

```go
var isReady bool
var hasDependencies bool
var canProceed bool
```


### Error Variables

* error variables must be named starting with `Err` followed by a description of the error; this is a common practice in Go;
* if the error variable is local to a function and is not going to be returned, consider naming it `err`.

```go
var ErrNotFound = errors.New("not found")
```


### Interfaces

Interfaces are a fundamental part of Go's type system. When naming them, aim for clarity and readability.

In Go, you must not use a prefix `I` for interfaces, like `IReader`, or a suffix like `ReaderInterface`. While this might be a convention in other languages (like C# or Java), it’s uncommon in Go and can make the code harder to read for those familiar with Go conventions.

Instead, Go favors simple, descriptive names for interfaces:

* interface names should be a noun or noun phrase that describes the behavior the interface represents;
* if the interface has a single method, it's common (though not required) to name the interface after the method with an `-er` suffix, like `Reader` or `Writer`;
* if the interface has multiple methods, it’s advisable to choose a name that accurately represents the behavior the interface abstracts, such as `Storage`, `Service`, `Factory`, etc.;
* avoid using vague or generic terms such as `Manager` for interface names.


```go
// Good
type Reader interface {
    Read(p []byte) (n int, err error)
}

// Bad
type IReader interface {
    Read(p []byte) (n int, err error)
}

// Bad
type ReaderInterface interface {
    Read(p []byte) (n int, err error)
}
```


### Struct Naming

* struct names should be a noun or noun phrase that clearly indicates what the struct represents; like other type names, struct names must be in `camelCase`;
* if the struct is primarily defined to implement an interface, its name should reflect that. a common pattern is to use a prefix or suffix that indicates the interface, but the best name depends on the specific use case;
* avoid using vague or generic terms such as `Manager` for struct names;
* struct names must not contain suffixes like `Impl` or `Implementation`.

```go
// Good
type Employee struct {} // Descriptive noun
type HTTPResponse struct {} // Descriptive noun phrase
type ByteReader struct {} // Implements the `Reader` interface for bytes
type HTTPRequestHandler struct {} // Implements the `Handler` interface specifically for HTTP requests

// Bad
type my_struct struct {} // Struct name with underscore
```

Consider an `Encoder` interface with different implementations for various encoding schemes.

```go
type Encoder interface {
    Encode(data []byte) ([]byte, error)
}

// Good
type JSONEncoder struct {} // Encodes data in JSON format
type XMLEncoder struct {} // Encodes data in XML format
type GobEncoder struct {} // Encodes data in Go's "gob" binary format

// Bad
type EncoderImpl struct {} // Not descriptive - what kind of encoder is this?
```

Example with `OrderService`:

```go
type OrderService interface {
    CreateOrder(context.Context, *CreateOrderRequest) (*Order, error)
    ChangeOrderPrice(context.Context, *ChangeOrdePricer) (*Order, error)
    CancelOrder(...) error
    GetNearOrdersWithFilter(...) ([]*Order, error)
    GetContractorOrder(...) (*Order, error)
}
```

If you have only one implementation of the `OrderService` interface, you could simply name the implementation as `OrderService`. This communicates that it is the primary or main implementation of the interface.

It's a common practice to name the struct with the same name as the interface, especially when there is only one implementation. In this case, it's understood that this is the primary (or even only) implementation of the `OrderService` interface.

```go
type OrderService struct {} // The primary implementation of the OrderService interface
type MockOrderService struct {} // Mock implementation of the OrderService interface for testing
```

This clearly indicates that `MockOrderService` is used for testing, while `OrderService` is the main implementation. This makes the code easier to read and understand.

If the interface and the implementing struct are in the same package, they cannot have the same name. Choose a descriptive name for the struct that indicates its specific behavior or its role as the primary implementation of the interface.

Consider the `OrderService` interface. If the implementing struct is in the same package, you might name it `OrderOperations` or `OrderProcessor`:

```go
type OrderService interface {
    // ...
}

type OrderOperations struct {} // The primary implementation of the OrderService interface
// or
type OrderProcessor struct {} // The primary implementation of the OrderService interface
```


### Functions and Methods


#### General Guidelines

* function names must be clear about what they do. A good function name describes its side effects and return values. Follow the principle: "A function name should describe everything that the function does";
* function names must not be too short or too verbose. They should be as concise as possible without losing the context of what they do.
* function names must be in `camelCase`. An exception is made for test functions, which may contain underscores for the purpose of grouping related test cases, e.g., `TestMyFunction_WhatIsBeingTested`;

    For more information refer to [Effective Go](https://go.dev/doc/effective_go#Getters)
    
    ```go
    func ReadFile(path string) ([]byte, error) // Good
    func RF(path string) ([]byte, error) // Bad
    func PerformReadingOfFileLocatedAt(path string) ([]byte, error) // Bad
    func TestOrderHandler_CreateOrder(t *testing.T) // Good
    ```

* if a function returns a boolean, the name must clearly indicate a true or false question.

    ```go
    func Check() bool // Bad
    
    func IsEmpty() bool // Good
    func Empty() bool // Also good
    ```


#### Function Grouping and Ordering

* functions should be sorted in rough call order;
* functions in a file must be grouped by receiver.

Therefore, exported functions should appear first in a file, following `struct`, `const`, and `var` definitions.

A `newXYZ()` / `NewXYZ()` function may appear after the type is defined but before the rest of the methods on that receiver.

Since functions are grouped by receiver, plain utility functions should be placed toward the end of the file.


#### Error Handling Functions

* if a function or method can return an error, it must be the last return value;
* if a function or method is meant to handle errors, it must have "handle" or "check" in the name:

    ```go
    func HandleError(err error) // good
    func CheckError(err error) // good
    ```


#### Function Argument Naming

* argument names must clearly express what the function expects;
* avoid single-letter argument names, except for very short functions and methods.

```go
func Write(w Writer, data []byte) error // Good
func Write(dst Writer, dataToWrite []byte) error // Good
func Write(x Writer, y []byte) error // Bad
```


### Avoid Using Built-In Names

The Go [language specification](https://golang.org/ref/spec) outlines several built-in [predeclared identifiers](https://golang.org/ref/spec#Predeclared_identifiers) that should not be used as names within Go programs.

Depending on context, reusing these identifiers as names will either shadow the original within the current lexical scope (and any nested scopes) or make affected code confusing. In the best case, the compiler will complain; in the worst case, such code may introduce latent, hard-to-grep bugs.

**Bad example**:

```go
var error string // `error` shadows the builtin
var url string // `url` shadows `net/url` package

func handleErrorMessage(error string) {
    // `error` shadows the builtin
}
```

**Good example**:

```go
var errorMessage string
// `error` refers to the builtin

// or

func handleErrorMessage(msg string) {
    // `error` refers to the builtin
}
```

**Bad example**:

```go
type Foo struct {
    // While these fields technically don't
    // constitute shadowing, grepping for
    // `error` or `string` strings is now
    // ambiguous.
    error  error
    string string
}

func (f Foo) Error() error {
    // `error` and `f.error` are
    // visually similar
    return f.error
}

func (f Foo) String() string {
    // `string` and `f.string` are
    // visually similar
    return f.string
}
```

**Good example**:

```go
type Foo struct {
    // `error` and `string` strings are
    // now unambiguous.
    err error
    str string
}

func (f Foo) Error() error {
    return f.err
}

func (f Foo) String() string {
    return f.str
}
```

Note that the compiler will not generate errors when using predeclared identifiers, but tools such as `go vet` should correctly identify these and other cases of shadowing.
