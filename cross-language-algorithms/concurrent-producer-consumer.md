---
created: 2026-02-06
---

# Concurrent Producer-Consumer Pipeline: A Cross-Language Study

_February 6, 2026_

> Part of a series: [overview](three-algorithms-overview.md) | [indices above mean](indices-above-mean.md) | [expression trees](expression-tree-evaluation.md) | **this document**

This document explores a single concurrent programming pattern implemented across dozens of programming languages, organized by language family. Where the companion documents showcased [array transformation](indices-above-mean.md) and [recursive data processing](expression-tree-evaluation.md), this algorithm — a concurrent producer-consumer pipeline — rewards languages with lightweight concurrency primitives, message passing, and safe shared-state abstractions.

## Table of Contents

1. [The Algorithm](#the-algorithm)
2. [The BEAM Family](#the-beam-family)
   - [Erlang](#erlang)
   - [Elixir](#elixir)
   - [Gleam](#gleam)
   - [BEAM Family Comparison](#beam-family-comparison)
3. [Modern Systems Languages](#modern-systems-languages)
   - [Go](#go)
   - [Rust](#rust)
   - [Zig](#zig)
   - [Systems Language Comparison](#systems-language-comparison)
4. [The ML Family](#the-ml-family)
   - [Haskell](#haskell)
   - [OCaml](#ocaml)
   - [F#](#f)
   - [ML Family Comparison](#ml-family-comparison)
5. [The JVM Family](#the-jvm-family)
   - [Java](#java)
   - [Scala](#scala)
   - [Kotlin](#kotlin)
   - [Clojure](#clojure)
   - [JVM Family Comparison](#jvm-family-comparison)
6. [The C Family](#the-c-family)
   - [C (pthreads)](#c-pthreads)
   - [C++ (std::thread)](#c-stdthread)
   - [C Family Comparison](#c-family-comparison)
7. [Scripting Languages](#scripting-languages)
   - [Python](#python)
   - [Ruby](#ruby)
   - [JavaScript / Node.js](#javascript--nodejs)
   - [TypeScript (Deno)](#typescript-deno)
   - [Perl](#perl)
   - [Lua](#lua)
   - [Scripting Language Comparison](#scripting-language-comparison)
8. [The Apple Ecosystem](#the-apple-ecosystem)
   - [Swift](#swift)
9. [The Wirth Family](#the-wirth-family)
   - [Ada](#ada)
10. [Newer Functional Languages](#newer-functional-languages)
    - [Roc](#roc)
    - [Unison](#unison)
11. [Other Notable Languages](#other-notable-languages)
    - [Julia](#julia)
    - [Forth](#forth)
    - [Prolog](#prolog)
12. [The APL Family](#the-apl-family)
13. [Cross-Family Comparison](#cross-family-comparison)
14. [Conclusion](#conclusion)

---

## The Algorithm

The task we will implement is a concurrent producer-consumer pipeline with the following structure:

Several **producers** each generate a sequence of items (integers, for simplicity) and place them into a shared **bounded buffer** (also called a bounded channel or bounded queue). Several **consumers** pull items from this buffer, process them (we will square each number, as a stand-in for real work), and contribute the results to a shared collection. When all producers have finished, consumers drain the remaining items and shut down. The final result is the collected outputs, sorted.

Concretely, given P=3 producers each generating items `[1..5]`, the pipeline should produce the squares of all 15 numbers: `[1, 4, 9, 16, 25, 1, 4, 9, 16, 25, 1, 4, 9, 16, 25]`, collected and sorted to `[1, 1, 1, 4, 4, 4, 9, 9, 9, 16, 16, 16, 25, 25, 25]`.

The algorithm decomposes into these concurrent coordination problems:

1. **Spawning**: Create multiple producer and consumer threads/processes/goroutines.
2. **Bounded buffering**: The buffer has a maximum capacity; producers must wait (block or backpressure) when it is full.
3. **Safe sharing**: Multiple producers write to the buffer concurrently; multiple consumers read from it concurrently. The data structure must handle this safely.
4. **Shutdown signaling**: Producers must signal when they are done. Consumers must know when to stop — not when the buffer is momentarily empty, but when all producers are finished *and* the buffer is empty.
5. **Result collection**: Consumers write results to a shared collection, which must also be concurrent-safe.

Different languages have radically different primitives for these tasks. Actor-model languages (Erlang, Elixir) use message passing between lightweight processes and never share memory. Channel-based languages (Go) use typed channels for communication. Systems languages (Rust, C) use mutexes, condition variables, and atomic operations. Functional languages (Haskell) offer Software Transactional Memory or async abstractions. Scripting languages are often constrained by global interpreter locks.

This problem reveals how languages think about coordination, safety, and the fundamental question of concurrent programming: how do independent units of computation communicate?

---

## The BEAM Family

The BEAM virtual machine was designed from the ground up for concurrency. Erlang processes are extraordinarily lightweight (a few hundred bytes of overhead), and message passing between them is the only coordination mechanism — there is no shared mutable state. This is the actor model in its purest form.

### Erlang

Erlang handles the producer-consumer pipeline with spawned processes, message passing, and a coordinator that manages the buffer logic.

```erlang
-module(pipeline).
-export([run/0]).

producer(Buffer, Items) ->
    lists:foreach(fun(Item) ->
        Buffer ! {produce, Item}
    end, Items),
    Buffer ! {producer_done}.

consumer(Buffer, Collector) ->
    Buffer ! {consume, self()},
    receive
        {item, Item} ->
            Result = Item * Item,
            Collector ! {result, Result},
            consumer(Buffer, Collector);
        done ->
            ok
    end.

%% Buffer process: manages a bounded queue with backpressure
buffer(Queue, Waiting, ProducersLeft, MaxSize) ->
    receive
        {produce, Item} when length(Queue) < MaxSize ->
            %% Space available: enqueue and serve any waiting consumer
            NewQueue = Queue ++ [Item],
            case Waiting of
                [Consumer | Rest] ->
                    [H | T] = NewQueue,
                    Consumer ! {item, H},
                    buffer(T, Rest, ProducersLeft, MaxSize);
                [] ->
                    buffer(NewQueue, [], ProducersLeft, MaxSize)
            end;
        {consume, Consumer} when length(Queue) > 0 ->
            [H | T] = Queue,
            Consumer ! {item, H},
            buffer(T, Waiting, ProducersLeft, MaxSize);
        {consume, Consumer} when ProducersLeft == 0 ->
            Consumer ! done;
        {consume, Consumer} ->
            buffer(Queue, Waiting ++ [Consumer], ProducersLeft, MaxSize);
        {producer_done} ->
            NewLeft = ProducersLeft - 1,
            case {NewLeft, Queue, Waiting} of
                {0, [], WaitList} ->
                    lists:foreach(fun(C) -> C ! done end, WaitList);
                _ ->
                    buffer(Queue, Waiting, NewLeft, MaxSize)
            end
    end.

collector(Results, ConsumersLeft) ->
    receive
        {result, R} ->
            collector([R | Results], ConsumersLeft);
        consumer_done ->
            case ConsumersLeft - 1 of
                0 -> lists:sort(Results);
                N -> collector(Results, N)
            end
    end.

run() ->
    NumProducers = 3,
    NumConsumers = 2,
    Items = lists:seq(1, 5),
    BufferSize = 4,
    
    CollectorPid = spawn(fun() -> collector([], NumConsumers) end),
    BufferPid = spawn(fun() -> buffer([], [], NumProducers, BufferSize) end),
    
    %% Spawn producers
    lists:foreach(fun(_) ->
        spawn(fun() -> producer(BufferPid, Items) end)
    end, lists:seq(1, NumProducers)),
    
    %% Spawn consumers
    lists:foreach(fun(_) ->
        spawn(fun() -> consumer(BufferPid, CollectorPid) end)
    end, lists:seq(1, NumConsumers)).
```

Every component — producers, consumers, the buffer, the collector — is its own process communicating via messages. The buffer process maintains its state (the queue, waiting consumers, producers remaining) as function arguments to a tail-recursive receive loop. There are no locks, no shared variables, and no possibility of data races. Each process owns its data exclusively.

The buffer implements backpressure by queuing consumer requests when the buffer is empty and queuing producer items when consumers are waiting. Shutdown propagates naturally: when all producers signal `producer_done` and the queue is empty, waiting consumers receive `done`.

This is the actor model working exactly as it was designed to. The code is longer than some alternatives, but every concurrent interaction is explicit in the message protocol. There are no hidden synchronization points, no lock ordering concerns, and no possibility of deadlock from competing mutex acquisitions.

### Elixir

Elixir provides the same actor model with more ergonomic syntax and standard library abstractions.

```elixir
defmodule Pipeline do
  def run do
    num_producers = 3
    num_consumers = 2
    items = Enum.to_list(1..5)
    
    # GenStage or simple Task-based approach
    results = 
      items
      |> List.duplicate(num_producers)
      |> List.flatten()
      |> Task.async_stream(fn item -> item * item end,
           max_concurrency: num_consumers,
           ordered: false)
      |> Enum.map(fn {:ok, result} -> result end)
      |> Enum.sort()
    
    results
  end
end
```

Elixir's `Task.async_stream` abstracts the entire producer-consumer pattern into a single function call. It takes an enumerable, applies a function concurrently with bounded parallelism (`max_concurrency`), and returns results as a stream. The bounded buffer, shutdown signaling, and result collection are all handled internally.

For a more explicit version that shows the actor model directly:

```elixir
defmodule Pipeline do
  def producer(buffer, items) do
    Enum.each(items, fn item ->
      send(buffer, {:produce, item})
    end)
    send(buffer, :producer_done)
  end

  def consumer(buffer, collector) do
    send(buffer, {:consume, self()})
    receive do
      {:item, item} ->
        send(collector, {:result, item * item})
        consumer(buffer, collector)
      :done ->
        send(collector, :consumer_done)
    end
  end

  def run do
    collector = spawn(fn -> collect([], 2) end)
    buffer = spawn(fn -> buffer_loop([], [], 3, 4) end)
    
    for _ <- 1..3, do: spawn(fn -> producer(buffer, 1..5) end)
    for _ <- 1..2, do: spawn(fn -> consumer(buffer, collector) end)
  end
  
  # ... buffer_loop and collect similar to Erlang version
end
```

Elixir's pipe operator (`|>`) and comprehensions make the high-level version remarkably clean. The explicit version mirrors Erlang's structure with lighter syntax. In production Elixir, you would likely use GenStage or Broadway — dedicated libraries for concurrent pipeline processing built on OTP supervision trees.

### Gleam

Gleam brings static types to the BEAM, using typed actors (subjects) for message passing.

```gleam
import gleam/erlang/process.{type Subject}
import gleam/otp/task
import gleam/list

pub type BufferMsg {
  Produce(Int)
  Consume(Subject(ConsumerMsg))
  ProducerDone
}

pub type ConsumerMsg {
  Item(Int)
  Done
}

pub fn run() -> List(Int) {
  let items = [1, 2, 3, 4, 5]
  
  // Simple approach using tasks
  let results =
    list.range(1, 3)
    |> list.flat_map(fn(_) { items })
    |> list.map(fn(item) {
      task.async(fn() { item * item })
    })
    |> list.map(task.await_forever)
    |> list.sort(int.compare)
  
  results
}
```

Gleam's typed message channels (`Subject(ConsumerMsg)`) ensure at compile time that only valid messages are sent to each actor. This adds ML-family type safety to the BEAM's actor model. The message types `BufferMsg` and `ConsumerMsg` are algebraic data types, and the compiler ensures exhaustive handling.

### BEAM Family Comparison

The BEAM family treats concurrency as its native element. Processes are cheap (millions can run simultaneously), message passing is the only coordination mechanism, and there is no shared mutable state to cause data races. The buffer process in Erlang manages all synchronization through its message-handling loop — there are no mutexes, no condition variables, and no atomics.

The progression from Erlang to Elixir to Gleam adds layers of abstraction and safety: Elixir adds ergonomic syntax and high-level combinators; Gleam adds compile-time message type checking. But the underlying model — lightweight processes, message passing, no shared state — is the same throughout.

This is the family's home turf. Erlang was designed for telephone switches that handle millions of concurrent connections. The producer-consumer pipeline is a toy version of problems Erlang solves in production at WhatsApp, Discord, and telecom infrastructure worldwide.

---

## Modern Systems Languages

### Go

Go's goroutines and channels were designed explicitly for concurrent communication patterns like producer-consumer pipelines.

```go
package main

import (
    "fmt"
    "sort"
    "sync"
)

func producer(id int, items []int, ch chan<- int, wg *sync.WaitGroup) {
    defer wg.Done()
    for _, item := range items {
        ch <- item  // blocks if buffer is full
    }
}

func consumer(ch <-chan int, results chan<- int, wg *sync.WaitGroup) {
    defer wg.Done()
    for item := range ch {  // exits when ch is closed
        results <- item * item
    }
}

func main() {
    items := []int{1, 2, 3, 4, 5}
    bufferSize := 4
    numProducers := 3
    numConsumers := 2

    // Bounded channel acts as the buffer
    ch := make(chan int, bufferSize)
    results := make(chan int, numProducers*len(items))

    // Spawn producers
    var prodWg sync.WaitGroup
    for i := 0; i < numProducers; i++ {
        prodWg.Add(1)
        go producer(i, items, ch, &prodWg)
    }

    // Close channel when all producers finish
    go func() {
        prodWg.Wait()
        close(ch)
    }()

    // Spawn consumers
    var consWg sync.WaitGroup
    for i := 0; i < numConsumers; i++ {
        consWg.Add(1)
        go consumer(ch, results, &consWg)
    }

    // Close results when all consumers finish
    go func() {
        consWg.Wait()
        close(results)
    }()

    // Collect results
    var collected []int
    for r := range results {
        collected = append(collected, r)
    }
    sort.Ints(collected)
    fmt.Println(collected)
}
```

Go's solution is clean and idiomatic. The bounded channel `make(chan int, bufferSize)` is the buffer, with built-in blocking on full/empty. Goroutines are lightweight (a few KB each). The `range ch` loop in consumers automatically exits when the channel is closed. `sync.WaitGroup` coordinates shutdown: a goroutine waits for all producers to finish, then closes the channel, which causes consumers' `range` loops to terminate.

The directional channel types (`chan<- int` for send-only, `<-chan int` for receive-only) provide compile-time enforcement that producers only send and consumers only receive. This is not as strong as Rust's ownership guarantees, but it catches common mistakes.

Go's concurrency model — goroutines communicating over channels, with the motto "don't communicate by sharing memory; share memory by communicating" — makes this pattern almost trivial. The bounded channel handles backpressure, the close-on-done pattern handles shutdown, and WaitGroups coordinate completion. This is Go at its best.

### Rust

Rust uses ownership and the type system to provide "fearless concurrency" — data races are impossible at compile time.

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let items = vec![1, 2, 3, 4, 5];
    let num_producers = 3;
    let num_consumers = 2;

    // Bounded channel (buffer size 4)
    let (tx, rx) = mpsc::sync_channel::<i32>(4);
    let (result_tx, result_rx) = mpsc::channel::<i32>();

    // Spawn producers
    let mut producer_handles = vec![];
    for _ in 0..num_producers {
        let tx = tx.clone();
        let items = items.clone();
        producer_handles.push(thread::spawn(move || {
            for item in items {
                tx.send(item).unwrap();  // blocks if buffer full
            }
            // tx is dropped here, decrementing sender count
        }));
    }
    drop(tx);  // Drop original sender so channel closes when producers finish

    // Spawn consumers
    let rx = std::sync::Arc::new(std::sync::Mutex::new(rx));
    let mut consumer_handles = vec![];
    for _ in 0..num_consumers {
        let rx = rx.clone();
        let result_tx = result_tx.clone();
        consumer_handles.push(thread::spawn(move || {
            loop {
                let item = {
                    let rx = rx.lock().unwrap();
                    rx.recv()
                };
                match item {
                    Ok(item) => result_tx.send(item * item).unwrap(),
                    Err(_) => break,  // Channel closed
                }
            }
        }));
    }
    drop(result_tx);

    // Collect results
    let mut results: Vec<i32> = result_rx.iter().collect();
    results.sort();
    println!("{:?}", results);
    
    for h in producer_handles { h.join().unwrap(); }
    for h in consumer_handles { h.join().unwrap(); }
}
```

Rust's ownership system makes several things explicit that Go hides. The `tx.clone()` is necessary because each producer needs its own sender — Rust's ownership rules prevent sharing a sender without cloning. The `drop(tx)` after spawning producers ensures the channel closes when all clones are dropped (each producer's `tx` is moved into the closure and dropped when the thread finishes, but the original `tx` must also be dropped). The `Arc<Mutex<rx>>` wraps the receiver for sharing among consumers, since `mpsc::Receiver` is not `Sync`.

The `move` keyword transfers ownership of captured variables into each thread's closure, and the compiler verifies that no thread accesses data it does not own. A data race is literally a compilation error. This is more verbose than Go, but the compiler guarantees are stronger. In Go, misusing channels or WaitGroups can produce runtime panics; in Rust, misusing ownership is a compile-time error.

For a cleaner solution, Rust's `crossbeam` crate provides multi-consumer channels:

```rust
use crossbeam::channel;
use std::thread;

fn main() {
    let items = vec![1, 2, 3, 4, 5];
    let (tx, rx) = channel::bounded::<i32>(4);
    let (result_tx, result_rx) = channel::unbounded::<i32>();

    // Spawn producers
    for _ in 0..3 {
        let tx = tx.clone();
        let items = items.clone();
        thread::spawn(move || {
            for item in items { tx.send(item).unwrap(); }
        });
    }
    drop(tx);

    // Spawn consumers — rx can be cloned directly
    for _ in 0..2 {
        let rx = rx.clone();
        let result_tx = result_tx.clone();
        thread::spawn(move || {
            for item in rx { result_tx.send(item * item).unwrap(); }
        });
    }
    drop(result_tx);

    let mut results: Vec<_> = result_rx.iter().collect();
    results.sort();
    println!("{:?}", results);
}
```

With `crossbeam`, the Rust version approaches Go's cleanliness while maintaining compile-time safety.

### Zig

Zig provides low-level concurrency primitives with explicit thread management.

```zig
const std = @import("std");
const Thread = std.Thread;
const Mutex = Thread.Mutex;
const Condition = Thread.Condition;

const BoundedQueue = struct {
    buffer: [64]i32 = undefined,
    head: usize = 0,
    tail: usize = 0,
    count: usize = 0,
    capacity: usize = 4,
    mutex: Mutex = .{},
    not_full: Condition = .{},
    not_empty: Condition = .{},
    closed: bool = false,

    fn push(self: *BoundedQueue, item: i32) void {
        self.mutex.lock();
        defer self.mutex.unlock();
        while (self.count == self.capacity) {
            self.not_full.wait(&self.mutex);
        }
        self.buffer[self.tail] = item;
        self.tail = (self.tail + 1) % self.capacity;
        self.count += 1;
        self.not_empty.signal();
    }

    fn pop(self: *BoundedQueue) ?i32 {
        self.mutex.lock();
        defer self.mutex.unlock();
        while (self.count == 0 and !self.closed) {
            self.not_empty.wait(&self.mutex);
        }
        if (self.count == 0) return null;
        const item = self.buffer[self.head];
        self.head = (self.head + 1) % self.capacity;
        self.count -= 1;
        self.not_full.signal();
        return item;
    }

    fn close(self: *BoundedQueue) void {
        self.mutex.lock();
        defer self.mutex.unlock();
        self.closed = true;
        self.not_empty.broadcast();
    }
};
```

Zig requires building the bounded queue from primitives: a mutex, two condition variables (not_full and not_empty), and manual buffer management. This is essentially the same approach as C with pthreads, but with Zig's `defer` for cleanup and a slightly more structured syntax. The queue is a circular buffer protected by explicit synchronization.

### Systems Language Comparison

The systems languages show a clear progression in abstraction level for concurrency. Zig and C require building bounded queues from mutexes and condition variables — the most explicit but most error-prone approach. Rust's standard library provides channels, and the ownership system prevents data races at compile time. Go provides channels as a language-level feature with goroutines that make spawning concurrent work trivial.

For this specific problem, Go is the most ergonomic: bounded channels, goroutines, and close semantics are all built into the language. Rust is safer: the compiler prevents data races, though the ergonomics are slightly worse with `Arc<Mutex<>>` wrappers. Zig is the most explicit: you see every mutex lock, every condition variable wait, every buffer index operation.

---

## The ML Family

### Haskell

Haskell offers Software Transactional Memory (STM), a fundamentally different concurrency primitive that composes transactional operations.

```haskell
import Control.Concurrent
import Control.Concurrent.STM
import Control.Monad (forM_, replicateM_)
import Data.List (sort)

producer :: TBQueue Int -> [Int] -> IO ()
producer queue items =
    forM_ items $ \item ->
        atomically $ writeTBQueue queue item

consumer :: TBQueue Int -> TVar [Int] -> MVar () -> IO ()
consumer queue results done = loop
  where
    loop = do
        mItem <- atomically $ tryReadTBQueue queue
        case mItem of
            Just item -> do
                atomically $ modifyTVar' results (item * item :)
                loop
            Nothing -> putMVar done ()

main :: IO ()
main = do
    let items = [1..5]
        numProducers = 3
        numConsumers = 2
    
    queue   <- atomically $ newTBQueue 4   -- bounded queue, capacity 4
    results <- atomically $ newTVar []
    
    -- Spawn producers
    prodDone <- newMVar ()
    forM_ [1..numProducers] $ \_ ->
        forkIO $ producer queue items
    
    -- Wait for producers, then signal consumers
    threadDelay 100000  -- simplified; real code uses async library
    
    -- Spawn consumers
    dones <- replicateM numConsumers newEmptyMVar
    forM_ dones $ \done ->
        forkIO $ consumer queue results done
    
    -- Wait for all consumers
    forM_ dones takeMVar
    
    -- Collect results
    finalResults <- atomically $ readTVar results
    print (sort finalResults)
```

The key abstraction is `TBQueue` — a bounded transactional queue from the STM library. The `atomically` block ensures that reads and writes to transactional variables are atomic and composable. Unlike mutexes, STM transactions compose: you can combine multiple STM operations into a single atomic block, and the runtime handles retries and conflict resolution.

A more robust version would use the `async` library:

```haskell
import Control.Concurrent.Async (mapConcurrently_, forConcurrently_)

pipeline :: IO [Int]
pipeline = do
    queue   <- atomically $ newTBQueue 4
    results <- atomically $ newTVar []
    
    -- Run producers concurrently, then close
    let produce = forConcurrently_ [1..3] $ \_ ->
            forM_ [1..5] $ \item ->
                atomically $ writeTBQueue queue item
    
    let consume = forConcurrently_ [1..2] $ \_ ->
            let loop = do
                    mItem <- atomically $ tryReadTBQueue queue
                    case mItem of
                        Just item -> do
                            atomically $ modifyTVar' results (item * item :)
                            loop
                        Nothing -> return ()
            in loop
    
    -- Run producers and consumers
    produce
    consume
    sort <$> atomically (readTVar results)
```

STM is Haskell's distinctive contribution to concurrency. Where channels are a specific coordination mechanism, STM is a general transactional framework that can express any coordination pattern without explicit locks. The composability guarantee — any combination of STM operations can be wrapped in `atomically` and will execute correctly — is unique to Haskell's approach.

### OCaml

OCaml 5 introduced domains (OS threads with parallelism) and effects for concurrent programming.

```ocaml
(* Using Eio, OCaml 5's effect-based concurrency library *)
open Eio

let producer stream items =
  List.iter (fun item ->
    Stream.add stream item
  ) items

let consumer stream results =
  let rec loop () =
    match Stream.take_nonblocking stream with
    | Some item ->
        Mutex.lock results.mutex;
        results.data <- (item * item) :: results.data;
        Mutex.unlock results.mutex;
        loop ()
    | None -> ()
  in loop ()

let run () =
  Eio_main.run @@ fun env ->
    let stream = Stream.create 4 in  (* bounded, capacity 4 *)
    let results = { data = []; mutex = Mutex.create () } in
    
    Fiber.both
      (fun () ->
        Fiber.all (List.init 3 (fun _ ->
          fun () -> producer stream [1;2;3;4;5]
        )))
      (fun () ->
        Fiber.all (List.init 2 (fun _ ->
          fun () -> consumer stream results
        )));
    
    List.sort compare results.data
```

OCaml 5's Eio library uses structured concurrency, where `Fiber.both` ensures that both the producer group and consumer group complete before proceeding. The `Stream` module provides bounded buffering. OCaml's approach to concurrency is newer and still evolving, but the effect-handler foundation is theoretically elegant.

### F#

F# provides `MailboxProcessor` (inspired by Erlang) and `Async` workflows.

```fsharp
open System.Collections.Concurrent

let run () = async {
    let queue = new BlockingCollection<int>(boundedCapacity = 4)
    let results = ConcurrentBag<int>()
    
    // Producers
    let producers =
        [1..3] |> List.map (fun _ ->
            async {
                for item in 1..5 do
                    queue.Add(item)  // blocks if full
            })
    
    // Run producers, then signal completion
    let! _ = Async.Parallel(producers |> Array.ofList)
    queue.CompleteAdding()
    
    // Consumers
    let consumers =
        [1..2] |> List.map (fun _ ->
            async {
                for item in queue.GetConsumingEnumerable() do
                    results.Add(item * item)
            })
    
    let! _ = Async.Parallel(consumers |> Array.ofList)
    return results |> Seq.toList |> List.sort
}
```

F# leverages .NET's `BlockingCollection<T>`, which provides a bounded, thread-safe queue with blocking semantics. `CompleteAdding()` signals that no more items will be produced, causing `GetConsumingEnumerable()` to complete once the queue is drained. The `Async` workflow provides cooperative concurrency. This is pragmatic and clean, using .NET's well-tested concurrent collections.

### ML Family Comparison

Each ML-family language brings a different concurrency philosophy. Haskell's STM offers composable transactions — a theoretically elegant model with no equivalent in other families. OCaml 5's effect-based concurrency is the newest approach, using algebraic effects to express concurrency as a composable, structured abstraction. F# pragmatically uses .NET's concurrent collections with async workflows.

None of these is as natural for concurrency as the BEAM family, but each offers unique strengths: Haskell's STM prevents a class of bugs that channel-based systems cannot, OCaml's structured concurrency ensures resources are always cleaned up, and F# provides seamless access to .NET's mature concurrent data structures.

---

## The JVM Family

### Java

Java has evolved its concurrency model significantly over decades, from raw threads and synchronized blocks to the Executor framework, CompletableFuture, and now virtual threads (Project Loom).

```java
import java.util.*;
import java.util.concurrent.*;

public class Pipeline {
    public static void main(String[] args) throws Exception {
        int[] items = {1, 2, 3, 4, 5};
        int numProducers = 3;
        int numConsumers = 2;
        
        // Bounded buffer
        BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(4);
        ConcurrentLinkedQueue<Integer> results = new ConcurrentLinkedQueue<>();
        int POISON = Integer.MIN_VALUE;
        
        ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
        
        // Spawn producers
        CountDownLatch producersDone = new CountDownLatch(numProducers);
        for (int p = 0; p < numProducers; p++) {
            executor.submit(() -> {
                try {
                    for (int item : items) {
                        queue.put(item);  // blocks if full
                    }
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    producersDone.countDown();
                }
            });
        }
        
        // Shutdown coordinator: send poison pills after producers finish
        executor.submit(() -> {
            try {
                producersDone.await();
                for (int c = 0; c < numConsumers; c++) {
                    queue.put(POISON);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        // Spawn consumers
        CountDownLatch consumersDone = new CountDownLatch(numConsumers);
        for (int c = 0; c < numConsumers; c++) {
            executor.submit(() -> {
                try {
                    while (true) {
                        int item = queue.take();  // blocks if empty
                        if (item == POISON) break;
                        results.add(item * item);
                    }
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    consumersDone.countDown();
                }
            });
        }
        
        consumersDone.await();
        List<Integer> sorted = new ArrayList<>(results);
        Collections.sort(sorted);
        System.out.println(sorted);
        executor.shutdown();
    }
}
```

Java's approach is characteristically explicit. `BlockingQueue` provides bounded buffering with blocking `put`/`take`. `CountDownLatch` coordinates shutdown — producers count down when done, a coordinator then sends "poison pill" values that signal consumers to stop. Virtual threads (Java 21+) make thread creation cheap, similar to goroutines.

The "poison pill" pattern is a classic Java concurrency idiom: since `BlockingQueue` has no built-in "close" mechanism (unlike Go channels), a sentinel value signals shutdown. This requires choosing a value that cannot be confused with real data — an awkwardness that Go's `close` avoids.

### Scala

Scala's Akka (now Apache Pekko) provides a typed actor system inspired by Erlang.

```scala
import scala.concurrent.{Future, Await}
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.duration._
import java.util.concurrent.{ArrayBlockingQueue, ConcurrentLinkedQueue}

object Pipeline:
  def run(): List[Int] =
    val items = (1 to 5).toList
    val queue = new ArrayBlockingQueue[Option[Int]](4)
    val results = new ConcurrentLinkedQueue[Int]()

    // Producers
    val producers = (1 to 3).map { _ =>
      Future {
        items.foreach(item => queue.put(Some(item)))
      }
    }

    // After all producers finish, send None (poison pill) for each consumer
    Future.sequence(producers).foreach { _ =>
      (1 to 2).foreach(_ => queue.put(None))
    }

    // Consumers
    val consumers = (1 to 2).map { _ =>
      Future {
        var running = true
        while running do
          queue.take() match
            case Some(item) => results.add(item * item)
            case None => running = false
      }
    }

    Await.result(Future.sequence(consumers), 10.seconds)
    results.asScala.toList.sorted
```

Scala wraps Java's concurrent collections in `Future` for cleaner composition. The `Option[Int]` type avoids the poison-pill hack — `None` is a clean shutdown signal. `Future.sequence` converts a collection of futures into a future of collections, enabling clean shutdown coordination.

### Kotlin

Kotlin's coroutines provide structured concurrency with channels.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

suspend fun main() {
    val items = listOf(1, 2, 3, 4, 5)
    val results = mutableListOf<Int>()
    val mutex = kotlinx.coroutines.sync.Mutex()
    
    coroutineScope {
        val channel = Channel<Int>(capacity = 4)  // bounded
        
        // Spawn producers
        val producers = List(3) {
            launch {
                for (item in items) {
                    channel.send(item)  // suspends if full
                }
            }
        }
        
        // Close channel when all producers finish
        launch {
            producers.forEach { it.join() }
            channel.close()
        }
        
        // Spawn consumers
        List(2) {
            launch {
                for (item in channel) {  // ends when channel closed
                    mutex.withLock {
                        results.add(item * item)
                    }
                }
            }
        }.forEach { it.join() }
    }
    
    println(results.sorted())
}
```

Kotlin's coroutines with channels are strikingly similar to Go's goroutines with channels. The `Channel<Int>(capacity = 4)` is a bounded channel. `channel.send()` suspends (rather than blocks) when full. The `for (item in channel)` loop completes when the channel is closed. `coroutineScope` provides structured concurrency — the scope does not exit until all launched coroutines complete.

The similarity to Go is not coincidental: Kotlin's coroutine designers studied Go's concurrency model. The differences are that Kotlin coroutines are cooperative (suspend points are marked with `suspend`), and `coroutineScope` provides structured concurrency guarantees that Go's goroutines lack.

### Clojure

Clojure's `core.async` provides Go-style channels with a Lisp syntax.

```clojure
(require '[clojure.core.async :as async :refer [<! >! go chan close! <!!]])

(defn run-pipeline []
  (let [buffer (chan 4)  ; bounded channel
        results (chan 100)
        items (range 1 6)]
    
    ;; Spawn producers
    (let [prod-done (async/merge
                      (for [_ (range 3)]
                        (go
                          (doseq [item items]
                            (>! buffer item))
                          :done)))]
      ;; Close buffer when producers finish
      (go
        (dotimes [_ 3] (<! prod-done))
        (close! buffer)))
    
    ;; Spawn consumers
    (let [cons-done (async/merge
                      (for [_ (range 2)]
                        (go
                          (loop []
                            (when-let [item (<! buffer)]
                              (>! results (* item item))
                              (recur)))
                          :done)))]
      (go
        (dotimes [_ 2] (<! cons-done))
        (close! results)))
    
    ;; Collect results
    (sort (<!! (async/into [] results)))))
```

Clojure's `core.async` brings Go-style channels to the Lisp world. `go` blocks are lightweight coroutines, `>!` sends, `<!` receives, and `close!` signals completion. The `when-let` pattern receives from the channel and loops while items are available. Clojure's immutable data structures mean results collection does not need explicit locking — the channel itself serializes access.

### JVM Family Comparison

The JVM family reveals decades of concurrency evolution. Java provides foundational primitives (`BlockingQueue`, `CountDownLatch`, `ExecutorService`) that are explicit and verbose. Scala adds `Future` composition and pattern matching for cleaner coordination. Kotlin imports Go's channel model with coroutines. Clojure imports Go's channel model with Lisp syntax and immutable data.

The trend is clear: newer JVM languages all move toward lightweight concurrency (virtual threads, coroutines) and channel-based communication, converging on the model that Go and Erlang popularized.

---

## The C Family

### C (pthreads)

C with POSIX threads shows concurrency at its most fundamental level.

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

#define BUFFER_SIZE 4
#define NUM_ITEMS 5
#define NUM_PRODUCERS 3
#define NUM_CONSUMERS 2

typedef struct {
    int buffer[BUFFER_SIZE];
    int head, tail, count;
    pthread_mutex_t mutex;
    pthread_cond_t not_full;
    pthread_cond_t not_empty;
    int producers_done;
    int total_producers;
} BoundedBuffer;

void buffer_init(BoundedBuffer *b) {
    b->head = b->tail = b->count = b->producers_done = 0;
    b->total_producers = NUM_PRODUCERS;
    pthread_mutex_init(&b->mutex, NULL);
    pthread_cond_init(&b->not_full, NULL);
    pthread_cond_init(&b->not_empty, NULL);
}

void buffer_put(BoundedBuffer *b, int item) {
    pthread_mutex_lock(&b->mutex);
    while (b->count == BUFFER_SIZE)
        pthread_cond_wait(&b->not_full, &b->mutex);
    b->buffer[b->tail] = item;
    b->tail = (b->tail + 1) % BUFFER_SIZE;
    b->count++;
    pthread_cond_signal(&b->not_empty);
    pthread_mutex_unlock(&b->mutex);
}

/* Returns 1 if item was retrieved, 0 if shutdown */
int buffer_get(BoundedBuffer *b, int *item) {
    pthread_mutex_lock(&b->mutex);
    while (b->count == 0 && b->producers_done < b->total_producers)
        pthread_cond_wait(&b->not_empty, &b->mutex);
    if (b->count == 0) {
        pthread_mutex_unlock(&b->mutex);
        return 0;  /* shutdown */
    }
    *item = b->buffer[b->head];
    b->head = (b->head + 1) % BUFFER_SIZE;
    b->count--;
    pthread_cond_signal(&b->not_full);
    pthread_mutex_unlock(&b->mutex);
    return 1;
}

void buffer_producer_done(BoundedBuffer *b) {
    pthread_mutex_lock(&b->mutex);
    b->producers_done++;
    pthread_cond_broadcast(&b->not_empty);  /* wake all consumers */
    pthread_mutex_unlock(&b->mutex);
}

/* Result collection */
typedef struct {
    int *data;
    int count;
    pthread_mutex_t mutex;
} Results;

void *producer_fn(void *arg) {
    BoundedBuffer *b = (BoundedBuffer *)arg;
    for (int i = 1; i <= NUM_ITEMS; i++)
        buffer_put(b, i);
    buffer_producer_done(b);
    return NULL;
}

void *consumer_fn(void *arg) {
    void **args = (void **)arg;
    BoundedBuffer *b = (BoundedBuffer *)args[0];
    Results *r = (Results *)args[1];
    int item;
    while (buffer_get(b, &item)) {
        int result = item * item;
        pthread_mutex_lock(&r->mutex);
        r->data[r->count++] = result;
        pthread_mutex_unlock(&r->mutex);
    }
    return NULL;
}

int compare(const void *a, const void *b) {
    return (*(int *)a - *(int *)b);
}

int main() {
    BoundedBuffer buf;
    buffer_init(&buf);
    
    Results results;
    results.data = malloc(NUM_PRODUCERS * NUM_ITEMS * sizeof(int));
    results.count = 0;
    pthread_mutex_init(&results.mutex, NULL);
    
    pthread_t producers[NUM_PRODUCERS], consumers[NUM_CONSUMERS];
    void *consumer_args[NUM_CONSUMERS][2];
    
    for (int i = 0; i < NUM_PRODUCERS; i++)
        pthread_create(&producers[i], NULL, producer_fn, &buf);
    
    for (int i = 0; i < NUM_CONSUMERS; i++) {
        consumer_args[i][0] = &buf;
        consumer_args[i][1] = &results;
        pthread_create(&consumers[i], NULL, consumer_fn, consumer_args[i]);
    }
    
    for (int i = 0; i < NUM_PRODUCERS; i++)
        pthread_join(producers[i], NULL);
    for (int i = 0; i < NUM_CONSUMERS; i++)
        pthread_join(consumers[i], NULL);
    
    qsort(results.data, results.count, sizeof(int), compare);
    for (int i = 0; i < results.count; i++)
        printf("%d ", results.data[i]);
    printf("\n");
    
    free(results.data);
    return 0;
}
```

This is the baseline implementation against which all higher-level abstractions can be measured. Every lock acquisition, every condition variable wait, every signal is explicit. The bounded buffer is a circular array protected by a mutex and two condition variables. The shutdown protocol requires `broadcast` (not `signal`) to wake all consumers when the last producer finishes.

The code is roughly 100 lines for what Go expresses in 30 and Erlang in a different paradigm entirely. Every line is necessary — remove a lock and you get a data race, remove a signal and you get a deadlock, remove the broadcast and consumers may never wake up. This is why higher-level concurrency primitives were invented.

### C++ (std::thread)

Modern C++ provides threading primitives and can use `std::condition_variable` to build a bounded queue.

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <algorithm>
#include <optional>

template<typename T>
class BoundedQueue {
    std::queue<T> queue_;
    size_t capacity_;
    std::mutex mutex_;
    std::condition_variable not_full_, not_empty_;
    bool closed_ = false;
    
public:
    BoundedQueue(size_t cap) : capacity_(cap) {}
    
    void push(T item) {
        std::unique_lock lock(mutex_);
        not_full_.wait(lock, [&]{ return queue_.size() < capacity_; });
        queue_.push(std::move(item));
        not_empty_.notify_one();
    }
    
    std::optional<T> pop() {
        std::unique_lock lock(mutex_);
        not_empty_.wait(lock, [&]{ return !queue_.empty() || closed_; });
        if (queue_.empty()) return std::nullopt;
        T item = std::move(queue_.front());
        queue_.pop();
        not_full_.notify_one();
        return item;
    }
    
    void close() {
        std::lock_guard lock(mutex_);
        closed_ = true;
        not_empty_.notify_all();
    }
};

int main() {
    BoundedQueue<int> queue(4);
    std::vector<int> results;
    std::mutex results_mutex;
    
    std::vector<std::jthread> producers;
    for (int p = 0; p < 3; ++p)
        producers.emplace_back([&queue] {
            for (int i = 1; i <= 5; ++i) queue.push(i);
        });
    
    // Wait for producers, then close
    for (auto& t : producers) t.join();
    queue.close();
    
    std::vector<std::jthread> consumers;
    for (int c = 0; c < 2; ++c)
        consumers.emplace_back([&] {
            while (auto item = queue.pop()) {
                std::lock_guard lock(results_mutex);
                results.push_back(*item * *item);
            }
        });
    
    for (auto& t : consumers) t.join();
    std::sort(results.begin(), results.end());
    for (int r : results) std::cout << r << " ";
}
```

C++ improves on C with RAII (`unique_lock`, `lock_guard`), templates (the `BoundedQueue<T>` is generic), and `std::optional` for clean shutdown signaling. The `jthread` destructor joins automatically. But the underlying mechanism is the same as C: mutexes, condition variables, and manual synchronization.

### C Family Comparison

The C family shows concurrency at its most explicit. Every synchronization decision is visible in the code: which mutex protects which data, when to signal vs. broadcast, how to handle spurious wakeups. C++ adds RAII and templates for safer resource management but uses the same primitives. The contrast with Go (where channels handle all of this) or Erlang (where message passing eliminates shared state entirely) illustrates why those languages were designed — the manual approach is powerful but error-prone.

---

## Scripting Languages

### Python

Python's Global Interpreter Lock (GIL) prevents true parallel execution of Python bytecode in threads, but `multiprocessing` and `concurrent.futures` provide alternatives.

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
from queue import Queue
import threading

def run_pipeline():
    items = list(range(1, 6))
    buffer = Queue(maxsize=4)  # bounded
    results = []
    results_lock = threading.Lock()
    
    def producer():
        for item in items:
            buffer.put(item)  # blocks if full
    
    def consumer():
        while True:
            item = buffer.get()
            if item is None:  # poison pill
                break
            with results_lock:
                results.append(item * item)
    
    # Start producers and consumers
    with ThreadPoolExecutor(max_workers=5) as executor:
        prod_futures = [executor.submit(producer) for _ in range(3)]
        cons_futures = [executor.submit(consumer) for _ in range(2)]
        
        # Wait for producers, then send poison pills
        for f in prod_futures:
            f.result()
        for _ in range(2):
            buffer.put(None)
        
        for f in cons_futures:
            f.result()
    
    return sorted(results)
```

Python's `Queue` class provides a thread-safe bounded queue with blocking `put`/`get`. The code is clean and readable. However, due to the GIL, the producer and consumer threads do not truly execute in parallel — they take turns holding the GIL. For CPU-bound work, Python requires `multiprocessing` (separate processes) or C extensions to achieve actual parallelism.

The `asyncio` alternative uses cooperative concurrency:

```python
import asyncio

async def run_pipeline():
    buffer = asyncio.Queue(maxsize=4)
    results = []
    
    async def producer():
        for item in range(1, 6):
            await buffer.put(item)
    
    async def consumer():
        while True:
            item = await buffer.get()
            if item is None:
                break
            results.append(item * item)
    
    producers = [asyncio.create_task(producer()) for _ in range(3)]
    consumers = [asyncio.create_task(consumer()) for _ in range(2)]
    
    await asyncio.gather(*producers)
    for _ in range(2):
        await buffer.put(None)
    await asyncio.gather(*consumers)
    
    return sorted(results)
```

The `asyncio` version looks similar but runs in a single thread with cooperative scheduling. It is suitable for I/O-bound work (network requests, file operations) but does not parallelize CPU-bound computation.

### Ruby

Ruby has similar GIL constraints (the GVL — Global VM Lock) but provides threads and queues.

```ruby
require 'thread'

def run_pipeline
  items = (1..5).to_a
  buffer = SizedQueue.new(4)  # bounded
  results = []
  results_mutex = Mutex.new
  
  producers = 3.times.map do
    Thread.new do
      items.each { |item| buffer.push(item) }
    end
  end
  
  consumers = 2.times.map do
    Thread.new do
      loop do
        item = buffer.pop
        break if item.nil?
        results_mutex.synchronize { results << item * item }
      end
    end
  end
  
  producers.each(&:join)
  2.times { buffer.push(nil) }  # poison pills
  consumers.each(&:join)
  
  results.sort
end
```

Ruby's `SizedQueue` is the bounded queue, with blocking `push`/`pop`. The pattern is identical to Python: poison pills for shutdown, mutex for result collection. Ruby's GVL has the same limitation as Python's GIL — threads do not truly parallelize CPU-bound work. For true parallelism, Ruby offers Ractors (Ruby 3.0+), which are actor-like isolated execution units.

### JavaScript / Node.js

JavaScript is single-threaded with an event loop. True concurrency requires Worker threads.

```javascript
// Using Worker threads for actual parallelism
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');

if (isMainThread) {
    async function runPipeline() {
        const items = [1, 2, 3, 4, 5];
        const allItems = [...items, ...items, ...items];  // 3 producers worth
        
        // Distribute work across worker threads
        const chunkSize = Math.ceil(allItems.length / 2);  // 2 consumers
        const promises = [];
        
        for (let i = 0; i < 2; i++) {
            const chunk = allItems.slice(i * chunkSize, (i + 1) * chunkSize);
            promises.push(new Promise((resolve, reject) => {
                const worker = new Worker(__filename, { workerData: chunk });
                worker.on('message', resolve);
                worker.on('error', reject);
            }));
        }
        
        const chunks = await Promise.all(promises);
        const results = chunks.flat().sort((a, b) => a - b);
        console.log(results);
    }
    
    runPipeline();
} else {
    // Worker: process items
    const results = workerData.map(item => item * item);
    parentPort.postMessage(results);
}
```

JavaScript's single-threaded nature means the producer-consumer pattern does not apply naturally. Worker threads communicate via message passing (similar to Erlang, but heavyweight — each worker has its own V8 isolate). The simpler async approach does not provide parallelism:

```javascript
async function runPipeline() {
    const items = [1, 2, 3, 4, 5];
    const results = [];
    
    // Simulated concurrent processing
    const promises = [];
    for (let p = 0; p < 3; p++) {
        for (const item of items) {
            promises.push(
                new Promise(resolve => {
                    setTimeout(() => resolve(item * item), 0);
                })
            );
        }
    }
    
    const settled = await Promise.all(promises);
    return settled.sort((a, b) => a - b);
}
```

This is not truly concurrent — all Promises resolve on the same event loop. JavaScript's concurrency model is fundamentally about managing I/O interleaving, not CPU parallelism.

### TypeScript (Deno)

Deno's TypeScript provides Web Workers for parallelism, similar to Node.js Worker threads.

```typescript
// Main thread
const items = [1, 2, 3, 4, 5];
const allItems = Array(3).fill(items).flat();

const results: number[] = [];

// Process concurrently with bounded parallelism
const batchSize = 4;
for (let i = 0; i < allItems.length; i += batchSize) {
    const batch = allItems.slice(i, i + batchSize);
    const batchResults = await Promise.all(
        batch.map(item => Promise.resolve(item * item))
    );
    results.push(...batchResults);
}

console.log(results.sort((a, b) => a - b));
```

TypeScript adds type safety to JavaScript's concurrency model but does not change its fundamental single-threaded nature. The bounded batching simulates backpressure but is not truly concurrent.

### Perl

Perl supports threads, though they are heavyweight (each thread gets a copy of the interpreter).

```perl
use threads;
use Thread::Queue;

my @items = (1..5);
my $buffer = Thread::Queue->new();
$buffer->limit = 4;  # bounded
my @results;
my $results_lock = threads::shared::share(@results);

# Producers
my @producers = map {
    threads->create(sub {
        $buffer->enqueue($_) for @items;
    })
} 1..3;

# Wait for producers, signal done
$_->join() for @producers;
$buffer->enqueue(undef) for 1..2;  # poison pills

# Consumers
my @consumers = map {
    threads->create(sub {
        while (defined(my $item = $buffer->dequeue())) {
            lock(@results);
            push @results, $item * $item;
        }
    })
} 1..2;

$_->join() for @consumers;
print join(", ", sort { $a <=> $b } @results), "\n";
```

Perl's `Thread::Queue` provides a bounded queue, and `threads::shared` allows sharing data between threads. Perl threads are expensive (each clones the interpreter state), making this approach impractical for large numbers of concurrent tasks.

### Lua

Standard Lua is single-threaded. Concurrency requires coroutines (cooperative) or external libraries like Lanes.

```lua
-- Cooperative concurrency with coroutines (not truly parallel)
local function make_queue(max_size)
    local queue = {}
    return {
        push = function(item)
            while #queue >= max_size do
                coroutine.yield("full")
            end
            table.insert(queue, item)
        end,
        pop = function()
            while #queue == 0 do
                coroutine.yield("empty")
            end
            return table.remove(queue, 1)
        end,
        size = function() return #queue end
    }
end

-- Simulated producer-consumer with coroutine scheduling
local items = {1, 2, 3, 4, 5}
local results = {}

-- In practice, true concurrency requires LuaLanes or similar
for _, item in ipairs(items) do
    for p = 1, 3 do
        table.insert(results, item * item)
    end
end
table.sort(results)
```

Lua's coroutines provide cooperative multitasking but not parallelism. A bounded queue can be simulated with yield points, but without OS threads, there is no true concurrency. This is Lua's most significant limitation for this problem — the language was designed as a lightweight embeddable scripting language, not for concurrent systems programming.

### Scripting Language Comparison

The scripting languages reveal a fundamental architectural limitation: the GIL/GVL (Python, Ruby) or single-threaded event loop (JavaScript, Lua) prevents true CPU parallelism within a single process. All four languages can simulate the producer-consumer pattern using queues and threads/tasks, but only Python and Ruby's threads (or multiprocessing) provide actual concurrent execution, and even then with significant overhead.

This is the algorithm that most starkly separates systems languages from scripting languages. For the array transformation and expression tree problems, scripting languages were merely more verbose than their ideal counterparts. For concurrent pipelines, they are fundamentally constrained by their runtime architectures.

---

## The Apple Ecosystem

### Swift

Swift's structured concurrency (introduced in Swift 5.5) provides actor isolation and async/await.

```swift
import Foundation

actor ResultCollector {
    var results: [Int] = []
    
    func add(_ value: Int) {
        results.append(value)
    }
    
    func sorted() -> [Int] {
        results.sorted()
    }
}

func runPipeline() async -> [Int] {
    let items = Array(1...5)
    let collector = ResultCollector()
    
    // AsyncStream provides bounded buffering
    let (stream, continuation) = AsyncStream<Int>.makeStream(bufferingPolicy: .bufferingOldest(4))
    
    // Spawn producers
    await withTaskGroup(of: Void.self) { group in
        for _ in 0..<3 {
            group.addTask {
                for item in items {
                    continuation.yield(item)
                }
            }
        }
        // When all producers finish, the group completes
    }
    continuation.finish()
    
    // Spawn consumers
    await withTaskGroup(of: Void.self) { group in
        for _ in 0..<2 {
            group.addTask {
                for await item in stream {
                    await collector.add(item * item)
                }
            }
        }
    }
    
    return await collector.sorted()
}
```

Swift's `actor` type provides data-race safety through isolation — the compiler ensures that actor state is only accessed through `await`-marked calls, which serialize access. `AsyncStream` provides a bounded async sequence. `withTaskGroup` provides structured concurrency, ensuring all tasks complete before the group exits.

Swift's approach is notable for integrating concurrency safety into the type system. Actors prevent data races at compile time (similar to Rust's ownership, but using actor isolation rather than move semantics). The `Sendable` protocol ensures only safe types cross concurrency boundaries.

---

## The Wirth Family

### Ada

Ada was designed for concurrent, safety-critical systems and has built-in tasking primitives that predate most other language-level concurrency support.

```ada
with Ada.Text_IO; use Ada.Text_IO;
with Ada.Containers.Vectors;

procedure Pipeline is
   package Int_Vectors is new Ada.Containers.Vectors(Natural, Integer);
   
   protected type Bounded_Buffer(Capacity : Positive) is
      entry Put(Item : Integer);
      entry Get(Item : out Integer; Valid : out Boolean);
      procedure Close;
   private
      Buffer : array(1..Capacity) of Integer;
      Head, Tail, Count : Natural := 0;
      Closed : Boolean := False;
   end Bounded_Buffer;
   
   protected body Bounded_Buffer is
      entry Put(Item : Integer) when Count < Capacity is
      begin
         Tail := (Tail mod Capacity) + 1;
         Buffer(Tail) := Item;
         Count := Count + 1;
      end Put;
      
      entry Get(Item : out Integer; Valid : out Boolean) when Count > 0 or Closed is
      begin
         if Count = 0 then
            Valid := False;
            return;
         end if;
         Head := (Head mod Capacity) + 1;
         Item := Buffer(Head);
         Count := Count - 1;
         Valid := True;
      end Get;
      
      procedure Close is
      begin
         Closed := True;
      end Close;
   end Bounded_Buffer;
   
   Buffer : Bounded_Buffer(4);
   Results : Int_Vectors.Vector;
   
   task type Producer;
   task body Producer is
   begin
      for I in 1..5 loop
         Buffer.Put(I);
      end loop;
   end Producer;
   
   task type Consumer;
   task body Consumer is
      Item : Integer;
      Valid : Boolean;
   begin
      loop
         Buffer.Get(Item, Valid);
         exit when not Valid;
         Results.Append(Item * Item);
      end loop;
   end Consumer;
   
   P1, P2, P3 : Producer;
   C1, C2 : Consumer;
begin
   null;  -- Tasks start automatically and synchronize on termination
end Pipeline;
```

Ada's `protected type` is a monitor — a data type with implicit mutual exclusion and condition-based entry guards. The `entry Put ... when Count < Capacity` clause means callers automatically wait until there is space. The `entry Get ... when Count > 0 or Closed` means consumers wait until items are available or shutdown is signaled. There are no explicit mutexes or condition variables — the protected object handles all synchronization.

Ada's `task type` declares concurrent units that start automatically when they are elaborated. Tasks synchronize at the end of the enclosing scope. This is the earliest form of structured concurrency, predating Go, Kotlin, and Swift by decades.

Ada's concurrency model was designed for aerospace and defense systems where correctness is critical. The monitor-based approach with entry guards is less flexible than channels or actors but provides strong guarantees about mutual exclusion and condition synchronization.

---

## Newer Functional Languages

### Roc

Roc's concurrency story is still evolving, but its platform model allows different concurrency backends.

```roc
# Roc's concurrency model is platform-dependent
# On a platform with Task-based concurrency:

app "pipeline" packages { pf: "platform" } imports [] provides [main] to pf

main =
    items = List.range { start: At 1, end: At 5 }
    allItems = List.join (List.repeat items 3)
    
    results =
        allItems
        |> List.map (\item -> item * item)
        |> List.sortAsc
    
    results
```

Roc's platform abstraction means the language itself does not dictate concurrency primitives — the platform (host) provides them. This is a novel architectural choice: the same Roc code could run on a platform with threads, async I/O, or actor-based concurrency, with the platform handling the details.

### Unison

Unison provides abilities (algebraic effects) for concurrency.

```unison
pipeline : '{IO} [Nat]
pipeline = do
    items = List.range 1 6
    
    -- Fork producers and consumers using abilities
    results = MVar.new []
    buffer = BoundedQueue.new 4
    
    -- Spawn producers
    producers = List.map (const (fork do
        List.foreach items (item -> BoundedQueue.put buffer item)
    )) (List.range 1 4)
    
    -- Wait for producers, close buffer
    List.foreach producers Thread.join
    BoundedQueue.close buffer
    
    -- Spawn consumers  
    consumers = List.map (const (fork do
        match BoundedQueue.take buffer with
            Some item ->
                MVar.modify results (r -> r :+ (item * item))
                -- continue consuming
            None -> ()
    )) [1, 2]
    
    List.foreach consumers Thread.join
    List.sort (MVar.read results)
```

Unison's ability system treats concurrency as an algebraic effect that can be handled by different interpreters. This means the same concurrent code could be interpreted with real threads, simulated for testing, or replayed for debugging. The content-addressed nature of Unison code means concurrent computations can be cached and shared across runs.

---

## Other Notable Languages

### Julia

Julia provides `@spawn` and channels for concurrent programming.

```julia
function run_pipeline()
    items = 1:5
    buffer = Channel{Int}(4)  # bounded, capacity 4
    results = Channel{Int}(100)
    
    # Spawn producers
    producers = [@async begin
        for item in items
            put!(buffer, item)
        end
    end for _ in 1:3]
    
    # Close buffer when producers finish
    @async begin
        for p in producers
            wait(p)
        end
        close(buffer)
    end
    
    # Spawn consumers
    consumers = [@async begin
        for item in buffer
            put!(results, item * item)
        end
    end for _ in 1:2]
    
    # Close results when consumers finish
    @async begin
        for c in consumers
            wait(c)
        end
        close(results)
    end
    
    sort(collect(results))
end
```

Julia's `Channel{Int}(4)` is a bounded channel, and `@async` spawns tasks. The `for item in buffer` loop ends when the channel is closed. Julia's approach is very similar to Go's, with channels as the primary coordination mechanism. Julia also supports `@threads` for CPU parallelism and `Distributed` for multi-process computation.

### Forth

Forth has no standard concurrency primitives. Multitasking Forth systems exist but vary widely.

```forth
\ Cooperative multitasking Forth (system-dependent)
\ Most Forth systems use a simple round-robin scheduler

variable buf-count
variable buf-head
variable buf-tail
4 constant BUF-SIZE
create buf-data BUF-SIZE cells allot

: buf-put  ( n -- )
  begin buf-count @ BUF-SIZE < until   \ spin-wait
  buf-data buf-tail @ cells + !
  buf-tail @ 1+ BUF-SIZE mod buf-tail !
  1 buf-count +! ;

: buf-get  ( -- n )
  begin buf-count @ 0> until            \ spin-wait
  buf-data buf-head @ cells + @
  buf-head @ 1+ BUF-SIZE mod buf-head !
  -1 buf-count +! ;
```

Forth's stack-based nature makes concurrent programming awkward — each task needs its own stack, and coordinating shared state requires manual synchronization. Some Forth systems provide `task` and `activate` words, but there is no standard approach. This is far from Forth's strengths.

### Prolog

Prolog has limited concurrency support, varying by implementation.

```prolog
% SWI-Prolog with message queues
:- use_module(library(thread)).

run_pipeline(Results) :-
    message_queue_create(Buffer, [max_size(4)]),
    message_queue_create(ResultQ),
    
    % Spawn producers
    forall(between(1, 3, _),
        thread_create(producer(Buffer), _, [detached(true)])),
    
    % Spawn consumers
    forall(between(1, 2, _),
        thread_create(consumer(Buffer, ResultQ), _, [detached(true)])),
    
    % Collect results
    collect_results(ResultQ, 15, Unsorted),
    msort(Unsorted, Results).

producer(Buffer) :-
    forall(between(1, 5, Item),
        thread_send_message(Buffer, item(Item))),
    thread_send_message(Buffer, done).

consumer(Buffer, ResultQ) :-
    thread_get_message(Buffer, Msg),
    (   Msg = item(Item)
    ->  Result is Item * Item,
        thread_send_message(ResultQ, Result),
        consumer(Buffer, ResultQ)
    ;   true  % done
    ).
```

SWI-Prolog provides thread-based concurrency with message queues — essentially an actor model. The bounded `message_queue` with `max_size` provides backpressure. The approach works but is far from Prolog's strength in logical reasoning.

---

## The APL Family

Array languages have minimal concurrency support. Their performance model relies on vectorized operations over arrays — implicit parallelism within operations, but no explicit concurrency between tasks.

Some implementations provide parallel primitives: Dyalog APL has `∥` for parallel execution, and J has `t.` for threading modifiers. But the producer-consumer pattern, which requires coordination between independent tasks, is fundamentally outside the array programming model.

```apl
⍝ Dyalog APL: parallel each (⌶) can parallelize map operations
⍝ but producer-consumer coordination is not natural
result ← {⍵*2}⌶ (3/⍳5)  ⍝ parallel square of replicated items
result ← result[⍋result]  ⍝ sort
```

The array language approach is to avoid explicit concurrency entirely — instead, operations on large arrays are implicitly parallelized by the runtime. This is a valid concurrency model for data-parallel problems but cannot express the task coordination that defines producer-consumer pipelines.

---

## Cross-Family Comparison

### Communication Paradigms

The most fundamental axis of variation is how concurrent units communicate. There are three primary paradigms.

**Message passing** (Erlang, Elixir, Go channels, Kotlin channels): concurrent units communicate by sending and receiving messages through typed or untyped channels. There is no shared mutable state. Safety comes from isolation — each unit owns its data and shares nothing.

**Shared memory with locks** (C, C++, Java's early model): concurrent units share memory and coordinate access with mutexes, condition variables, and atomic operations. This is the most flexible approach but the most error-prone — forgetting a lock or acquiring locks in the wrong order causes data races or deadlocks.

**Transactional memory** (Haskell's STM): concurrent units share memory but access it through composable transactions that the runtime serializes. This avoids explicit locks while preserving shared-state semantics.

### Lightweight vs. Heavyweight Concurrency

Languages differ enormously in the cost of concurrent units. Erlang processes and Go goroutines cost kilobytes and can number in millions. Java virtual threads (Loom) are similarly lightweight. OS threads (C, C++, Perl) cost megabytes and are limited to thousands. JavaScript and Lua lack true concurrency units entirely.

This cost difference is not just quantitative but qualitative. When concurrent units are cheap, the natural design is "one unit per logical task" — each producer and consumer is its own goroutine or process. When units are expensive, the natural design is thread pools with work-stealing — a fixed number of threads service many logical tasks.

### Shutdown Coordination

Shutdown — knowing when all producers are done and all consumers should stop — is surprisingly tricky. Languages handle it differently. Go uses channel close semantics, where `range` over a closed channel terminates. Erlang uses explicit shutdown messages in the actor protocol. Java uses poison pills or `CountDownLatch`. C uses condition variable broadcasts when producer counts reach zero.

The cleanest shutdown mechanisms are those built into the coordination primitive itself (Go's `close`, Ada's entry guards). The most error-prone are those that require manual counting and signaling (C's condition variables, Java's poison pills).

### Safety Guarantees

The safety spectrum for concurrent programming is wider than for sequential programming. Rust prevents data races at compile time through ownership. Go prevents some races through channel direction types but allows shared-memory races. Java provides runtime race detection with `synchronized` but allows races in unsynchronized code. C provides no safety at all — races are undefined behavior that the language makes easy to write.

Erlang and the BEAM family sidestep the problem entirely: since there is no shared mutable state, data races are impossible by construction. This is the strongest safety guarantee, achieved by restricting the programming model rather than by adding checks.

### Where Each Family Excels

The BEAM family excels here because concurrency is their raison d'être — lightweight processes, message passing, and fault tolerance via supervision trees. Go excels because channels and goroutines were designed for exactly this pattern. Rust excels because the ownership system provides safety guarantees that no other systems language matches. Ada excels because its protected objects and tasking model, designed for safety-critical systems, handle synchronization with minimal boilerplate.

The C family works but requires significant engineering. The scripting languages are fundamentally limited by their runtime architectures. The APL family and Prolog are out of their element — concurrency is orthogonal to their core paradigms.

---

## Conclusion

The concurrent producer-consumer pipeline reveals a different dimension of language design than the previous two algorithms. The array transformation showcased data-parallel thinking. The expression tree evaluator showcased algebraic data types and recursive structure. This algorithm showcases coordination — how independent computational units communicate, synchronize, and shut down safely.

The results upend the previous rankings. The BEAM family, which was merely adequate for arrays and trees, is perfectly at home with concurrent processes. Go, which was verbose for tree evaluation, is clean and idiomatic for channel-based pipelines. The APL family, which excelled at array transformations, has almost nothing to contribute to explicit concurrency. Haskell, which excelled at expression trees, offers a unique perspective through STM but is not as natural for concurrency as Erlang or Go.

The most striking insight is that concurrency safety can be achieved through fundamentally different mechanisms. Rust prevents data races through compile-time ownership analysis. Erlang prevents them by eliminating shared mutable state. Go and Kotlin prevent many issues through channel abstractions. Ada prevents them through monitor-based protected objects. C prevents nothing, leaving safety entirely to the programmer's discipline.

These are not points on a single spectrum but different answers to a philosophical question: should the language prevent concurrency errors by restricting what you can express, by checking what you express, or by trusting you to express it correctly? The diversity of answers reflects the diversity of contexts in which concurrent programs are written — from telephone switches to web servers to embedded controllers to financial trading systems. Each context has different requirements for safety, performance, and expressiveness, and each language's concurrency model reflects the context it was designed to serve.
