---
title: 别再写 Go 反模式代码了
description: 本文介绍 Go 中常见的反模式，帮助开发者编写更健壮、可维护的代码。
---

# 别再写 Go 反模式代码了

在软件开发中，我们不仅要学习最佳实践，还要了解并避免常见的“反模式”（Anti-Patterns）。反模式是指那些在实践中常常出现但会导致问题的解决方案。它们看起来似乎是正确的，但长期来看会引入不必要的复杂性、降低代码质量，甚至引发难以调试的 Bug。

本文将结合多个 Go 社区的讨论和文章，为你梳理出 20 个常见的 Go 反模式，并提供相应的“好”与“坏”的代码示例，帮助你识别并避免这些陷阱，写出更地道、更高效的 Go 代码。

## 1. 使用 `make([]T, 0)` 初始化空切片

在 Go 中，一个 `nil` 切片在很多情况下和一个空切片（长度为 0）的行为是一致的，例如 `append`、`len`、`cap` 和 `for...range`。使用 `nil` 切片可以减少不必要的内存分配。

**反模式 (Bad Case):**

```go
// 不必要的内存分配
s := make([]int, 0)
```

**推荐实践 (Good Case):**

```go
var s []int
// 或者 s := []int(nil)
```

**解释:**

`nil` 切片不需要分配内存，而 `make([]int, 0)` 会分配一个虽然为空但有底层数组的切片。在大多数情况下，`nil` 切片是更简洁、更高效的选择。

## 2. 在非必要情况下使用 `panic`

`panic` 在 Go 中用于表示不可恢复的错误，它会中断正常的程序执行流程。滥用 `panic` 会使错误处理变得混乱，难以维护。

**反模式 (Bad Case):**

```go
func GetUser(id int) (*User, error) {
    user, err := db.QueryUser(id)
    if err != nil {
        // 对于可预见的错误使用 panic
        panic("database error")
    }
    return user, nil
}
```

**推荐实践 (Good Case):**

```go
func GetUser(id int) (*User, error) {
    user, err := db.QueryUser(id)
    if err != nil {
        // 返回错误，让调用者决定如何处理
        return nil, fmt.Errorf("failed to get user: %w", err)
    }
    return user, nil
}
```

**解释:**

Go 的设计哲学是明确地处理错误。对于可预见的错误（如数据库连接失败、文件未找到等），应该返回一个 `error` 值，由调用者决定如何处理。`panic` 只应用于程序无法继续运行的场景，例如初始化失败。

## 3. 忽略错误

Go 的多返回值机制强制你关注错误，但有时开发者会为了“方便”而忽略它们。

**反模式 (Bad Case):**

```go
// 忽略了 f.Close() 可能返回的错误
f, _ := os.Open("file.txt")
defer f.Close() 
```

**推荐实践 (Good Case):**

```go
f, err := os.Open("file.txt")
if err != nil {
    // 处理错误
}
defer func() {
    if err := f.Close(); err != nil {
        // 处理关闭文件时的错误
    }
}()
```

**解释:**

忽略错误可能会导致资源泄露、数据损坏或其他静默失败。即使是像 `f.Close()` 这样的操作也可能失败（例如，在写入数据到缓冲区后），因此检查所有返回的错误是至关重要的。

## 4. 对多种职责使用单一模型

在 Web 开发中，经常会有一个模型（struct）同时用于数据库操作（如 GORM）、JSON 序列化和业务逻辑。这会导致模型变得臃肿，并且高度耦合。

**反模式 (Bad Case):**

```go
type User struct {
    ID        int    `json:"id" gorm:"primaryKey"`
    Email     string `json:"email" gorm:"uniqueIndex" validate:"required,email"`
    Password  string `json:"-" gorm:"column:password_hash"` // 在 JSON 中隐藏
    CreatedAt time.Time `json:"created_at"`
}
```

**推荐实践 (Good Case):**

```go
// 数据库模型
type UserDB struct {
    ID           int
    Email        string
    PasswordHash string
    CreatedAt    time.Time
}

// API 请求模型
type CreateUserRequest struct {
    Email    string `json:"email" validate:"required,email"`
    Password string `json:"password" validate:"required"`
}

// API 响应模型
type UserResponse struct {
    ID        int       `json:"id"`
    Email     string    `json:"email"`
    CreatedAt time.Time `json:"created_at"`
}
```

**解释:**

将不同职责的模型分开可以实现关注点分离。数据库模型只关心数据存储，API 模型只关心数据传输和验证。这样可以使代码更清晰、更易于维护，并减少因修改一个模型的某个部分而意外影响其他部分的风险。

## 5. 过度使用 `init()` 函数

`init()` 函数在包被导入时自动执行，用于初始化包级别的状态。过度使用 `init()` 会使代码难以测试和理解，因为它的执行是隐式的。

**反模式 (Bad Case):**

```go
var db *sql.DB

func init() {
    // 隐式地初始化全局变量
    var err error
    db, err = sql.Open("mysql", "user:password@/dbname")
    if err != nil {
        panic(err)
    }
}
```

**推荐实践 (Good Case):**

```go
func setupDB() (*sql.DB, error) {
    db, err := sql.Open("mysql", "user:password@/dbname")
    if err != nil {
        return nil, err
    }
    return db, nil
}

func main() {
    db, err := setupDB()
    if err != nil {
        log.Fatalf("failed to setup database: %v", err)
    }
    // ... 使用 db
}
```

**解释:**

显式的初始化函数（如 `setupDB`）使依赖关系更清晰，也更容易进行单元测试（例如，通过传入一个 mock 的数据库连接）。

## 6. 返回指向切片或映射的指针

在 Go 中，切片和映射本身就是引用类型。返回它们的指针通常是不必要的，并且会增加代码的复杂性。

**反模式 (Bad Case):**

```go
func getUsers() *[]User {
    users := []User{...}
    return &users
}
```

**推荐实践 (Good Case):**

```go
func getUsers() []User {
    users := []User{...}
    return users
}
```

**解释:**

切片和映射在函数间传递时，不会复制底层的数据。返回一个切片或映射的副本通常开销很小，代码也更简洁。只有当你需要修改调用者函数中的切片头（即长度、容量或指向底层数组的指针）时，才需要使用指向切片的指针。

## 7. 在请求处理程序中传递 `context.Background()`

`context.Context` 对于在 API 调用中处理超时、取消和传递请求范围的值至关重要。`context.Background()` 是一个空的上下文，通常用作上下文链的根。在请求处理中，应该使用请求自带的上下文。

**反模式 (Bad Case):**

```go
func (h *Handler) HandleRequest(w http.ResponseWriter, r *http.Request) {
    // 忽略了请求的上下文
    err := h.service.DoSomething(context.Background(), "some-data")
    // ...
}
```

**推荐实践 (Good Case):**

```go
func (h *Handler) HandleRequest(w http.ResponseWriter, r *http.Request) {
    // 使用请求的上下文
    err := h.service.DoSomething(r.Context(), "some-data")
    // ...
}
```

**解释:**

`r.Context()` 包含了来自传入请求的上下文信息，例如超时或客户端断开连接的信号。通过传递它，下游的函数可以优雅地处理这些情况，避免不必要的工作。

## 8. 滥用 `context.WithValue`

`context.WithValue` 用于在上下文中传递请求范围的数据，但不应该用它来传递可选参数或依赖项。这会使函数签名不清晰，并引入隐式依赖。

**反模式 (Bad Case):**

```go
func processRequest(ctx context.Context) {
    // 从上下文中获取 logger，这是隐式依赖
    logger := ctx.Value("logger").(*log.Logger)
    logger.Println("processing request")
}
```

**推荐实践 (Good Case):**

```go
func processRequest(ctx context.Context, logger *log.Logger) {
    // logger 作为显式参数传递
    logger.Println("processing request")
}
```

**解释:**

将依赖项作为显式参数传递，可以使代码的依赖关系更清晰，也更容易测试。`context.WithValue` 只应用于传递那些真正属于请求范围的元数据，例如请求 ID 或用户身份信息。

## 9. 在循环中不正确地使用 `defer`

`defer` 语句会将其后的函数调用推迟到所在函数返回之前执行。如果在循环中使用 `defer`，这些调用会不断堆积，直到函数结束才执行，可能导致资源耗尽。

**反模式 (Bad Case):**

```go
func processFiles(filenames []string) error {
    for _, filename := range filenames {
        f, err := os.Open(filename)
        if err != nil {
            return err
        }
        // defer 会在函数退出时才执行，而不是循环迭代结束时
        defer f.Close() 
        // ... process file
    }
    return nil
}
```

**推荐实践 (Good Case):**

```go
func processFiles(filenames []string) error {
    for _, filename := range filenames {
        err := func() error {
            f, err := os.Open(filename)
            if err != nil {
                return err
            }
            defer f.Close()
            // ... process file
            return nil
        }()
        if err != nil {
            return err
        }
    }
    return nil
}
```

**解释:**

通过将循环体内的逻辑包装在一个匿名函数中，`defer` 会在该匿名函数返回时执行，从而确保资源在每次迭代后都能被及时释放。

## 10. 协程（Goroutine）泄漏

当一个协程在后台启动后，如果它没有明确的退出条件，就可能会永远运行下去，这就是协程泄漏。

**反模式 (Bad Case):**

```go
func process() {
    ch := make(chan int)
    go func() {
        // 这个协程可能会因为 ch 从未被接收而永远阻塞
        ch <- 1 
    }()
}
```

**推荐实践 (Good Case):**

```go
func process(ctx context.Context) {
    ch := make(chan int)
    go func() {
        select {
        case ch <- 1:
        case <-ctx.Done(): // 添加退出机制
            return
        }
    }()
    // ...
}
```

**解释:**

确保每个协程都有一个明确的退出路径。使用 `context`、`channel` 或 `sync.WaitGroup` 等机制来管理协程的生命周期，是避免泄漏的关键。

## 11. 过度使用 `// nolint`

`// nolint` 注释用于在特定代码行上禁用静态分析工具（linter）的检查。过度使用它会掩盖潜在的问题，降低代码质量。

**反模式 (Bad Case):**

```go
// nolint: some-check
x := someFunction() // 隐藏了一个 linter 警告
```

**推荐实践 (Good Case):**

```go
// 修复 linter 警告，而不是忽略它
x, err := someFunction()
if err != nil {
    // ...
}
```

**解释:**

Linter 的警告通常指向潜在的 bug、性能问题或不符合 Go 风格的代码。应该优先修复这些问题，而不是简单地忽略它们。只有在极少数情况下，当 linter 的警告确实是误报时，才应该使用 `// nolint`，并附上解释。

## 12. 不使用接口进行解耦

直接依赖具体的实现（struct）会增加代码的耦合度，使其难以测试和替换。

**反模式 (Bad Case):**

```go
type UserService struct {
    db *MySQLDatabase // 直接依赖具体的数据库实现
}

func (s *UserService) GetUser(id int) (*User, error) {
    return s.db.QueryUser(id)
}
```

**推荐实践 (Good Case):**

```go
// 定义一个接口
type UserStore interface {
    QueryUser(id int) (*User, error)
}

type UserService struct {
    store UserStore // 依赖接口
}

func (s *UserService) GetUser(id int) (*User, error) {
    return s.store.QueryUser(id)
}
```

**解释:**

通过依赖接口，`UserService` 不再关心底层的数据库是 MySQL、PostgreSQL 还是一个内存中的 mock。这使得单元测试变得非常容易，也提高了代码的灵活性和可维护性。

## 13. 在已知具体类型时滥用 `any` (`interface{}`)

`any` (或 `interface{}`) 是 Go 的空接口类型，可以表示任何类型的值。当你知道或可以约束类型时，滥用空接口会牺牲类型安全。

**反模式 (Bad Case):**

```go
func process(data any) {
    // 需要类型断言，容易出错
    s, ok := data.(string)
    if !ok {
        // handle error
    }
    // ...
}
```

**推荐实践 (Good Case):**

```go
// 使用泛型（Go 1.18+）
func process[T string | int](data T) {
    // ...
}

// 或者使用具体的类型
func processString(data string) {
    // ...
}
```

**解释:**

尽可能使用具体的类型或泛型来保证类型安全。这可以使编译器在编译时就发现类型错误，而不是在运行时才通过类型断言失败来发现。

## 14. 创建“工具”包（`utils`, `helpers`）

将没有明确归属的函数随意地扔进一个名为 `utils` 或 `helpers` 的包里，是一种常见的坏习惯。这会导致包的职责不明确，最终变成一个大杂烩。

**反模式 (Bad Case):**

```
/pkg
  /utils
    - string.go
    - time.go
    - http.go
```

**推荐实践 (Good Case):**

```
/pkg
  /strutil  // 专门处理字符串的包
    - reverse.go
  /httputil // 专门处理 HTTP 的包
    - client.go
```

**解释:**

包名应该描述它的功能。好的包名应该是名词，并且能够清晰地表达包的用途。例如，`strutil` (string utility) 比 `utils` 更具描述性。将相关的函数组织在有明确职责的包中，可以提高代码的可发现性和可维护性。

## 15. 过度使用 Getter 和 Setter

在 Go 中，直接访问 struct 的导出字段是惯例。无脑地为每个字段都创建 Getter 和 Setter 方法（像在 Java 或 C# 中那样）是不符合 Go 风格的。

**反模式 (Bad Case):**

```go
type User struct {
    name string
}

func (u *User) SetName(name string) {
    u.name = name
}

func (u *User) GetName() string {
    return u.name
}
```

**推荐实践 (Good Case):**

```go
type User struct {
    Name string // 直接导出字段
}

// 只有在需要额外逻辑时才使用方法
func (u *User) SetName(name string) {
    u.Name = strings.TrimSpace(name)
}
```

**解释:**

只有当获取或设置字段需要额外的逻辑（如验证、计算、同步等）时，才需要使用 Getter 和 Setter 方法。否则，直接访问导出的字段更简洁、更地道。

## 16. 未关闭资源

忘记关闭文件、网络连接或其他需要显式释放的资源，是导致资源泄漏的常见原因。

**反模式 (Bad Case):**

```go
func readFile(path string) ([]byte, error) {
    f, err := os.Open(path)
    if err != nil {
        return nil, err
    }
    // 忘记 defer f.Close()
    return io.ReadAll(f)
}
```

**推荐实践 (Good Case):**

```go
func readFile(path string) ([]byte, error) {
    f, err := os.Open(path)
    if err != nil {
        return nil, err
    }
    defer f.Close() // 确保文件被关闭
    return io.ReadAll(f)
}
```

**解释:**

使用 `defer` 是确保资源被释放的最可靠方法。它保证了即使函数在中间因为错误而返回，关闭操作也一定会被执行。

## 17. 在循环中不正确地使用 `time.After`

`time.After` 会创建一个通道，并在指定时间后发送一个值。如果在循环中直接调用它，每次迭代都会创建一个新的计时器和通道，但只有在超时发生时才会被垃圾回收，这可能导致资源泄漏。

**反模式 (Bad Case):**

```go
func doWork(ch <-chan int) {
    for {
        select {
        case v := <-ch:
            fmt.Println(v)
        case <-time.After(1 * time.Second): // 每次循环都创建新的 Timer
            fmt.Println("timeout")
            return
        }
    }
}
```

**推荐实践 (Good Case):**

```go
func doWork(ch <-chan int) {
    timer := time.NewTimer(1 * time.Second)
    defer timer.Stop()

    for {
        select {
        case v := <-ch:
            fmt.Println(v)
            // 重置计时器
            if !timer.Stop() {
                <-timer.C
            }
            timer.Reset(1 * time.Second)
        case <-timer.C:
            fmt.Println("timeout")
            return
        }
    }
}
```

**解释:**

在循环外部创建一个 `time.Timer`，然后在循环内部使用 `timer.Reset()` 来重用它。这避免了在每次迭代中都创建新的资源。

## 18. `for...range` 循环中的指针问题

在 `for...range` 循环中，迭代变量在每次迭代中都会被重用。如果直接取该变量的地址，你会发现所有指针都指向同一个内存地址，其值为最后一次迭代的值。

**反模式 (Bad Case):**

```go
values := []int{1, 2, 3}
var pointers []*int

for _, v := range values {
    // 所有指针都指向了最后一次迭代时 v 的地址
    pointers = append(pointers, &v) 
}

// 打印出来会是 [3, 3, 3] (的地址)
```

**推荐实践 (Good Case):**

```go
values := []int{1, 2, 3}
var pointers []*int

for _, v := range values {
    // 在循环内部创建一个新的变量
    v := v 
    pointers = append(pointers, &v)
}

// 打印出来会是 [1, 2, 3] (的地址)
```

**解释:**

通过在循环体内部重新声明一个同名变量 (`v := v`)，你为每次迭代都创建了一个新的、独立的变量，因此取它的地址是安全的。这是 Go 中一个常见且重要的技巧。

## 19. 将业务逻辑与实现细节混合

将核心业务逻辑与具体的技术实现（如 HTTP 处理、数据库查询）紧密耦合，会使代码难以测试、重用和演进。

**反模式 (Bad Case):**

```go
func HandleCreateUser(w http.ResponseWriter, r *http.Request) {
    // 解析 HTTP 请求
    var req CreateUserRequest
    json.NewDecoder(r.Body).Decode(&req)

    // 业务逻辑：验证用户
    if req.Password != req.ConfirmPassword {
        http.Error(w, "passwords do not match", http.StatusBadRequest)
        return
    }

    // 数据库操作
    db.Exec("INSERT INTO users ...")

    // 返回 HTTP 响应
    w.WriteHeader(http.StatusCreated)
}
```

**推荐实践 (Good Case):**

```go
// 业务逻辑层
type UserService struct { /* ... */ }

func (s *UserService) CreateUser(ctx context.Context, email, password string) (*User, error) {
    // 纯粹的业务逻辑
    if password == "" {
        return nil, errors.New("password cannot be empty")
    }
    // ...
    return s.store.CreateUser(ctx, email, password)
}

// HTTP 处理层
func HandleCreateUser(service *UserService) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        // 只负责 HTTP 相关的事情
        var req CreateUserRequest
        json.NewDecoder(r.Body).Decode(&req)

        user, err := service.CreateUser(r.Context(), req.Email, req.Password)
        // ... 处理错误和响应
    }
}
```

**解释:**

通过分层（例如，将代码分为 HTTP 层、业务逻辑层和数据存储层），可以使每一层的职责都更单一。业务逻辑层不关心数据来自 HTTP 还是 gRPC，也不关心数据存储在哪个数据库中。这种分离使得代码更清晰、更灵活。

## 20. 从数据库模式而不是领域开始

在设计软件时，如果首先考虑的是数据库表结构，而不是业务领域本身，很容易创建出所谓的“贫血领域模型”，即只有数据没有行为的对象。

**反模式 (Bad Case):**

- 先设计 `users`、`orders`、`order_items` 表。
- 然后创建对应的 `User`, `Order`, `OrderItem` struct。
- 业务逻辑散落在各种服务函数中，例如 `CreateOrder(userID, productIDs)`。

**推荐实践 (Good Case):**

- 先思考业务概念：一个“客户”（Customer）可以“下订单”（Place an Order）。一个“订单”（Order）包含多个“订单项”（LineItem），并且有自己的生命周期（待支付、已支付、已发货）。
- 然后设计对应的领域对象（struct）和方法：`customer.PlaceOrder(...)`，`order.AddLineItem(...)`，`order.MarkAsPaid()`。
- 最后再考虑如何将这些领域对象持久化到数据库中。

**解释:**

领域驱动设计（DDD）强调以业务领域为核心来构建软件。你的代码应该反映业务语言和流程。从领域开始设计，可以创建出更健壮、更能适应业务变化的模型。数据库只是实现持久化的一个细节。

## 结论

避免反模式和遵循最佳实践同样重要。通过识别并纠正这些常见的 Go 编程陷阱，你可以编写出更清晰、更健壮、更易于维护的代码。希望本文能帮助你成为一名更出色的 Go 开发者。
