# Concurrency

## Joinable thread

A new thread class `jthread` that automatically joins thread (if it is joinable)
in its destructor as well as supports cooperative cancellation.

## Support for cooperative cancellation

- `stop_source` (analogous to `promise`)
  - Producer of stop request
  - Owns the shared state (if any)

- `stop_token` (analogous to `future`)
  - Handle to a `stop_source`
  - Query for stop requests, but can't make them (consumer only)

- `stop_callback` (analogous to `future::then`)
  - Mechanism for registering invocables to be run upon receiving a stop request

```cpp
void Sleep(const chrono::seconds seconds) {
  this_thread::sleep_for(seconds);
}

int main() {
  jthread t{[](stop_token st) {
    while (!st.stop_requested()) {
      cout << "Working...\n";
      Sleep(1s);
    }
  }};

  Sleep(3s);

  t.request_stop();
}
```

:paperclip: Cooperative cancellation was added for conditional variable methods as well

```cpp
template<class Lock, class Predicate>
bool wait(Lock& lock, stop_token stoken, Predicate pred);

template<class Lock, class Rep, class Period, class Predicate>
bool wait_for(Lock& lock,
              stop_token stoken,
              const chrono::duration<Rep, Period>& rel_time,
              Predicate pred);

template<class Lock, class Clock, class Duration, class Pred>
void wait_until(Lock& lock,
                stop_token stoken,
                const chrono::time_point<Clock, Duration>& timeout_time,
                Pred pred);
```

## New synchronization facilities

  ### Latches

  `latch` is a counter that allows threads to wait for the count to reach zero.

  ```cpp
  void DoStuff() {
    latch done{threadsCount};

    vector<jthread> threads;
    for (uint16_t i = 0; i < threadsCount; ++i)
      threads.push_back([&, i] {
        auto data = RequestData(i);

        done.count_down();

        DoMoreStuff(data);
      });
    // Threads block at a latch point, untill a given number of threads reach the latch point
    done.wait();
    // At which point all threads are allowed to continue
    ProcessData();
  }
  ```

  ### Barriers
  
  `barrier` is a sequence of phases, in each phase:

  - a number of threads block untill the requested number of threads arrive at the barries, then
  - a phase completion callback is executed
  - the thread counter is reset
  - the next phase starts
  - thread can continue

  ```cpp
  const uint16_t numberThreads = ...;
  void FinishTask() {}

  barrier<function<void()>> barrier{numberThreads, FinishTask};

  void WorkerThread(stop_token st, uint16_t i) {
    while (!st.stop_requested()) {
        cout << "Working...\n";
        barrier.arrive_and_wait();
    }
  }
  ```

  ### Semaphores

  Semaphores is a lightweight synchronization primitives that can be used to implement any other synchronization concepts (mutex, latches, barries, ...). There are two types:
  - **counting** semaphore models a non-negative resource count
  - **binary** semaphore has only 1 slot, i.e. two possible states - free and not free
    ```cpp
    using binary_semaphore = counting_semaphore<1>;
    ```

  ```cpp
  counting_semaphore<5> slots{5};

  void Foo() {
    slots.acquire();
    DoSomething(); // at most 5 threads can be here
    slots.release();
  }
  ```

## Updates for `atomic`

- `atomic<shared_ptr<T>>` and `atomic<weak_ptr<T>>`

  > Might use a mutex internally

  > Global non-member atomic operations are deprecated

- Waiting and notifying on `atomic`

  > Wait/block for an atomic object to change its value, notified by a notification function

- `atomic_ref` provides atomic operations on a non-atomic objects

> **Note**: If you use `atomic_ref` to access an object, all accesses to this object must use `atomic_ref`

<p align="right"><a href="../README.md#contents">:arrow_left: Back to Contents</a></p>
