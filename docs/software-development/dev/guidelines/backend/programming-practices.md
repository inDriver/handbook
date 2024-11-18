# Programming Practices


## Reduce Nesting

Deeply nested conditions or loops can make code harder to read and understand. It can also increase the cognitive load required to understand the flow of the program. To reduce nesting:

* **should not** exceed a depth of 3 levels. Any function exceeding this limit **should** be refactored to reduce complexity and improve readability;
* **use early returns**: rather than wrapping large blocks of code in `if` statements, check for the opposite condition and return or continue early. This reduces the level of indentation and makes the code more readable.

  **Bad example**:

    ```go
    func process(data []byte) error {
        if len(data) > 0 {
            // lots of code here
        }
        return nil
    }
    ```

  **Good example**:

    ```go
    func process(data []byte) error {
        if len(data) == 0 {
            return nil
        }
        // lots of code here
        return nil
    }
    ```

* **flatten loops**: avoid loops within loops where possible. If a loop within a loop is unavoidable, consider breaking the inner loop out into a separate function.

  **Bad example**:

    ```go
    for i := 0; i < len(data); i++ {
        for j := 0; j < len(data[i]); j++ {
            // process data[i][j]
        }
    }
    ```

  **Good example**:

    ```go
    for i := 0; i < len(data); i++ {
        processRow(data[i])
    }

    func processRow(row []byte) {
        for j := 0; j < len(row); j++ {
            // process row[j]
        }
    }
    ```

* **avoid deeply nested callbacks**: in some cases, you might be tempted to nest callbacks (like when working with goroutines or channels). This can lead to "callback hell". Instead, consider using techniques like "promises" or async/await-style functionality provided by libraries.


## Error Handling

Proper error handling is essential for building robust applications. Here are some best practices to follow when working with errors in Go:

* **should not** return `nil, nil`: if there's an error condition, always return an error. This approach avoids panics.

    ```go
    // Bad
    func doSomething() (*Result, error) {
        // if something goes wrong
        return nil, nil
    }

    // Good
    var errSomethingWrong = errors.New("something went wrong")
    func doSomething() (*Result, error) {
        // if something goes wrong
        return nil, errSomethingWrong
    }
    ```

* **should** use error types that convey meaningful information: it's best to use custom error types that can provide more context about the error, such as additional information or categorization to help in handling the error more effectively.

    ```go
    type ErrNotFound struct {
        ID string
    }

    func (e *ErrNotFound) Error() string {
        return fmt.Sprintf("resource with ID %v not found", e.ID)
    }
    ```

* **should** use error wrapping to provide context: this can be used to provide additional context, such as where the error occurred.

    ```go
    _, err := doSomething()
    if err != nil {
        return fmt.Errorf("failed to do something: %w", err)
    }
    ```

* **should not** use the word "error" in error messages for wrapping errors, as this creates unreadable logs with excessive repetition of "error" at each level, such as "error something: error something:... etc."

    ```go
    fmt.Errorf("error when doing something: %w", err) // bad

    fmt.Errorf("failed to do something: %w", err) // good
    fmt.Errorf("unable to do something: %w", err) // good
    ```

* **must** handle errors once: when an error is returned from a function, handle it only once and at the level where it makes sense, as high as possible. This prevents redundant error handling and keeps the error handling code cleaner.

    ```go
    func doTask() error {
        if err := doSubtask1(); err != nil {
            return fmt.Errorf("failed to do subtask 1: %w", err)
        }
        if err := doSubtask2(); err != nil {
            return fmt.Errorf("failed to do subtask 2: %w", err)
        }
        return nil
    }

    func main() {
        err := doTask()
        if err != nil {
            // log and process error here only once
        }
    }
    ```

* **should not** allow error leakage; e.g., if we have a repository using SQL, it should not return an `sqlx.ErrNotFound`, but rather an error near the interface that can be used at other levels;
* **must not** compare errors with `==` operator; use `errors.Is()` or `errors.As()` instead.


## Avoid Mutable Globals

* **should** avoid mutating global variables, instead opting for dependency injection.

**Bad example**:

```go
// sign.go

var _timeNow = time.Now

func sign(msg string) string {
  now := _timeNow()
  return signWithTime(msg, now)
}
```

```go
// sign_test.go

func TestSign(t *testing.T) {
  oldTimeNow := _timeNow
  _timeNow = func() time.Time {
    return someFixedTime
  }
  defer func() { _timeNow = oldTimeNow }()

  assert.Equal(t, want, sign(give))
}
```

**Good example**:

```go
// sign.go

type signer struct {
  now func() time.Time
}

func newSigner() *signer {
  return &signer{
    now: time.Now,
  }
}

func (s *signer) Sign(msg string) string {
  now := s.now()
  return signWithTime(msg, now)
}
```

```go
// sign_test.go

func TestSigner(t *testing.T) {
  s := newSigner()
  s.now = func() time.Time {
    return someFixedTime
  }

  assert.Equal(t, want, s.Sign(give))
}
```


## Reduce Scope of Variables

* **should** reduce the scope of variables. **Should not** reduce the scope if it conflicts with [reduce nesting](#reduce-nesting).

**Bad example**:

```go
err := os.WriteFile(name, data, 0644)
if err != nil {
 return err
}
```

**Good example**:

```go
if err := os.WriteFile(name, data, 0644); err != nil {
 return err
}
```

If you need the result of a function call outside of the `if` block, you should not try to reduce the scope.

**Bad example**:

```go
if data, err := os.ReadFile(name); err == nil {
  err = cfg.Decode(data)
  if err != nil {
    return err
  }

  fmt.Println(cfg)
  return nil
} else {
  return err
}
```

**Good example**:

```go
data, err := os.ReadFile(name)
if err != nil {
   return err
}

if err := cfg.Decode(data); err != nil {
  return err
}

fmt.Println(cfg)
return nil
```


## Reduce Complexity of Functions

Maintaining manageable complexity within functions is crucial for code maintainability and readability. Here are some guidelines:

* **Single Responsibility Principle (SRP)**: each function **should** have a single responsibility. It **should** do one thing and do it well.

    ```go
    // Recommended
    func calculateSum(numbers []int) int {
        sum := 0
        for _, num := range numbers {
            sum += num
        }
        return sum
    }

    func printSum(numbers []int) {
        sum := calculateSum(numbers)
        fmt.Println(sum)
    }
    ```

* **Limit Function Length**: functions **should not** exceed 50 lines of code, excluding comments and whitespace. This makes functions easier to understand, test, and maintain. If a function does exceed this limit, it **should** be refactored into smaller functions. Linter configuration: for golangci-lint, you can use the `funlen` linter and set the `max-lines` setting to `50`.

* **Limit Function Complexity**: functions **should** have a cyclomatic complexity of 10 or less. High cyclomatic complexity **may** indicate that a function is doing too much and **should** be refactored. Linter configuration: for golangci-lint, you can use the `gocyclo` linter and set the `min-complexity` setting to `10`.

    ```go
    // Discouraged
    func processNumbers(numbers []int) {
        for _, num := range numbers {
            if num > 10 {
                // ...
                if num > 20 {
                    // ...
                }
            }
        }
    }

    // Recommended
    func processNumbers(numbers []int) {
        for _, num := range numbers {
            processNumber(num)
        }
    }

    func processNumber(num int) {
        if num > 10 {
            // ...
        }
        if num > 20 {
            // ...
        }
    }
    ```


## Function Parameters

When designing functions, care should be taken to limit the number of parameters that a function requires. This makes the function easier to use and understand.

* functions **should not** have more than three positional parameters, excluding special parameters like `context.Context` and `sync.WaitGroup`. If needed, consider passing a single struct with all parameters.

    ```go
    func CreateOrder(ctx context.Context, customerName, customerAddress, productName, productCode string) {  // Not recommended
        ...
    }
    
    type OrderParams struct {
        CustomerName    string
        CustomerAddress string
        ProductName     string
        ProductCode     string
    }
    
    func CreateOrder(ctx context.Context, params OrderParams) {  // Recommended
        ...
    }
    ```

* **should**: prefer multiple functions with fewer parameters over single functions with more parameters.

    ```go
    func CreateOrderWithCustomerAndProduct(ctx context.Context, customer Customer, product Product) {  // Not recommended
        ...
    }
    
    func CreateOrderWithCustomer(ctx context.Context, customer Customer) {  // Recommended
        ...
    }
    
    func CreateOrderWithProduct(ctx context.Context, product Product) {  // Recommended
        ...
    }
    ```

* **must not**: functions must not use boolean parameters to determine behavior. Instead, consider creating separate functions.

    ```go
    func ReturnData(data []byte, asJSON bool) {  // Not recommended
        if asJSON {
            // Do something
        } else {
            // Do something else
        }
    }
    
    func ReturnJSON(data []byte) {  // Recommended
        // Do something
    }
    
    func ReturnProto(data []byte) {  // Recommended
        // Do something else
    }
    ```

* **must not** return more than three parameters. Although Go allows this, it is considered bad practice in most cases;
* **must** return an error as the last return value if a method returns an error.


## Functional Options Pattern

Functional options are an idiomatic way of creating APIs with options on types. This design pattern was first presented in an article by Rob Pike called [Self-referential functions and the design of options](https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html) and in Dave Cheney's [Functional options for friendly APIs](https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis).

Benefits:

* functional options let you write APIs that can grow over time;
* they enable the default use case to be the simplest;
* they provide meaningful configuration parameters;
* they give you access to the full power of the language to initialize complex values.

    ```go
    package main
    
    import "fmt"
    
    type Foo struct {
        Host                 string
        Enabled              bool
        DaysWithoutIncidents int
    }
    
    type option func(*Foo) error
    
    func WithEnabled(enabled bool) option {
        return func(f *Foo) error {
            f.Enabled = enabled
            return nil
        }
    }
    
    func WithDaysWithoutIncidents(daysCount int) option {
        return func(f *Foo) error {
            if daysCount < 1 {
                return fmt.Errorf("invalid value for option: %d", daysCount)
            }
            f.DaysWithoutIncidents = daysCount
            return nil
        }
    }
    
    func NewFoo(host string, opts ...option) (*Foo, error) {
        // Default options
        f := &Foo{
            Host:                 host,
            Enabled:              false,
            DaysWithoutIncidents: 0,
        }
        for _, opt := range opts {
            if err := opt(f); err != nil {
                return nil, err
            }
        }
        return f, nil
    }
    
    func main() {
        f, err := NewFoo("foo", WithEnabled(true), WithDaysWithoutIncidents(20))
    }
    ```


## Forbidden Dependency Injection Containers

Avoid using dependency injection libraries like `facebook-go/inject` or `uber-go/dig`. These can obfuscate dependencies and make the code harder to understand. Dependencies **should** be passed as arguments to constructor functions.

It is also recommended to follow the [dependency inversion principle](https://readmedium.com/the-dependency-inversion-principle-dip-in-golang-fb0bdc503972).


## Accept Interfaces, Return Structs

* constructor functions **must not** return interfaces;
* constructor functions **should** accept interfaces as arguments.

    ```go
    type Repository interface {
        Get(id int) error
    }
    
    type Repo struct {}
    
    func (r *Repo) Get(id int) error { return nil }
    
    func NewRepo() Repository { // Not recommended
        return &Repo{}
    }
    
    func NewRepo() *Repo { // Recommended
        return &Repo{}
    }
    ```


## Forbidden `return` without Values and Named Returnable Variables

Avoid using `return` statements without explicit values.


## ORM vs SQL

Should use ORM only for internal tools like admin panels, CRUD forms, and projects without heavy load. All other repositories must use plain SQL queries. We recommend using `database/sql` or `github.com/jmoiron/sqlx` packages to work with RDBMS.

    ```go
    // Bad
    type User struct {
        gorm.Model
        Name  string
        Email string
    }
    
    func (u *User) TableName() string {
        return "user"
    }
    
    func GetUser(id uint) (User, error) {
        var user User
        result := db.First(&user, id)
        return user, result.Error
    }
    
    // Good
    type User struct {
        ID    int
        Name  string
        Email string
    }
    
    func GetUser(db *sql.DB, id int) (*User, error) {
        row := db.QueryRow("SELECT id, name, email FROM users WHERE id = ?", id)
    
        var user User
        err := row.Scan(&user.ID, &user.Name, &user.Email)
        if err != nil {
            return nil, err
        }
    
        return &user, nil
    }
    ```


## Use of Global Variables is Forbidden

Global variables make code harder to understand and debug. Avoid them whenever possible.

* should not use global variables;
* should not use `init` functions, as they should not perform actions that could cause an error.

Exceptions include package errors and registries, like `prometheus.Registry`.

```go 
package something

var ErrNotFound = errors.New("not found")
```

**Bad Example**

```go
func main() {
	db := // ...
	handlers := Handlers{DB: db}
	http.HandleFunc("/drop", handlers.DropHandler)
	// ...
}

type Handlers struct {
	DB *sql.DB
}

func (h *Handlers) DropHandler(w http.ResponseWriter, r *http.Request) {
	h.DB.Exec("DROP DATABASE foo")
}
```


## Avoid Using `new` Keyword

The `new` keyword in Go allocates zeroed storage for a new item of a specified type and returns its address, a value of type `*T`.

While `new` can be useful in certain scenarios, it's generally recommended to avoid using it, as it can lead to code that is harder to read and understand. Instead, prefer to use composite literals, which are more readable and expressive.

Here is an example:

```go
// Discouraged
user := new(User)
user.Name = "Alice"
user.Age = 30

// Recommended
user := &User{
    Name: "Alice",
    Age:  30,
}
```


## `nil` is a Valid Slice

`nil` is a valid slice of length 0. This means that:

* you **should not** return a slice of length zero explicitly. Return `nil` instead.

**Bad example**:

```go
if x == "" {
    return []int{}
}
```

**Good example**:

```go
if x == "" {
    return nil
}
```

* to check if a slice is empty, **must** use `len(s) == 0`. Do not check for `nil`.

**Bad example**

```go
func isEmpty(s []string) bool {
    return s == nil
}
```

**Good example**

```go
func isEmpty(s []string) bool {
    return len(s) == 0
}
```


## Struct Fields Must Be Named (in Assignment)

When you instantiate a struct, you must always name the fields, even if the order in the code matches the order in the struct declaration. This makes the code more robust against changes in struct field order and more readable. Exceptions could be made for tests and single-field structures.

Here is an example:

```go
// Discouraged
user := User{"Alice", 30}

// Recommended
user := User{
    Name: "Alice",
    Age:  30,
}
```


## Don't Panic

Code running in production must avoid panics. Panics are a major source of [cascading failures](https://en.wikipedia.org/wiki/Cascading_failure). If an error occurs, the function must return an error and allow the caller to decide how to handle it. Avoid using `os.Exit()` and `log.Fatal()` as well, because they do not allow the application to release resources properly.

**Bad example**

```go
func run(args []string) {
    if len(args) == 0 {
        panic("an argument is required")
    }
    // ...
}

func main() {
    run(os.Args[1:])
}
```

**Good example**:

```go
func run(args []string) error {
    if len(args) == 0 {
        return errors.New("an argument is required")
    }
    // ...
    return nil
}

func main() {
    os.Exit(gracefulMain())
}

func gracefulMain() int {
    if err := run(os.Args[1:]); err != nil {
        fmt.Fprintln(os.Stderr, err)
        return 1
    }
    return 0
}
```

Panic / recover is not an error handling strategy. A program should panic only when something irrecoverable happens, such as a `nil` dereference. An exception to this is program initialization: fatal issues at startup that should abort the program may cause a panic.

```go
var _statusTemplate = template.Must(template.New("name").Parse("_statusHTML"))
```

Even in tests, prefer `t.Fatal` or `t.FailNow` over panics to ensure that the test is marked as failed.

**Bad example**:

```go
// func TestFoo(t *testing.T)

f, err := os.CreateTemp("", "test")
if err != nil {
    panic("failed to set up test")
}
```

**Good example**

```go
// func TestFoo(t *testing.T)

f, err := os.CreateTemp("", "test")
if err != nil {
    t.Fatal("failed to set up test")
}
```


## Context Handling and Propagation

Context is a powerful tool in Go for managing and canceling long-running processes, especially in concurrent and distributed systems. It allows you to pass request-scoped values, deadlines, and cancellation signals through the call chain of goroutines.

When working with context, it is important to follow these best practices:

1. **Create Context**: always create a context at the entry point of a request or operation. The context should be derived from `context.Background()` or an existing context.

   ```go
   ctx := context.Background()
   ```

2. **Pass context**: pass the context as an argument to all functions and methods that need access to it. This ensures that the context is available to all relevant parts of the code. Context must be the first argument of a function.

   ```go
   func DoSomething(ctx context.Context) {
       // Use the context within the function
   }
   ```

3. **Context values**: use context values to store request-scoped information.

    * **must not** pass context values through function arguments;
    * **must not** use context values for passing optional parameters.

   ```go
   type key string

   // Define a key for the request-scoped value
   const userIDKey key = "userID"

   func SetUserID(ctx context.Context, userID int) context.Context {
       return context.WithValue(ctx, userIDKey, userID)
   }

   func GetUserID(ctx context.Context) (int, bool) {
       userID, ok := ctx.Value(userIDKey).(int)
       return userID, ok
   }
   ```

4. **Cancellation**: use context cancellation to terminate long-running processes. Propagate the cancellation signal through the context tree to ensure that all goroutines are properly cleaned up.

    * **must** check for cancellation during long-running operations.
    * **must** return the cancellation error to the caller.
    * **must not** ignore the cancellation error.

   ```go
   func DoSomething(ctx context.Context) error {
       for {
           // Check for cancelation
           if ctx.Err() != nil {
               return ctx.Err()
           }

           // Do some work

           // Check for cancelation again
           if ctx.Err() != nil {
               return ctx.Err()
           }

            // Do another work
       }
   }
   ```

Avoid the following when working with context:

* **must not** store context in a struct: context is designed to be passed explicitly and scoped to a specific request or operation. Storing a context in a struct can lead to context leaks and unexpected behavior.
* **must not** pass a nil context: always make sure to create and pass a valid context. Passing a nil context will result in a panic.

By using context correctly, you can effectively manage and cancel long-running processes in a clean and controlled manner.

For more information, refer to the [Context package documentation](https://pkg.go.dev/context) and [Effective Go](https://golang.org/doc/effective_go#concurrency).


## Slices of Pointers

A slice of pointers is a slice where each element is a pointer to a value, rather than the value itself. This means that the slice stores the memory addresses of several values, rather than the values themselves.

Should not return a slice of pointers: Returning a slice of pointers can lead to memory leaks and unexpected errors. Always check for `nil` before dereferencing a pointer.


### Potential Issues and Complications

* **memory leaks**: creating pointers to values that are never used can lead to memory leaks;
* **increased GC pressure**: using pointers increases heap allocations, which can add pressure on the garbage collector, leading to more frequent GC cycles and higher pause times;
* **garbage collection**: having fewer pointers reduces GC pause times, as the garbage collector traces fewer pointers to determine which objects are still in use;
* **nil pointer dereferences**: pointers can be `nil`, and dereferencing a nil pointer causes a runtime panic. Always check for `nil` before dereferencing pointers to avoid panics;
* **mutation side effects**: since pointers store memory addresses, inadvertently modifying the underlying value from different parts of your code can lead to unexpected side effects and bugs. This can make debugging more challenging;
* **slower access times**: dereferencing pointers is slightly slower than accessing values directly, especially when frequent, due to the additional indirection;
* **larger memory footprint**: pointers take up memory (usually 4 or 8 bytes, depending on the architecture). Therefore, a slice of pointers will generally use more memory than a slice of values;
* **concurrency issues**: in a concurrent program, pointers can lead to race conditions if multiple goroutines access and modify the same memory locations concurrently. Careful synchronization is necessary to prevent data races.

**Bad example**:

```go
func ReturnSliceWithPointers(size int) []*Person {
    res := make([]*Person, size)
    for i := 0; i < size; i++ {
        res[i] = &Person{}
    }
    return res
}
```

**Good example**

```go
func ReturnSliceWithStructs(size int) []Person {
    res := make([]Person, size)
    for i := 0; i < size; i++ {
        p := Person{}
        res[i] = p
    }
    return res
}
```


### Benchmark Results

```text
Benchmark_ReturnSliceWithPointers-12      10000    106092 ns/op    161921 B/op   10001 allocs/op
Benchmark_ReturnSliceWithStructs-12       10000     10478 ns/op     81920 B/op       1 allocs/op
Benchmark_ReturnSliceWithPointersBoth-12  10000     29902 ns/op    163840 B/op       2 allocs/op
```


## Handling Pointers

For more information, refer to the [Pointers](https://go.dev/doc/faq#Pointers) section of the Go FAQ.