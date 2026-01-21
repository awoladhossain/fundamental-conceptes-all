# Go (Golang) In-Depth: Junior to Senior Level Knowledge

A comprehensive guide to mastering Go from fundamentals to advanced concepts for building complex backend systems.

---

## 1. GO BASICS & FUNDAMENTALS

### What is Go?

Go (Golang) is a statically-typed, compiled language created by Google. It emphasizes:
- **Simplicity**: Easy to learn and read
- **Speed**: Fast compilation and execution
- **Concurrency**: Built-in support for concurrent programming
- **Efficiency**: Small binary size, low memory footprint

Go is ideal for:
- Backend services and APIs
- Microservices
- CLI tools
- System tools
- Cloud infrastructure

### Your First Program

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

**Key Points**:
- Every Go program needs a `main` package
- The `main()` function is the entry point
- `import` statements import packages

### Variables and Constants

```go
// Variable declaration (explicit type)
var name string = "John"
var age int = 30
var salary float64 = 50000.50

// Variable declaration (type inference)
var city = "New York"

// Short declaration (inside functions only)
message := "Hello"
count := 5

// Multiple declarations
var (
    firstName string = "John"
    lastName  string = "Doe"
    age       int    = 30
)

// Constants (values that cannot change)
const pi = 3.14159
const maxUsers int = 1000

// Multiple constants
const (
    statusActive   = "active"
    statusInactive = "inactive"
    statusPending  = "pending"
)
```

**Important**: Use `:=` for short variable declaration inside functions. Use `var` at package level.

### Zero Values

Each type has a zero value (default value when not initialized):

```go
var str string      // ""
var num int         // 0
var flag bool       // false
var ptr *int        // nil
var slice []int     // nil
var m map[string]int // nil
```

### Data Types

**Numeric Types**:
```go
// Integers
var i8 int8 = 127              // -128 to 127
var i16 int16 = 32767          // -32,768 to 32,767
var i32 int32 = 2147483647     // -2^31 to 2^31-1
var i64 int64 = 9223372036854775807
var ui uint = 100               // Unsigned (0 and positive)
var ui8 uint8 = 255
var ui16 uint16 = 65535
var ui32 uint32 = 4294967295
var ui64 uint64 = 18446744073709551615

// The 'int' and 'uint' types are platform-dependent
// On 32-bit systems: 32 bits
// On 64-bit systems: 64 bits

// Floating point
var f32 float32 = 3.14
var f64 float64 = 2.71828

// Complex numbers
var c64 complex64 = 1 + 2i
var c128 complex128 = 3 + 4i
```

**Boolean**:
```go
var flag bool = true
var active bool = false
```

**String**:
```go
// String literal (with escape sequences)
var str string = "Hello\nWorld"

// Raw string literal (no escape sequences)
var raw string = `Hello\nWorld` // Prints literally with \n
```

**Type Conversion**:
```go
// Go requires explicit type conversion
var i int = 42
var f float64 = float64(i)  // Convert int to float64

var s string = "123"
var num int64
num, _ = strconv.ParseInt(s, 10, 64) // Convert string to int

// Using fmt for type conversion
str := fmt.Sprintf("%d", 42) // Convert int to string
```

---

## 2. FUNCTIONS & METHODS

### Function Declaration

```go
// Basic function
func add(a int, b int) int {
    return a + b
}

// Shorthand for same types
func multiply(a, b int) int {
    return a * b
}

// Multiple return values
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

// Named return values
func swap(a, b string) (first, second string) {
    first = b
    second = a
    return // Naked return (implicit return of named return values)
}

// Variadic functions (variable number of arguments)
func sum(numbers ...int) int {
    total := 0
    for _, num := range numbers {
        total += num
    }
    return total
}

sum(1, 2, 3, 4, 5) // 15

// Passing slices to variadic functions
nums := []int{1, 2, 3}
sum(nums...) // Unpack slice into variadic arguments
```

### Defer Statement

`defer` postpones function execution until the surrounding function returns.

```go
func readFile(filename string) {
    file, err := os.Open(filename)
    if err != nil {
        return
    }
    defer file.Close()  // Close file when function returns

    // Use file...
}

// Defer with multiple statements
func example() {
    defer fmt.Println("Third")
    defer fmt.Println("Second")
    fmt.Println("First")
}
// Output:
// First
// Second
// Third
// Defers are executed in LIFO (Last In, First Out) order

// Defer with function arguments (evaluated immediately)
func deferDemo() {
    i := 10
    defer fmt.Println(i)  // Prints 10 (value captured at defer time)
    i = 20
}

// Useful pattern: Acquire/Release
func acquireResource() Resource {
    r := getResource()
    defer r.Release()
    return r
}
```

### Methods

Methods are functions with a receiver argument.

```go
type Rectangle struct {
    width, height float64
}

// Method with value receiver
func (r Rectangle) Area() float64 {
    return r.width * r.height
}

// Method with pointer receiver (can modify the receiver)
func (r *Rectangle) Scale(factor float64) {
    r.width *= factor
    r.height *= factor
}

// Usage
rect := Rectangle{10, 5}
fmt.Println(rect.Area())  // 50

rect.Scale(2)
fmt.Println(rect.Area())  // 200

// Pointer receivers are useful when:
// 1. You need to modify the receiver
// 2. You want to avoid copying large structs
```

### Anonymous Functions & Closures

```go
// Anonymous function
func() {
    fmt.Println("Anonymous function")
}()  // Immediately invoked

// Closure
func counter() func() int {
    count := 0
    return func() int {
        count++
        return count
    }
}

c := counter()
fmt.Println(c()) // 1
fmt.Println(c()) // 2
fmt.Println(c()) // 3
```

---

## 3. PACKAGES & IMPORTS

### Package Structure

Go code is organized in packages. A package is a collection of source files in the same directory.

```go
// main.go
package main

import (
    "fmt"
    "math"
    "myapp/util"  // Local package
)

func main() {
    fmt.Println(math.Pi)
    fmt.Println(util.Add(5, 3))
}

// util/math.go
package util

func Add(a, b int) int {
    return a + b
}

// Note: Exported functions/variables start with UPPERCASE
// Unexported (private) start with lowercase
```

### Blank Identifier

```go
// Import for side effects
import _ "myapp/migrations"

// Ignore return values
_, err := someFunction()
```

### Package Visibility

```go
// Public (exported) - starts with uppercase
func PublicFunction() {}
var PublicVariable = 10

// Private (unexported) - starts with lowercase
func privateFunction() {}
var privateVariable = 10
```

---

## 4. DATA STRUCTURES: ARRAYS, SLICES, AND MAPS

### Arrays

Arrays have fixed length.

```go
// Declare array
var arr [5]int  // [0 0 0 0 0]

// Array literal
arr := [5]int{1, 2, 3, 4, 5}

// Size inferred from initializer
arr := [...]int{1, 2, 3, 4, 5}

// Access elements
arr[0] = 10
value := arr[1]

// Length
len(arr) // 5

// Iterate
for i := 0; i < len(arr); i++ {
    fmt.Println(arr[i])
}

// Iterate with range
for i, v := range arr {
    fmt.Printf("Index: %d, Value: %d\n", i, v)
}

// Ignore index
for _, v := range arr {
    fmt.Println(v)
}
```

### Slices

Slices are dynamic arrays with variable length.

```go
// Slice declaration
var slice []int  // nil slice

// Create slice
slice := []int{1, 2, 3, 4, 5}

// From array
arr := [5]int{1, 2, 3, 4, 5}
slice := arr[1:4]  // Elements at index 1, 2, 3 (not 4)

// Using make
slice := make([]int, 5)      // Length 5, capacity 5
slice := make([]int, 5, 10)  // Length 5, capacity 10

// Append (grows slice if needed)
slice = append(slice, 6)      // [1 2 3 4 5 6]
slice = append(slice, 7, 8)   // [1 2 3 4 5 6 7 8]

// Append another slice
slice = append(slice, otherSlice...)

// Length and capacity
len(slice)  // Current number of elements
cap(slice)  // Maximum elements before reallocation

// Copy slices
original := []int{1, 2, 3}
copy := make([]int, len(original))
copy(copy, original)

// Slice operations
arr := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
slice := arr[2:6]     // [2 3 4 5]
slice := arr[2:]      // [2 3 4 5 6 7 8 9]
slice := arr[:6]      // [0 1 2 3 4 5]
slice := arr[:]       // [0 1 2 3 4 5 6 7 8 9]
```

**Important**: Slices are references to underlying arrays. Modifying a slice modifies the underlying array.

```go
arr := [5]int{1, 2, 3, 4, 5}
slice := arr[1:4]
slice[0] = 99

fmt.Println(arr) // [1 99 3 4 5] - Original array is modified!
```

### Maps

Maps are key-value data structures (like dictionaries/hash tables).

```go
// Declare map
var m map[string]int  // nil map

// Map literal
m := map[string]int{
    "one":   1,
    "two":   2,
    "three": 3,
}

// Using make
m := make(map[string]int)
m["one"] = 1
m["two"] = 2

// Get value
value := m["one"]  // 1

// Check if key exists
value, ok := m["one"]
if ok {
    fmt.Println("Key exists:", value)
}

// Delete key
delete(m, "one")

// Length
len(m)

// Iterate
for key, value := range m {
    fmt.Printf("Key: %s, Value: %d\n", key, value)
}

// Only keys
for key := range m {
    fmt.Println(key)
}

// Map of slices
m := map[string][]int{
    "even": {2, 4, 6},
    "odd":  {1, 3, 5},
}

// Nested maps
m := map[string]map[string]int{
    "user1": {"age": 30, "score": 95},
    "user2": {"age": 25, "score": 87},
}
```

---

## 5. STRUCTS AND INTERFACES

### Structs

A struct is a collection of fields.

```go
// Define struct
type Person struct {
    Name string
    Age  int
    City string
}

// Create instance
person := Person{
    Name: "John",
    Age:  30,
    City: "New York",
}

// Positional initialization (order matters)
person := Person{"John", 30, "New York"}

// Partial initialization
person := Person{Name: "John", Age: 30}
// City will be empty string (zero value)

// Accessing fields
fmt.Println(person.Name)
person.Age = 31

// Pointer to struct
var p *Person
p = &person
p.Age = 32  // Can access fields via pointer (automatic dereferencing)

// New function
p := new(Person)  // Returns *Person
p.Name = "Alice"

// Struct embedding (composition)
type Address struct {
    Street string
    City   string
}

type Employee struct {
    Name    string
    Address  // Embedded struct (promotes fields)
}

emp := Employee{
    Name: "Bob",
    Address: Address{
        Street: "123 Main St",
        City:   "Boston",
    },
}

fmt.Println(emp.City)  // Can access embedded struct fields directly
fmt.Println(emp.Address.City)  // Or explicitly

// Anonymous structs
user := struct {
    Name  string
    Email string
}{
    Name:  "John",
    Email: "john@example.com",
}
```

### Interfaces

An interface defines a set of method signatures (contracts).

```go
// Define interface
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

// Implement interface
type File struct {
    name string
}

func (f *File) Read(p []byte) (int, error) {
    // Implementation
    return len(p), nil
}

// Interfaces are implicitly satisfied
// File implements Reader because it has Read method

// Multiple interfaces
type ReadWriter interface {
    Reader
    Writer
}

// Empty interface (can hold any type)
var i interface{}
i = 42
i = "hello"
i = []int{1, 2, 3}

// Type assertion
value := i.(string)  // Assert i is string

value, ok := i.(string)  // Safe type assertion
if ok {
    fmt.Println("Value is string:", value)
}

// Type switch
switch v := i.(type) {
case string:
    fmt.Println("String:", v)
case int:
    fmt.Println("Int:", v)
case []int:
    fmt.Println("Slice:", v)
default:
    fmt.Println("Unknown type")
}

// Interface composition
type Reader interface {
    Read() string
}

type Writer interface {
    Write(string) error
}

type ReadWriter interface {
    Reader
    Writer
}

// Implementing ReadWriter
type File struct {}

func (f *File) Read() string {
    return "data"
}

func (f *File) Write(data string) error {
    return nil
}
```

---

## 6. ERROR HANDLING

### The Error Interface

```go
type error interface {
    Error() string
}

// Most functions return error as last return value
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

// Check error immediately
result, err := divide(10, 0)
if err != nil {
    fmt.Println("Error:", err)
    // Handle error
}
```

### Custom Errors

```go
type ValidationError struct {
    Field   string
    Message string
}

func (e ValidationError) Error() string {
    return fmt.Sprintf("validation error: %s - %s", e.Field, e.Message)
}

// Usage
func validateEmail(email string) error {
    if !strings.Contains(email, "@") {
        return ValidationError{
            Field:   "email",
            Message: "must contain @",
        }
    }
    return nil
}

// Type assertion on error
err := validateEmail("invalid")
if err != nil {
    if ve, ok := err.(ValidationError); ok {
        fmt.Println("Field:", ve.Field)
    }
}
```

### Error Wrapping (Go 1.13+)

```go
import "fmt"

// Wrap error with context
result, err := someOperation()
if err != nil {
    return fmt.Errorf("failed to do something: %w", err)
}

// Check if error is of specific type
if errors.Is(err, os.ErrNotExist) {
    // Handle file not found
}

// Get error of specific type
var validationErr ValidationError
if errors.As(err, &validationErr) {
    fmt.Println(validationErr.Field)
}
```

### Panic and Recover

```go
// Panic stops execution and causes program to crash
func riskyOperation() {
    panic("Something went wrong!")
}

// Recover catches panic
func safe() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered from:", r)
        }
    }()

    riskyOperation()
}

// Use sparingly - prefer explicit error handling
```

---

## 7. GOROUTINES & CONCURRENCY

### Goroutines

A goroutine is a lightweight thread managed by the Go runtime.

```go
// Start goroutine
go function()

func main() {
    go sayHello()
    fmt.Println("Main function")
    time.Sleep(1 * time.Second)  // Give goroutine time to finish
}

func sayHello() {
    fmt.Println("Hello from goroutine")
}

// Without sleep, program exits before goroutine completes
```

### Channels

Channels are the primary way goroutines communicate.

```go
// Create channel
var ch chan int
ch := make(chan int)

// Send value
ch <- 42

// Receive value
value := <-ch

// Buffered channel
ch := make(chan int, 5)  // Can hold 5 values before blocking

// Close channel
close(ch)

// Range over channel
for value := range ch {
    fmt.Println(value)
}

// Check if channel is closed
value, ok := <-ch
if !ok {
    fmt.Println("Channel is closed")
}
```

### Channel Directions

```go
// Send-only channel
func send(ch chan<- int) {
    ch <- 42
}

// Receive-only channel
func receive(ch <-chan int) {
    value := <-ch
    fmt.Println(value)
}

// Bidirectional channel (default)
func bidirectional(ch chan int) {
    ch <- 42
    value := <-ch
}
```

### Worker Pattern

```go
func main() {
    jobs := make(chan int, 10)
    results := make(chan string, 10)

    // Start 3 workers
    for i := 1; i <= 3; i++ {
        go worker(i, jobs, results)
    }

    // Send jobs
    for j := 1; j <= 5; j++ {
        jobs <- j
    }
    close(jobs)

    // Collect results
    for i := 0; i < 5; i++ {
        fmt.Println(<-results)
    }
}

func worker(id int, jobs <-chan int, results chan<- string) {
    for job := range jobs {
        time.Sleep(1 * time.Second)
        results <- fmt.Sprintf("Worker %d completed job %d", id, job)
    }
}
```

### Select Statement

```go
// Select waits for multiple channel operations
select {
case value := <-ch1:
    fmt.Println("Got from ch1:", value)
case value := <-ch2:
    fmt.Println("Got from ch2:", value)
case ch3 <- 42:
    fmt.Println("Sent to ch3")
default:
    fmt.Println("No channel ready")
}

// Timeout pattern
select {
case result := <-done:
    fmt.Println("Done:", result)
case <-time.After(5 * time.Second):
    fmt.Println("Timeout!")
}

// Non-blocking check
select {
case value := <-ch:
    fmt.Println(value)
default:
    fmt.Println("Channel not ready")
}
```

### Mutex (Mutual Exclusion)

Protect shared data from race conditions.

```go
import "sync"

type Counter struct {
    mu    sync.Mutex
    value int
}

func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.value++
}

func (c *Counter) Get() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.value
}

// Safe concurrent access
counter := &Counter{}
go counter.Increment()
go counter.Increment()
time.Sleep(100 * time.Millisecond)
fmt.Println(counter.Get())  // 2
```

### RWMutex (Read-Write Mutex)

Multiple readers, exclusive writer.

```go
type UserDB struct {
    mu    sync.RWMutex
    users map[int]string
}

func (db *UserDB) GetUser(id int) string {
    db.mu.RLock()  // Lock for reading
    defer db.mu.RUnlock()
    return db.users[id]
}

func (db *UserDB) SetUser(id int, name string) {
    db.mu.Lock()  // Lock for writing
    defer db.mu.Unlock()
    db.users[id] = name
}
```

---

## 8. CONTEXT PACKAGE

Context is used for cancellation, timeouts, and deadlines.

```go
import "context"

// Context with timeout
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

// Use in function
select {
case <-ctx.Done():
    fmt.Println("Context cancelled:", ctx.Err())
case result := <-ch:
    fmt.Println("Got result:", result)
}

// Context with deadline
deadline := time.Now().Add(10 * time.Second)
ctx, cancel := context.WithDeadline(context.Background(), deadline)
defer cancel()

// Pass context to function
func doWork(ctx context.Context) error {
    select {
    case <-ctx.Done():
        return ctx.Err()
    case <-time.After(3 * time.Second):
        fmt.Println("Work done")
        return nil
    }
}

// Context with value
ctx := context.WithValue(context.Background(), "user_id", 123)
userID := ctx.Value("user_id").(int)
```

---

## 9. SYNC PACKAGE

### WaitGroup

Synchronize multiple goroutines.

```go
import "sync"

func main() {
    var wg sync.WaitGroup

    for i := 1; i <= 5; i++ {
        wg.Add(1)  // Add goroutine to wait group
        go func(id int) {
            defer wg.Done()  // Mark as done
            fmt.Printf("Goroutine %d\n", id)
        }(i)
    }

    wg.Wait()  // Wait for all goroutines to finish
}
```

### Once

Execute a function exactly once.

```go
import "sync"

var (
    once sync.Once
    instance *Singleton
)

func getInstance() *Singleton {
    once.Do(func() {
        instance = &Singleton{}
    })
    return instance
}

// getInstance() will only create instance once
```

### Pool

Reuse objects to reduce allocation.

```go
import "sync"

func main() {
    bufferPool := sync.Pool{
        New: func() interface{} {
            return make([]byte, 0, 1024)
        },
    }

    // Get from pool
    buf := bufferPool.Get().([]byte)

    // Use buffer...

    // Return to pool
    buf = buf[:0]  // Reset
    bufferPool.Put(buf)
}
```

---

## 10. FILE I/O & STREAMS

### Reading Files

```go
import (
    "bufio"
    "io/ioutil"
    "os"
)

// Read entire file
data, err := ioutil.ReadFile("file.txt")
if err != nil {
    log.Fatal(err)
}
content := string(data)

// Read file with os.Open
file, err := os.Open("file.txt")
if err != nil {
    log.Fatal(err)
}
defer file.Close()

// Read line by line
scanner := bufio.NewScanner(file)
for scanner.Scan() {
    line := scanner.Text()
    fmt.Println(line)
}
if err := scanner.Err(); err != nil {
    log.Fatal(err)
}

// Read with buffer
reader := bufio.NewReader(file)
for {
    line, err := reader.ReadString('\n')
    if err != nil {
        if err == io.EOF {
            break
        }
        log.Fatal(err)
    }
    fmt.Print(line)
}
```

### Writing Files

```go
import (
    "bufio"
    "os"
)

// Write entire file
err := ioutil.WriteFile("file.txt", []byte("Hello, World!"), 0644)
if err != nil {
    log.Fatal(err)
}

// Write with os.Create
file, err := os.Create("file.txt")
if err != nil {
    log.Fatal(err)
}
defer file.Close()

writer := bufio.NewWriter(file)
writer.WriteString("Hello, ")
writer.WriteString("World!")
writer.Flush()  // Write buffered data

// Append to file
file, err := os.OpenFile("file.txt", os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
if err != nil {
    log.Fatal(err)
}
defer file.Close()

file.WriteString("\nNew line")
```

---

## 11. STRINGS & REGULAR EXPRESSIONS

### String Manipulation

```go
import (
    "fmt"
    "strings"
)

// Concatenation
s1 := "Hello"
s2 := "World"
result := s1 + " " + s2  // "Hello World"

// String functions
str := "Hello, World!"

len(str)                      // 13
strings.ToUpper(str)          // "HELLO, WORLD!"
strings.ToLower(str)          // "hello, world!"
strings.TrimSpace("  hello ")  // "hello"
strings.Split("a,b,c", ",")   // ["a" "b" "c"]
strings.Join([]string{"a", "b", "c"}, ",")  // "a,b,c"
strings.Contains(str, "World")  // true
strings.Index(str, "World")     // 7
strings.HasPrefix(str, "Hello") // true
strings.HasSuffix(str, "!")     // true
strings.Replace(str, "World", "Go", -1)  // "Hello, Go!"

// String formatting
name := "Alice"
age := 30
message := fmt.Sprintf("Name: %s, Age: %d", name, age)
```

### Regular Expressions

```go
import "regexp"

// Compile regex
re := regexp.MustCompile(`\d+`)

// Find matches
text := "Price: 100, Quantity: 5"
matches := re.FindAllString(text, -1)  // ["100" "5"]

// Find first match
firstMatch := re.FindString(text)  // "100"

// Replace
result := re.ReplaceAllString(text, "X")  // "Price: X, Quantity: X"

// Check if matches
isMatch := re.MatchString(text)  // true

// Common patterns
emailRegex := regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)
phoneRegex := regexp.MustCompile(`^\d{3}-\d{3}-\d{4}$`)
```

---

## 12. JSON HANDLING

### Marshal and Unmarshal

```go
import (
    "encoding/json"
    "fmt"
)

type Person struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
    City string `json:"city,omitempty"`  // Omit if empty
}

// Struct to JSON (Marshal)
person := Person{
    Name: "John",
    Age:  30,
    City: "New York",
}

data, err := json.Marshal(person)
if err != nil {
    log.Fatal(err)
}
fmt.Println(string(data))  // {"name":"John","age":30,"city":"New York"}

// Pretty print JSON (MarshalIndent)
data, _ := json.MarshalIndent(person, "", "  ")
fmt.Println(string(data))

// JSON to Struct (Unmarshal)
jsonStr := `{"name":"Alice","age":25,"city":"Boston"}`
var p Person
err := json.Unmarshal([]byte(jsonStr), &p)
if err != nil {
    log.Fatal(err)
}
fmt.Println(p.Name)  // Alice

// Struct tags
type Product struct {
    ID    int    `json:"product_id"`
    Name  string `json:"product_name"`
    Price float64 `json:"price,omitempty"`
}

// Ignore fields
type Config struct {
    Host string
    Port int
    Secret string `json:"-"`  // Not included in JSON
}
```

### Working with Dynamic JSON

```go
// Unmarshal to map
var data map[string]interface{}
json.Unmarshal(jsonBytes, &data)
name := data["name"].(string)

// Unmarshal to []interface{}
var items []interface{}
json.Unmarshal(jsonBytes, &items)

// Use json.RawMessage for custom parsing
type Event struct {
    Type string          `json:"type"`
    Data json.RawMessage `json:"data"`
}

// Parse later
var e Event
json.Unmarshal(eventBytes, &e)

// Different parsing based on type
if e.Type == "user" {
    var user User
    json.Unmarshal(e.Data, &user)
}
```

---

## 13. DATABASE INTERACTION

### SQL Database

```go
import (
    "database/sql"
    _ "github.com/go-sql-driver/mysql"  // Import driver
)

// Connect
db, err := sql.Open("mysql", "user:password@tcp(localhost:3306)/dbname")
if err != nil {
    log.Fatal(err)
}
defer db.Close()

// Test connection
if err := db.Ping(); err != nil {
    log.Fatal(err)
}

// Query single row
var name string
var age int
err := db.QueryRow("SELECT name, age FROM users WHERE id = ?", 1).Scan(&name, &age)
if err != nil {
    log.Fatal(err)
}

// Query multiple rows
rows, err := db.Query("SELECT id, name FROM users")
if err != nil {
    log.Fatal(err)
}
defer rows.Close()

for rows.Next() {
    var id int
    var name string
    if err := rows.Scan(&id, &name); err != nil {
        log.Fatal(err)
    }
    fmt.Printf("ID: %d, Name: %s\n", id, name)
}

// Insert
result, err := db.Exec("INSERT INTO users (name, age) VALUES (?, ?)", "John", 30)
if err != nil {
    log.Fatal(err)
}
lastID, _ := result.LastInsertId()
rowsAffected, _ := result.RowsAffected()

// Update
result, err := db.Exec("UPDATE users SET age = ? WHERE id = ?", 31, 1)

// Delete
result, err := db.Exec("DELETE FROM users WHERE id = ?", 1)

// Prepared statements (safer, faster for multiple queries)
stmt, err := db.Prepare("SELECT name FROM users WHERE id = ?")
if err != nil {
    log.Fatal(err)
}
defer stmt.Close()

for i := 1; i <= 3; i++ {
    var name string
    stmt.QueryRow(i).Scan(&name)
}
```

### ORMs (Object-Relational Mapping)

GORM is the most popular ORM for Go:

```go
import "gorm.io/gorm"

type User struct {
    ID   int
    Name string
    Age  int
}

// Connect
db, err := gorm.Open(mysql.Open("user:password@tcp(localhost:3306)/dbname"), &gorm.Config{})

// Create
user := User{Name: "John", Age: 30}
db.Create(&user)

// Read
var user User
db.First(&user, 1)  // Get by ID

db.Where("name = ?", "John").First(&user)  // Get by condition

// Update
db.Model(&user).Update("age", 31)

// Delete
db.Delete(&user)

// Query multiple
var users []User
db.Where("age > ?", 25).Find(&users)

// Relationships
type Profile struct {
    ID     int
    UserID int
    Bio    string
}

type User struct {
    ID       int
    Name     string
    Profile  Profile
}

db.Preload("Profile").First(&user)  // Eager load
```

---

## 14. HTTP & WEB DEVELOPMENT

### HTTP Server

```go
import (
    "fmt"
    "log"
    "net/http"
)

func main() {
    // Simple handler
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello, World!")
    })

    // Another route
    http.HandleFunc("/users", handleUsers)

    // Start server
    log.Fatal(http.ListenAndServe(":8080", nil))
}

func handleUsers(w http.ResponseWriter, r *http.Request) {
    if r.Method == "GET" {
        w.Header().Set("Content-Type", "application/json")
        fmt.Fprintf(w, `[{"id":1,"name":"John"}]`)
    }
}
```

### Using Multiplexer

```go
func main() {
    mux := http.NewServeMux()

    mux.HandleFunc("/api/users", handleUsers)
    mux.HandleFunc("/api/posts", handlePosts)

    server := &http.Server{
        Addr:    ":8080",
        Handler: mux,
    }

    log.Fatal(server.ListenAndServe())
}
```

### Request and Response

```go
func handleRequest(w http.ResponseWriter, r *http.Request) {
    // Request info
    fmt.Println("Method:", r.Method)
    fmt.Println("URL:", r.URL.Path)
    fmt.Println("Query:", r.URL.Query())  // ?name=John&age=30

    // Get query parameter
    name := r.URL.Query().Get("name")

    // Get path parameter (need router for this)
    // id := chi.URLParam(r, "id")

    // Get request body
    var data map[string]string
    json.NewDecoder(r.Body).Decode(&data)

    // Headers
    contentType := r.Header.Get("Content-Type")

    // Response
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusOK)
    w.Write([]byte(`{"status":"ok"}`))
}
```

### Popular Frameworks

**Gin** (most popular):
```go
import "github.com/gin-gonic/gin"

func main() {
    r := gin.Default()

    r.GET("/users/:id", func(c *gin.Context) {
        id := c.Param("id")
        c.JSON(200, gin.H{
            "id": id,
            "name": "John",
        })
    })

    r.POST("/users", func(c *gin.Context) {
        var user User
        c.BindJSON(&user)
        c.JSON(200, user)
    })

    r.Run(":8080")
}
```

**Echo**:
```go
import "github.com/labstack/echo/v4"

func main() {
    e := echo.New()

    e.GET("/users/:id", getUser)
    e.POST("/users", createUser)

    e.Logger.Fatal(e.Start(":8080"))
}

func getUser(c echo.Context) error {
    id := c.Param("id")
    return c.JSON(200, map[string]string{"id": id})
}
```

---

## 15. TESTING

### Unit Testing

```go
import "testing"

// Function to test
func Add(a, b int) int {
    return a + b
}

// Test function (in _test.go file)
func TestAdd(t *testing.T) {
    result := Add(2, 3)
    expected := 5

    if result != expected {
        t.Errorf("Expected %d, got %d", expected, result)
    }
}

// Multiple test cases
func TestAddCases(t *testing.T) {
    tests := []struct {
        a, b, expected int
    }{
        {2, 3, 5},
        {-1, 1, 0},
        {0, 0, 0},
    }

    for _, test := range tests {
        if Add(test.a, test.b) != test.expected {
            t.Errorf("Add(%d, %d) = %d, want %d", test.a, test.b, Add(test.a, test.b), test.expected)
        }
    }
}
```

### Table-Driven Tests

```go
func TestDivide(t *testing.T) {
    tests := []struct {
        name      string
        a, b      float64
        expected  float64
        shouldErr bool
    }{
        {"Normal division", 10, 2, 5, false},
        {"Division by zero", 10, 0, 0, true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result, err := Divide(tt.a, tt.b)

            if tt.shouldErr && err == nil {
                t.Errorf("Expected error, got none")
            }

            if !tt.shouldErr && result != tt.expected {
                t.Errorf("Expected %f, got %f", tt.expected, result)
            }
        })
    }
}
```

### Mocking

```go
import "testing"

// Interface for mocking
type UserStore interface {
    GetUser(id int) (*User, error)
}

// Mock implementation
type MockUserStore struct{}

func (m *MockUserStore) GetUser(id int) (*User, error) {
    return &User{ID: id, Name: "John"}, nil
}

// Function that uses interface
func GetUserInfo(store UserStore, id int) string {
    user, _ := store.GetUser(id)
    return user.Name
}

func TestGetUserInfo(t *testing.T) {
    mock := &MockUserStore{}
    result := GetUserInfo(mock, 1)

    if result != "John" {
        t.Errorf("Expected John, got %s", result)
    }
}
```

### Benchmarking

```go
func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Add(2, 3)
    }
}

// Run: go test -bench=. -benchmem
```

---

## 16. REFLECTION

Reflection allows examining and manipulating types at runtime.

```go
import "reflect"

var x int = 42

// Get type
t := reflect.TypeOf(x)
fmt.Println(t)  // int

// Get value
v := reflect.ValueOf(x)
fmt.Println(v)  // 42

// Type name
fmt.Println(t.Name())  // int

// Kind
fmt.Println(t.Kind())  // int

// Check interface
var i interface{} = "hello"
if t.String() == "string" {
    s := v.String()
}

// Struct reflection
type Person struct {
    Name string
    Age  int
}

p := Person{"John", 30}
v := reflect.ValueOf(p)

// Iterate fields
for i := 0; i < v.NumField(); i++ {
    field := v.Field(i)
    fmt.Println(field.Interface())  // John, 30
}

// Get field by name
ageField, _ := v.Type().FieldByName("Age")
fmt.Println(v.FieldByName("Age"))

// Set field value (requires pointer)
pv := reflect.ValueOf(&p)
pv.Elem().FieldByName("Age").SetInt(31)

// Call method via reflection
method := reflect.ValueOf(p).MethodByName("String")
result := method.Call(nil)
```

---

## 17. DESIGN PATTERNS IN GO

### Singleton Pattern

```go
var (
    once     sync.Once
    instance *Singleton
)

type Singleton struct {
    value string
}

func GetInstance() *Singleton {
    once.Do(func() {
        instance = &Singleton{value: "singleton"}
    })
    return instance
}
```

### Factory Pattern

```go
type Shape interface {
    Area() float64
}

type Circle struct {
    radius float64
}

func (c Circle) Area() float64 {
    return math.Pi * c.radius * c.radius
}

type Rectangle struct {
    width, height float64
}

func (r Rectangle) Area() float64 {
    return r.width * r.height
}

func CreateShape(shapeType string, args ...float64) Shape {
    switch shapeType {
    case "circle":
        return Circle{radius: args[0]}
    case "rectangle":
        return Rectangle{width: args[0], height: args[1]}
    default:
        return nil
    }
}
```

### Observer Pattern

```go
type Subject struct {
    observers []Observer
}

type Observer interface {
    Update(message string)
}

func (s *Subject) Subscribe(o Observer) {
    s.observers = append(s.observers, o)
}

func (s *Subject) Notify(message string) {
    for _, o := range s.observers {
        o.Update(message)
    }
}

type User struct {
    name string
}

func (u User) Update(message string) {
    fmt.Printf("%s received: %s\n", u.name, message)
}
```

### Builder Pattern

```go
type User struct {
    Name  string
    Email string
    Age   int
}

type UserBuilder struct {
    user *User
}

func NewUserBuilder() *UserBuilder {
    return &UserBuilder{user: &User{}}
}

func (ub *UserBuilder) WithName(name string) *UserBuilder {
    ub.user.Name = name
    return ub
}

func (ub *UserBuilder) WithEmail(email string) *UserBuilder {
    ub.user.Email = email
    return ub
}

func (ub *UserBuilder) WithAge(age int) *UserBuilder {
    ub.user.Age = age
    return ub
}

func (ub *UserBuilder) Build() *User {
    return ub.user
}

// Usage
user := NewUserBuilder().
    WithName("John").
    WithEmail("john@example.com").
    WithAge(30).
    Build()
```

### Strategy Pattern

```go
type PaymentStrategy interface {
    Pay(amount float64) error
}

type CreditCardPayment struct{}

func (c CreditCardPayment) Pay(amount float64) error {
    fmt.Printf("Paid %.2f with credit card\n", amount)
    return nil
}

type PayPalPayment struct{}

func (p PayPalPayment) Pay(amount float64) error {
    fmt.Printf("Paid %.2f via PayPal\n", amount)
    return nil
}

type Order struct {
    strategy PaymentStrategy
}

func (o *Order) SetPaymentStrategy(strategy PaymentStrategy) {
    o.strategy = strategy
}

func (o *Order) Checkout(amount float64) error {
    return o.strategy.Pay(amount)
}
```

---

## 18. DEPENDENCY INJECTION

Dependency injection improves testability and loose coupling.

```go
// Bad: Hard-coded dependency
type UserService struct {
    db *Database
}

func NewUserService() *UserService {
    db := NewDatabase()
    return &UserService{db: db}
}

// Good: Inject dependency
type UserService struct {
    db UserRepository
}

type UserRepository interface {
    GetUser(id int) (*User, error)
}

func NewUserService(repo UserRepository) *UserService {
    return &UserService{db: repo}
}

// Allows easy testing
type MockRepository struct{}

func (m MockRepository) GetUser(id int) (*User, error) {
    return &User{ID: id, Name: "Test"}, nil
}

service := NewUserService(&MockRepository{})
```

---

## 19. LOGGING & DEBUGGING

### Built-in Log Package

```go
import "log"

log.Println("Info message")
log.Printf("Formatted: %s\n", "message")
log.Fatal("Fatal error")  // Logs and exits
log.Panic("Panic error")  // Logs and panics

// Custom logger
logger := log.New(os.Stdout, "PREFIX: ", log.LstdFlags)
logger.Println("Custom log")
```

### Structured Logging with Logrus

```go
import "github.com/sirupsen/logrus"

log := logrus.New()

log.Info("Information message")
log.WithFields(logrus.Fields{
    "user": "john",
    "id": 123,
}).Info("User logged in")

log.WithError(err).Error("Operation failed")

// Log levels
log.SetLevel(logrus.DebugLevel)
```

---

## 20. DEPLOYMENT & PERFORMANCE

### Building for Production

```bash
# Build optimized binary
go build -o myapp

# Strip debug info for smaller size
go build -ldflags="-s -w" -o myapp

# Cross-compile
GOOS=linux GOARCH=amd64 go build -o myapp

# Set version info
go build -ldflags="-X main.Version=1.0.0" -o myapp
```

### Profiling

```go
import _ "net/http/pprof"

// In main()
go func() {
    log.Println(http.ListenAndServe("localhost:6060", nil))
}()

// Access at http://localhost:6060/debug/pprof/
```

### Benchmarking Code

```go
import (
    "testing"
    "fmt"
)

func BenchmarkStringConcat(b *testing.B) {
    for i := 0; i < b.N; i++ {
        str := ""
        for j := 0; j < 100; j++ {
            str += "a"
        }
    }
}

func BenchmarkStringBuilder(b *testing.B) {
    for i := 0; i < b.N; i++ {
        var sb strings.Builder
        for j := 0; j < 100; j++ {
            sb.WriteString("a")
        }
        sb.String()
    }
}

// Run: go test -bench=. -benchmem
```

---

## 21. ADVANCED CONCURRENCY PATTERNS

### Pipeline Pattern

```go
func generate(numbers ...int) <-chan int {
    out := make(chan int)
    go func() {
        for _, n := range numbers {
            out <- n
        }
        close(out)
    }()
    return out
}

func square(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        for n := range in {
            out <- n * n
        }
        close(out)
    }()
    return out
}

func main() {
    for result := range square(generate(1, 2, 3, 4)) {
        fmt.Println(result)  // 1, 4, 9, 16
    }
}
```

### Fan-Out/Fan-In

```go
// Fan-out: distribute work
func fanOut(ch <-chan int, workers int) []<-chan int {
    channels := make([]<-chan int, workers)
    for i := 0; i < workers; i++ {
        channels[i] = process(ch)
    }
    return channels
}

// Fan-in: combine results
func fanIn(channels ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    for _, ch := range channels {
        wg.Add(1)
        go func(ch <-chan int) {
            for n := range ch {
                out <- n
            }
            wg.Done()
        }(ch)
    }

    go func() {
        wg.Wait()
        close(out)
    }()

    return out
}
```

### Rate Limiting

```go
import "time"

// Simple rate limiter
limiter := time.NewTicker(time.Second)
for request := range requests {
    <-limiter.C
    processRequest(request)
}

// Token bucket pattern
type RateLimiter struct {
    tokens chan struct{}
    ticker *time.Ticker
}

func NewRateLimiter(rps int) *RateLimiter {
    rl := &RateLimiter{
        tokens: make(chan struct{}, rps),
        ticker: time.NewTicker(time.Second / time.Duration(rps)),
    }

    go func() {
        for range rl.ticker.C {
            select {
            case rl.tokens <- struct{}{}:
            default:
            }
        }
    }()

    return rl
}

func (rl *RateLimiter) Wait() {
    <-rl.tokens
}
```

---

## 22. MIDDLEWARE PATTERN

```go
type Handler func(http.ResponseWriter, *http.Request)

// Middleware wrapper
func LoggingMiddleware(next Handler) Handler {
    return func(w http.ResponseWriter, r *http.Request) {
        log.Printf("%s %s", r.Method, r.URL.Path)
        next(w, r)
    }
}

func AuthMiddleware(next Handler) Handler {
    return func(w http.ResponseWriter, r *http.Request) {
        token := r.Header.Get("Authorization")
        if token == "" {
            http.Error(w, "Unauthorized", http.StatusUnauthorized)
            return
        }
        next(w, r)
    }
}

// Chaining middleware
func Chain(h Handler, middleware ...func(Handler) Handler) Handler {
    for _, m := range middleware {
        h = m(h)
    }
    return h
}

// Usage
handler := Chain(
    handleUser,
    LoggingMiddleware,
    AuthMiddleware,
)

http.HandleFunc("/user", handler)
```

---

## 23. BEST PRACTICES & CONVENTIONS

### Code Organization

```
myapp/
├── main.go              // Entry point
├── go.mod              // Module definition
├── go.sum              // Dependency checksums
├── config/             // Configuration
│   └── config.go
├── models/             // Data structures
│   └── user.go
├── handlers/           // HTTP handlers
│   └── user.go
├── services/           // Business logic
│   └── user.go
├── repositories/       // Data access
│   └── user.go
├── middleware/         // HTTP middleware
│   └── auth.go
└── tests/              // Test files
    └── user_test.go
```

### Error Handling Best Practices

```go
// Check errors immediately
if err != nil {
    return err  // Propagate
}

// Wrap errors with context
if err != nil {
    return fmt.Errorf("failed to fetch user: %w", err)
}

// Handle specific errors
if errors.Is(err, sql.ErrNoRows) {
    // Handle not found
}

// Don't ignore errors
_ = os.Remove("file.txt")  // Discouraged

// Better
if err := os.Remove("file.txt"); err != nil {
    log.Printf("Failed to remove file: %v", err)
}
```

### Naming Conventions

```go
// Package names: lowercase, short
package user
package models

// Functions/Variables: camelCase
func getUserByID() {}
var userID = 1

// Constants: camelCase or SCREAMING_SNAKE_CASE for groups
const defaultTimeout = 30 * time.Second
const (
    StatusActive   = "active"
    StatusInactive = "inactive"
)

// Interfaces: end with 'er'
type Reader interface {}
type Writer interface {}
type UserStore interface {}

// Exported: PascalCase
func ExportedFunction() {}
var ExportedVariable = 1

// Unexported: camelCase
func unexportedFunction() {}
var unexportedVariable = 1
```

### Code Quality

```go
// Formatting
go fmt ./...

// Linting
go vet ./...
golint ./...

// Testing
go test ./...
go test -v ./...  // Verbose
go test -cover ./...  // Coverage

// Test coverage
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

---

## 24. BUILDING A BACKEND SYSTEM (PRACTICAL EXAMPLE)

```go
package main

import (
    "context"
    "fmt"
    "log"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"

    "github.com/gin-gonic/gin"
    "github.com/joho/godotenv"
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
)

// Models
type User struct {
    ID    uint
    Name  string
    Email string
}

// Repository
type UserRepository struct {
    db *gorm.DB
}

func (r *UserRepository) GetUser(id uint) (*User, error) {
    var user User
    if err := r.db.First(&user, id).Error; err != nil {
        return nil, err
    }
    return &user, nil
}

func (r *UserRepository) CreateUser(user *User) error {
    return r.db.Create(user).Error
}

// Service
type UserService struct {
    repo *UserRepository
}

func (s *UserService) GetUserInfo(id uint) (*User, error) {
    user, err := s.repo.GetUser(id)
    if err != nil {
        return nil, fmt.Errorf("failed to get user: %w", err)
    }
    return user, nil
}

func (s *UserService) RegisterUser(user *User) error {
    if err := s.repo.CreateUser(user); err != nil {
        return fmt.Errorf("failed to register user: %w", err)
    }
    return nil
}

// Handler
func NewUserHandler(service *UserService) func(*gin.Context) {
    return func(c *gin.Context) {
        var user User
        if err := c.ShouldBindJSON(&user); err != nil {
            c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
            return
        }

        if err := service.RegisterUser(&user); err != nil {
            c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
            return
        }

        c.JSON(http.StatusCreated, user)
    }
}

// Main application
type App struct {
    router  *gin.Engine
    db      *gorm.DB
    service *UserService
}

func NewApp(db *gorm.DB) *App {
    repo := &UserRepository{db: db}
    service := &UserService{repo: repo}

    router := gin.Default()
    router.POST("/users", NewUserHandler(service))

    return &App{
        router:  router,
        db:      db,
        service: service,
    }
}

func (app *App) Start(addr string) error {
    server := &http.Server{
        Addr:    addr,
        Handler: app.router,
    }

    go func() {
        if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            log.Fatalf("Server error: %v", err)
        }
    }()

    // Graceful shutdown
    sigChan := make(chan os.Signal, 1)
    signal.Notify(sigChan, os.Interrupt, syscall.SIGTERM)

    <-sigChan
    log.Println("Shutting down server...")

    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    return server.Shutdown(ctx)
}

func main() {
    // Load config
    if err := godotenv.Load(); err != nil {
        log.Println("No .env file found")
    }

    // Connect database
    dsn := "user:password@tcp(localhost:3306)/dbname"
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatalf("Failed to connect to database: %v", err)
    }

    // Auto migrate
    db.AutoMigrate(&User{})

    // Create and start app
    app := NewApp(db)
    if err := app.Start(":8080"); err != nil {
        log.Fatalf("Application error: %v", err)
    }
}
```
