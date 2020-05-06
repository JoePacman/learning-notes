# Golang notes
*following https://tour.golang.org/welcome/1*

## BASICS

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

**Basic types in Go**
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
