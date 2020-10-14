## I. Package import

1. By convention, the package name is the same as the last element of the import path. For instance, the `"math/rand"` package comprises files that begin with the statement `package rand`

```go
package main

// This code groups the imports into a parenthesized, "factored" import statement.
// import "fmt"
// import "math"

import ( 
	"fmt"
	"math"
)

func main() {
	fmt.Println(math.Pi) // In Go, a name is exported if it begins with a capital letter. For example, `Pizza` is an exported name, as is `Pi`, which is exported from the `math` package.
}
```



## II. Function

1. Arguments

```go
// normal form
func add(x int, y int) int {
	return x + y
}


// in case x and y is the same type
func add(x, y int) int {
	return x + y
}
```

2. Multiple results

```go
// A function can return any number of results

func swap(x, y string) (string, string) {
	return y, x
}

func main() {
	a, b := swap("hello", "world")
	fmt.Println(a, b)
}

```

3. Named return values & `Naked return`
   1. Go's return values may be named. If so, they are treated as variables defined at the top of the function.
   2. These names should be used to document the meaning of the return values.
   3. A `return` statement without arguments returns the named return values. This is known as a "naked" return.
   4. Naked return statements should be used only in short functions, as with the example shown here. They can harm readability in longer functions.

```go
func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}

func main() {
	fmt.Println(split(17))
}
```

## III. Variables

1. Declare & Initialize

```go
var c, python, java bool // declare multiple varibles. type is at the end

func main() {
	var i int
	fmt.Println(i, c, python, java) // 0 false false false
}


var i, j int = 1, 2 // initialize multiple variables in one line

func main() {
	var c, python, java = true, false, "no!" // you can omit type if you initialize with values
	fmt.Println(i, j, c, python, java)
}

```

2. Declare & initialize with `:=` inside function

```go
k := 3 // It does not work outside of a function

func main() {
	var i, j int = 1, 2
	k := 3 // It works only inside a function
	c, python, java := true, false, "no!"

	fmt.Println(i, j, k, c, python, java)
}

```

3. types
   1. bool
   2. The int, uint, and uintptr types are usually 32 bits wide on 32-bit systems and 64 bits wide on 64-bit systems. When you need an integer value you should use int unless you have a specific reason to use a sized or unsigned integer type.
      1. int int8 int16 int 32 int64
      2. uint uint8 uint16 uint32 uint64 uintptr
   3. byte // alias for uint8
   4. rune // alias for int32. Represents a Unicode code point
   5. float32 float64
   6. complex64 complex128

```go
package main

import (
	"fmt"
	"math/cmplx"
)

var (
	ToBe   bool       = false
	MaxInt uint64     = 1<<64 - 1
	z      complex128 = cmplx.Sqrt(-5 + 12i)
)

func main() {
	fmt.Printf("Type: %T Value: %v\n", ToBe, ToBe)
	fmt.Printf("Type: %T Value: %v\n", MaxInt, MaxInt)
	fmt.Printf("Type: %T Value: %v\n", z, z)
}
```

4. default Null type

```go
func main() {
	var i int // 0
	var f float64 // 0
	var b bool // false
	var s string // ""
	fmt.Printf("%v %v %v %q\n", i, f, b, s)
}
```

5. Type conversions

```go
// The expression T(v) converts the value v to the type T.
var i int = 42
var f float64 = float64(i)
var u uint = uint(f)


```

6. Type Inference

```go
// Numeric type inference depending on initializing input

i := 42           // int
f := 3.142        // float64
g := 0.867 + 0.5i // complex128
```

7. Constants

```go
// Constants cannot be declared using the := syntax.

const Pi = 3.14

func main() {
	const World = "世界"
	fmt.Println("Hello", World)
	fmt.Println("Happy", Pi, "Day")

	const Truth = true
	fmt.Println("Go rules?", Truth)
}

// An untyped constant takes the type needed by its context.

const (
	// Create a huge number by shifting a 1 bit left 100 places.
	// In other words, the binary number that is 1 followed by 100 zeroes.
	Big = 1 << 10
	// Shift it right again 99 places, so we end up with 1<<1, or 2.
	Small = Big >> 99
)

func needInt(x int) int { return x*10 + 1 }
func needFloat(x float64) float64 {
	return x * 0.1
}

func main() {
	fmt.Println(needInt(Small)) // 21
	fmt.Println(needInt(Big)) // constant 1267650600228229401496703205376 overflows int
	fmt.Println(needFloat(Small)) // 0.2
	fmt.Println(needFloat(Big)) // 1.2676506002282295e+29
}
```



## IV. Flow

1. Forloop

```go
func main() {
	// normal form
  for i := 0; i < 10; i++ {
		fmt.Println(i)
	}
	
	// Skip init and post statement
	sum := 1
	for ; sum < 1000; {
		sum += sum
	}
	
  fmt.Println(sum)
  
  // Infinite for loop
  for {
	}
}
```

2. if 

```go
	if x < 0 {
		return sqrt(-x) + "i"
	}


// Start with a short statement to execute before the condition. 
// Variables declared by the statement are only in scope until the end of the 
func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v + 1 // v is available here
	} else {
		return v + 2 // also available inside any of the "else" blocks.
	}

  return v // Exception - undefined: v
}
```

3. switch

```go
// No need to put "break;" because go does not run the followings
// Another important difference is that Go's switch cases need not be constants, and the values involved need not be integers
switch os := runtime.GOOS; os {
  case "darwin":
		fmt.Println("OS X.") 
  case "linux":
  	fmt.Println("Linux.")
	case f():	
		// You can even put a function
  default:
  fmt.Printf("%s.\n", os)
}


// You don't have to put a condition
t := time.Now()
	switch {
	case t.Hour() < 12:
		fmt.Println("Good morning!")
	case t.Hour() < 17:
		fmt.Println("Good afternoon.")
	default:
		fmt.Println("Good evening.")
	}

```

4. Defer

```
// A defer statement defers the execution of a function until the surrounding function returns.

// The deferred call's arguments are evaluated immediately, but the function call is not executed until the surrounding function returns.

func main() {
	defer fmt.Println("world")

	fmt.Println("hello")
}

// Result:
// hello
// world



// Deferred function calls are pushed onto a stack. When a function returns, its deferred calls are executed in last-in-first-out order.

func main() {
	fmt.Println("counting")

	for i := 0; i < 10; i++ {
		defer fmt.Println(i)
	}

	fmt.Println("done")
}

// Result:
// counting
// done
// 9
// 8
// 7
// 6
// 5
// 4
// 3
// 2
// 1
// 0

```

## V. Pointer

```go
func main() {
	i, j := 42, 2701

	p := &i         // point to i
	fmt.Println(*p) // 42
	fmt.Println(p) // 0xc000094010
	*p = 21         // set i through the pointer
	fmt.Println(i)  // see the new value of i

	p = &j         // point to j
	*p = *p / 37   // divide j through the pointer
	fmt.Println(j) // 73
}
```

## VI. Structs

A `struct` is a collection of fields.

```go
type Vertex struct {
	X int
	Y int
	Z string
}

func main() {
	fmt.Println(Vertex{1, 2, "3"}) // {1 2 3}

  // Accessing the field inside
  v := Vertex{1, 2}
	v.X = 4
	fmt.Println(v.X) // 4
  
  // To access the field X of a struct when we have the struct pointer p we could write (*p).X. 
  // However, that notation is cumbersome, 
  // so the language permits us instead to write just p.X, without the explicit dereference.
	p := &v
	p.X = 1e9
  fmt.Println(v) // {1000000000 2}
}
```

## VII. Arrays

The type `[n]T` is an array of `n` values of type `T`.

An array's length is part of its type, so arrays cannot be resized. 

```go
package main

import "fmt"

func main() {
	var a [2]string
	a[0] = "Hello"
	a[1] = "World"
	fmt.Println(a[0], a[1])
	fmt.Println(a)

	primes := [6]int{2, 3, 5, 7, 11, 13}
	fmt.Println(primes)
}

```

## VIII. Slices

An array has a fixed size. A slice, on the other hand, is a dynamically-sized, flexible view into the elements of an array. In practice, slices are much more common than arrays.

```go
func main() {
  
  // Creating from array
	primes := [6]int{2, 3, 5, 7, 11, 13}

  var s []int = primes[1:4] //inclusive : exclusive
	fmt.Println(s) // 3, 5, 7 
  
  // reference to arrays
  // A slice does not store any data, it just describes a section of an underlying array.
  // Changing the elements of a slice modifies the corresponding elements of its underlying array.
  // Other slices that share the same underlying array will see those changes.
 	names := [4]string{
		"John",
		"Paul",
		"George",
		"Ringo",
	}

  fmt.Println(names) // ["John", "Paul", "George",	"Ringo"]

	a := names[0:2]
	b := names[1:3]
	fmt.Println(a, b) // ["John", "Paul"], ["Paul", "George"]

	b[0] = "XXX"
	fmt.Println(a, b) // ["John", "XXX" ], ["XXX",	"Ringo"]
	fmt.Println(names) // ["John", "XXX", "George",	"Ringo"]
  
  // Slice literals
 	q := []int{2, 3, 5, 7, 11, 13}
	fmt.Println(q) // [2 3 5 7 11 13]

	r := []bool{true, false, true, true, false, true}
	fmt.Println(r) // [true false true true false true]

	s := []struct {
		i int
		b bool
	}{
		{2, true},
		{3, false},
		{5, true},
		{7, true},
		{11, false},
		{13, true},
	}
  fmt.Println(s) // [{2 true} {3 false} {5 true} {7 true} {11 false} {13 true}]

  // When slicing, you may omit the high or low bounds to use their defaults instead.
	// The default is zero for the low bound and the length of the slice for the high bound.
	s := []int{2, 3, 5, 7, 11, 13}

	s = s[1:4]
  fmt.Println(s) // [3 5 7]

	s = s[:2]
	fmt.Println(s) // [3 5]

	s = s[1:]
	fmt.Println(s) // [5]

  // The capacity of a slice is the number of elements in the underlying array, counting from the first element in the slice.
  // The length and capacity of a slice s can be obtained using the expressions len(s) and cap(s).
  
 	s := []int{2, 3, 5, 7, 11, 13}

  // Slice the slice to give it zero length.
	s = s[:0] // [ ]

	// Extend its length.
	s = s[:4] // [2 3 5 7]

	// Drop its first two values.
	s = s[2:] // [5 7]

  // The zero value of a slice is nil.
  // A nil slice has a length and capacity of 0 and has no underlying array.
  
 	var s []int
	fmt.Println(s, len(s), cap(s)) // [ ] 0 0
	if s == nil { // true
		fmt.Println("nil!")
	}
  
}
```







