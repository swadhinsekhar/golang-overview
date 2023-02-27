# golang-overview

Basics of golang concepts and question

---

## What is interface in golang and example implementation of interface?

An interface type is defined as a set of method signatures. A value of interface type can hold any value that implements those methods.

```go
package main

import (
    "fmt"
)

//set of method signatures
type Mathops interface {
    Add(a, b int32) int32
    Sub(a, b int64) int64
}

type Adder struct {
    id int32
}

//go:noinline
func (adder Adder) Add(a, b int32) int32 { return a + b }
//go:noinline
func (adder Adder) Sub(a, b int64) int64 { return a - b }

func main() {
    m := Mathops(Adder{
            id: 6754
        })

    // This call just makes sure that the interface is actually used.
    // Without this call, the linker would see that the interface defined above
    // is in fact never used, and thus would optimize it out of the final executable.
    fmt.Println(m.Add(10, 32))
    fmt.Println(m.Sub(10, 8))
}
```

---

---

## When to use os.Exit() and panic()?

Now, os.Exit and panic are quite different. panic is used when the program, or its part, has reached an unrecoverable state.

When panic is called, including implicitly for run-time errors such as indexing a slice out of bounds or failing a type assertion, it immediately stops execution of the current function and begins unwinding the stack of the goroutine, running any deferred functions along the way. If that unwinding reaches the top of the goroutine's stack, the program dies.

os.Exit is used when you need to abort the program immediately, with no possibility of recovery or running a deferred clean-up statement, and also return an error code (that other programs can use to report what happened). This is useful in tests, when you already know that after this one test fails, the other will fail as well, so you might as well just exit now. This can also be used when your program has done everything it needed to do, and now just needs to exit, i.e. after printing a help message.

Most of the time you won't use panic (you should return an error instead), and you almost never need os.Exit outside of some cases in tests and for quick program termination.

---

---

## Explain Type Assertions in Go

#### Short answer In one line:

- x.(T) asserts that x is not nil and that the value stored in x is of type T.

#### Why would I use them:

- to check x is nil
- to check what is the dynamic type held by interface x
- to extract the dynamic type from x

#### What exactly they return:

- t := x.(T) => t is of type T; if x is nil, it panics.
- t,ok := x.(T) => if x is nil or not of type T => ok is false otherwise ok is true and t is of type T.

---

## What Is a Race Condition?

Race conditions are the outcomes of two different concurrent contexts reading and writing to the same shared data at the same time,
resulting in an unexpected output. In Golang two concurrent goroutines that access the same variable concurrently will produce
a data race in the program.

### How to Avoid Race Conditions in Golang?

- The `sync.Mutex` package provides a mechanism to guard a block of code, making it concurrency-safe,
  meaning write operations within that block will be safe. The primitives that the sync package provides
  allow you to write concurrent code using memory access synchronization to avoid data race conditions.

- This mechanism constitutes using the `Lock` and `Unlock`, methods from the package.

- The Lock method will establish that the goroutine who calls this method has just acquired the lock
  and no other goroutines can use the lock until it is released.

- The Unlock method releases the lock so that other goroutines can use it.

- When one goroutine is using the lock and another one tries to acquire the lock too,
  the goroutine will block until the other goroutine releases the lock.

### Example of race Condition

```go
// race condition scenario
func TestDataRaceCondition(t *testing.T) {
    var counter int32
    for i := 0; i < 10; i++ {
        go func (i int) {
            counter += int32(i)
        }(i)
    }
}
```

NOTE: instead of `sync.Mutex` use `"sync/atomic"` package to avoid race condition

```go
func TestDataRaceCondition(t *testing.T) {
    var counter int32
    for i := 0; i < 10; i++ {
        go func (i int) {
            atomic.AddInt32(&counter, int32(i))
        }(i)
    }
}
```

---

## WAP where two goroutine will print print 1 to 100, one goroutine printing odd numbers and other one even numbers.

```go
package main                                                                                                                                        [30/274]

import (
        "fmt"
        "sync"
)

type sysout struct {
    oddChan chan int
    evenChan chan int
    exitOdd chan bool
}

func oddRoutine(wg *sync.WaitGroup, s *sysout) {
    done := false
    for !done {
        select {
        case oddData := <- s.oddChan:
            fmt.Println(oddData)
            s.evenChan <- oddData + 1
        case <- s.exitOdd:
            done = true
        }
    }
    wg.Done()
}

func evenRoutine(wg *sync.WaitGroup, s *sysout) {
    done := false
    for !done {
        select {
        case evenData := <- s.evenChan:
            fmt.Println(evenData)
            if (evenData == 100) {
                done = true
                s.exitOdd <- true
            }
            s.oddChan <- evenData + 1
        }
    }
    wg.Done()
}

func main() {
    fmt.Println("2 * goroutine print 1 to 100 synchronizingly ")

    s := &sysout {
        oddChan: make(chan int, 100),
        evenChan: make(chan int, 100),
        exitOdd: make(chan bool, 1),
    }
    wg := &sync.WaitGroup{}

    wg.Add(2)

    go oddRoutine(wg, s)
    go evenRoutine(wg, s)

    //printing odd numbers
    s.oddChan <- int(1)

    //closing channels or freeing on memories
    close(s.oddChan)
    close(s.evenChan)
    close(s.exitOdd)

    //waiting for goroutine to Done()
    wg.Wait()
}
```

---
