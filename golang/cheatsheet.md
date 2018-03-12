# Go Programming Language

- GO program made of packages
- GO program start running in `main` package

In Golang, there is two type of name `Exported name` and `Un-exported name`.
- `Exported name`: Begin with a capital letter. Ex: `fmt.Println`
- `Un-exported name`: Do not start with capital letter. Ex: `pizza`

This `exported` thing is related when you are importing a package. You can refer only to its exported names such `fmt.Println`. Any `un-exported name` is not accessible from outside the package.

Ex: `math.Pi` vs `math.pi`

-----

In Golang you need to define data type such as `int` and `string` when defining parameters.
Ex: `var pi int`, `var x string`

There is some difference with C language about how to define a variable which the data type comes after variable name while C language is the opposite. For example in C language is `int x`.


----

A function can return any number of results
There is two type of return in Golang.
- `named return`

```go
func swap(x, y string) (string, string) {
  return x, y
}
```

- `naked return`

```go
func swap(sum int) (x, y int) {
  x = sum * 2
    y = sum + 2
    return
}
```

Function in golang can be a closures.
A closure is a function value that references variables from outside its body. 
The function may access and assign to the referenced variables; in this sense the function is "bound" to the variables.
Ex:
```go
func adder() func(int) int {
  sum := 0
  return func(x int) int {
    sum += x
    return sum
  }
}

func main() {
  pos, neg := adder(), adder()
  for i := 0; i < 10; i++ {
    fmt.Println(
      pos(i),
      neg(-2*i),
    )
  }
}

```


----

Variables are defined with `var` keyword and can be inside package or function level. Variables can have an initializer, one per variable.
If variables defined with an initializer, then we can omit the data type and the variable will take the type of the initializer
Ex: 

```go
var i, k int = 1, 2
var pi, is_okay = 3.14, true
```

Along with `var` keyword, there is another way to define a variable. That is `:=` the short variable declaration.
Short variable declaration only works inside a function
Ex:

```go
func sum(x string) string {
  i := x
  return x
}
```

Some notes about `:=` operator is after you assign to a variable then you can't re-assign it again with `:=` operator.
Ex:

```go
func sum(x int) int {
  i := x
  i := i + 10
  // You will get error "no new variable on the left side of :="
  return i
}
```

-----

Zero value in Golang:

- 0 for numeric
- false for boolean type
- `""` (empty string) for string


----

Golang also supports type conversion.
Ex:

```go
  var i int = 10
  var f float64 = float64(i)
```

----

In Golang also exists constant with keyword `const` and can be character, numeric, string, boolean. Constant cannot be declared with `:=` syntax.

----

`for` is the only one looping structure in Golang. How you write `for` syntax is almost same like you do in JavaScript.
Ex:

```go
for i := 0; i < 10; i++ {
  fmt.Println(i)
}
```

`for` has three components:
- `i := 0` is the init statement. Executed before the first iteration (optional)
- `i < 10` is the condition expression. Executed  before every iteration
- `i++` is the post statement. Executed at the end of every iteration (optional)

There is no need surround the three components with parentheses unlike JavaScript, but the curly braces are always required.
If you want to create an infinite looping, then leave out the three component.


-----

`if` statements are like `for`, no need to surround the expression with parentheses `()` but braces is required. Variables declared inside `if` statements is not accessible outside the `if` and/or `else` scope.
Ex:

```go
if a > 10 {
  fmt.Println("OK")
} else {
  fmt.Println("Not OK")
}

// With short statement
if a = math.Sqrt(100); a > 10 {
  fmt.Println("OK")
} else if a == 0 {
  fmt.Println("Hmm OK")
} else {
  fmt.Println("Not OK")
}
```


----

`switch` also exists in Golang. `switch` declaration is pretty similar with `if`. They can have a short statement before condition statement. `switch` in Golang only runs the selected case, not all the cases that follow so the `break` statement in each `case` is don't needed. Also, the switch cases are not need to be constants.
Ex:

```go
fmt.Print("Go runs on ")
switch os := runtime.GOOS; os {
  case "darwin":
    fmt.Println("OS X.")
  case "linux":
    fmt.Println("Linux.")
  default:
    // freebsd, openbsd,
    // plan9, windows...
    fmt.Printf("%s.", os)
}
```

`switch` statement that don't have conditions are called `switch true`.
Ex:

```go
  func main() {
    t := time.Now()
    switch {
    case t.Hour() < 12:
      fmt.Println("Good morning!")
    case t.Hour() < 17:
      fmt.Println("Good afternoon.")
    default:
      fmt.Println("Good evening.")
    }
}
```

----

In Golang there is called Defer (`defer`). `defer` command is used to defer a function until the surrounding function returns or finish executed.
Ex:

```go
func main() {
  defer fmt.Println("world")

  fmt.Println("hello")
}
// - This will print
// hello
// world
// - Get it?
```

It is like you hold some function and execute them until the function where `defer` function is written finish or return some result.
`defer` can also be stacked and the execution order is **last-in-first-out**.
Ex:
```go
func main() {
  fmt.Println("counting")

  for i := 0; i < 3; i++ {
    defer fmt.Println(i)
  }

  fmt.Println("done")
}
/*
Print:
counting
2
1
0
done
 */
```

----

Golang has pointers. Pointers holds the memory address of value.
Ex:
```go
var p *int

i := 24
p = &i

// This is known as "dereferencing" or "indirecting"
*p = 21 // Change value i through pointer p
```

How to read pointers? (Thanks to mas Iman for this tips :thumbsup:)
- `*int` means `pointer of int`
- `&angka` (while `angka` is variable) means `address of angka`
- `*angka` means `value of angka`


----

There is also `struct` in Golang. `struct` is collection of fields.
Ex: 
```go
type Str struct {
  number int
  code string
}

s := Str{1, "hello"}
fmt.Println(s.number)
```

`struct` field is accessed with a dot. `struct` field can also be accessed through pointer.
Ex:
```go
type Vertex struct {
  X int
  Y int
}

func main() {
  v := Vertex{1, 2}
  p := &v
  p.X = 1e9
  fmt.Println(v)
}
```

`&` in `struct` is not a pointer to the struct but it returns a pointer to the struct value.
```go
type Vertex struct {
  X, Y int
}

var (
  v1 = Vertex{1, 2}  // has type Vertex
  v2 = Vertex{X: 1}  // Y:0 is implicit
  v3 = Vertex{}      // X:0 and Y:0
  p  = &Vertex{1, 2} // has type *Vertex
)

func main() {
  fmt.Println(v1, p, v2, v3)
}
```

----

#### Array and Slices

Array is defined with `[n]T` with `n` is the size for the array and `T` is the array type.
Ex:
```go
var a [5]int // Will create an array of 5 integers

var a [5]int{1, 2, 3, 4, 5} // Create an array of 5 integers with initial value
```

In Golang array has fixed size. Means that they cannot be resized after they created.
Evertime an array is created GO will allocated as big as the size of the array.

Meanwhile Slice has dynamic size. Slice created with `[]T` with `T` is the array type.
When creating slice we don't set the size of the slice like array do.
Ex:
```go
var s []int
```

Slices actually a pointer/referrences to an arrays. We can also select range of slice with `[high:low]` syntax
but this will exlude the last one. The `high` and `low` is optional actually. The default for those two is zero and the length on the slice
respectively
Ex:
```go
var s []{1, 2, 3, 4, 5}
slc := s[1:3] // This will create [2, 3] but exclude 4 as its in index 3

s[0:] // => [1, 2, 3, 4, 5]
s[:3] // => [1, 2, 3]
s[:] // => [1, 2, 3, 4, 5]
s[3:] // => [4, 5]

```

Slice also don't store any data like array do.
When you change the slices that referrences to an array you will make a change to that array too.
Ex:
```go
func main() {
  names := [4]string{
    "John",
    "Paul",
    "George",
    "Ringo",
  }
  fmt.Println(names)

  a := names[0:2]
  b := names[1:3]
  fmt.Println(a, b)

  b[0] = "XXX"
  fmt.Println(a, b)
  fmt.Println(names)

  // This will print
  /*
  [John Paul George Ringo]
  [John Paul] [Paul George]
  [John XXX] [XXX George]
  [John XXX George Ringo]
  */

}

```

Slices has both length (`len(s)`) and capacity (`cap(s)`) just like Arrays. Slices and arrays can also be created with `make`
keyword.
Ex:
```go
a := make([]int, 5)
```

We can append slice using built-in `append`(`func append(s []T, vs ...T) []T`) function.
First parameter is a slice you want to append, and the rest is the value you want to append into the slice.
If the backing array of slice is too small to fit all the given values bigger array will be allocated. The returned
slice will point to the newly allocated array.
Ex:
```go
var s[]int

s = append(s, 0) // [0]
s = append(s, 1) // The slice size is growing. [0, 1]
```

----

__*TIPS SECTION*__ from mas Iman:

Use `%+v` when debugging.
Ex:
```go

type Str struct {
  name string
  age int
}

func main() {
  fmt.Printf("Hello, playground %v\n", Str{}) // This will print "Hello, playground { 0}"
  fmt.Printf("Hello, playground %+v", Str{}) // This will print "Hello, playground {name: age:0}"
}
```
See the difference between `%+v` and `%v`?
While `%v` will print only the value of struct, `%+v` will print the fields with the values of the struct

If you want to use interpolation in string like `%v` or `%d` use `fmt.Printf()` instead `fmt.Println`

---

In Golang you can also loop through slice or map with `range` keyword. When looping with `range` you will get
two value for each iteration just like you do `each_with_index` in Ruby.
Ex:
```go
var s []int{1, 2, 3, 4, 5}

for index, value := range s {
  fmt.Printf("index: %v, value: %v", index, value)
}
```
You can skip the `value` or the `index` by assign it to `_` and if you want to skip the value then just don't write it.

```go
var s []int{1, 2, 3, 4, 5}

for _, value := range s {
  fmt.Printf("value: %v", value)
}

for index := range s {
  fmt.Printf("index: %v", index)
}
```

----

In Golang there is called `map`. This is like `hash` in Ruby with key-value pairs. The default value for `map` is `nil`.
Ex:
```go
type Vertex struct {
  Lat, Long float64
}

var m map[string]Vertex

func main() {
  m = make(map[string]Vertex)
  m["Bell Labs"] = Vertex{ // To set map element value
    40.68433, -74.39967,
  }
  fmt.Println(m["Bell Labs"])


  // You can also define the map like this
  m = map[string]Vertex{
    "Bell Labs": Vertex{
      40.68433, -74.39967,
    }
  }
  fmt.Println(m["Bell Labs"]) // To get map element value

  delete(m, "Bell Labs") // Delete map element
}
```

`map` is pretty similar with `struct` but in `map` the keys are required.
You can also test the is present with two-value assignment with `elm, ok = m[key]`

----

#### Methods

In Golang there is no class but you can create methods on types. A method is a function with __special receiver__ argument.
Ex:
```go
type Vertex struct {
  X, Y float64
}

func (v Vertex) Abs() float64 {
  return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
  v := Vertex{3, 4}
  fmt.Println(v.Abs())
}
```

Methods also can be created on non-struct types such as `type MyFloat float64`. You can only declare a method with a receiver whose type
is defined in the same package as the method. And also you can't declare method with receiver with built-in type such `int` or type
that inside another package.

You can't also declare methods with pointer receiver (`*T`) like `*int`.

There is a difference when method and function use *pointer argument*.
When function use pointer argument, the function must take a pointer while method can take either a value or a pointer.
And function that take a value argument must take a value of the specific type.
Ex:
```go
var v Vertex

Function(v, 5) // Error!
Function(&v, 5) // OK!

v.Method(5) // OK!
p := &v
p.Method(10) // OK!


/*
Code below means we define new method `Sum()`
*/
type Struct struct {
  x, y int
}

func (s *Struct) Sum() int {

}

s := Struct{1, 2}
s.Sum()

func Sum(s *Struct) int {

}

Sum(s)
```

---

#### Interface

Interface in Golang is an collection of methods. Interface is like *duck typing* in Ruby. For example,
if you declare interface `human` with `Speak()` as the method inside the interface. Then if any type implement 
`Speak()` method then that is type *implicitly* is a `human`. So it's like *if you walk like a duck, swim like a duck then you are duck*.
Ex:
```go
type Human interface {
  Speak()
}

type Str string

func (a Str) Speak() string {
  return fmt.Printf("String is %v", a)
}

var a Str

// At this point `a` is already as a `Human` IMPLICITLY
// because `a` is implemented `Speak()` method.
a.Speak()
```

