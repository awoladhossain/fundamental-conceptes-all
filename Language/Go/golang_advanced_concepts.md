# Go Advanced Concepts: Mastering Backend Development

---

## PART 1: MASTER CONCURRENCY (DEEP DIVE)

### 1.1 Understanding Goroutines Under the Hood

Goroutines are lightweight threads managed by the Go runtime. They're NOT OS threads.

```go
// OS thread vs Goroutine
// - 1000s of goroutines: Light, fast creation/switching
// - 1000s of OS threads: Heavy, slow, expensive context switching

// Goroutine creation is cheap
func main() {
    for i := 0; i < 100000; i++ {
        go func(id int) {
            // Do work
        }(i)
    }
    time.Sleep(1 * time.Second)
}
```

**How Goroutines Work**:
1. Multiple goroutines are mapped to fewer OS threads
2. Go runtime schedules goroutines on available threads
3. When a goroutine blocks (I/O, channel), the runtime switches to another
4. This is called "M:N scheduling" (M goroutines on N OS threads)

**Memory Footprint**:
- OS Thread: ~1-2 MB
- Goroutine: ~2 KB

### 1.2 Advanced Channel Patterns

#### Timeout Pattern

```go
// Problem: Goroutine might never return
result := <-channel  // Blocks forever if channel never sends

// Solution: Timeout
select {
case result := <-channel:
    fmt.Println("Got result:", result)
case <-time.After(5 * time.Second):
    fmt.Println("Timeout! No result received")
}

// Real-world example: API call with timeout
func FetchData(url string) (string, error) {
    resultChan := make(chan string, 1)
    errChan := make(chan error, 1)

    go func() {
        resp, err := http.Get(url)
        if err != nil {
            errChan <- err
            return
        }
        defer resp.Body.Close()

        body, _ := ioutil.ReadAll(resp.Body)
        resultChan <- string(body)
    }()

    select {
    case result := <-resultChan:
        return result, nil
    case err := <-errChan:
        return "", err
    case <-time.After(10 * time.Second):
        return "", errors.New("request timeout")
    }
}
```

#### Done Channel Pattern

```go
// Signal for graceful shutdown
func worker(id int, done <-chan struct{}) {
    for {
        select {
        case <-done:
            fmt.Printf("Worker %d shutting down\n", id)
            return
        default:
            // Do work
            fmt.Printf("Worker %d working\n", id)
            time.Sleep(1 * time.Second)
        }
    }
}

func main() {
    done := make(chan struct{})

    // Start 3 workers
    for i := 1; i <= 3; i++ {
        go worker(i, done)
    }

    time.Sleep(5 * time.Second)

    // Signal shutdown
    close(done)  // Unblocks all <-done operations

    time.Sleep(1 * time.Second)
}
```

#### Pipeline Pattern (Advanced)

```go
// Create a data processing pipeline
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

func filter(in <-chan int, predicate func(int) bool) <-chan int {
    out := make(chan int)
    go func() {
        for n := range in {
            if predicate(n) {
                out <- n
            }
        }
        close(out)
    }()
    return out
}

func main() {
    // Generate 1-10 -> square -> filter even
    pipeline := filter(
        square(generate(1, 2, 3, 4, 5)),
        func(n int) bool { return n%2 == 0 },
    )

    for result := range pipeline {
        fmt.Println(result)  // 4, 16, 100
    }
}
```

#### Merging Channels

```go
// Merge multiple channels into one
func merge(channels ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    for _, ch := range channels {
        wg.Add(1)
        go func(ch <-chan int) {
            defer wg.Done()
            for value := range ch {
                out <- value
            }
        }(ch)
    }

    go func() {
        wg.Wait()
        close(out)
    }()

    return out
}

// Example
ch1 := generate(1, 2, 3)
ch2 := generate(4, 5, 6)

for value := range merge(ch1, ch2) {
    fmt.Println(value)  // 1, 4, 2, 5, 3, 6 (order may vary)
}
```

#### Broadcast Channel

```go
// Send message to many listeners
type Broadcaster struct {
    mu      sync.RWMutex
    clients []chan string
}

func (b *Broadcaster) Register(ch chan string) {
    b.mu.Lock()
    defer b.mu.Unlock()
    b.clients = append(b.clients, ch)
}

func (b *Broadcaster) Broadcast(message string) {
    b.mu.RLock()
    defer b.mu.RUnlock()

    for _, client := range b.clients {
        select {
        case client <- message:
        default:
            // Client not reading, skip
        }
    }
}

// Usage
br := &Broadcaster{}

// Register listeners
for i := 0; i < 3; i++ {
    ch := make(chan string, 1)
    br.Register(ch)

    go func(ch <-chan string) {
        for msg := range ch {
            fmt.Println(msg)
        }
    }(ch)
}

br.Broadcast("Hello everyone!")
```

### 1.3 Advanced Synchronization Patterns

#### Rate Limiting with Token Bucket

```go
type RateLimiter struct {
    tokens chan struct{}
    ticker *time.Ticker
}

func NewRateLimiter(rps int) *RateLimiter {
    rl := &RateLimiter{
        tokens: make(chan struct{}, rps),
        ticker: time.NewTicker(time.Second / time.Duration(rps)),
    }

    // Fill tokens at fixed rate
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

func (rl *RateLimiter) Allow() bool {
    select {
    case <-rl.tokens:
        return true
    default:
        return false
    }
}

func (rl *RateLimiter) Wait() {
    <-rl.tokens
}

// Usage: 100 requests per second
limiter := NewRateLimiter(100)

for i := 0; i < 1000; i++ {
    limiter.Wait()
    // Process request
}
```

#### Semaphore Pattern

```go
// Limit concurrent goroutines
type Semaphore chan struct{}

func NewSemaphore(n int) Semaphore {
    return make(Semaphore, n)
}

func (s Semaphore) Acquire() {
    s <- struct{}{}
}

func (s Semaphore) Release() {
    <-s
}

// Usage: Max 3 concurrent operations
sem := NewSemaphore(3)

for i := 0; i < 10; i++ {
    go func(id int) {
        sem.Acquire()
        defer sem.Release()

        fmt.Printf("Processing %d\n", id)
        time.Sleep(1 * time.Second)
    }(i)
}
```

#### Barrier Synchronization

```go
// Wait for all goroutines to reach a point
type Barrier struct {
    mu     sync.Mutex
    count  int
    arrive chan struct{}
    depart chan struct{}
}

func NewBarrier(n int) *Barrier {
    return &Barrier{
        count:  n,
        arrive: make(chan struct{}),
        depart: make(chan struct{}),
    }
}

func (b *Barrier) Wait() {
    b.mu.Lock()
    b.count--
    b.mu.Unlock()

    if b.count == 0 {
        close(b.arrive)
    }

    <-b.arrive
    close(b.depart)
}

// Usage
barrier := NewBarrier(3)

for i := 0; i < 3; i++ {
    go func(id int) {
        fmt.Printf("Goroutine %d waiting\n", id)
        barrier.Wait()
        fmt.Printf("Goroutine %d proceeding\n", id)
    }(i)
}
```

### 1.4 Avoiding Deadlocks and Race Conditions

**Deadlock Example**:
```go
// DEADLOCK: Both goroutines waiting for each other
ch1 := make(chan int)
ch2 := make(chan int)

go func() {
    ch1 <- 1
    <-ch2  // Waits forever
}()

go func() {
    ch2 <- 2
    <-ch1  // Waits forever
}()
```

**Fix**: Use buffered channels or guarantee send/receive order.

**Race Condition Example**:
```go
// RACE CONDITION: Multiple goroutines access without synchronization
var counter = 0

for i := 0; i < 1000; i++ {
    go func() {
        counter++  // Not atomic! Race condition!
    }()
}

// Detection: go run -race main.go
```

**Fix**: Use mutex
```go
var (
    mu      sync.Mutex
    counter = 0
)

for i := 0; i < 1000; i++ {
    go func() {
        mu.Lock()
        counter++
        mu.Unlock()
    }()
}
```

### 1.5 Context for Advanced Control

```go
// Context tree for cancellation propagation
func main() {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    // Create child contexts
    ctx1, _ := context.WithTimeout(ctx, 5*time.Second)
    ctx2, _ := context.WithDeadline(ctx, time.Now().Add(10*time.Second))

    // Cancelling parent cancels all children
    go doWork(ctx1)
    go doWork(ctx2)

    time.Sleep(3 * time.Second)
    cancel()  // Cancels ctx, ctx1, ctx2

    time.Sleep(1 * time.Second)
}

func doWork(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            fmt.Println("Context cancelled:", ctx.Err())
            return
        default:
            // Do work
            fmt.Println("Working...")
            time.Sleep(1 * time.Second)
        }
    }
}
```

---

## PART 2: PERFORMANCE OPTIMIZATION

### 2.1 CPU Profiling

```go
import (
    "runtime/pprof"
    "os"
)

func main() {
    // Start CPU profile
    f, err := os.Create("cpu.prof")
    if err != nil {
        log.Fatal(err)
    }
    defer f.Close()

    if err := pprof.StartCPUProfile(f); err != nil {
        log.Fatal(err)
    }
    defer pprof.StopCPUProfile()

    // Your code here
    expensive()
}

func expensive() {
    sum := 0
    for i := 0; i < 1000000000; i++ {
        sum += i
    }
}

// Analysis:
// go tool pprof cpu.prof
// (pprof) top10
// (pprof) list expensive
```

### 2.2 Memory Profiling

```go
import "runtime/pprof"

func main() {
    // Write memory profile
    f, err := os.Create("mem.prof")
    if err != nil {
        log.Fatal(err)
    }
    defer f.Close()

    // Run your program
    allocateMemory()

    // Get current memory stats
    runtime.GC()
    if err := pprof.WriteHeapProfile(f); err != nil {
        log.Fatal(err)
    }
}

func allocateMemory() {
    var slices [][]int
    for i := 0; i < 1000; i++ {
        slices = append(slices, make([]int, 1000))
    }
}

// Analysis:
// go tool pprof mem.prof
// (pprof) alloc_space   // Total allocated
// (pprof) alloc_objects // Number of allocations
```

### 2.3 HTTP Profiling

```go
import _ "net/http/pprof"

func main() {
    // Start pprof server on :6060
    go func() {
        log.Println(http.ListenAndServe("localhost:6060", nil))
    }()

    // Your server code here
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}

// Access profiles:
// http://localhost:6060/debug/pprof/
// http://localhost:6060/debug/pprof/heap
// http://localhost:6060/debug/pprof/goroutine
// http://localhost:6060/debug/pprof/profile?seconds=30  (CPU profile)

// Download and analyze:
// go tool pprof http://localhost:6060/debug/pprof/heap
```

### 2.4 Benchmarking

```go
import "testing"

func BenchmarkStringConcat(b *testing.B) {
    for i := 0; i < b.N; i++ {
        str := ""
        for j := 0; j < 100; j++ {
            str += "x"  // Inefficient!
        }
    }
}

func BenchmarkStringBuilder(b *testing.B) {
    for i := 0; i < b.N; i++ {
        var sb strings.Builder
        for j := 0; j < 100; j++ {
            sb.WriteString("x")  // Efficient!
        }
        sb.String()
    }
}

// Run benchmarks:
// go test -bench=. -benchmem
// go test -bench=. -benchmem -cpuprofile=cpu.prof
// go test -bench=. -benchmem -memprofile=mem.prof

// Output example:
// BenchmarkStringConcat-8    1000  1000000 ns/op  102400 B/op  100 allocs/op
// BenchmarkStringBuilder-8 100000   10000 ns/op    1024 B/op    1 allocs/op
```

### 2.5 Optimization Techniques

#### Reduce Allocations

```go
// Bad: Creates new slice every time
func Process(data []int) []int {
    result := []int{}  // Allocates
    for _, v := range data {
        result = append(result, v*2)
    }
    return result
}

// Better: Pre-allocate
func Process(data []int) []int {
    result := make([]int, len(data))  // Single allocation
    for i, v := range data {
        result[i] = v * 2
    }
    return result
}

// Best: Reuse buffer
func Process(data []int, result []int) {
    for i, v := range data {
        result[i] = v * 2
    }
}
```

#### Use sync.Pool for Object Reuse

```go
var bufferPool = sync.Pool{
    New: func() interface{} {
        return new(bytes.Buffer)
    },
}

func ProcessLarge(data string) {
    buf := bufferPool.Get().(*bytes.Buffer)
    defer bufferPool.Put(buf)

    buf.Reset()
    buf.WriteString(data)
    // Use buffer
}
```

#### Minimize String Copies

```go
// Bad: Multiple allocations
func Concat(strs ...string) string {
    result := ""
    for _, s := range strs {
        result += s  // Allocates new string each time
    }
    return result
}

// Good: Single allocation
func Concat(strs ...string) string {
    var sb strings.Builder
    for _, s := range strs {
        sb.WriteString(s)
    }
    return sb.String()
}
```

#### Cache Expensive Computations

```go
type Cache struct {
    mu    sync.RWMutex
    items map[string]interface{}
    ttl   time.Duration
}

func (c *Cache) Get(key string, fn func() interface{}) interface{} {
    c.mu.RLock()
    if val, ok := c.items[key]; ok {
        c.mu.RUnlock()
        return val
    }
    c.mu.RUnlock()

    // Compute value
    val := fn()

    c.mu.Lock()
    c.items[key] = val
    c.mu.Unlock()

    return val
}
```

---

## PART 3: ADVANCED TESTING

### 3.1 Property-Based Testing

Property-based testing generates random test cases to find edge cases.

```go
import "testing"
import "math/rand"

// Property: Adding 0 should return original number
func TestAddZeroProperty(t *testing.T) {
    for i := 0; i < 1000; i++ {
        x := rand.Int63()
        if x+0 != x {
            t.Fatalf("Add(0) failed for %d", x)
        }
    }
}

// Property: Sorting is idempotent
func TestSortIdempotent(t *testing.T) {
    for i := 0; i < 100; i++ {
        arr := generateRandomArray(100)
        arr1 := make([]int, len(arr))
        copy(arr1, arr)

        sort.Ints(arr1)

        arr2 := make([]int, len(arr1))
        copy(arr2, arr1)
        sort.Ints(arr2)

        if !equal(arr1, arr2) {
            t.Fatal("Sorting is not idempotent")
        }
    }
}

// Using gopter for advanced property testing
import "github.com/leanovate/gopter"
import "github.com/leanovate/gopter/gen"
import "github.com/leanovate/gopter/prop"

func TestAdditionProperty(t *testing.T) {
    properties := gopter.NewProperties(nil)

    properties.Property("addition is commutative",
        prop.ForAll(
            func(x, y int) bool {
                return x+y == y+x
            },
            gen.Int(),
            gen.Int(),
        ))

    if !properties.Run(gopter.DefaultTestParameters()) {
        t.Fail()
    }
}
```

### 3.2 Fuzzing

Fuzzing is automated testing with random data to find crashes.

```go
// Go 1.18+ built-in fuzzing
func FuzzJSON(f *testing.F) {
    // Seed corpus
    f.Add(`{"name":"John","age":30}`)
    f.Add(`[]`)

    f.Fuzz(func(t *testing.T, input string) {
        var data interface{}
        // This shouldn't panic for any input
        _ = json.Unmarshal([]byte(input), &data)
    })
}

// Run:
// go test -fuzz=FuzzJSON -fuzztime=30s

// Custom fuzzer
func FuzzHTTPServer(f *testing.F) {
    f.Add("GET", "/users")
    f.Add("POST", "/users")

    f.Fuzz(func(t *testing.T, method, path string) {
        req := httptest.NewRequest(method, path, nil)
        w := httptest.NewRecorder()

        // Server shouldn't panic on any request
        handler(w, req)

        if w.Code < 200 || w.Code > 599 {
            t.Errorf("Invalid status code: %d", w.Code)
        }
    })
}
```

### 3.3 Benchmark-Driven Development

```go
func BenchmarkFibRecursive(b *testing.B) {
    for i := 0; i < b.N; i++ {
        fibRecursive(30)
    }
}

func BenchmarkFibMemoized(b *testing.B) {
    for i := 0; i < b.N; i++ {
        fibMemoized(30)
    }
}

// Results show which is faster
// Use benchmarks to drive optimization decisions

// Comparing benchmarks:
// go test -bench=. -benchmem > old.txt
// (make optimizations)
// go test -bench=. -benchmem > new.txt
// benchstat old.txt new.txt
```

### 3.4 Integration Testing

```go
func TestDatabaseIntegration(t *testing.T) {
    // Setup
    db := setupTestDatabase(t)
    defer db.Close()

    // Test
    user := &User{Name: "John", Email: "john@example.com"}
    err := db.CreateUser(user)
    if err != nil {
        t.Fatalf("Failed to create user: %v", err)
    }

    // Verify
    retrieved, _ := db.GetUserByEmail("john@example.com")
    if retrieved.Name != "John" {
        t.Errorf("Expected John, got %s", retrieved.Name)
    }
}

// Using Docker for integration tests
func TestWithDocker(t *testing.T) {
    if testing.Short() {
        t.Skip("Skipping integration test")
    }

    // Start Docker container
    // Connect to service
    // Run tests
    // Cleanup
}
```

---

## PART 4: MICROSERVICES & DISTRIBUTED SYSTEMS

### 4.1 gRPC (Modern RPC Framework)

```protobuf
// user.proto
syntax = "proto3";

package user;

service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc ListUsers(Empty) returns (UserList);
  rpc CreateUser(CreateUserRequest) returns (User);
}

message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
}

message GetUserRequest {
  int32 id = 1;
}

message CreateUserRequest {
  string name = 1;
  string email = 2;
}

message UserList {
  repeated User users = 1;
}

message Empty {}
```

**Server Implementation**:
```go
import "github.com/yourname/user"

type UserServer struct {
    user.UnimplementedUserServiceServer
    db *Database
}

func (s *UserServer) GetUser(ctx context.Context, req *user.GetUserRequest) (*user.User, error) {
    u, err := s.db.GetUser(int(req.Id))
    if err != nil {
        return nil, status.Errorf(codes.NotFound, "user not found")
    }

    return &user.User{
        Id:    int32(u.ID),
        Name:  u.Name,
        Email: u.Email,
    }, nil
}

func (s *UserServer) CreateUser(ctx context.Context, req *user.CreateUserRequest) (*user.User, error) {
    u := &User{
        Name:  req.Name,
        Email: req.Email,
    }

    if err := s.db.CreateUser(u); err != nil {
        return nil, status.Errorf(codes.Internal, "failed to create user")
    }

    return &user.User{
        Id:    int32(u.ID),
        Name:  u.Name,
        Email: u.Email,
    }, nil
}

func main() {
    lis, err := net.Listen("tcp", ":50051")
    if err != nil {
        log.Fatalf("failed to listen: %v", err)
    }

    s := grpc.NewServer(
        grpc.MaxConcurrentStreams(100),
        grpc.KeepaliveParams(keepalive.ServerParameters{
            Time:    20 * time.Second,
            Timeout: 10 * time.Second,
        }),
    )

    db := ConnectDatabase()
    user.RegisterUserServiceServer(s, &UserServer{db: db})

    if err := s.Serve(lis); err != nil {
        log.Fatalf("failed to serve: %v", err)
    }
}
```

**Client Implementation**:
```go
func main() {
    conn, err := grpc.Dial("localhost:50051", grpc.WithInsecure())
    if err != nil {
        log.Fatalf("failed to connect: %v", err)
    }
    defer conn.Close()

    client := user.NewUserServiceClient(conn)

    // Get user
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    resp, err := client.GetUser(ctx, &user.GetUserRequest{Id: 1})
    if err != nil {
        log.Fatalf("failed to get user: %v", err)
    }

    fmt.Printf("User: %s (%s)\n", resp.Name, resp.Email)
}
```

### 4.2 Service Discovery

Service discovery helps services find each other.

```go
// Simple Consul-based service discovery
import "github.com/hashicorp/consul/api"

type ServiceRegistry struct {
    client *api.Client
}

func (sr *ServiceRegistry) Register(name, addr string, port int) error {
    reg := &api.AgentServiceRegistration{
        ID:      fmt.Sprintf("%s-%s", name, addr),
        Name:    name,
        Address: addr,
        Port:    port,
        Check: &api.AgentServiceCheck{
            HTTP:     fmt.Sprintf("http://%s:%d/health", addr, port),
            Interval: "10s",
            Timeout:  "5s",
        },
    }

    return sr.client.Agent().ServiceRegister(reg)
}

func (sr *ServiceRegistry) Discover(name string) ([]*api.ServiceEntry, error) {
    entries, _, err := sr.client.Health().Service(name, "", true, nil)
    return entries, err
}

// Usage
registry, _ := api.NewClient(api.DefaultConfig())
sr := &ServiceRegistry{client: registry}

sr.Register("user-service", "localhost", 8080)

// Later, discover service
services, _ := sr.Discover("user-service")
for _, svc := range services {
    fmt.Printf("Found service at %s:%d\n", svc.Service.Address, svc.Service.Port)
}
```

### 4.3 Load Balancing

```go
type LoadBalancer struct {
    mu       sync.Mutex
    servers  []string
    current  int
}

// Round-robin
func (lb *LoadBalancer) RoundRobin() string {
    lb.mu.Lock()
    defer lb.mu.Unlock()

    server := lb.servers[lb.current]
    lb.current = (lb.current + 1) % len(lb.servers)
    return server
}

// Least connections
type ServerStats struct {
    mu          sync.Mutex
    connections map[string]int
}

func (ss *ServerStats) LeastConnected(servers []string) string {
    ss.mu.Lock()
    defer ss.mu.Unlock()

    min := math.MaxInt
    var selected string

    for _, server := range servers {
        if ss.connections[server] < min {
            min = ss.connections[server]
            selected = server
        }
    }

    ss.connections[selected]++
    return selected
}
```

### 4.4 Circuit Breaker Pattern

```go
type CircuitBreaker struct {
    mu           sync.RWMutex
    state        string  // "closed", "open", "half-open"
    failCount    int
    failThresh   int
    lastFailTime time.Time
    timeout      time.Duration
}

func NewCircuitBreaker(failThresh int, timeout time.Duration) *CircuitBreaker {
    return &CircuitBreaker{
        state:      "closed",
        failThresh: failThresh,
        timeout:    timeout,
    }
}

func (cb *CircuitBreaker) Call(fn func() error) error {
    cb.mu.RLock()
    state := cb.state
    cb.mu.RUnlock()

    // Circuit is open
    if state == "open" {
        if time.Since(cb.lastFailTime) > cb.timeout {
            cb.mu.Lock()
            cb.state = "half-open"
            cb.mu.Unlock()
        } else {
            return errors.New("circuit breaker is open")
        }
    }

    // Try the operation
    err := fn()

    cb.mu.Lock()
    defer cb.mu.Unlock()

    if err != nil {
        cb.failCount++
        cb.lastFailTime = time.Now()

        if cb.failCount >= cb.failThresh {
            cb.state = "open"
        }
        return err
    }

    // Success: reset
    cb.failCount = 0
    cb.state = "closed"
    return nil
}

// Usage
breaker := NewCircuitBreaker(5, 10*time.Second)

err := breaker.Call(func() error {
    return callExternalService()
})
```

### 4.5 Retrying with Exponential Backoff

```go
type Retry struct {
    MaxAttempts int
    BaseDelay   time.Duration
    MaxDelay    time.Duration
}

func (r *Retry) Do(fn func() error) error {
    var lastErr error

    for attempt := 1; attempt <= r.MaxAttempts; attempt++ {
        err := fn()
        if err == nil {
            return nil
        }

        lastErr = err

        if attempt < r.MaxAttempts {
            delay := r.calculateBackoff(attempt)
            time.Sleep(delay)
        }
    }

    return lastErr
}

func (r *Retry) calculateBackoff(attempt int) time.Duration {
    delay := r.BaseDelay * time.Duration(math.Pow(2, float64(attempt-1)))

    if delay > r.MaxDelay {
        delay = r.MaxDelay
    }

    // Add jitter to prevent thundering herd
    jitter := time.Duration(rand.Int63n(int64(delay / 10)))
    return delay + jitter
}

// Usage
retry := &Retry{
    MaxAttempts: 5,
    BaseDelay:   100 * time.Millisecond,
    MaxDelay:    30 * time.Second,
}

err := retry.Do(func() error {
    return callUnreliableService()
})
```

---

## PART 5: ARCHITECTURE (DDD, SOLID, CLEAN ARCHITECTURE)

### 5.1 Domain-Driven Design (DDD)

DDD structures code around business domains, not technical layers.

```go
// Domain layer - Core business logic
package domain

// Value Object: No identity, immutable
type Money struct {
    amount   decimal.Decimal
    currency string
}

func (m Money) Add(other Money) (Money, error) {
    if m.currency != other.currency {
        return Money{}, errors.New("currencies must match")
    }
    return Money{
        amount:   m.amount.Add(other.amount),
        currency: m.currency,
    }, nil
}

// Entity: Has identity, mutable
type Order struct {
    id        OrderID
    customerId CustomerID
    items     []OrderItem
    status    OrderStatus
    total     Money
}

func (o *Order) AddItem(item OrderItem) error {
    if o.status != OrderStatusOpen {
        return errors.New("cannot add item to closed order")
    }
    o.items = append(o.items, item)
    return nil
}

func (o *Order) Complete() error {
    if len(o.items) == 0 {
        return errors.New("order must have items")
    }
    o.status = OrderStatusCompleted
    return nil
}

// Aggregate: Cluster of entities with single root
type OrderAggregate struct {
    order      *Order
    customer   *Customer
    items      []OrderItem
}

// Repository interface (defined in domain)
type OrderRepository interface {
    Save(order *Order) error
    FindByID(id OrderID) (*Order, error)
}

// Application layer - Use cases
package application

type CreateOrderService struct {
    orderRepo  domain.OrderRepository
    itemRepo   domain.ItemRepository
    eventBus   EventBus
}

func (s *CreateOrderService) Execute(cmd CreateOrderCommand) error {
    // Fetch aggregates
    order, err := s.orderRepo.FindByID(cmd.OrderID)
    if err != nil {
        return err
    }

    // Business logic
    for _, itemID := range cmd.ItemIDs {
        item, _ := s.itemRepo.FindByID(itemID)
        order.AddItem(item)
    }

    // Save aggregate
    if err := s.orderRepo.Save(order); err != nil {
        return err
    }

    // Publish domain events
    s.eventBus.Publish(domain.OrderCreatedEvent{
        OrderID:  order.id,
        Total:    order.total,
        Time:     time.Now(),
    })

    return nil
}

// Infrastructure layer - Implementation details
package infrastructure

type PostgresOrderRepository struct {
    db *sql.DB
}

func (r *PostgresOrderRepository) Save(order *domain.Order) error {
    // SQL implementation
    _, err := r.db.Exec(
        "INSERT INTO orders (id, customer_id, status, total) VALUES ($1, $2, $3, $4)",
        order.ID(), order.CustomerID(), order.Status(), order.Total(),
    )
    return err
}

// Presentation layer - HTTP handlers
package presentation

type OrderHandler struct {
    createOrderService *application.CreateOrderService
}

func (h *OrderHandler) CreateOrder(w http.ResponseWriter, r *http.Request) {
    var req CreateOrderRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }

    cmd := application.CreateOrderCommand{
        OrderID: domain.OrderID(req.OrderID),
        ItemIDs: req.ItemIDs,
    }

    if err := h.createOrderService.Execute(cmd); err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(map[string]string{"status": "created"})
}
```

### 5.2 SOLID Principles

**S - Single Responsibility Principle**:
```go
// Bad: Multiple responsibilities
type User struct {
    ID    int
    Name  string
}

func (u *User) Save(db *sql.DB) error {
    // Database logic mixed with entity
    return nil
}

func (u *User) SendEmail() error {
    // Email logic mixed with entity
    return nil
}

// Good: Separate concerns
type User struct {
    ID   int
    Name string
}

type UserRepository interface {
    Save(user *User) error
}

type EmailService interface {
    SendWelcomeEmail(user *User) error
}
```

**O - Open/Closed Principle**:
```go
// Bad: Needs modification for new types
func CalculateDiscount(userType string, amount float64) float64 {
    if userType == "regular" {
        return amount * 0.1
    } else if userType == "premium" {
        return amount * 0.2
    } else if userType == "vip" {
        return amount * 0.3
    }
    return 0
}

// Good: Open for extension, closed for modification
type DiscountCalculator interface {
    Calculate(amount float64) float64
}

type RegularDiscount struct{}
func (d RegularDiscount) Calculate(amount float64) float64 {
    return amount * 0.1
}

type PremiumDiscount struct{}
func (d PremiumDiscount) Calculate(amount float64) float64 {
    return amount * 0.2
}

func CalculateDiscount(calc DiscountCalculator, amount float64) float64 {
    return calc.Calculate(amount)
}
```

**L - Liskov Substitution Principle**:
```go
// Bad: Violates LSP
type Bird interface {
    Fly()
}

type Penguin struct{}
func (p Penguin) Fly() {
    panic("penguins can't fly!")
}

// Good: Proper abstraction
type FlyingBird interface {
    Fly()
}

type SwimmingBird interface {
    Swim()
}

type Penguin struct{}
func (p Penguin) Swim() {}
```

**I - Interface Segregation Principle**:
```go
// Bad: Fat interface
type Worker interface {
    Work()
    Manage()
    Report()
}

// Good: Smaller, focused interfaces
type Worker interface {
    Work()
}

type Manager interface {
    Manage()
}

type Reporter interface {
    Report()
}

// Combine as needed
type Employee interface {
    Worker
    Manager
}
```

**D - Dependency Inversion Principle**:
```go
// Bad: Depends on concrete implementation
type OrderService struct {
    emailService *EmailService
}

// Good: Depends on abstraction
type EmailSender interface {
    Send(to, subject, body string) error
}

type OrderService struct {
    emailSender EmailSender
}

func NewOrderService(emailSender EmailSender) *OrderService {
    return &OrderService{emailSender: emailSender}
}
```

### 5.3 Clean Architecture

```
project/
├── cmd/
│   └── api/
│       └── main.go
├── internal/
│   ├── domain/           # Business rules
│   │   ├── models.go
│   │   └── repositories.go
│   ├── usecases/         # Application logic
│   │   ├── create_user.go
│   │   └── list_users.go
│   ├── gateways/         # External services
│   │   ├── http_handler.go
│   │   └── database.go
│   └── infrastructure/   # Implementation details
│       ├── postgres.go
│       └── redis.go
├── pkg/                  # Reusable packages
│   └── errors/
└── go.mod
```

```go
// Domain layer (no dependencies)
package domain

type UserRepository interface {
    Save(user *User) error
    FindByID(id int) (*User, error)
}

// Use case layer
package usecases

type CreateUserUseCase struct {
    repo domain.UserRepository
}

func (uc *CreateUserUseCase) Execute(name, email string) (*domain.User, error) {
    user := domain.NewUser(name, email)
    return user, uc.repo.Save(user)
}

// Gateway (HTTP handler)
package gateways

type UserHandler struct {
    createUserUseCase *usecases.CreateUserUseCase
}

func (h *UserHandler) Create(w http.ResponseWriter, r *http.Request) {
    var req CreateUserRequest
    json.NewDecoder(r.Body).Decode(&req)

    user, err := h.createUserUseCase.Execute(req.Name, req.Email)
    if err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }

    json.NewEncoder(w).Encode(user)
}

// Infrastructure (Database implementation)
package infrastructure

type PostgresUserRepository struct {
    db *sql.DB
}

func (r *PostgresUserRepository) Save(user *domain.User) error {
    _, err := r.db.Exec(
        "INSERT INTO users (name, email) VALUES ($1, $2)",
        user.Name, user.Email,
    )
    return err
}
```

---

## PART 6: ADVANCED PATTERNS (EVENT SOURCING & CQRS)

### 6.1 Event Sourcing

Instead of storing current state, store all changes (events).

```go
package domain

// Events
type Event interface {
    AggregateID() string
    Timestamp() time.Time
}

type AccountCreatedEvent struct {
    id        string
    owner     string
    timestamp time.Time
}

func (e AccountCreatedEvent) AggregateID() string {
    return e.id
}

func (e AccountCreatedEvent) Timestamp() time.Time {
    return e.timestamp
}

type MoneyDepositedEvent struct {
    id        string
    amount    decimal.Decimal
    timestamp time.Time
}

func (e MoneyDepositedEvent) AggregateID() string {
    return e.id
}

// Aggregate that rebuilds from events
type Account struct {
    id       string
    owner    string
    balance  decimal.Decimal
    version  int
    events   []Event
}

func (a *Account) ApplyEvent(event Event) error {
    switch e := event.(type) {
    case AccountCreatedEvent:
        a.id = e.AggregateID()
        a.owner = e.owner
        a.balance = decimal.Zero
    case MoneyDepositedEvent:
        a.balance = a.balance.Add(e.amount)
    }
    a.version++
    return nil
}

func (a *Account) Deposit(amount decimal.Decimal) error {
    event := MoneyDepositedEvent{
        id:        a.id,
        amount:    amount,
        timestamp: time.Now(),
    }
    return a.ApplyEvent(event)
}

// Event store
type EventStore interface {
    Append(events ...Event) error
    GetEvents(aggregateID string) ([]Event, error)
}

type InMemoryEventStore struct {
    mu     sync.RWMutex
    events map[string][]Event
}

func (es *InMemoryEventStore) Append(events ...Event) error {
    es.mu.Lock()
    defer es.mu.Unlock()

    for _, event := range events {
        id := event.AggregateID()
        es.events[id] = append(es.events[id], event)
    }
    return nil
}

func (es *InMemoryEventStore) GetEvents(aggregateID string) ([]Event, error) {
    es.mu.RLock()
    defer es.mu.RUnlock()

    return es.events[aggregateID], nil
}

// Repository that uses event store
type AccountRepository struct {
    eventStore EventStore
}

func (r *AccountRepository) Save(account *Account) error {
    return r.eventStore.Append(account.events...)
}

func (r *AccountRepository) FindByID(id string) (*Account, error) {
    events, err := r.eventStore.GetEvents(id)
    if err != nil {
        return nil, err
    }

    account := &Account{}
    for _, event := range events {
        account.ApplyEvent(event)
    }
    return account, nil
}
```

### 6.2 CQRS (Command Query Responsibility Segregation)

Separate read and write models.

```go
package application

// Write side - Commands
type CreateAccountCommand struct {
    ID    string
    Owner string
}

type DepositMoneyCommand struct {
    AccountID string
    Amount    decimal.Decimal
}

type CommandHandler interface {
    Handle(cmd interface{}) error
}

type CreateAccountCommandHandler struct {
    repo EventStore
}

func (h *CreateAccountCommandHandler) Handle(cmd *CreateAccountCommand) error {
    account := &domain.Account{}

    event := domain.AccountCreatedEvent{
        id:        cmd.ID,
        owner:     cmd.Owner,
        timestamp: time.Now(),
    }

    account.ApplyEvent(event)
    return h.repo.Append(event)
}

// Read side - Queries and views
type GetAccountQuery struct {
    ID string
}

type AccountView struct {
    ID      string
    Owner   string
    Balance decimal.Decimal
}

type QueryHandler interface {
    Handle(query interface{}) (interface{}, error)
}

type AccountProjection struct {
    mu       sync.RWMutex
    accounts map[string]AccountView
}

func (p *AccountProjection) Handle(query *GetAccountQuery) (*AccountView, error) {
    p.mu.RLock()
    defer p.mu.RUnlock()

    view, ok := p.accounts[query.ID]
    if !ok {
        return nil, errors.New("account not found")
    }
    return &view, nil
}

// Event handler updates projections
type AccountProjectionUpdater struct {
    projection *AccountProjection
}

func (u *AccountProjectionUpdater) OnAccountCreated(event domain.AccountCreatedEvent) {
    u.projection.mu.Lock()
    defer u.projection.mu.Unlock()

    u.projection.accounts[event.AggregateID()] = AccountView{
        ID:      event.AggregateID(),
        Owner:   event.owner,
        Balance: decimal.Zero,
    }
}

func (u *AccountProjectionUpdater) OnMoneyDeposited(event domain.MoneyDepositedEvent) {
    u.projection.mu.Lock()
    defer u.projection.mu.Unlock()

    view := u.projection.accounts[event.AggregateID()]
    view.Balance = view.Balance.Add(event.amount)
    u.projection.accounts[event.AggregateID()] = view
}
```

### 6.3 Saga Pattern

For distributed transactions across services.

```go
package application

type PaymentSaga struct {
    mu       sync.Mutex
    state    string
    orderID  string
    paymentID string
}

// Orchestration-based Saga
func (s *PaymentSaga) Start(order Order, payment Payment) error {
    s.mu.Lock()
    s.state = "started"
    s.orderID = order.ID
    s.mu.Unlock()

    // Step 1: Process payment
    if err := s.processPayment(payment); err != nil {
        return s.compensatePayment()
    }

    // Step 2: Reserve inventory
    if err := s.reserveInventory(order); err != nil {
        s.compensatePayment()
        return s.compensateInventory(order)
    }

    // Step 3: Create shipment
    if err := s.createShipment(order); err != nil {
        s.compensatePayment()
        s.compensateInventory(order)
        return s.compensateShipment(order)
    }

    s.mu.Lock()
    s.state = "completed"
    s.mu.Unlock()

    return nil
}

func (s *PaymentSaga) processPayment(payment Payment) error {
    // Call payment service
    return paymentService.Charge(payment)
}

func (s *PaymentSaga) compensatePayment() error {
    // Call payment service to refund
    return paymentService.Refund(s.paymentID)
}

func (s *PaymentSaga) reserveInventory(order Order) error {
    return inventoryService.Reserve(order.Items)
}

func (s *PaymentSaga) compensateInventory(order Order) error {
    return inventoryService.Release(order.Items)
}

func (s *PaymentSaga) createShipment(order Order) error {
    return shipmentService.Create(order)
}

func (s *PaymentSaga) compensateShipment(order Order) error {
    return shipmentService.Cancel(order.ID)
}

// Choreography-based Saga using events
type OrderService struct {
    eventBus EventBus
}

func (s *OrderService) CreateOrder(order Order) error {
    if err := s.saveOrder(order); err != nil {
        return err
    }

    // Publish event - others listen and act
    return s.eventBus.Publish(OrderCreatedEvent{OrderID: order.ID})
}

// Payment service listens and charges
type PaymentServiceListener struct {
    paymentService PaymentService
}

func (l *PaymentServiceListener) OnOrderCreated(event OrderCreatedEvent) {
    order := getOrder(event.OrderID)
    l.paymentService.Charge(order.Total)
}

// Inventory service listens and reserves
type InventoryServiceListener struct {
    inventoryService InventoryService
}

func (l *InventoryServiceListener) OnOrderCreated(event OrderCreatedEvent) {
    order := getOrder(event.OrderID)
    l.inventoryService.Reserve(order.Items)
}
```

---

## PART 7: PRODUCTION SKILLS

### 7.1 Structured Logging

```go
import "github.com/sirupsen/logrus"

type StructuredLogger struct {
    logger *logrus.Logger
}

func NewStructuredLogger() *StructuredLogger {
    logger := logrus.New()
    logger.SetFormatter(&logrus.JSONFormatter{
        TimestampFormat: time.RFC3339,
    })
    logger.SetLevel(logrus.InfoLevel)
    return &StructuredLogger{logger: logger}
}

func (sl *StructuredLogger) LogUserCreated(userID, email string) {
    sl.logger.WithFields(logrus.Fields{
        "event":   "user_created",
        "user_id": userID,
        "email":   email,
        "timestamp": time.Now(),
    }).Info("User successfully created")
}

func (sl *StructuredLogger) LogError(operation string, err error, metadata map[string]interface{}) {
    fields := logrus.Fields{
        "operation": operation,
        "error":     err.Error(),
    }
    for k, v := range metadata {
        fields[k] = v
    }
    sl.logger.WithFields(fields).Error("Operation failed")
}

// Output:
// {"event":"user_created","user_id":"123","email":"john@example.com","level":"info","msg":"User successfully created","time":"2024-01-20T10:30:45Z"}
```

### 7.2 Distributed Tracing

```go
import (
    "github.com/opentelemetry/otel"
    "github.com/opentelemetry/otel/trace"
)

type TracedUserService struct {
    tracer trace.Tracer
    repo   UserRepository
}

func NewTracedUserService(tracer trace.Tracer, repo UserRepository) *TracedUserService {
    return &TracedUserService{tracer: tracer, repo: repo}
}

func (us *TracedUserService) GetUser(ctx context.Context, userID string) (*User, error) {
    ctx, span := us.tracer.Start(ctx, "GetUser")
    defer span.End()

    // Add attributes for debugging
    span.SetAttributes(
        attribute.String("user_id", userID),
    )

    // Trace child operation
    ctx, dbSpan := us.tracer.Start(ctx, "FetchFromDatabase")
    user, err := us.repo.FindByID(ctx, userID)
    dbSpan.End()

    if err != nil {
        span.RecordError(err)
        return nil, err
    }

    return user, nil
}

// Setup tracing exporter
func InitTracing() {
    exporter, _ := jaeger.New(
        jaeger.WithAgentHost("localhost"),
    )

    tp := tracesdk.NewTracerProvider(
        tracesdk.WithBatcher(exporter),
    )

    otel.SetTracerProvider(tp)
}
```

### 7.3 Metrics and Monitoring

```go
import "github.com/prometheus/client_golang/prometheus"

type MetricsCollector struct {
    requestCount   prometheus.Counter
    requestLatency prometheus.Histogram
    activeRequests prometheus.Gauge
}

func NewMetricsCollector() *MetricsCollector {
    return &MetricsCollector{
        requestCount: prometheus.NewCounter(prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total HTTP requests",
        }),
        requestLatency: prometheus.NewHistogram(prometheus.HistogramOpts{
            Name:    "http_request_duration_seconds",
            Help:    "HTTP request latency",
            Buckets: prometheus.DefBuckets,
        }),
        activeRequests: prometheus.NewGauge(prometheus.GaugeOpts{
            Name: "http_requests_active",
            Help: "Active HTTP requests",
        }),
    }
}

func (mc *MetricsCollector) RecordRequest(duration time.Duration) {
    mc.requestCount.Inc()
    mc.requestLatency.Observe(duration.Seconds())
}

// Middleware
func MetricsMiddleware(mc *MetricsCollector, next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        mc.activeRequests.Inc()
        defer mc.activeRequests.Dec()

        start := time.Now()
        next.ServeHTTP(w, r)
        mc.RecordRequest(time.Since(start))
    })
}

// Expose Prometheus endpoint
func main() {
    mc := NewMetricsCollector()
    prometheus.MustRegister(
        mc.requestCount,
        mc.requestLatency,
        mc.activeRequests,
    )

    http.Handle("/metrics", promhttp.Handler())
    http.ListenAndServe(":8080", nil)
}
```

### 7.4 Error Handling at Scale

```go
type AppError struct {
    Code    string // Machine-readable error code
    Message string // User-friendly message
    Details map[string]interface{}
    Cause   error  // Original error
}

func (e AppError) Error() string {
    return fmt.Sprintf("%s: %s", e.Code, e.Message)
}

// Error codes
const (
    ErrCodeValidation   = "VALIDATION_ERROR"
    ErrCodeNotFound     = "NOT_FOUND"
    ErrCodeUnauthorized = "UNAUTHORIZED"
    ErrCodeInternal     = "INTERNAL_ERROR"
    ErrCodeDatabase     = "DATABASE_ERROR"
)

// Create errors with context
func NewValidationError(field, reason string) AppError {
    return AppError{
        Code:    ErrCodeValidation,
        Message: fmt.Sprintf("Validation failed for field '%s'", field),
        Details: map[string]interface{}{
            "field":  field,
            "reason": reason,
        },
    }
}

// Error handler middleware
func ErrorHandlerMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        defer func() {
            if err := recover(); err != nil {
                handleError(w, AppError{
                    Code:    ErrCodeInternal,
                    Message: "Internal server error",
                    Cause:   fmt.Errorf("%v", err),
                })
            }
        }()

        next.ServeHTTP(w, r)
    })
}

func handleError(w http.ResponseWriter, appErr AppError) {
    w.Header().Set("Content-Type", "application/json")

    statusCode := http.StatusInternalServerError
    switch appErr.Code {
    case ErrCodeValidation:
        statusCode = http.StatusBadRequest
    case ErrCodeNotFound:
        statusCode = http.StatusNotFound
    case ErrCodeUnauthorized:
        statusCode = http.StatusUnauthorized
    }

    w.WriteHeader(statusCode)
    json.NewEncoder(w).Encode(map[string]interface{}{
        "code":    appErr.Code,
        "message": appErr.Message,
        "details": appErr.Details,
    })
}

// Usage
func GetUser(w http.ResponseWriter, r *http.Request) {
    userID := r.URL.Query().Get("id")

    if userID == "" {
        handleError(w, NewValidationError("id", "required"))
        return
    }

    user, err := userRepo.FindByID(userID)
    if err != nil {
        handleError(w, AppError{
            Code:    ErrCodeDatabase,
            Message: "Failed to fetch user",
            Cause:   err,
        })
        return
    }

    if user == nil {
        handleError(w, AppError{
            Code:    ErrCodeNotFound,
            Message: "User not found",
            Details: map[string]interface{}{"id": userID},
        })
        return
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(user)
}
```

### 7.5 Graceful Shutdown

```go
type App struct {
    server     *http.Server
    db         *sql.DB
    logger     *logrus.Logger
    shutdownCh chan os.Signal
}

func (app *App) Start(addr string) error {
    app.server = &http.Server{
        Addr:         addr,
        Handler:      app.setupRoutes(),
        ReadTimeout:  15 * time.Second,
        WriteTimeout: 15 * time.Second,
        IdleTimeout:  60 * time.Second,
    }

    // Start server in goroutine
    go func() {
        if err := app.server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            app.logger.Fatalf("Server error: %v", err)
        }
    }()

    app.logger.Infof("Server started on %s", addr)

    // Wait for shutdown signal
    <-app.shutdownCh
    app.logger.Info("Shutdown signal received")

    return app.Shutdown()
}

func (app *App) Shutdown() error {
    app.logger.Info("Starting graceful shutdown")

    // Create shutdown context with timeout
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()

    // Stop accepting new requests
    if err := app.server.Shutdown(ctx); err != nil {
        app.logger.Errorf("Server shutdown error: %v", err)
        return err
    }

    // Close database
    if err := app.db.Close(); err != nil {
        app.logger.Errorf("Database close error: %v", err)
        return err
    }

    app.logger.Info("Graceful shutdown completed")
    return nil
}

func (app *App) NotifyShutdown(sig os.Signal) {
    app.shutdownCh <- sig
}

// Usage
func main() {
    app := NewApp()

    // Handle signals
    sigChan := make(chan os.Signal, 1)
    signal.Notify(sigChan, os.Interrupt, syscall.SIGTERM)

    go func() {
        sig := <-sigChan
        app.NotifyShutdown(sig)
    }()

    if err := app.Start(":8080"); err != nil {
        log.Fatalf("Application error: %v", err)
    }
}
```

### 7.6 Health Checks and Readiness Probes

```go
type HealthChecker struct {
    checks map[string]func(context.Context) error
    mu     sync.RWMutex
}

func NewHealthChecker() *HealthChecker {
    return &HealthChecker{
        checks: make(map[string]func(context.Context) error),
    }
}

func (hc *HealthChecker) Register(name string, check func(context.Context) error) {
    hc.mu.Lock()
    defer hc.mu.Unlock()
    hc.checks[name] = check
}

func (hc *HealthChecker) CheckLiveness(ctx context.Context) bool {
    // Check if app is alive (basic checks)
    return true
}

func (hc *HealthChecker) CheckReadiness(ctx context.Context) (map[string]bool, error) {
    // Check if app is ready to serve traffic
    hc.mu.RLock()
    checks := hc.checks
    hc.mu.RUnlock()

    results := make(map[string]bool)

    for name, check := range checks {
        err := check(ctx)
        results[name] = err == nil
    }

    return results, nil
}

// HTTP handlers
func (hc *HealthChecker) LivenessHandler(w http.ResponseWriter, r *http.Request) {
    if hc.CheckLiveness(r.Context()) {
        w.WriteHeader(http.StatusOK)
        w.Write([]byte("OK"))
    } else {
        w.WriteHeader(http.StatusServiceUnavailable)
    }
}

func (hc *HealthChecker) ReadinessHandler(w http.ResponseWriter, r *http.Request) {
    results, _ := hc.CheckReadiness(r.Context())

    ready := true
    for _, ok := range results {
        if !ok {
            ready = false
            break
        }
    }

    w.Header().Set("Content-Type", "application/json")

    if ready {
        w.WriteHeader(http.StatusOK)
    } else {
        w.WriteHeader(http.StatusServiceUnavailable)
    }

    json.NewEncoder(w).Encode(results)
}

// Usage
func main() {
    hc := NewHealthChecker()

    // Register checks
    hc.Register("database", func(ctx context.Context) error {
        return db.Ping(ctx)
    })

    hc.Register("redis", func(ctx context.Context) error {
        return redisClient.Ping(ctx).Err()
    })

    http.HandleFunc("/live", hc.LivenessHandler)
    http.HandleFunc("/ready", hc.ReadinessHandler)
}
```

---

## INTEGRATION: Building a Production System

```go
package main

import (
    "context"
    "log"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"

    "github.com/sirupsen/logrus"
    "go.opentelemetry.io/otel"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

type App struct {
    server         *http.Server
    db             *sql.DB
    logger         *logrus.Logger
    metrics        *MetricsCollector
    healthChecker  *HealthChecker
    shutdownCh     chan os.Signal
}

func NewApp() *App {
    logger := logrus.New()
    logger.SetFormatter(&logrus.JSONFormatter{})

    db, _ := sql.Open("postgres", "...")

    metrics := NewMetricsCollector()

    hc := NewHealthChecker()
    hc.Register("database", db.Ping)

    return &App{
        logger:        logger,
        db:            db,
        metrics:       metrics,
        healthChecker: hc,
        shutdownCh:    make(chan os.Signal, 1),
    }
}

func (app *App) setupRoutes() http.Handler {
    router := http.NewServeMux()

    // Setup middleware chain
    var handler http.Handler = router
    handler = MetricsMiddleware(app.metrics, handler)
    handler = ErrorHandlerMiddleware(handler)
    handler = LoggingMiddleware(app.logger, handler)

    // Routes
    router.HandleFunc("/api/users", app.handleCreateUser)
    router.HandleFunc("/live", app.healthChecker.LivenessHandler)
    router.HandleFunc("/ready", app.healthChecker.ReadinessHandler)
    router.Handle("/metrics", promhttp.Handler())

    return handler
}

func (app *App) handleCreateUser(w http.ResponseWriter, r *http.Request) {
    ctx, span := otel.Tracer("app").Start(r.Context(), "CreateUser")
    defer span.End()

    var req CreateUserRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        handleError(w, NewValidationError("body", err.Error()))
        return
    }

    user := &User{
        Name:  req.Name,
        Email: req.Email,
    }

    if err := app.createUser(ctx, user); err != nil {
        app.logger.WithError(err).Error("Failed to create user")
        handleError(w, AppError{
            Code:    ErrCodeDatabase,
            Message: "Failed to create user",
            Cause:   err,
        })
        return
    }

    app.logger.WithFields(logrus.Fields{
        "user_id": user.ID,
        "email":   user.Email,
    }).Info("User created")

    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(user)
}

func (app *App) createUser(ctx context.Context, user *User) error {
    // Business logic with retries and timeouts
    retry := &Retry{
        MaxAttempts: 3,
        BaseDelay:   100 * time.Millisecond,
        MaxDelay:    10 * time.Second,
    }

    return retry.Do(func() error {
        ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
        defer cancel()

        result, err := app.db.ExecContext(
            ctx,
            "INSERT INTO users (name, email) VALUES ($1, $2)",
            user.Name,
            user.Email,
        )
        if err != nil {
            return err
        }

        id, _ := result.LastInsertId()
        user.ID = int(id)
        return nil
    })
}

func (app *App) Start(addr string) error {
    app.server = &http.Server{
        Addr:         addr,
        Handler:      app.setupRoutes(),
        ReadTimeout:  15 * time.Second,
        WriteTimeout: 15 * time.Second,
    }

    go func() {
        if err := app.server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            app.logger.Fatalf("Server error: %v", err)
        }
    }()

    app.logger.Infof("Server started on %s", addr)

    <-app.shutdownCh
    return app.Shutdown()
}

func (app *App) Shutdown() error {
    app.logger.Info("Starting graceful shutdown")

    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()

    if err := app.server.Shutdown(ctx); err != nil {
        return err
    }

    return app.db.Close()
}

func main() {
    app := NewApp()

    sigChan := make(chan os.Signal, 1)
    signal.Notify(sigChan, os.Interrupt, syscall.SIGTERM)

    go func() {
        app.shutdownCh <- <-sigChan
    }()

    if err := app.Start(":8080"); err != nil {
        log.Fatalf("Application error: %v", err)
    }
}
```
