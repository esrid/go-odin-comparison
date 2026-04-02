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

// Range with half-open interval [0, 3)
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
