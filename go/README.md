# Golang notes
*following https://tour.golang.org/welcome/1*

## Basics

**Capitalised variables are exported. Lower case are not**
``` go
func main() {
	fmt.Println(math.pi)
}
```
would result in
./prog.go:9:14: cannot refer to unexported name math.pi
./prog.go:9:14: undefined: math.pi

and would need to be swappet out with
``` go
fmt.Println(math.Pi)
```

**Function return types appear after variables, and can return > 1 result**
``` go
func add(x int, y int) (int, error) {...}
```

**Two or more consecutive parameters can share a type, with only one declaration**
``` go
func add(x, y, z int) int {...}
```

**'Naked' return**
``` go
func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}
```

**var statements declare variable(s)**

``` go
var c, python, java bool
// with initializers
var i, j int = 1, 2
// with mix of inferred types
var c, python, java = true, false, "no!"
//short assignment - only available WITHIN functions
k := 3
```

**constants can be declared inside or out of functions, but not with := syntax**
``` go
const Pi = 3.14
```

**constants can also be declared in constant blocks**
``` go
const (
	k = 200
	z = 42
)
```

Constants cannot be initialised to a variables value. They are initialised at  
compile time.

## Basic types
``` go
bool //if uninitialised 'zero' value is false

string //if uninitialised 'zero' value is ""

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr
// generally int should be used unless you have a specific reason to define the memory size.
// unsigned ints don't have +ve or -ve.

byte // alias for uint8

rune // alias for int32
     // represents a Unicode code point

float32 float64

complex64 complex128 //complex #s containing a real and imaginary part

// if uninitialised 'zero' value of all numeric types is 0
```

## Loops, Conditionals:

## For
``` go
// init; condition; post;
for i := 0; i < 10; i++ {
		// do stuff
    }

// the init and post statement are optional
for ; sum < 1000; {
	sum += sum
}

// but you would generally right the above as
for sum < 1000 {
	sum += sum
}
// which is Go's version of while
```
## If
``` go
// simple if
if x < 0 {
	// do stuff
}

// if with init statement (as in for) is allowed
if v := math.Pow(x, n); v < lim {
		return v
	} else {
        // variables declared in if are also available in else
    }
```
## Switches
``` go
	switch os := runtime.GOOS; os {
	case "darwin":
        fmt.Println("OS X.")
        // switches in Go only run the case, they don't need a break statement as in Java/ other langs.
	case "linux":
		fmt.Println("Linux.")
	default:
		// freebsd, openbsd,
		// plan9, windows...
		fmt.Printf("%s.\n", os)
    }

// switches without a statement are a clean way of doing a long if-then-else chain
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

## The defer statement
The defer statement only executes after the function call has been completed.
However its arguments are evaluated immediately.

``` go
func main() {
	defer fmt.Println("world")

	fmt.Println("hello")
}
```

 
## Data structures:

## Pointers

The * operator denotes the pointer's underlying value.
 
 The & operator generators a pointer to an operand

``` go
i := 42
p = &i
fmt.Println(*p) // read i through the pointer p. Prints 42
*p = 21         // set i through the pointer p
// this is known as dereferncing or indirecting
fmt.Println(i) // prints 21
```

## A struct is a collection of fields
``` go
type Vertex struct {
	X int
	Y int
}
```
when referencing a field value in a struct, Go allows us to not use a pointer as the notation would be cumbersome
``` go
func main() {
	v := Vertex{1, 2}
	p := &v
	p.X = 1e9 // allowed rather than (*p).X (which still works)
	fmt.Println(v)
}
```

Struct literals denote a newly allocated struct value by listing the values of its fields:
``` go
var (
	v1 = Vertex{1, 2}  // has type Vertex
	v2 = Vertex{X: 1}  // Y:0 is implicit
	v3 = Vertex{}      // X:0 and Y:0
)
```

## Arrays

Arrays have a **FIXED** size.

``` go
var a[2]string //array declaration. Size 2
a[0] = "Hello"
a[1] = "World"

b := [2]string{"good", "morning"} //in line. This is called a 'literal'
```

## Slices

Slices are **DYNAMICALLY SIZED** flexible views into the elements of an array.

``` go
primes := [6]int{2, 3, 5, 7, 11, 13}
var s []int = primes[1:4] //slice
fmt.Println(s)

// prints [3, 5, 7]
s[0] = 0
fmt.Println(primes)
//prints [2 0 5 7 11 13]

// values can be omitted e.g.
fmt.Println(primes[:3])
//prints [2 0 5]

//this creates an array and then builds a slice that references it
c := []bool{true, true, false}
```
As you can see in the above altering a slice, alters the underlying array elements.

Slices have length and capcity. 
 
Length is the number of elements contained in the slice.
Capacity is the number of elements in the underlying array.

``` go
// The built-in make function can be used to create dynamically sized arrays and return a slice that refers to that array.
a := make([]int, 5)  // len(a)=5
b := make([]int, 0, 5) // len(b)=0, cap(b)=5
```

Slices can contain any type, e.g. another slice
``` go
board := [][]string{
	[]string{"_", "_", "_"},
	[]string{"_", "_", "_"},
	[]string{"_", "_", "_"},
}

board[0][0] = "X"
fmt.Println(board) // [[X _ _] [_ _ _] [_ _ _]]
```
More than one element can be appended to a slice
``` go
	var s []int

	// append works on nil slices.
	s = append(s, 0)
	printSlice(s) // [0]

	// We can add more than one element at a time.
	s = append(s, 2, 3, 4)
	printSlice(s) // [0, 2, 3, 4]
```

## Range

The range form of the for loop iterates over a slice or map.  

When ranging over a slice, two values are returned for each iteration. The first is the index, and the second is a copy of the element at that index.

```go
var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}

func main() {
	for i, v := range pow {
		fmt.Printf(" index: %d, value: %d \n", i, v)
	}
}
/*
prints:
	index: 0, value: 1 
 	index: 1, value: 2 
 	index: 2, value: 4 
 	index: 3, value: 8 
 	index: 4, value: 16 
 	index: 5, value: 32 
 	index: 6, value: 64 
 	index: 7, value: 128
 */
```
The index or value can be ommited with _.

## Maps
Maps map keys to values
``` go

type Vertex struct {
	Lat, Long float64
}

var m map[string]Vertex

func main() {
	m = make(map[string]Vertex)
	m["Bell Labs"] = Vertex{
		40.68433, -74.39967,
	}
	fmt.Println(m["Bell Labs"])
	// prints {40.68433 -74.39967}
}

```

Insert or update an element in map m:
``` go 
m[key] = elem
```

Retrieve an element:
``` go 
elem = m[key]
```

Delete an element:
``` go 
delete(m, key)
```

Test that a key is present with a two-value assignment:
``` go 
elem, ok = m[key]
```
If key is in m, ok is true. If not, ok is false.

## Functions

Functions are values and can be passed around like values.

``` go

func compute(fn func(float64, float64) float64) float64 {
	return fn(3, 4)
}

func main() {
	hypot := func(x, y float64) float64 {
		return math.Sqrt(x*x + y*y)
	}

	fmt.Println(compute(hypot)) // (9 + 16)^0.5 = 5
}

```

Go functions may be closures.  
A closure is a function value that references variables from outside its body.  
The function may access and assign to the referenced variables;  
in this sense the function is "bound" to the variables.

Read this: https://stackoverflow.com/questions/36636/what-is-a-closure

``` go
func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		fmt.Println(sum)
		return sum
	}
}

func main() {
	pos := adder()
	// 	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		pos(i)
	}
}
/* prints 
0
1
3
6
10
15
21
28
36
45
*/
```