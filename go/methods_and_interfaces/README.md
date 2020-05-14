# Golang notes
*following https://tour.golang.org/welcome/1*

# Methods and Interfaces

Go **DOES NOT HAVE CLASSES** but you can define methods on types (they are still just functions):

``` go
type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
```

Methods with pointer receivers can modify the value to which the receiver points. Since methods often need to modify their receiver, pointer receivers are more common than value receivers.

``` go
type Vertex struct {
	X, Y float64
}

func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}
```

This is basically a method on the Vertex object altering its underlying X, Y values.

Functions with pointer arguments must take a pointer:

``` go
var v Vertex
ScaleFunc(v, 5)  // Compile error!
ScaleFunc(&v, 5) // OK
```

here ScaleFunc is a rewrite of Scale that does the same thing but is not a method on Vertex:
``` go
func ScaleFunc(v *Vertex, f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}
```

while methods with pointer receivers take either a value or a pointer as the receiver when they are called:

``` go
var v Vertex
v.Scale(5)  // OK
p := &v
p.Scale(10) // OK
// Here as a convenience Go interprets the statement v.Scale(5) as (&v).Scale(5) 
```

Reversing this, with *pointer indirection*, functions that take a value argument  
must take a value of that specific type.
``` go
var v Vertex
fmt.Println(AbsFunc(v))  // OK
fmt.Println(AbsFunc(&v)) // Compile error!
```

whil methods with value receivers can take a value or a pointer
``` go
var v Vertex
fmt.Println(v.Abs()) // OK
p := &v
fmt.Println(p.Abs()) // OK
```

There are two reasons to use a pointer receiver.

The first is so that the method can modify the value that its receiver points to.

The second is to avoid copying the value on each method call. This can be more efficient if the receiver is a large struct, for example.

## Interfaces

There is no explicit declaration of implementing an interface, it is explicit.

``` go

type I interface {
	M()
}

type T struct {
	S string
}

// This method means type T implements the interface I,
// but we don't need to explicitly declare that it does so.
func (t T) M() {
	fmt.Println(t.S)
}

func main() {
	var i I = T{"hello"}
    i.M()
}
```

Under the hood, interface values can be thought of as a tuple of a value and a concrete type:
```
(value, type)
```
The interface type that specifies zero methods is known as the empty interface:

```
interface{}
```
An empty interface may hold values of any type and are used by code that handles values of unknown type.For example, ``fmt.Print`` takes any number of arguments of type ``interface{}``.

A type assertion provides access to an interface value's underlying concrete value.
``` go
t := i.(T) // access to underlying value. Will panic if not of expected type.
// e.g. i is a String and i.(float64) called

t, ok := i.(T) //get underlying value and boolean on whether assertion succeeded. If not ok t will be zero type of T - and no panic :)
```

Type switches allow several type assertions in series:
``` go
switch v := i.(type) {
case T:
    // here v has type T
case S:
    // here v has type S
default:
    // no match; here v has the same type as i
}
```

One of the most ubiquitous interfaces is Stringer defined by the fmt package.
``` go
type Stringer interface {
    String() string
}
```
This is essentiall toString() in Java.

## Errors

The error type is a built-in interface similar to fmt.Stringer:

``` go
type error interface {
    Error() string
}
```

An example of a user created error:

``` go
type MyError struct {
	When time.Time
}

func (e *MyError) Error() string {
	return fmt.Sprintf("My custom error happened at %v",
		e.When)
}

func run() error {
	return &MyError{
		time.Now(),
	}
}

func main() {
	if err := run(); err != nil {
		fmt.Println(err) //prints error message on MyError
	}
}
```
