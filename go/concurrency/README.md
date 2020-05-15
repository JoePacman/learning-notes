# Golang notes
*following https://tour.golang.org/welcome/1*

# Concurrency

## Go Routines
A goroutine is a function that is capable of running concurrently with other functions. To create a goroutine we use the keyword go followed by a function invocation:
``` go
func say(s string) {
	for i := 0; i < 5; i++ {
		time.Sleep(100 * time.Millisecond)
		fmt.Printf("%v: %v\n", i, s)
	}
}

func main() {
	go say("world")
	say("hello")
}
```
```
0: hello
0: world
1: world
1: hello
2: hello
2: world
3: world
3: hello
4: hello
```
The ordering can vary. The Go routine is started and then the program continues.
Notice how the last 'world' does not print here. The program ends when the main Go Routine is complete, preventing the one we've created from completing.

## Channels

Channels are a typed conduit through which you can send and receive values with the channel operator, <-.

``` go
ch := make(chan int) // create channel
ch <- v    // Send v to channel ch.
v := <-ch  // Receive from ch, and assign value to v.
```

By default, sends and receives block until the other side is ready. This allows goroutines to synchronize **without explicit locks or condition variables**.

A sender can close a channel to indicate that no more values will be sent. Receivers can test whether a channel has been closed by assigning a second parameter to the receive expression
``` go
v, ok := <-ch //ok is false if there are no more values to receivea and the channel is closed.
```

This example code sums the numbers in a slice, distributing the work between two goroutines. Once both goroutines have completed their computation, it calculates the final result.

``` go
func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum // send sum to c
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int)
	go sum(s[:3], c)
	go sum(s[3:], c)
	x, y := <-c, <-c // receive from c

	fmt.Println(x, y, x+y)
}

```

By **default channels are unbuffered**, meaning that they will only accept sends (chan <-) if there is a corresponding receive (<- chan) ready to receive the sent value.  

**Buffered channels** accept a limited number of values without a corresponding receiver for those values. Sends to a buffered channel block only when the buffer is full.

``` go
func main() {

    messages := make(chan string, 2) //up to 2 values

    messages <- "buffered"
	messages <- "channel"
	// fmt.Println(<-messages) //N.B. 2. UNLESS we added this to receive the value first.
	// message <- "deadlock" // N.B. 1. adding this would result in a DEADLOCK. 

    fmt.Println(<-messages) //receive values as usual
    fmt.Println(<-messages)
}
```
```
buffered
channel
```

## Range, Close and Select
The loop ``for i := range c`` receives values from the channel repeatedly until it is closed. Simple example:


``` go
func clever_count(n int, c chan int) {
	for i := 0; i < n; i++ {
		c <- i
	}
	close(c) // *
}

func main() {
	c := make(chan int, 10)
	go clever_count(cap(c), c)
	for i := range c {
		fmt.Println(i)
	}
}

```
*Note that closing is only necessary when the receiver must be told there are no more values coming, such as to terminate a range loop.

The `` select `` statement lets a goroutine wait on multiple communication operations.

A ``select`` blocks until one of its cases can run, then it executes that case. It chooses one at random if multiple are ready.

Select block example:

``` go
func count_or_quit(c, quit chan int) {
	x := 1
	for {
		select {
		case c <- x:
			x = x + 1
		case <- quit:
			fmt.Println("quit")
			return			
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
	
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<- c)
		}
		quit <- 0 // this number could be anything. Just note it's something received by the quit channel.
	}()
	
	count_or_quit(c, quit)
}
```
```
1
2
3
4
5
6
7
8
9
10
quit
```

The ``default`` case in a select is run if no other case is ready. Use a default case to try a send or receive without blocking.

``` go
func main() {
	tick := time.Tick(100 * time.Millisecond) // creates a new Timer channel
	boom := time.After(500 * time.Millisecond) // creates a new Timer channel
	for { // remember this is Go's infinite While loop
		select {
		case <-tick:
			fmt.Println("tick.")
		case <-boom:
			fmt.Println("BOOM!")
			return
		default:
			fmt.Println("    .")
			time.Sleep(50 * time.Millisecond)
		}
	}
}

```
```
    .
    .
tick.
    .
    .
tick.
    .
    .
tick.
    .
    .
tick.
    .
    .
BOOM!
```

## sync.Mutex

What if we need to make sure only one goroutine can access variable at a time?

This is the concept of *mutual exclusion* and the datastructure used in go is the ``mutex`` which has methods ``Lock`` and ``Unlock``.

The ``defer`` method can be used to ensure the mutex is unlocked.

``` go
// SafeCounter is safe to use concurrently.
type SafeCounter struct {
	v   map[string]int
	mux sync.Mutex
}

// Inc increments the counter for the given key.
func (c *SafeCounter) Inc(key string) {
	c.mux.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	c.v[key]++
	c.mux.Unlock()
}

// Value returns the current value of the counter for the given key.
func (c *SafeCounter) Value(key string) int {
	c.mux.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	defer c.mux.Unlock()
	return c.v[key]
}

func main() {
	c := SafeCounter{v: make(map[string]int)}
	for i := 0; i < 1000; i++ {
		go c.Inc("somekey")
	}

	time.Sleep(time.Second)
	fmt.Println(c.Value("somekey"))
}
```
```
1000
```

## Binary Trees

Golang's concurrency langauage allows for much easier implementation of binary trees. 

See:
https://tour.golang.org/concurrency/7
https://godoc.org/golang.org/x/tour/tree#Tree
