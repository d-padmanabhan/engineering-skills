# Go Patterns

## Context & Cancellation

```go
// Context as first parameter
func fetchUser(ctx context.Context, id string) (*User, error) {
    req, err := http.NewRequestWithContext(ctx, "GET", "/users/"+id, nil)
    if err != nil {
        return nil, err
    }
    // ...
}

// Check context in loops
for _, item := range items {
    select {
    case <-ctx.Done():
        return ctx.Err()
    default:
        process(item)
    }
}

// Timeout context
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()
result, err := fetchData(ctx)
```

## Concurrency

```go
// WaitGroup for goroutine coordination
var wg sync.WaitGroup
for _, item := range items {
    wg.Add(1)
    go func(item Item) {
        defer wg.Done()
        process(item)
    }(item)
}
wg.Wait()

// Mutex for shared state
type SafeCounter struct {
    mu    sync.RWMutex
    count map[string]int
}

func (c *SafeCounter) Inc(key string) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count[key]++
}

func (c *SafeCounter) Get(key string) int {
    c.mu.RLock()
    defer c.mu.RUnlock()
    return c.count[key]
}

// Channels for communication
results := make(chan Result, len(items))
for _, item := range items {
    go func(item Item) {
        results <- process(item)
    }(item)
}
for range items {
    result := <-results
    handle(result)
}
```

## Interfaces

```go
// Small, focused interfaces
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

// Compose interfaces
type ReadWriter interface {
    Reader
    Writer
}

// Accept interfaces, return structs
func ProcessData(r io.Reader) (*Result, error) {
    // ...
}
```

## Testing

```go
// Table-driven tests
func TestParse(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        want    int
        wantErr bool
    }{
        {"valid", "123", 123, false},
        {"negative", "-5", -5, false},
        {"invalid", "abc", 0, true},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := Parse(tt.input)
            if (err != nil) != tt.wantErr {
                t.Errorf("Parse() error = %v, wantErr %v", err, tt.wantErr)
                return
            }
            if got != tt.want {
                t.Errorf("Parse() = %v, want %v", got, tt.want)
            }
        })
    }
}

// Subtests and parallelism
func TestAPI(t *testing.T) {
    t.Run("Create", func(t *testing.T) {
        t.Parallel()
        // ...
    })
    t.Run("Update", func(t *testing.T) {
        t.Parallel()
        // ...
    })
}
```

## HTTP Handlers

```go
// Handler with dependency injection
type UserHandler struct {
    userService UserService
    logger      *log.Logger
}

func (h *UserHandler) GetUser(w http.ResponseWriter, r *http.Request) {
    id := r.PathValue("id")
    user, err := h.userService.Get(r.Context(), id)
    if err != nil {
        http.Error(w, "User not found", http.StatusNotFound)
        return
    }
    json.NewEncoder(w).Encode(user)
}

// Middleware pattern
func LoggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        next.ServeHTTP(w, r)
        log.Printf("%s %s %v", r.Method, r.URL.Path, time.Since(start))
    })
}
```

## Struct Tags

```go
type User struct {
    ID        string    `json:"id" db:"id"`
    Name      string    `json:"name" db:"name" validate:"required"`
    Email     string    `json:"email" db:"email" validate:"email"`
    CreatedAt time.Time `json:"created_at" db:"created_at"`
}
```

## Generics (Go 1.18+)

```go
// Generic function
func Filter[T any](items []T, predicate func(T) bool) []T {
    result := make([]T, 0)
    for _, item := range items {
        if predicate(item) {
            result = append(result, item)
        }
    }
    return result
}

// Generic type with constraint
type Number interface {
    int | int64 | float64
}

func Sum[T Number](numbers []T) T {
    var sum T
    for _, n := range numbers {
        sum += n
    }
    return sum
}
```

## golangci-lint Configuration

```yaml
# .golangci.yml
linters:
  enable:
    - errcheck
    - gosimple
    - govet
    - ineffassign
    - staticcheck
    - unused
    - gosec
    - gofmt
    - goimports

linters-settings:
  errcheck:
    check-type-assertions: true
  govet:
    check-shadowing: true

issues:
  exclude-use-default: false
```

