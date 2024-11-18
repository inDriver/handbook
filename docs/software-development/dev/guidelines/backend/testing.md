# GoLang Test Guide


## General

1. The `testing` package must be imported to provide testing functionality.
2. Generated mock files should not be placed in a `mocks` directory; instead, place them close to the interface.
3. Mock files should be included in code coverage.


## Libraries

1. The `github.com/stretchr/testify` library should be used to make tests clearer and more readable. It provides a comprehensive set of functions and is widely adopted in the Go community.
2. The `github.com/stretchr/testify/mock` library should be used for creating mocks in tests, offering a flexible and easy-to-use API for mocking.
3. `reflect.DeepEqual` must not be used when `assert.Equal` or `assert.EqualValues` are suitable alternatives. The `reflect.DeepEqual` function can have subtle nuances, potentially causing false positives and negatives in tests.

   ```go
   // DO THIS
   assert.Equal(t, expected, actual)
   
   // NOT THIS
   if !reflect.DeepEqual(expected, actual) {
       t.Errorf("Expected and actual do not match.")
   }
   ```

4. If the test cannot proceed after an assertion failure, the `require` package should be used instead of `assert`.

   ```go
   // DO THIS
   require.Equal(t, expected, actual)
   
   // NOT THIS
   assert.Equal(t, expected, actual)
   ```

The `require` package stops test execution after a failing assertion, which is preferable when subsequent tests depend on previous assertions.


## Mocking with Testify Mockery

>**NOTE**
> 
> This section is not complete, as the common library is not fully adopted.

When mocking dependencies for testing, the following guidelines should be observed:

1. Mocks must be created using the `mockery` tool, a flexible library for creating and using mock objects in tests.
2. Mock objects should be used to isolate the code under test from its dependencies.

Example:

```go
type DBMock struct {
    mock.Mock
}

func (db *DBMock) FetchData(id string) (Data, error) {
    args := db.Called(id)
    return args.Get(0).(Data), args.Error(1)
}

func TestFetchData(t *testing.T) {
    db := new(DBMock)
    db.On("FetchData", "123").Return(Data{"123", "test data"}, nil)

    res, err := db.FetchData("123")

    db.AssertExpectations(t)
    assert.NoError(t, err)
    assert.Equal(t, Data{"123", "test data"}, res)
}
```


## Structure

1. Each test function should test only a single function or method to promote clarity and ease of maintenance.
2. Table-driven tests should be used when testing multiple scenarios for the same function, improving readability and making it easier to add new cases.
3. Table-driven tests should not be used for complex functions that require extensive setup. In such cases, use subtests with `t.Run()` instead.
4. For each table-driven test, use a struct for input and expected results.

   ```go
   tests := []struct {
       name     string
       args     args
       expected Type
   }{
       // test cases here
   }
   ```

5. Test functions should not return any values; they should use the `Error`, `Fail`, or related methods to signal failure.


## Naming

1. Test function names must be descriptive so that the purpose of a test is clear without reading the implementation.
2. Error messages should provide clear context for failures, making diagnosis easier.
3. If testing a specific method of a struct, the test function should follow the naming convention `TestStruct_Method`. For unexported structs, should use `Test_struct_method` format.


## Test Encapsulation

1. Mocks should be used when testing functions that depend on external services.
2. Use interfaces to create decoupled code that is easier to test with mock implementations.
3. Avoid using global variables as they can cause issues with maintaining state between tests. If needed, encapsulate global state within a struct and mock it in the test.

## Test Cases

1. You should include both positive and negative test cases.
2. Test cases must cover all edge cases and boundary conditions. If a function behaves differently with an empty slice, zero value, or a specific boundary case, a test case must cover this scenario.
3. Test cases should verify the correctness of a function rather than its performance. Performance tests can be written separately as benchmarks.


## Execution

1. Tests should be able to run in any order and must not rely on the state from other tests or the order of execution.
2. Tests should be designed to run quickly, encouraging frequent test runs and faster feedback.
3. Tests should not rely on time checks, which can lead to flaky tests and false positives.

      ```go
      func TestCheckRateLimiterStatus(t *testing.T) {
          ...
          <-time.After(time.Second) // Bad practice
          doCheck()
          ...
      }
      ```
   
4. Tests must not rely on map keys or struct fields for order.

   Following these guidelines will help ensure your Go tests are effective, maintainable, and easy to understand. Each rule should be considered in the context of your specific use case, with exceptions permissible when appropriate.


## Integration Tests

1. Should have a build tag "integration" to separate them from unit tests.
2. Should be placed in the same package as the code they’re testing.
3. To set up the environment for integration tests, should use a repeatable environment, such as [Dockertest](https://github.com/ory/dockertest) or Docker Compose.
4. The [Testify](https://github.com/stretchr/testify) suite should be used to organize integration tests, including setup and teardown steps to manage the lifecycle of Dockertest containers.

   Here’s a simplified example:

   ```go
   type MyTestSuite struct {
       suite.Suite
       pool     *dockertest.Pool
       resource *dockertest.Resource
   }

   // SetupTest runs before each test in the suite
   func (s *MyTestSuite) SetupTest() {
       var err error
       s.pool, err = dockertest.NewPool("")
       s.Require().NoError(err)

       s.resource, err = s.pool.Run("mysql", "5.7", []string{"MYSQL_ROOT_PASSWORD=secret"})
       s.Require().NoError(err)

       // Allow time for the database to initialize
       time.Sleep(time.Second * 10)
   }

   // TearDownTest runs after each test in the suite
   func (s *MyTestSuite) TearDownTest() {
       if err := s.pool.Purge(s.resource); err != nil {
           log.Fatalf("Could not purge resource: %s", err)
       }
   }

   // An example test within the suite
   func (s *MyTestSuite) TestSomething() {
       // Test logic here
   }

   func TestMyTestSuite(t *testing.T) {
       suite.Run(t, new(MyTestSuite))
   }
   ```