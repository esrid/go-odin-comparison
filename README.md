# Golang Vs Odin: Syntax Comparison

A quick cheat-sheet comparing the fundamental syntax of the Go and Odin programming languages.

## Sommaire

* [Naming Convention](#naming-convention)
* [Package](#package)
* [Import](#import)
* [Basic Types](#basic-types)
* [Custom Type](#custom-type)
* [Enum](#enum)
* [Variable](#variable)
* [Function / Method](#function--method)
* [If Statement](#if-statement)
* [Loops](#loops)
* [Switch](#switch)
* [Struct](#struct)
* [Pointer](#pointer)
* [Array & Slice](#array--slice)
* [Map](#map)
* [Union](#union)
* [Error Handling](#error-handling)
* [Generics](#generics)
* [Closures & Anonymous Functions](#closures--anonymous-functions)
* [Defer](#defer)
* [Concurrency](#concurrency)
* [Memory Management & Allocators](#memory-management--allocators)
* [Type Casting & Conversion](#type-casting--conversion)
* [String Operations](#string-operations)
* [Bit Sets (Odin only)](#bit-sets-odin-only)
* [Context System (Odin only)](#context-system-odin-only)
* [Generics](#generics)
* [Closures & Anonymous Functions](#closures--anonymous-functions)
* [Defer](#defer)
* [Concurrency](#concurrency)
* [Memory Management & Allocators](#memory-management--allocators)
* [Type Casting & Conversion](#type-casting--conversion)
* [String Operations](#string-operations)
* [Context System (Odin only)](#context-system-odin-only)

---

## Naming Convention

| | Go | Odin |
|---|---|---|
| Local identifiers | `camelCase` | `snake_case` |
| Exported / Public | `PascalCase` | `Pascal_Case` or `PascalCase` |
| Constants | `PascalCase` or `ALL_CAPS` | `ALL_CAPS` or `SCREAMING_SNAKE` |
| Types / Structs | `PascalCase` | `Pascal_Case` |

In **Go**, visibility is determined by the first letter: uppercase = exported (public), lowercase = unexported (package-private).

In **Odin**, visibility is controlled explicitly with attributes:
```odin
@(private)         // restrict to package
@(private="file")  // restrict to file only
```

---

## Package

Declare the package at the top of every file. Same keyword in both languages.

```go
package main
```

```odin
package main
```

---

## Import

Both use the `import` keyword.

**Go:**
```go
import "fmt"
import "log"

// Multiple imports (preferred style)
import (
    "fmt"
    "log"
)
```

**Odin:**
```odin
// "core:" prefix is the standard library collection
import "core:fmt"
import "core:log"

// Custom alias
import foo "core:fmt"

// Odin does not support grouped import blocks import (...)
```

---

## Basic Types

Both languages have a zero value for every type (`0`, `false`, `""`, `nil`).

| Category | Go | Odin | Notes |
|---|---|---|---|
| **Boolean** | `bool` | `bool`, `b8`, `b16`, `b32`, `b64` | |
| **Signed Integer** | `int`, `int8`, `int16`, `int32`, `int64` | `int`, `i8`, `i16`, `i32`, `i64`, `i128` | `int` is pointer-sized in both |
| **Unsigned Integer** | `uint`, `uint8`, `uint16`, `uint32`, `uint64`, `uintptr` | `uint`, `u8`, `u16`, `u32`, `u64`, `u128`, `uintptr` | |
| **Byte / Rune** | `byte` (alias `uint8`), `rune` (alias `int32`) | `rune` (signed 32-bit) | |
| **Floating-Point** | `float32`, `float64` | `f16`, `f32`, `f64` | Go has no f16 |
| **Complex** | `complex64`, `complex128` | `complex32`, `complex64`, `complex128` | |
| **Quaternion** | none | `quaternion64`, `quaternion128`, `quaternion256` | Odin only |
| **String** | `string` (immutable byte sequence) | `string` (ptr + len), `cstring` (null-terminated) | |
| **Pointer / Raw** | `*T` | `^T`, `rawptr` | Different syntax |
| **Any** | `any` (alias `interface{}`) | `any`, `typeid` | |

---

## Custom Type

**Go:**
```go
package main

import "fmt"

// Distinct type (not just alias)
type Age int16

// Struct
type Person struct {
    name string
    age  Age
}

// Method on struct
func (p Person) GetAge() Age {
    return p.age
}

type Animal struct {
    name string
    age  Age
}

func (a Animal) GetAge() Age {
    return a.age
}

// Interface (implicit — no implements keyword)
type Ager interface {
    GetAge() Age
}

func printAge(a Ager) {
    fmt.Println("Age:", a.GetAge())
}

func main() {
    p := Person{name: "Alice", age: 30}
    dog := Animal{name: "Rex", age: 5}
    printAge(p)
    printAge(dog)
}
```

**Odin:**
```odin
package main

import "core:fmt"

// Type alias (same type, interchangeable)
My_Int :: int

// Distinct type (different type, not interchangeable)
Age :: distinct i16

// Struct type
Person :: struct {
    name: string,
    age:  Age,
}

// Odin has no methods — just procedures that take the type
get_age :: proc(p: Person) -> Age {
    return p.age
}

main :: proc() {
    p := Person{
        name = "odin",
        age  = 10,
    }
    fmt.println(p)
    fmt.println(get_age(p))
}
```

> **Key difference:** Go achieves polymorphism via interfaces (implicit). Odin has no methods or interfaces — use procedures and unions instead.

---

## Enum

**Go** — uses `iota` inside a `const` block:
```go
type OS int

const (
    MacOs   OS = iota // 0
    Windows            // 1
    Linux              // 2
)

// Bit flags
const (
    Read    = 1 << iota // 1
    Write               // 2
    Execute             // 4
)
```

**Odin** — first-class `enum` type:
```odin
OS :: enum {
    MacOs,
    Windows,
    Linux,
}

// Specify backing type (default is int)
OS :: enum i16 {
    MacOs,
    Windows,
    Linux,
}

// Explicit values
Foo :: enum {
    A,
    B = 4,
    C = 7,
}

// Enumerated arrays: index an array by enum value
Direction :: enum { North, East, South, West }

Direction_Vectors :: [Direction][2]int{
    .North = { 0, -1},
    .East  = {+1,  0},
}

// Iterate over all enum values
for dir, i in Direction {
    fmt.println(i, dir)
}
```

---

## Variable

**Go:**
```go
// Untyped constant
const BLUE = "#0000FF"

// Typed constant
const RED string = "#FF0000"

// Grouped constants
const (
    MaxSize = 1024
    Pi      = 3.14
)

// Variable declaration
var yellow string
yellow = "#FFFF00"

// Short declaration (inside function only, type inferred)
green := "#00FF00"

// Multiple assignment
x, y := 1, 2
```

**Odin:**
```odin
// Untyped constant (compile-time)
BLUE :: "#0000FF"

// Typed constant
RED : string : "#FF0000"

// Computed constant
Z :: RED + " suffix"  // evaluated at compile-time

// Variable declaration
yellow: string
yellow = "#FFFF00"

// Short declaration (inside procedure only, type inferred)
green := "#00FF00"

// Multiple assignment
x, y := 1, 2
```

---

## Function / Method

**Go:**
```go
// Basic
func someFunction() int { return 0 }

// Multiple return values
func someFunction() (int, float32) { return 0, 0.0 }

// With parameters
func someFunction(x int8, y int8) (int, float32) { return 0, 0.0 }

// Type grouping for same-type params
func someFunction(x, y int8) (int, float32) { return 0, 0.0 }

// Named return parameters
func someFunction(x, y int8) (a int, b float32) {
    a = int(x)
    b = float32(y)
    return // bare return
}

// Variadic
func sum(nums ...int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

// Method (receiver syntax)
func (s *SomeStruct) someMethod(x int) {}
```

**Odin** — functions are called **procedures**:
```odin
// Basic
some_procedure :: proc() -> i8 { return 0 }

// Multiple return values
some_procedure :: proc() -> (i8, f32) { return 0, 0 }

// With parameters
some_procedure :: proc(x: i8, y: i8) -> (i8, f32) { return 0, 0 }

// Type grouping
some_procedure :: proc(x, y: i8) -> (i8, f32) { return 0, 0 }

// Named return parameters + bare return
some_procedure :: proc(x, y: i8) -> (a: i8, b: f32) {
    a = x
    b = f32(y)
    return
}

// Default parameter values
create_window :: proc(
    title:   string,
    width  := 854,
    height := 480,
) {}

// Named arguments at call site
create_window(title = "Hello", width = 640, height = 360)

// Variadic
sum :: proc(nums: ..int) -> int {
    result: int
    for n in nums { result += n }
    return result
}
odds := []int{1, 3, 5}
sum(..odds) // spread a slice

// Procedure overloading
bool_to_string :: proc(b: bool) -> string { return "bool" }
int_to_string  :: proc(i: int)  -> string { return "int" }
to_string :: proc{bool_to_string, int_to_string}

// No methods — just a procedure that accepts the type
some_method :: proc(cs: CustomType) {}
```

---

## If Statement

Both languages have the same structure. Braces are mandatory. No parentheses around the condition.

**Go:**
```go
if x > 0 {
    fmt.Println("positive")
} else if x == 0 {
    fmt.Println("zero")
} else {
    fmt.Println("negative")
}

// Short init statement (x scoped to if block)
if x := compute(); x < 0 {
    fmt.Println("negative:", x)
}
```

**Odin:**
```odin
if x > 0 {
    fmt.println("positive")
} else if x == 0 {
    fmt.println("zero")
} else {
    fmt.println("negative")
}

// Same init statement pattern
if x := compute(); x < 0 {
    fmt.println("negative:", x)
}
```

---

## Loops

Both languages use `for` as the only loop keyword.

**Go:**
```go
// C-style
for i := 0; i < 10; i++ {
    fmt.Println(i)
}

// While-style
for sum < 1000 {
    sum += sum
}

// Infinite
for {
    break
}

// Range over slice/array (index first, value second)
arr := [3]int{1, 2, 3}
for i, v := range arr {
    fmt.Println(i, v)
    // v is a copy — modifying it does NOT affect arr
    arr[i] *= 2 // use index to mutate
}

// Range over map
for k, v := range m {
    fmt.Println(k, v)
}

// Range over string (yields runes)
for i, ch := range "hello" {
    fmt.Println(i, ch)
}

// Discard index or value
for _, v := range arr {}
for i := range arr   {}
```

**Odin:**
```odin
// C-style
for i := 0; i < 10; i += 1 {
    fmt.println(i)
}

// While-style
i := 0
for i < 10 {
    i += 1
}

// Infinite
for {
    break
}

// Range with half-open interval [0, 2)
for i in 0..<3 {
    fmt.println(i)
}

// Range with closed interval [0, 3]
for i in 0..=3 {
    fmt.println(i)
}

// Range over array/slice (value first, index second — opposite of Go)
arr := [3]int{1, 2, 3}
for v, i in arr {
    fmt.println(i, v)
}

// By-reference — modify elements in place
for &v in arr {
    v *= 2
}

// Range over map
for key, &value in some_map {
    value += 1
}

// Reverse iteration
#reverse for x in arr {
    fmt.println(x)
}

// Compile-time loop unrolling
#unroll for i in 0..<4 {
    x[i] ~= y[i]
}
```

> **Key difference:** In Go, `range` gives `(index, value)`. In Odin, `in` gives `(value, index)` — reversed order.

---

## Switch

**Go:**
```go
// Expression switch (no implicit fall-through)
computer := MacOs
switch computer {
case MacOs:
    fmt.Println("Mac")
case Windows:
    fmt.Println("Windows")
case Linux:
    fmt.Println("Linux")
default:
    fmt.Println("Unknown")
}

// Multiple values per case
switch c {
case 'a', 'e', 'i', 'o', 'u':
    fmt.Println("vowel")
}

// No-tag switch (acts like if-else chain)
switch {
case x < 0:
    fmt.Println("negative")
case x == 0:
    fmt.Println("zero")
default:
    fmt.Println("positive")
}

// Explicit fall-through
switch i {
case 0:
    foo()
    fallthrough
case 1:
    bar()
}

// Type switch
switch v := x.(type) {
case int:
    fmt.Println("int:", v)
case string:
    fmt.Println("string:", v)
default:
    fmt.Println("other")
}
```

**Odin:**
```odin
// Enum switch — must cover all cases (exhaustive)
computer: OS
switch computer {
case .MacOs:
    fmt.println("Mac")
case .Windows:
    fmt.println("Windows")
case .Linux:
    fmt.println("Linux")
case:
    fmt.println("Unknown") // default
}

// Partial switch — opt out of exhaustiveness check
#partial switch computer {
case .MacOs:
    fmt.println("Mac")
case .Linux:
    fmt.println("Linux")
}

// Range in cases
switch c {
case 'A'..='Z', 'a'..='z', '0'..='9':
    fmt.println("alphanumeric")
}

switch x {
case 0..<10:
    fmt.println("units")
case 10..<13:
    fmt.println("pre-teens")
case 13..<20:
    fmt.println("teens")
}

// No-tag switch
switch {
case x < 0:
    fmt.println("negative")
case x == 0:
    fmt.println("zero")
case:
    fmt.println("positive")
}

// Explicit fall-through
switch i {
case 0:
    foo()
    fallthrough
case 1:
    bar()
}

// Union type switch
c: Custom_Union = u8(10)
switch v in c {
case u8:
    // do something with v as u8
case f32:
    // do something with v as f32
}
```

> **Key difference:** Odin enum switches are **exhaustive by default** — all variants must be handled. Use `#partial` to relax this. Go switches are never exhaustive.

---

## Struct

**Go:**
```go
type Vector2 struct {
    X, Y float64
}

// Instantiation
v := Vector2{1.0, 2.0}    // positional
v := Vector2{X: 1.0, Y: 2.0} // named fields

// Pointer to struct (auto-dereference for field access)
p := &v
p.X = 99.0 // no need for (*p).X

// Embedded struct (promotes fields and methods)
type Entity struct {
    Vector2           // embedded — fields X, Y promoted
    Name   string
}

e := Entity{Vector2: Vector2{1, 2}, Name: "hero"}
fmt.Println(e.X) // promoted field

// Field tags (used by encoding/json etc.)
type User struct {
    Name string `json:"name"`
    Age  int    `json:"age,omitempty"`
}
```

**Odin:**
```odin
Vector2 :: struct {
    x: f32,
    y: f32,
}

// Instantiation
v := Vector2{1, 2}                   // positional
v := Vector2{x = 1, y = 2}          // named fields
v := Vector2{}                        // zero value

// Pointer (^ is pointer, auto-dereference for fields)
p := &v
p.x = 99 // equivalent to p^.x

// Using — embeds fields of another struct into scope
Entity :: struct {
    using position: Vector2,
    name: string,
}

e: Entity
e.x = 1 // x comes from embedded Vector2

// Field tags
User :: struct {
    name: string `json:"username"`,
    age:  int    "custom info",
}

// Struct directives
struct #align(4) { ... }   // force alignment
struct #packed { ... }     // remove padding
struct #raw_union { ... }  // C-style union (shared memory)
```

---

## Pointer

**Go:**
```go
var p *int    // pointer to int, zero value is nil

i := 42
p = &i        // address-of
fmt.Println(*p) // dereference: prints 42
*p = 100      // modify through pointer
```

**Odin:**
```odin
p: ^int       // pointer to int (^ instead of *)

i := 42
p = &i        // address-of
fmt.println(p^) // dereference: ^ suffix
p^ = 100      // modify through pointer
```

| | Go | Odin |
|---|---|---|
| Pointer type | `*T` | `^T` |
| Address of | `&x` | `&x` |
| Dereference | `*p` | `p^` |
| Raw pointer | n/a | `rawptr` |
| Arithmetic | No | No (use `ptr_offset` from `core:mem`) |

---

## Array & Slice

**Go:**
```go
// Fixed array (length is part of the type)
var a [5]int
a := [3]int{1, 2, 3}

// Slice (dynamic view over array)
s := a[1:3]          // elements at index 1 and 2
s := a[:]            // all elements
s := make([]int, 5)  // len=5
s := make([]int, 0, 10) // len=0, cap=10

// Append
s = append(s, 4, 5, 6)

// Spreading a slice
other := []int{7, 8}
s = append(s, other...)

// Length and capacity
len(s)
cap(s)
```

**Odin:**
```odin
// Fixed array
x := [5]int{1, 2, 3, 4, 5}
x := [?]int{1, 2, 3}        // length inferred from literal

// Designated initializer
animals := [?]string{
    0      = "Raven",
    3..=5  = "Frog",
    6..<8  = "Cat",
}

// Slice (view into array)
a: [6]int
s := a[1:4]    // half-open [1, 4)
s := a[:]      // full array

// Dynamic array (heap-allocated, resizable)
x: [dynamic]int
append(&x, 1, 2, 3)
append(&x, ..other_slice[:]) // spread
pop(&x)                      // remove last
ordered_remove(&x, 0)        // O(n), preserves order
unordered_remove(&x, 0)      // O(1), swaps with last
delete(x)                    // free memory
clear(&x)                    // len=0, keep capacity

x = make([dynamic]int, 0, 10) // len=0, cap=10

// Fixed-capacity dynamic array (stack-allocated)
x: [dynamic; 8]int // max capacity 8
```

> **Key difference:** Go uses `append` (returns new slice). Odin uses `append(&x, ...)` (passes pointer). Odin also has a `[dynamic]` type distinct from slices.

---

## Map

**Go:**
```go
// Create
m := make(map[string]int)
m := map[string]int{"a": 1, "b": 2} // literal

// Operations
m["key"] = 42
value := m["key"]
delete(m, "key")
clear(m)        // Go 1.21+

// Check existence
value, ok := m["key"] // ok is false if key absent

len(m)
```

**Odin:**
```odin
// Create
m := make(map[string]int)
defer delete(m) // common pattern — free at end of scope

// Literal (requires #+feature dynamic-literals at top of file)
m := map[string]int{
    "Bob"   = 2,
    "Chloe" = 5,
}

// Operations
m["key"] = 42
value := m["key"]
delete_key(&m, "key")
clear(&m)

// Check existence
elem, ok := m["key"]
ok = "key" in m     // membership test

len(m)
cap(m)
reserve(&m, 64)
```

---

## Union

**Go** — no tagged unions. Uses interfaces and type assertions:
```go
// An interface acts as an open union
type Shape interface{}

type Circle struct{ Radius float64 }
type Square struct{ Width float64 }

var s Shape = Circle{Radius: 5}

// Type assertion
c, ok := s.(Circle)

// Type switch
switch v := s.(type) {
case Circle:
    fmt.Println("circle radius:", v.Radius)
case Square:
    fmt.Println("square width:", v.Width)
default:
    fmt.Println("unknown")
}

// Go 1.18+ union constraint (type parameters only, not runtime)
type Float interface {
    ~float32 | ~float64
}
```

**Odin** — first-class tagged union:
```odin
// Closed set of types, zero value is nil
Shape :: union {
    Circle,
    Square,
}

Circle :: struct { radius: f32 }
Square :: struct { width: f32 }

shape: Shape // nil by default

shape = Circle{radius: 5}

// Type assert (panics if wrong type)
c := shape.(Circle)

// Safe type assert
c, ok := shape.(Circle)

// Type switch (exhaustive)
switch v in shape {
case Circle:
    fmt.println("circle:", v.radius)
case Square:
    fmt.println("square:", v.width)
case:
    fmt.println("nil")
}

// Prevent nil with #no_nil
Shape :: union #no_nil {
    Circle,
    Square,
}

// Shared nil semantics for error unions
Result :: union #shared_nil { Error_A, Error_B }
```

---

## Error Handling

**Go** — errors are values implementing the `error` interface:
```go
// The error interface
type error interface {
    Error() string
}

// Functions return error as the last value
f, err := os.Open("file.txt")
if err != nil {
    log.Fatal(err)
}

// Custom error
type MyError struct{ Msg string }
func (e *MyError) Error() string { return e.Msg }

// Panic and recover (for unrecoverable errors)
func safe() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("recovered:", r)
        }
    }()
    panic("something went wrong")
}
```

**Odin** — errors are ordinary enum/union values with special operators:
```odin
Error :: enum { None, File_Not_Found, Permission_Denied }

open_file :: proc(path: string) -> (Handle, Error) {
    // ...
    return {}, .File_Not_Found
}

// or_return — if last return value is non-zero, return it to caller
foo :: proc() -> Error {
    handle := open_file("x.txt") or_return
    return .None
}

// or_else — provide a default value
i := m["key"] or_else 0
val := v.(int) or_else -1

// or_continue — skip to next iteration on error
for &job in jobs {
    result := process(&job) or_continue
    fmt.println(result)
}

// or_break — break out of loop on error
for &job in jobs {
    result := process(&job) or_break
    fmt.println(result)
}
```

> **Key difference:** Go error handling is explicit `if err != nil` checks. Odin provides `or_return`/`or_else`/`or_continue` operators to propagate errors with less boilerplate, similar to Rust's `?` operator.

---

## Generics

**Go** — type parameters introduced in Go 1.18, written in square brackets:
```go
// Type parameter T constrained to any type
// Type parameter U constrained to any type
func Map[T, U any](s []T, f func(T) U) []U {
    result := make([]U, len(s))
    for i, v := range s {
        result[i] = f(v)
    }
    return result
}

// Use the ~ (tilde) for underlying-type constraints
// ~int means "int or any named type whose underlying type is int"
type Ordered interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
    ~float32 | ~float64 | ~string
}

func Min[T Ordered](a, b T) T {
    if a < b {
        return a
    }
    return b
}

// comparable: types that support == and != (maps, channels, structs of comparable fields)
func InMap[K comparable, V any](m map[K]V, key K) bool {
    _, ok := m[key]
    return ok
}

// Generic type with methods
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}

func (s *Stack[T]) Pop() (T, bool) {
    if len(s.items) == 0 {
        var zero T
        return zero, false
    }
    n := len(s.items) - 1
    item := s.items[n]
    s.items = s.items[:n]
    return item, true
}

// Standard library example: slices.Contains uses generics
// func Contains[S ~[]E, E comparable](s S, v E) bool
// ~[]E means "a slice of E or any named type whose underlying type is []E"

// Instantiation — type can often be inferred
result := Map([]int{1, 2, 3}, func(n int) string {
    return strconv.Itoa(n)
}) // result: []string{"1","2","3"}

s := Stack[int]{}
s.Push(10)
v, ok := s.Pop() // v=10, ok=true
```

**Odin** — parametric polymorphism ("parapoly") with `$` prefix. The compiler generates specialised versions at compile time — no runtime overhead, similar to C++ templates:
```odin
import "base:intrinsics"

// $T binds to the concrete type of the first argument
// The dollar sign ($) appears only once — that's where the type is inferred
clamp :: proc(val: $T, min: T, max: T) -> T {
    if val <= min { return min }
    if val >= max { return max }
    return val
}

// Restrict T to numeric types via a where clause
clamp_numeric :: proc(val: $T, min: T, max: T) -> T
    where intrinsics.type_is_numeric(T) {
    if val <= min { return min }
    if val >= max { return max }
    return val
}

// Explicitly pass a type as an argument ($T: typeid)
// Caller writes: make_slice(f32) — passes the type itself
make_slice :: proc($T: typeid, n: int) -> []T {
    return make([]T, n)
}

// Compile-time integer constant ($N: int)
// where clause used to restrict the constant value
require_more_than_two :: proc(x: [$N]int) -> bool
    where N > 2 {
    return true
}

// Polymorphic struct — generates a new struct type per unique (T, N) pair
Special_Array :: struct($T: typeid, $N: int) {
    items:          [N]T,
    num_items_used: int,
}

// Procedure accepting any Special_Array, capturing its T and N
find_first :: proc(arr: Special_Array($T, $N), target: T) -> int {
    for v, i in arr.items[:arr.num_items_used] {
        if v == target { return i }
    }
    return -1
}

// Polymorphic union — Odin's built-in Maybe uses this pattern
Maybe :: union($T: typeid) { T }

// Usage
n := clamp(7, 2, 5)                  // T inferred as int → generates clamp(int,int,int)->int
f := clamp(1.5, 0.0, 1.0)            // T inferred as f64

ints := make_slice(int, 10)           // explicit type parameter
defer delete(ints)

arr: Special_Array(f64, 128)          // concrete type with T=f64, N=128

m: Maybe(f64)
m = 4.32
if val, ok := m.?; ok {
    fmt.println(val)  // 4.32
}
```

> **Key differences:**
> - Go uses `[T Constraint]` syntax with interfaces as constraints; Odin uses `$T` prefix with `where` clauses for type-level assertions.
> - Go generics involve some runtime decisions (interface dispatch, shape-based code sharing). Odin's parapoly is fully resolved at compile time — one generated specialisation per unique argument combination, similar to C++ templates.
> - Go's `~T` means "any type whose underlying type is T"; Odin's `$T/BaseType` specialises `T` to be constrained to a specific form (e.g. `$T/[dynamic]$E` means T must be a dynamic array of element type E).
> - Go requires explicit type parameter lists (`func Foo[T any](...)`); in Odin the `$` in the parameter list is sufficient — no separate bracket list.
> - Go methods on generic types require the type parameter from the type definition; Odin has no methods at all — all procedures are standalone.
> - Odin can pass a **type itself** as a compile-time argument (`$T: typeid`) — Go cannot; it can only infer types from values.

---

## Closures & Anonymous Functions

**Go** — closures are function literals that capture variables by reference:
```go
// Anonymous function assigned to a variable
double := func(x int) int {
    return x * 2
}
fmt.Println(double(5)) // 10

// Closure: the returned function captures `sum` by reference
func makeAccumulator() func(int) int {
    sum := 0
    return func(x int) int {
        sum += x // sum is captured — persists across calls
        return sum
    }
}

acc := makeAccumulator()
fmt.Println(acc(1)) // 1
fmt.Println(acc(2)) // 3
fmt.Println(acc(3)) // 6

// Immediately invoked function expression (IIFE)
result := func(a, b int) int { return a + b }(3, 4)
fmt.Println(result) // 7

// Passing functions as values
func apply(nums []int, f func(int) int) []int {
    out := make([]int, len(nums))
    for i, n := range nums {
        out[i] = f(n)
    }
    return out
}

doubled := apply([]int{1, 2, 3}, func(n int) int { return n * 2 })
// doubled: [2 4 6]

// Common pitfall: loop variable capture (pre-Go 1.22)
// In Go 1.22+ each loop iteration has its own variable
funcs := make([]func(), 3)
for i := range funcs {
    i := i // re-declare to capture per-iteration copy (pre-1.22 idiom)
    funcs[i] = func() { fmt.Println(i) }
}
```

**Odin** — anonymous procedures are procedure literals. **Odin does not support automatic variable capture** from enclosing scopes. Nested procedures can only access global variables, global constants, and their own parameters — never a parent procedure's locals. This is a deliberate design choice (no GC, no hidden complexity). To share mutable state, pass a pointer explicitly:
```odin
// Procedure type declaration (equivalent to a function type)
Callback :: proc() -> int

// Anonymous procedure assigned to a variable
a: Callback
a = proc() -> int { return 0 }
fmt.println(a()) // 0

// Procedure as a parameter (first-class value)
apply :: proc(nums: []int, f: proc(int) -> int) -> []int {
    out := make([]int, len(nums))
    for n, i in nums {
        out[i] = f(n)
    }
    return out
}

doubled := apply([]int{1, 2, 3}, proc(n: int) -> int { return n * 2 })

// IMPORTANT: Odin anonymous procedures do NOT automatically close over
// variables in the enclosing scope. To share state, pass a pointer explicitly.
make_accumulator :: proc() -> proc(^int, int) {
    return proc(sum: ^int, x: int) {
        sum^ += x
    }
}

sum: int
acc := make_accumulator()
acc(&sum, 1)
acc(&sum, 2)
acc(&sum, 3)
fmt.println(sum) // 6

// Immediate invocation
result := (proc(a, b: int) -> int { return a + b })(3, 4)
fmt.println(result) // 7

// Using a procedure variable for callbacks/dispatch tables
ops := [?]proc(int, int) -> int {
    proc(a, b: int) -> int { return a + b },
    proc(a, b: int) -> int { return a * b },
}
fmt.println(ops[0](3, 4)) // 7
fmt.println(ops[1](3, 4)) // 12
```

> **Key differences:**
> - Go closures automatically capture variables from the enclosing scope by reference — mutation is visible to the outer scope.
> - Odin anonymous procedures do NOT close over outer variables automatically. To share mutable state, explicitly pass a pointer as a parameter.
> - Both treat procedures/functions as first-class values that can be stored in variables and passed as arguments.
> - Go function types use `func(params) returns` syntax; Odin uses `proc(params) -> returns`.

---

## Defer

**Go** — `defer` schedules a function call to run when the surrounding function returns (LIFO order, arguments evaluated immediately):
```go
// Arguments are evaluated at the defer statement, not at execution
func example() {
    i := 0
    defer fmt.Println("deferred i =", i) // captures i=0 right now
    i = 10
    fmt.Println("current i =", i) // prints 10
}
// Output:
//   current i = 10
//   deferred i = 0

// LIFO order: last defer runs first
func cleanup() {
    defer fmt.Println("first")  // runs third
    defer fmt.Println("second") // runs second
    defer fmt.Println("third")  // runs first
}
// Output: third, second, first

// Classic resource cleanup pattern
func copyFile(src, dst string) error {
    in, err := os.Open(src)
    if err != nil {
        return err
    }
    defer in.Close() // guaranteed to close even if error occurs below

    out, err := os.Create(dst)
    if err != nil {
        return err
    }
    defer out.Close()

    _, err = io.Copy(out, in)
    return err
}

// defer + anonymous function to capture current variable by reference
func mutable() {
    x := 0
    defer func() { fmt.Println("x at return:", x) }() // captures &x
    x = 42
}
// Output: x at return: 42

// defer with panic/recover
func safeDiv(a, b int) (result int, err error) {
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("recovered: %v", r)
        }
    }()
    return a / b, nil // panics if b==0, caught by defer above
}
```

**Odin** — same LIFO semantics, plus `defer if` and block defer; cannot modify named return values:
```odin
// Basic defer — runs at end of surrounding scope (not just function)
{
    defer fmt.println("1") // runs last in this block
    defer fmt.println("2") // runs second
    defer fmt.println("3") // runs first
}
// Output: 3, 2, 1

// Scope-level: defer runs at end of any scope, not just the procedure
proc_example :: proc() {
    {
        f, err := os.open("file.txt")
        if err != os.ERROR_NONE {
            return
        }
        defer os.close(f)     // closes when this inner block exits
        // ... work with f ...
    }
    // f is already closed here
}

// defer block — multiple statements in one defer
defer {
    fmt.println("step 1 of cleanup")
    fmt.println("step 2 of cleanup")
}

// defer if — conditional defer (evaluated at declaration time)
verbose := true
defer if verbose {
    fmt.println("verbose cleanup")
}

// Common map/dynamic-array cleanup idiom
m := make(map[string]int)
defer delete(m)

arr: [dynamic]int
defer delete(arr)

// NOTE: defer cannot modify named return values (they are already returned)
// This is different from Go where defer+closure can modify named returns
foo :: proc() -> (result: int) {
    defer result = 42  // this does NOT work as expected in Odin
    return 0
}
```

> **Key differences:**
> - Both use LIFO order and both evaluate the deferred expression when `defer` is encountered.
> - Odin `defer` runs at the end of **any scope** (block, if, for, etc.), not just at function return. Go `defer` only runs at function return.
> - Odin adds `defer if condition { ... }` for conditional cleanup and `defer { ... }` for multi-statement blocks.
> - In Go, a `defer func() { named_return = x }()` closure can modify named return values. Odin explicitly forbids this.

---

## Concurrency

### Go — Goroutines & Channels

Go concurrency is built into the language with the `go` keyword, channel types, and the `select` statement:
```go
// Launch a goroutine — lightweight thread managed by the Go runtime
go someFunction()

// Anonymous goroutine
go func(msg string) {
    fmt.Println(msg)
}("hello")

// Unbuffered channel: send and receive block until both sides are ready
ch := make(chan int)

go func() {
    ch <- 42 // send — blocks until receiver is ready
}()

v := <-ch // receive — blocks until sender is ready
fmt.Println(v) // 42

// Buffered channel: send does not block until buffer is full
buf := make(chan string, 3)
buf <- "a"
buf <- "b"
buf <- "c"
// buf <- "d" would block — buffer is full

// Directional channels
func producer(out chan<- int) { // send-only
    out <- 100
}
func consumer(in <-chan int) { // receive-only
    fmt.Println(<-in)
}

// Range over channel — receives until channel is closed
jobs := make(chan int, 5)
for j := 1; j <= 5; j++ {
    jobs <- j
}
close(jobs)

for j := range jobs {
    fmt.Println("job", j)
}

// Check if channel is closed
v, ok := <-ch
if !ok {
    fmt.Println("channel closed")
}

// select — multiplex on multiple channels
timeout := time.After(1 * time.Second)
select {
case msg := <-ch1:
    fmt.Println("from ch1:", msg)
case ch2 <- data:
    fmt.Println("sent to ch2")
case <-timeout:
    fmt.Println("timed out")
default:
    fmt.Println("no channel ready — non-blocking")
}

// WaitGroup: wait for multiple goroutines to finish
var wg sync.WaitGroup
for i := 0; i < 5; i++ {
    wg.Add(1)
    go func(n int) {
        defer wg.Done()
        fmt.Println("goroutine", n)
    }(i)
}
wg.Wait()
```

### Odin — core:thread

Odin has no built-in concurrency primitives in the language itself. Threads are managed via the `core:thread` package:
```odin
import "core:thread"
import "core:fmt"
import "core:time"

// Thread procedure signature: proc(t: ^thread.Thread)
worker :: proc(t: ^thread.Thread) {
    for iteration in 1..=5 {
        fmt.printf("Thread %d — iteration %d\n", t.user_index, iteration)
        time.sleep(1 * time.Millisecond)
    }
}

basic_threads :: proc() {
    threads := make([dynamic]^thread.Thread, 0, 4)
    defer delete(threads)

    for i in 0..<4 {
        t := thread.create(worker) // allocate + associate procedure
        if t != nil {
            t.init_context = context // pass current context to the thread
            t.user_index = i        // custom data slot on Thread struct
            append(&threads, t)
            thread.start(t)         // begin execution
        }
    }

    // Poll until all threads are done
    for len(threads) > 0 {
        for i := 0; i < len(threads); {
            if thread.is_done(threads[i]) {
                thread.destroy(threads[i]) // free resources
                ordered_remove(&threads, i)
            } else {
                i += 1
            }
        }
    }
}

// Thread Pool — distribute tasks across N worker threads
pool_example :: proc() {
    task_proc :: proc(t: thread.Task) {
        fmt.printf("Task %d executing\n", t.user_index)
    }

    pool: thread.Pool
    thread.pool_init(&pool, allocator = context.allocator, thread_count = 4)
    defer thread.pool_destroy(&pool)

    for i in 0..<16 {
        thread.pool_add_task(
            &pool,
            allocator  = context.allocator,
            procedure  = task_proc,
            data       = nil,
            user_index = i,
        )
    }

    thread.pool_start(&pool)
    thread.pool_finish(&pool) // block until all tasks complete
}
```

> **Key differences:**
> - Go goroutines are M:N green threads scheduled by the Go runtime — extremely cheap to create (a few KB of stack).
> - Odin threads map 1:1 to OS threads — heavier but with predictable behavior.
> - Go channels are a language-level primitive for communication. Odin has no built-in channel type; synchronization uses `core:sync` (mutexes, semaphores, atomics) or message queues built on top.
> - Go's `select` statement has no Odin equivalent — implement with condition variables or channels from `core:sync`.
> - Odin passes `init_context` to threads so they inherit the parent's allocator and logger.

---

## Memory Management & Allocators

**Go** — garbage-collected; `new` and `make` allocate on the heap:
```go
// new(T): allocates zeroed memory for T, returns *T
p := new(int)     // p is *int, *p == 0
*p = 42

// make: initializes slice/map/channel (returns the value, not a pointer)
s := make([]int, 5)        // len=5, cap=5
s := make([]int, 0, 10)    // len=0, cap=10
m := make(map[string]int)
ch := make(chan int, 8)     // buffered channel with capacity 8

// The GC handles all deallocation — no free(), no delete()
// GOGC=100 (default): collect when heap doubles since last collection
// GOMEMLIMIT=512MiB: soft total memory limit

// Tune GC at runtime
import "runtime/debug"
debug.SetGCPercent(50)          // more aggressive GC
debug.SetMemoryLimit(512 << 20) // 512 MiB soft limit

// Force a GC cycle (rarely needed)
runtime.GC()

// Escape analysis: compiler decides stack vs heap
// If a value's address escapes the function, it goes to the heap
func stackAlloc() int {
    x := 42       // likely stays on the stack
    return x
}
func heapAlloc() *int {
    x := 42
    return &x     // x escapes to heap because its address is returned
}
```

**Odin** — no GC; all allocation is explicit through a pluggable allocator system:
```odin
import "core:mem"
import "core:fmt"

// --- Basic allocation ---
// new(T): allocates one T using context.allocator, returns ^T
p := new(int)           // p is ^int, p^ == 0
defer free(p)           // must free manually

p^ = 42

// make: allocates slices, dynamic arrays, maps
s := make([]int, 5)              // slice, len=5
s := make([]int, 0, 10)          // slice, len=0 cap=10
d := make([dynamic]int, 0, 10)   // dynamic array
m := make(map[string]int)

defer delete(s)
defer delete(d)
defer delete(m)

// --- context.allocator ---
// Every allocation goes through context.allocator.
// Replace it per-scope to control where memory comes from.
my_allocator_proc :: proc(
    mode: mem.Allocator_Mode,
    size, alignment: int,
    old_memory: rawptr,
    old_size: int,
    location: Source_Code_Location,
) -> ([]byte, mem.Allocator_Error) { ... }

context.allocator = mem.Allocator{procedure = my_allocator_proc, data = nil}

// --- Temporary allocator ---
// context.temp_allocator: a scratch allocator, reset at end of frame.
// Allocations survive the current scope but are freed when you call
// free_all(context.temp_allocator).
str := fmt.tprintf("hello %d", 42) // allocated on temp_allocator by default in fmt.tprintf
// ... use str ...
free_all(context.temp_allocator)   // bulk-free all temp allocations

// --- Arena allocator ---
// Allocate a big block once; sub-allocate from it; free the whole block at once.
arena: mem.Arena
mem.arena_init(&arena, make([]byte, mem.Megabyte))
defer mem.arena_destroy(&arena)

arena_alloc := mem.arena_allocator(&arena)
context.allocator = arena_alloc

data := new(int)   // allocated from arena
data^ = 99

mem.free_all(arena_alloc) // reset arena — O(1), all sub-allocations freed

// --- Tracking allocator (debug/development) ---
// Wraps another allocator to detect leaks and invalid frees.
track: mem.Tracking_Allocator
mem.tracking_allocator_init(&track, context.allocator)
defer mem.tracking_allocator_destroy(&track)

context.allocator = mem.tracking_allocator(&track)

// ... run code that allocates ...

// Report leaks at end
for _, entry in track.allocation_map {
    fmt.printf("LEAK: %v bytes at %v\n", entry.size, entry.location)
}
for entry in track.bad_free_array {
    fmt.printf("BAD FREE: %v\n", entry.location)
}
```

> **Key differences:**
> - Go allocates on the heap automatically (escape analysis) and the GC reclaims memory. You never call `free`.
> - Odin has no GC — every `new`/`make` must be paired with a `free`/`delete`. The tracking allocator helps catch mistakes during development.
> - Odin's allocator is a first-class value (`mem.Allocator`) passed via the implicit `context`. You can swap allocators per-scope without changing call sites.
> - The temp allocator (`context.temp_allocator`) is an efficient scratch arena — allocate freely, bulk-free with `free_all`. Ideal for per-frame or per-request scratch work.
> - Arena allocators give O(1) bulk deallocation with no per-object `free` calls — useful for loading game levels, HTTP requests, etc.

---

## Type Casting & Conversion

**Go** — explicit conversion with `T(x)` syntax; type assertions for interfaces:
```go
// Numeric conversion: always explicit
var i int = 42
f := float64(i)   // int -> float64
u := uint32(f)    // float64 -> uint32 (truncates)

// String conversion
s := string(65)          // rune to string: "A"
s := string([]byte{72, 105}) // []byte to string: "Hi"
b := []byte("Hello")     // string to []byte

// Type assertion: extract concrete type from interface
var any interface{} = "hello"

// Asserting form (panics if wrong type)
s := any.(string)   // s == "hello"
// i := any.(int)   // panic: interface conversion

// Safe comma-ok form (never panics)
s, ok := any.(string) // s="hello", ok=true
i, ok := any.(int)    // i=0,       ok=false

// Type switch (dispatches on dynamic type)
switch v := any.(type) {
case int:
    fmt.Println("int:", v)
case string:
    fmt.Println("string:", v)
case bool:
    fmt.Println("bool:", v)
default:
    fmt.Printf("unknown: %T\n", v)
}

// No implicit conversions — this does NOT compile:
// var x int = 1.0   // error: cannot use 1.0 (untyped float) as int
// var y float64 = x  // error: cannot use x (int) as float64
```

**Odin** — three distinct cast operators with different semantics:
```odin
// T(x) or cast(T)x — value-aware type conversion
i := 123
f := f64(i)         // int -> f64  (standard conversion syntax)
f := cast(f64)i     // identical — cast() is an explicit alternative form
u := cast(u32)f     // f64 -> u32 (truncates toward zero)

// transmute(T)x — reinterpret raw bits (types must have same size)
// Equivalent to *(^T)(&x) — a pointer reinterpret, no value conversion
f32_val := f32(123)
u32_bits := transmute(u32)f32_val   // same 4 bytes, interpreted as u32

// Practical transmute: reinterpret quaternion as [4]f64
q := 1 + 2i + 3j + 4k
arr := transmute([4]f64)q

// auto_cast — compiler infers the target type from context
// NOTE: only for prototyping — avoid in production code
x: f32 = 123
y: int = auto_cast x  // compiler figures out the conversion

// Safe union type assertion (see Union section)
shape: Shape  // Shape :: union { Circle, Square }
shape = Circle{radius: 5}

c := shape.(Circle)        // panics if shape is not Circle
c, ok := shape.(Circle)    // safe: ok=false if wrong type

// any type assertion
val: any = 42
n, ok := val.(int)         // n=42, ok=true

// No implicit conversions — Odin is strict
a: int = 5
b: f64 = f64(a)  // must be explicit
```

> **Key differences:**
> - Both languages require explicit conversions — no implicit numeric promotions.
> - Go uses a single `T(x)` form for all safe conversions. Odin separates `cast(T)x` (value conversion) from `transmute(T)x` (bit reinterpretation), making unsafe operations more visible.
> - `transmute` has no Go equivalent; the closest is `unsafe.Pointer` chains or `*(*T)(unsafe.Pointer(&x))`.
> - Go type assertions (`x.(T)`) work on interface values. Odin union assertions (`x.(T)`) work on tagged union values — semantically similar but applied to a different type system feature.
> - `auto_cast` has no Go equivalent — Go never infers conversion targets.

---

## String Operations

**Go** — immutable `string`, `strings` package, `strings.Builder` for efficient concatenation:
```go
import (
    "fmt"
    "strings"
    "unicode/utf8"
)

s := "Hello, 世界"

// Length (in bytes, not runes)
fmt.Println(len(s))                    // 13

// Rune count
fmt.Println(utf8.RuneCountInString(s)) // 9

// Iterate by rune (Unicode codepoint)
for i, r := range s {
    fmt.Printf("index=%d rune=%c\n", i, r)
}

// Byte slice access
b := []byte(s)

// strings package — most common operations
fmt.Println(strings.Contains(s, "世界"))       // true
fmt.Println(strings.HasPrefix(s, "Hello"))    // true
fmt.Println(strings.HasSuffix(s, "界"))        // true
fmt.Println(strings.Index(s, "世"))            // 7

fmt.Println(strings.ToUpper("hello"))          // HELLO
fmt.Println(strings.ToLower("WORLD"))          // world
fmt.Println(strings.TrimSpace("  hi  "))       // "hi"
fmt.Println(strings.Replace("aabbcc", "b", "x", 1)) // "aaxbcc"
fmt.Println(strings.ReplaceAll("aabbcc", "b", "x"))  // "aaxxcc"

parts := strings.Split("a,b,c", ",")           // ["a","b","c"]
joined := strings.Join(parts, " | ")           // "a | b | c"

// Efficient string building (avoids repeated allocation from +)
var sb strings.Builder
sb.Grow(64)              // pre-allocate capacity
for i := range 3 {
    fmt.Fprintf(&sb, "item%d ", i)
}
result := sb.String()   // "item0 item1 item2 "

// Formatted string (allocates)
msg := fmt.Sprintf("x=%d, y=%.2f", 10, 3.14)

// Check if empty
if s == "" { ... }

// Substring (slicing)
sub := s[0:5] // "Hello" — shares underlying memory
```

**Odin** — `string` is `(ptr, len)`, `cstring` is null-terminated; `core:strings` package; `fmt.tprintf` for temp-allocated formatted strings:
```odin
import "core:fmt"
import "core:strings"
import "core:unicode/utf8"

s := "Hello, 世界"

// Length in bytes
fmt.println(len(s)) // 13

// Iterate by rune (preferred — Odin strings are UTF-8)
for r, i in s {
    fmt.printf("index=%d rune=%r\n", i, r)
}

// Iterate by byte
for b, i in transmute([]u8)s {
    fmt.printf("byte[%d]=%d\n", i, b)
}

// core:strings operations
fmt.println(strings.contains(s, "世界"))        // true
fmt.println(strings.has_prefix(s, "Hello"))    // true
fmt.println(strings.has_suffix(s, "界"))        // true
fmt.println(strings.index(s, "世"))             // 7

// Case conversion — allocates, use with defer delete()
upper, _ := strings.to_upper(s)
defer delete(upper)

lower, _ := strings.to_lower(s)
defer delete(lower)

// Clone a string
cloned, _ := strings.clone(s)
defer delete(cloned)

// Split and join — allocate; caller must free
parts, _ := strings.split(s, ", ")
defer delete(parts)

joined, _ := strings.join(parts, " | ")
defer delete(joined)

// Trim
trimmed := strings.trim_space("  hi  ") // no allocation

// Replace — allocates
replaced, _ := strings.replace(s, "Hello", "Hi", 1)
defer delete(replaced)

// Efficient string building with strings.Builder
b: strings.Builder
strings.builder_init(&b)
defer strings.builder_destroy(&b)

strings.write_string(&b, "Hello")
strings.write_byte(&b, ',')
strings.write_string(&b, " World")
result := strings.to_string(b)  // view into builder's buffer — no new allocation

// fmt.tprintf — formatted string allocated on the temp allocator
// (valid until free_all(context.temp_allocator) is called)
msg := fmt.tprintf("x=%d, y=%.2f", 10, 3.14)
// use msg ...
// no explicit free needed — lives until temp_allocator is reset

// cstring interop (for C FFI)
odin_str   := "Hello"
c_str      := strings.clone_to_cstring(odin_str) // allocates
defer delete(c_str)

back_again := string(c_str)                      // O(n): scans for null terminator
```

> **Key differences:**
> - Both `string` types are (pointer, length) internally, but Go strings are backed by read-only memory while Odin strings can point to any byte sequence.
> - Go's `strings` package functions return new strings without allocation concerns (GC handles cleanup). Odin's `strings` package functions that allocate return `(string, Allocator_Error)` and require explicit `delete`.
> - `fmt.tprintf` is unique to Odin — it allocates on `context.temp_allocator`, making it effectively free to use as long as you reset the temp allocator regularly.
> - Odin has a separate `cstring` type for null-terminated C strings; conversion between `string` and `cstring` is explicit and has cost.
> - `strings.Builder` in Go is a `bytes.Buffer` variant; Odin's `strings.Builder` wraps a `[dynamic]byte` with write helpers.

---

## Bit Sets (Odin only)

Odin's `bit_set` is a memory-efficient way to store a set of boolean-style flags backed by the bits of a single integer. Go has no equivalent — the closest Go pattern is either a bitmask constant block with `iota` and manual bitwise operations, or a `map[T]bool`.

**Go** — manual bitmask via `iota`:
```go
type Permission int

const (
    Read    Permission = 1 << iota // 1
    Write                          // 2
    Execute                        // 4
)

// Manual bitwise operations
perms := Read | Write
perms |= Execute       // add Execute
perms &^= Write        // clear Write (bit-clear operator)

hasRead := perms&Read != 0 // check membership
```

**Odin** — first-class `bit_set` backed by an integer:
```odin
Permission :: enum { Read, Write, Execute }

// bit_set[Enum] automatically picks the smallest backing integer
perms: bit_set[Permission]

perms += {.Read, .Write}   // add flags
perms += {.Execute}        // add one more
perms -= {.Write}          // remove a flag

// Membership test
fmt.println(.Read in perms)         // true
fmt.println(.Write not_in perms)    // true

// Initialize at declaration
flags := bit_set[Permission]{.Read, .Execute}

// Set operations
a := bit_set[Permission]{.Read, .Write}
b := bit_set[Permission]{.Write, .Execute}

union_set        := a | b   // {Read, Write, Execute}
intersection_set := a & b   // {Write}
difference_set   := a - b   // {Read}

// Subset / superset checks
fmt.println(a <= b)  // false — a is not a subset of b
fmt.println(b >= b)  // true  — b is a superset of itself

// Force a specific backing integer (e.g. u8)
Keycard :: enum { Red, Green, Blue, Yellow }
hand: bit_set[Keycard; u8]  // explicitly use a u8 (8 bits) as backing type

// bit_set with a range of integers
digits: bit_set['0'..'9']   // each bit represents one digit character
digits += {'3', '7'}
fmt.println('3' in digits)  // true

// bit_set with a range of characters (e.g. vowels)
vowels := bit_set['a'..'z']{'a', 'e', 'i', 'o', 'u'}
```

| Operation | Go (manual) | Odin `bit_set` |
|---|---|---|
| Add flag | `s \|= Flag` | `s += {.Flag}` |
| Remove flag | `s &^= Flag` | `s -= {.Flag}` |
| Check membership | `s & Flag != 0` | `.Flag in s` |
| Intersection | `a & b` | `a & b` |
| Union | `a \| b` | `a \| b` |
| Difference | `a &^ b` | `a - b` |
| Subset | manual | `a <= b` |

> **Key difference:** Odin's `bit_set` is built into the type system — exhaustiveness, enum names in debug output, and set operators are all handled by the compiler. In Go you manage the bit manipulation manually and lose type safety.

---

## Context System (Odin only)

Odin's context system has no equivalent in Go. Every procedure using the `"odin"` calling convention (the default) receives an implicit `Context` value passed by pointer. This allows per-scope control of allocators, logging, random state, and user data without changing function signatures.

```odin
import "core:mem"
import "core:fmt"
import "core:log"

// The implicit `context` variable is always in scope.
// Its type (simplified) is:
//
// Context :: struct {
//     allocator:               mem.Allocator,
//     temp_allocator:          mem.Allocator,
//     logger:                  log.Logger,
//     random_generator:        rand.Generator,
//     user_data:               rawptr,
//     user_index:              int,
//     assertion_failure_proc:  proc(...),
//     ...
// }

// --- Reading and copying context ---
show_context :: proc() {
    // context is local to each scope and passed implicitly
    fmt.println(context.user_index) // prints the current user_index

    // Take a copy before modifying
    saved := context
    context.user_index = 99
    inner()                         // inner() sees user_index=99
    assert(context.user_index == 99)

    // Restoring is NOT automatic — use a copy if you need to restore
    context = saved
    assert(context.user_index == saved.user_index)
}

// --- Swapping the allocator for a scope ---
use_temp_allocator :: proc() {
    // Everything allocated inside this block uses the temp allocator
    context.allocator = context.temp_allocator

    data := make([]int, 1024) // goes to temp_allocator
    _ = data
    // No delete needed — will be freed by free_all(context.temp_allocator)
}

// --- Passing context across scopes (child inherits parent's context) ---
parent :: proc() {
    context.user_index = 42
    child()             // child sees user_index=42
}

child :: proc() {
    // context is the same value the parent had (passed by implicit pointer)
    fmt.println(context.user_index) // 42
}

// --- Modifying context in a nested scope ---
nested_scope_demo :: proc() {
    context.user_index = 1
    {
        context.user_index = 2   // modifies only this scope's copy
        fmt.println(context.user_index) // 2
    }
    fmt.println(context.user_index) // 1 — outer scope unchanged
}

// --- Custom allocator injection (intercept third-party code) ---
using_custom_allocator :: proc() {
    track: mem.Tracking_Allocator
    mem.tracking_allocator_init(&track, context.allocator)
    defer mem.tracking_allocator_destroy(&track)

    // Redirect ALL allocations in this scope (and every proc called from here)
    context.allocator = mem.tracking_allocator(&track)

    // third_party_lib_proc() and everything it calls now uses our allocator
    // without third_party_lib_proc knowing about it
    third_party_lib_proc()

    for _, entry in track.allocation_map {
        fmt.printf("leak: %v bytes at %v\n", entry.size, entry.location)
    }
}

// --- Logger ---
setup_logger :: proc() {
    // Replace the logger for this scope and everything called from it
    context.logger = log.create_console_logger(.Debug)
    defer log.destroy_console_logger(context.logger)

    log.info("this uses the console logger")
    do_work() // do_work() and its callees use the same logger
}

// --- C interop: manual context setup ---
// Procedures with "c" calling convention do NOT receive context implicitly.
// You must set it manually before calling Odin code.
@(export)
my_c_callback :: proc "c" () {
    context = runtime.default_context() // set up Odin context manually
    fmt.println("called from C, context is now valid")
}
```

> **Why this matters:**
> - Allocator injection: pass any allocator (arena, tracking, pool) to any library without modifying the library's API. This is Odin's answer to dependency injection for memory.
> - Logger injection: replace the logger per-request (e.g., in a web server) so all log calls in the call tree automatically go to the right destination.
> - Zero-cost: the context pointer is passed in a register under the `"odin"` calling convention — no heap allocation, no overhead compared to a plain function call.
> - Go has no equivalent. The closest Go patterns are `context.Context` (passed explicitly as first argument) for cancellation/deadlines, and global allocator replacement via build tags.
