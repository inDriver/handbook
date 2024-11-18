# Guideline for Comments and Annotations in Go


## General Recommendations

Comments must start with a capital letter and end with a period. They should be full sentences, as they serve as documentation. Here's an example:

```go
// This is a correctly formatted comment.
```

Good comments explain why, not what. They also provide context and explain the purpose of a piece of code.

* comments should not be redundant; do not comment on something that can be clearly understood from the code itself;
* try to express meaning directly in code. If a comment is needed to explain what a piece of code does, consider rewriting the code to make it clearer;
* avoid writing comments when you should be writing a function or finding a better name. Comments should not make up for unclear code;
* commented-out code must not be left in the final version. It clutters the codebase and confuses other developers.

In accordance with these principles, here’s an example of a well-commented piece of code:

```go
// ComputeAverage calculates the average of an array of numbers.
// It returns zero if the array is empty.
func ComputeAverage(numbers []int) float64 {
    if len(numbers) == 0 {
        return 0
    }
    sum := 0
    for _, num := range numbers {
        sum += num
    }
    return float64(sum) / float64(len(numbers))
}
```


## Types of Comments


### Package Comments

**Purpose:** describe the package's functionality and any essential details.

```go
// Package calculator provides basic operations like addition and subtraction.
package calculator
```

### Function Comments

**Function header comments:** required for all exported functions. describe the function, its parameters, and return values.

```go
// Add returns the sum of two integers.
func Add(a, b int) int {
    return a + b
}
```

### Type Comments

**Type descriptions:** explain the purpose and structure of exported types.

```go
// Point represents a point in 2D space.
type Point struct {
    X, Y int
}
```


### Method Comments

**Method header comments:** describe the method, its parameters, and return values.

```go
// Distance calculates the Euclidean distance between two points.
func (p *Point) Distance(q *Point) float64 {
    dx := p.X - q.X
    dy := p.Y - q.Y
    return math.Sqrt(float64(dx*dx + dy*dy))
}
```


### Inline Comments

**Explain complex logic:** use sparingly to explain non-obvious parts of the code.

```go
for i := 0; i < 10; i++ {
    // Skip even numbers.
    if i%2 == 0 {
        continue
    }
    process(i)
}
```


## Annotations


### TODO Comments

**Mark incomplete work:** indicate areas that need further development or improvement.

```go
// TODO: optimize this algorithm for better performance.
```


### FIXME Comments

**Highlight known issues:** draw attention to parts of the code that have known issues.

```go
// FIXME: this function does not handle edge cases correctly.
```


### Deprecated Comments

**Indicate deprecated code:** mark deprecated functions or features with alternatives if available.

```go
// Deprecated: use NewFunction instead.
```


## Style and Formatting

* **[use GoDoc style](https://go.dev/doc/comment):** ensure comments are formatted for GoDoc compatibility;
* **proper grammar:** use complete sentences, correct grammar, and punctuation;
* **consistent style:** maintain a uniform commenting style throughout the codebase.