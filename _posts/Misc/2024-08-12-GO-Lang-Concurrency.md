---
layout: post
title: GO-LANG-Concuurency Explained
date: 2025-01-05
categories: categories: [Misc , Go-Lang] 
tags: [Cloud, GoLang]
---
# Understanding Go-Routines:
```go
package main

import(

"fmt"

"time"

)

func say(s string){

for i:= 0; i< 10 ;i++ {

time.Sleep(20*time.Millisecond)

fmt.Println(s)

}

}

func main(){

go say ("world")

say("hello")

}
```

when sleeping time is 20 ms the output is

```bash
world hello hello world hello world world hello hello world world hello hello world world hello hello world world hello
```

but when it is 1000 ms the output is
```bash
world hello hello world world hello hello world world hello hello world hello world world hello hello world world hello
```

and on 100ms
```bash
hello world world hello hello world world hello world hello hello world hello world hello world hello world world hello
```
hello world world hello hello world world hello world hello hello world hello world hello world hello world world hello


# Buffered channels
```go

package main

  

import "fmt"

  

func main() {

ch := make(chan int, 2)

ch <- 1

ch <- 2

go func() {

ch <- 3 // This will block until space becomes available

}()

  

// Give some time for the goroutine to attempt sending

// In a real program, you should use synchronization, but this is just for demonstration

fmt.Println("Attempting to send 3 into the channel...")

  

// Receive values from the channel

fmt.Println(<-ch)

fmt.Println(<-ch)

  

// At this point, the buffer will have space, and the blocked goroutine can proceed

fmt.Println("Sending 3 into the channel now")

fmt.Println(<-ch)

}

  ```

# Range and Close
```go

package main

  

import (

"fmt"

)

  

func fibonacci(n int, c chan int) {

x, y := 0, 1

for i := 0; i < n; i++ {

c <- x

x, y = y, x+y

}

close(c)

}

  

func main() {

c := make(chan int, 10)

go fibonacci(cap(c), c)

for i := range c {

fmt.Println(i)

}

}
```

explaining it

The main function starts by creating a buffered channel c with a capacity of 50.

It then starts a goroutine running the fibonacci function, which will generate Fibonacci numbers and send them to the channel c.

The main function then enters a loop where it continuously reads from the channel c and prints each received value until the channel is closed.

we can say that close is kind of like the break statement in c++ not completely analogous when will the panic situation occur
```go

package main

  

import (

"fmt"

)

  

func fibonacci(n int, c chan int) {

x, y := 0, 1

for i := 0; i < n; i++ {

c <- x

x, y = y, x+y

}

close(c) // Close the channel after sending all values

  

// Attempting to send more values to the closed channel

// This will cause a runtime panic

c <- x // This line will cause the panic

}

  

func main() {

c := make(chan int, 50)

go fibonacci(cap(c), c)

for i := range c {

fmt.Println(i)

}

}
```

# Select 
```go
package main

import "fmt"

func main() {
    // Create channels
    ch1 := make(chan int)
    ch2 := make(chan int)
    ch3 := make(chan int)

    // Start a goroutine to send data into channels
    go func() {
        ch1 <- 1
        ch2 <- 2
        ch3 <- 0 // this will be received by the select
    }()

    // Use select to handle channel operations
    select {
    case msg1 := <-ch1:
        fmt.Println("Received from ch1:", msg1)
    case msg2 := <-ch2:
        fmt.Println("Received from ch2:", msg2)
    case <-ch3:
        // To demonstrate sending to ch3, you can use this case instead
        fmt.Println("Received from ch3")
    default:
        fmt.Println("No communication")
    }
}

```
this will always print ```No communication``` as it is selecting the default case because no other case is ready.
```go
package main
import "fmt"
func main() {

ch1 := make(chan int)

ch2 := make(chan int)

ch3 := make(chan int)


go func() {

ch2 <- 2

ch1 <- 1

ch3 <- 0 

}()


select {

case msg1 := <-ch1:

fmt.Println("Received from ch1:", msg1)

case msg2 := <-ch2:

fmt.Println("Received from ch2:", msg2)

case ch3 <- 42:

fmt.Println("Sent 42 to ch3")

}

}
```

```go
time.After vs time.Tick 
```

``` 
time.After creates a single event after the specified time while time.Tick creates a repeating event at regular intervals.
```


# Implementing tree in GO

```go 
package main
import ("golang.org/x/tour/tree"
		"fmt")

// Walk walks the tree t sending all values
// from the tree to the channel ch.
func Walk(t *tree.Tree, ch chan int) {
	defer close(ch)
	walk(t, ch)
}
func walk(t *tree.Tree, ch chan int){
	if t == nil {return}
	walk(t.Left , ch)
	ch <- t.Value
	walk(t.Right,ch)
	
}

// Same determines whether the trees
// t1 and t2 contain the same values.
func Same(t1, t2 *tree.Tree) bool{
	ch1:= make(chan int)
	ch2:= make(chan int)
	go Walk(t1,ch1)
	go Walk(t2,ch2)
	for{
		v1,ok1 := <- ch1
		v2,ok2 := <- ch2
		if ok1!= ok2 || (ok1 && v1!=v2){
			return false}
		if !ok1 && !ok2 {return true}
	}
}
func main() {
	// Test Walk function
	ch := make(chan int)
	go Walk(tree.New(1), ch)
	for i := 0; i < 10; i++ {
		fmt.Println(<-ch)
	}
	ch2 := make(chan int)
	go Walk(tree.New(5), ch2)
	for i := 0; i < 10; i++ {
		fmt.Println(<-ch2)
	}

	// Test Same function
	fmt.Println(Same(tree.New(1), tree.New(1))) // Should return true
	fmt.Println(Same(tree.New(1), tree.New(2))) // Should return false
}```
```
# defer:
What do we mean by ```defer```  
A `defer` statement adds the function call following the `defer` keyword onto a stack. All of the calls on that stack are called when the function in which they were added returns. Because the calls are placed on a stack, they are called in last-in-first-out order.
For example : 
```go
package main
import (
"fmt"
)
func main(){
defer fmt.Println("BYE")
fmt.Println("HI")
}
```
It will print
```bash 
HI
BYE
```
This is because any statement that is preceded by the `defer` keyword isn’t invoked until the end of the function in which `defer` was used.

Using `defer` to Clean Up Resources :
The main function of defer is cleaning up, it is used to ensure that certain cleanup operations are performed when a function exits, regardless of whether it exits normally or due to an error.
It is useful when we are releasing resources, closing files or for unlocking mutexes.
In other languages there are some other methods for cleaning up such as:
- **C++**: We can  use RAII (Resource Acquisition Is Initialisation) to manage resource cleanup automatically. Smart pointers and containers handle their own cleanup when they go out of scope.
- **Python**: You can use `try/finally` blocks to ensure cleanup, or context managers (using the `with` statement).

If multiple `defer` statements are used : 
As we know that defer uses a LIFO Stack to store so if we use multiple defer statements for eg : 
```go
package main

import "fmt"

func main() {
	defer fmt.Println("one")
	defer fmt.Println("two")
	defer fmt.Println("three")
}
```
it will print 
```bash
three
two
one
```
As the stack will be of the form [one,two,three].

# Mutex 

### So what does a mutex does:
Lets's say we are working on something where multiple go-routines are needed to access something now let's say if one go-routine accesses it and changes it then for the other ones they will now access the changed value but if two or more routines accesses it at a same time(also called as concurrent access) then what will happen , it will throw a error.
So to prevent this we use mutex which is kind of a lock where if one routine is accessing something no one else can access it, that is it is locked.
It is somewhat similar deadlocks in OS where threads are used. 

### How does Mutexes work?:
Mutex is available in the `sync` package and there are 2 methods defined for mutex i.e. `lock` and `unlock`  Any code that is present between a call to `Lock` and `Unlock` will be executed by only one Go-routine, thus avoiding race condition.
Eg. 
```go
package main
import (
	"fmt"
	"sync"
	)
var x  = 0
func increment(wg *sync.WaitGroup) {
	x = x + 1
	wg.Done()
}
func main() {
	var w sync.WaitGroup
	for i := 0; i < 1000; i++ {
		w.Add(1)		
		go increment(&w)
	}
	w.Wait()
	fmt.Println("final value of x:", x)
}
```
Here if we run this code the output will be different every time as we spawn 1000 `increment` Go-routines from line no. 15 of the program above. Each of these Go-routines run concurrently and the race condition occurs.
Now for preventing it 
```go
package main
import (
	"fmt"
	"sync"
	)
var x  = 0
func increment(wg *sync.WaitGroup, m *sync.Mutex) {
	m.Lock()
	x = x + 1
	m.Unlock()
	wg.Done()	
}
func main() {
	var w sync.WaitGroup
	var m sync.Mutex
	for i := 0; i < 1000; i++ {
		w.Add(1)		
		go increment(&w, &m)
	}
	w.Wait()
	fmt.Println("final value of x:", x)
}
```
we used `Lock()` then `Unlock()` around `x=x+1` so that only one routine can access it.
It is important to pass the address of the mutex. If the mutex is passed by value instead of passing the address, each Go-routine will have its own copy of the mutex and the race condition will still occur.
We can also solve this problem using channels by creating a bool like
```go
package main
import (
	"fmt"
	"sync"
	)
var x  = 0
func increment(wg *sync.WaitGroup, ch chan bool) {
	ch <- true
	x = x + 1
	<- ch
	wg.Done()	
}
func main() {
	var w sync.WaitGroup
	ch := make(chan bool, 1)
	for i := 0; i < 1000; i++ {
		w.Add(1)		
		go increment(&w, ch)
	}
	w.Wait()
	fmt.Println("final value of x", x)
}
```

So which one to use?
In general use channels when Go-routines need to communicate with each other and mutexes when only one Go-routine should access the critical section of code.
