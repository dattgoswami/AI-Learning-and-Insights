# Concurrency

Concurrency is a property of systems in which several independent tasks are executing in overlapping time intervals. Concurrency is not parallelism, though it enables parallelism. Concurrency is about dealing with lots of things at once, while parallelism is about doing lots of things at once.

Concurrency is crucial for designing and building software that is efficient, responsive, and performant. It helps us manage and structure interactions between multiple independently running tasks. In the world where computer systems are multi-core and networked, it's essential to make use of these capabilities. 

However, writing concurrent programs can be challenging because there are many potential pitfalls, such as race conditions, deadlocks, and resource starvation.

## Concurrency in Go

Go is designed to make it easier to write concurrent programs. It provides primitives, like goroutines and channels, that help manage the complexities of concurrent execution.

1. **Goroutines**: These are functions or methods that run concurrently with other functions or methods. Goroutines are extremely lightweight — they're only a few kB in stack size, and the stack can grow and shrink according to needs of the application. Creating a new goroutine is very cheap in terms of memory and CPU resources.

   Here is an example of a goroutine:

   ```go
   package main

   import (
       "fmt"
       "time"
   )

   func sayHello() {
       fmt.Println("Hello, world!")
   }

   func main() {
       go sayHello()
       time.Sleep(time.Second) // wait for sayHello goroutine to finish
   }
   ```
   In this example, `sayHello` function runs concurrently with `main` function.

2. **Channels**: These are the pipes that connect goroutines. You can send values into channels from one goroutine and receive those values into another goroutine.

   Here is an example of using channels:

   ```go
   package main

   import "fmt"

   func produce(p chan<- int) {
       for i := 0; i < 10; i++ {
           p <- i
       }
       close(p)
   }

   func main() {
       ch := make(chan int)
       go produce(ch) // concurrent producer
       
       for i := range ch {
           fmt.Println("Received:", i)
       }
   }
   ```
   In this example, we create a producer function that sends integers on a channel. The main function then starts this producer as a goroutine and receives and prints the integers from the channel.

In conclusion, Go's concurrency model, grounded in the concept of goroutines (independent tasks) and channels (communication between tasks), allows us to write concurrent code that is simpler to understand, more idiomatic, and less prone to typical concurrency bugs.

## Communicating Sequential Processes 
CSP is a formal language for describing patterns of interaction in concurrent systems. It was developed by Tony Hoare, a British computer scientist, and introduced in a 1978 paper. Go's concurrency model, particularly goroutines and channels, is heavily influenced by CSP.

In Go, goroutines act as independent entities or 'processes'. Goroutines are lightweight threads managed by the Go runtime, not by the underlying operating system. This makes them much cheaper than traditional OS threads, and you can easily create thousands or even millions of goroutines in a single program.

Channels in Go act as the connections between these processes. A channel provides a way for two goroutines to communicate with each other and synchronize their operations. Here is a simple example of two goroutines communicating over a channel:

```go
package main

import "fmt"

func worker(messages chan string) {
    messages <- "ping"
}

func main() {
    messages := make(chan string)

    go worker(messages)

    msg := <-messages
    fmt.Println(msg)  // Outputs: ping
}
```

In this example, we create a channel of strings called `messages`. We then start a goroutine with the `go worker(messages)` call. This runs the `worker` function in its own goroutine, which sends a string "ping" on the channel.

Back in the main goroutine, we receive the "ping" message from the channel using `<-messages` and print it. This demonstrates the basic idea of CSP: we have two independent processes (the main goroutine and the `worker` goroutine) communicating via a channel.

The CSP model helps in ensuring that only one goroutine has access to a particular piece of data at any given time. This idea is also called 'Do not communicate by sharing memory; instead, share memory by communicating.' It provides a way to tackle the complexity in multi-threaded programming by avoiding the explicit use of locks.

## Go's Concurrency Building Blocks

1. **Goroutines:** A Goroutine is a lightweight thread managed by the Go runtime. You can launch a Goroutine simply by prefixing a function call with the keyword `go`. Here is an example:

    ```go
    package main
    
    import (
        "fmt"
        "time"
    )
    
    func printNumbers() {
        for i := 1; i <= 5; i++ {
            time.Sleep(1 * time.Second)
            fmt.Println(i)
        }
    }
    
    func main() {
        go printNumbers()  // Launching the goroutine
        time.Sleep(6 * time.Second) // Waiting for the goroutine to finish
    }
    ```
    In this example, `printNumbers` is executed as a goroutine concurrently with the rest of the `main` function.

2. **Channels:** Channels are the pipes that connect concurrent goroutines. You can send values into channels from one goroutine and receive those values into another goroutine. Here is an example:

    ```go
    package main
    
    import "fmt"
    
    func main() {
        messages := make(chan string)
    
        go func() {
            messages <- "Hello, World!" // Sending a message into the channel
        }()
    
        msg := <-messages // Receiving the message from the channel
        fmt.Println(msg)
    }
    ```

3. **The sync package:** The `sync` package provides concurrency primitives, like `WaitGroup`, `Mutex`, `RWMutex`, etc. For example, `WaitGroup` is used to wait for a collection of Goroutines to finish executing:

    ```go
    package main
    
    import (
        "fmt"
        "sync"
        "time"
    )
    
    func worker(id int, wg *sync.WaitGroup) {
        defer wg.Done() // Notify that this worker is done
        fmt.Printf("Worker %d starting\n", id)
        time.Sleep(time.Second)
        fmt.Printf("Worker %d done\n", id)
    }
    
    func main() {
        var wg sync.WaitGroup
        for i := 1; i <= 5; i++ {
            wg.Add(1) // Add a count to the WaitGroup counter
            go worker(i, &wg)
        }
        wg.Wait() // Wait until the counter is zero (all workers are done)
    }
    ```

4. **The select statement:** The `select` statement lets a Goroutine wait on multiple channel operations. It's like a switch for channels:

    ```go
    package main
    
    import "fmt"
    import "time"
    
    func main() {
        tick := time.Tick(100 * time.Millisecond)
        boom := time.After(500 * time.Millisecond)
    
        for {
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

5. **GOMAXPROCS:** This sets the maximum number of CPUs that can be executing simultaneously. For instance, `runtime.GOMAXPROCS(4)` would set it to 4.

## Concurrency Patterns in Go:

1. **Pipelines:** A pipeline is a series of stages connected by channels, where each stage is a group of goroutines running the same function. In each stage, the goroutines receive values from upstream, perform some function, and send values downstream.

    Here is an example of a simple integer pipeline:

    ```go
    package main
    
    import "fmt"
    
    func gen(nums ...int) <-chan int {
        out := make(chan int)
        go func() {
            for _, n := range nums {
                out <- n
            }
            close(out)
        }()
        return out
    }
    
    func sq(in <-chan int) <-chan int {
        out := make(chan int)
        go func() {
            for n := range in {
                out <- n * n
            }
            close(out)
        }()
        return out
    }
    
    func main() {
        for n := range sq(gen(2, 3, 4, 5)) {
            fmt.Println(n)
        }
    }
    ```

2. **Fan-out/fan-in:** Fan-out is having multiple goroutines read from the same channel until the channel is closed. Fan-in is multiplexing multiple inputs into one channel.

    ```go
    package main
    
    import (
        "fmt"
        "sync"
        "time"
    )
    
    func producer(n int) <-chan int {
        out := make(chan int)
        go func() {
            defer close(out)
            for i := 0; i < n; i++ {
                out <- i
            }
        }()
        return out
    }
    
    func consumer(in <-chan int) <-chan int {
        out := make(chan int)
        go func() {
            defer close(out)
            for n := range in {
                out <- n * n
            }
        }()
        return out
    }
    
    func merge(cs ...<-chan int) <-chan int {
        var wg sync.WaitGroup
        out := make(chan int)
    
        output := func(c <-chan int) {
            for n := range c {
                out <- n
            }
            wg.Done()
        }
    
        wg.Add(len(cs))
        for _, c := range cs {
            go output(c)
        }
    
        go func() {
            wg.Wait()
            close(out)
        }()
        return out
    }
    
    func main() {
        in := producer(10)
    
        // Fanning out
        c1 := consumer(in)
        c2 := consumer(in)
    
        // Fanning in
        for result := range merge(c1, c2) {
            fmt.Printf("%d ", result)
            time.Sleep(50 * time.Millisecond)
        }
        fmt.Println()
    }
    ```
3. **Context package:** The context package in Go is used for carrying deadlines, cancellation signals, and request-scoped values across API boundaries and between processes. Here is an example of using a context to implement a cancellation:

    ```go
    package main
    
    import (
        "context"
        "fmt"
        "time"
    )
    
    func main() {
        // Create a new context
        ctx, cancel := context.WithCancel(context.Background())
        // Create a new channel
        ch := make(chan string)
    
        // Launch a new goroutine
        go func(ctx context.Context, ch chan string) {
            for {
                select {
                case <-ctx.Done():
                    fmt.Println("Done!")
                    return
                default:
                    ch <- "work"
                    time.Sleep(500 * time.Millisecond)
                }
            }
        }(ctx, ch)
    
        // Print five "work"s and then cancel
        for i := 0; i < 5; i++ {
            fmt.Println(<-ch)
        }
        cancel()
    
        // Wait to see the Done! message
        time.Sleep(1 * time.Second)
    }
    ```

In Go, the `context` package is often used to manage and carry deadlines, cancellation signals, and other request-scoped values across API boundaries. This is especially relevant in programs dealing with networking, such as web servers or clients, where individual requests from users or to other services need to carry associated data and may need to be canceled partway through. 

3.1 **Cancellation signals:** The `context` package allows one goroutine to cancel others by sending a cancellation signal. Here's a simple example:

   ```go
   package main
   
   import (
       "context"
       "fmt"
       "time"
   )
   
   func doWork(ctx context.Context) {
       for {
           select {
           case <-time.After(1 * time.Second):
               fmt.Println("Doing some work")
           case <-ctx.Done():
               fmt.Println("Work cancelled:", ctx.Err())
               return
           }
       }
   }
   
   func main() {
       ctx, cancel := context.WithCancel(context.Background())
       go doWork(ctx)
   
       time.Sleep(5 * time.Second) // Let it work for a while
       cancel() // Now cancel the work
       time.Sleep(1 * time.Second) // Give it a moment to print cancellation message
   }
   ```
   In this example, a worker goroutine runs in a loop, periodically printing a message. When the main function cancels the context (using the `cancel` function that `context.WithCancel` returned), the worker sees that signal and returns, stopping its work.

3.2 **Request-scoped values:** Contexts can also carry arbitrary key-value pairs of data, allowing data associated with a request to be passed across API boundaries and between different parts of your program. Here's a simple example:

   ```go
   package main
   
   import (
       "context"
       "fmt"
   )
   
   type keyType struct{ name string }
   
   func doWork(ctx context.Context, key keyType) {
       if value := ctx.Value(key); value != nil {
           fmt.Println("found value:", value)
           return
       }
       fmt.Println("key not found")
   }
   
   func main() {
       key := keyType{"language"}
       ctx := context.WithValue(context.Background(), key, "Go")
   
       doWork(ctx, key)
       doWork(context.Background(), key)
   }
   ```
   In this example, the `doWork` function takes a context and a key, and attempts to print the value associated with that key in the context. In the `main` function, a context is created with a value for that key ("Go"), so the first call to `doWork` finds and prints that value, while the second call (with a 'background' context that has no values) does not.
   
Using the `context` package effectively can help your Go programs handle cancellation cleanly and manage request-scoped data appropriately. It's a powerful tool for managing the lifecycle of requests in concurrent Go programs.

## Concurrency at Scale

As a Go program scales, new challenges related to concurrency emerge. Here are some concepts and strategies to handle these challenges:

1. **Error Propagation:** In a concurrent system, it's important to ensure that errors are properly communicated between goroutines. Channels are one way to do this:

    ```go
    package main
    
    import (
        "fmt"
        "time"
    )
    
    func work() error {
        time.Sleep(time.Second)
        return fmt.Errorf("an error occurred")
    }
    
    func main() {
        errc := make(chan error)
        go func() { errc <- work() }()
    
        select {
        case err := <-errc:
            if err != nil {
                fmt.Println("An error happened:", err)
            }
        case <-time.After(2 * time.Second):
            fmt.Println("Work finished without errors")
        }
    }
    ```

2. **Timeouts and Cancellation:** The `context` package in Go provides mechanisms to set timeouts and cancellation signals. This is vital for preventing operations from taking too long and potentially hanging your system:

    ```go
    package main
    
    import (
        "context"
        "fmt"
        "time"
    )
    
    func work(ctx context.Context) error {
        for {
            select {
            case <-time.After(2 * time.Second):
                fmt.Println("Work finished")
                return nil
            case <-ctx.Done():
                fmt.Println("Canceled")
                return ctx.Err()
            }
        }
    }
    
    func main() {
        ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
        defer cancel() // cancel when we are finished consuming integers
        
        fmt.Println(work(ctx))
    }
    ```

3. **Rate Limiting:** This is an important technique to limit the number of requests a system can handle concurrently. A buffered channel can act as a simple rate limiter:

    ```go
    package main
    
    import (
        "fmt"
        "time"
    )
    
    func main() {
        requests := make(chan int, 5) // a buffered channel acts as a rate limiter
        for i := 1; i <= 5; i++ {
            requests <- i
        }
        close(requests)
    
        limiter := time.Tick(200 * time.Millisecond) // a rate of 200ms
        for req := range requests {
            <-limiter
            fmt.Println("request", req, time.Now())
        }
    }
    ```

## Goroutines and the Go Runtime

1. **Dynamic stack sizes:** Goroutines are cheaper than threads because they have smaller, dynamic stack sizes. While a typical thread might have a stack size on the order of a few megabytes, a goroutine starts with a stack of only a few kilobytes. As goroutines run and their stack needs grow, the Go runtime will dynamically allocate more memory for the stack, up to a limit.

2. **Work stealing:** The Go scheduler uses a work-stealing algorithm, which helps in achieving efficient use of system resources. If a processor finds its local run queue of goroutines empty (i.e., it has no work), it tries to steal work from other processors' local run queues.

This is a more advanced topic and isn't directly visible to Go programmers. But it's part of what makes Go's concurrency model powerful and efficient. While I can't provide a code example of work stealing (as it's part of the internals of the Go runtime, not something directly controlled by Go code), you can think of it as a way that Go manages to efficiently distribute work among available CPU cores.