# Concurrency

- A new thread class `jthread` that automatically joins thread (if it is joinable)
  in its destructor as well as supports cooperative cancellation.

- Support for cooperative cancellation of threads:

  - `stop_source` (analogous to a promise)
    - Producer of stop request
    - Owns the shared state (if any)

  - `stop_token` (analogous to future)
    - Handle to a `stop_source`
    - Query for stop requests, but can't make them (consumer only)
  
  - `stop_callback` (analogous to `future::then`)
    - Mechanism for registering invocables to be run upon receiving a stop request

  > **Note**: Cooperative cancellation was added for conditional variable methods
  ```cpp
  template<class Lock, class Predicate>
  bool wait(Lock& lock, std::stop_token stoken, Predicate pred);

  template<class Lock, class Rep, class Period, class Predicate>
  bool wait_for(Lock& lock,
                std::stop_token stoken,
                const std::chrono::duration<Rep, Period>& rel_time,
                Predicate pred);

  template<class Lock, class Clock, class Duration, class Pred>
  void wait_until(Lock& lock,
                  std::stop_token stoken,
                  const std::chrono::time_point<Clock, Duration>& timeout_time,
                  Pred pred);
  ```

  ```cpp
  void Sleep(const std::chrono::seconds seconds) {
    std::this_thread::sleep_for(seconds);
  }

  int main() {
    using namespace std::chrono_literals;

    std::jthread t{[](std::stop_token st) {
      while (!st.stop_requested()) {
        std::cout << "Working...\n";
        Sleep(1s);
      }
    }};

    Sleep(3s);

    t.request_stop();
  }
  ```

- New synchronization facilities:

  - **Latches**

    `std::latch` is a single-use counter that allows threads to wait for the count to reach zero.

    > Threads block at a latch point, untill a given number of threads reach the latch point, at which point all threads are allowed to continue
    
     ```cpp
     void Foo() {
       const uint16_t threadCount = ...
       std::latch done{threadCount};
       Data data[threadCount];
       std::vector<std::jthread> threads;
       for (uint32_t i = 0; i < threadsCount; ++i)
         threads.push_back([&, i] {
	     data[i] = MakeData(i);
	     done.count_down();
	     DoMoreStuff();
         });
       done.wait();
       ProcessData();
     }
     ```

  - **Barriers** - a sequence of phases, in each phase:
    - a number of threads block untill the requested number of threads arrive at the barries, then
    - a phase completion callback is executed
    - the thread counter is reset
    - the next phase starts
    - thread can continue
  
    ```cpp
    const uint16_t numberThreads = ...;
    void FinishTask() {}

    std::barrier<std::function<void()>> barrier{numberThreads, FinishTask};

    void WorkerThread(std::stop_token st, uint16_t i) {
      while (!st.stop_requested()) {
          std::cout << "Working...\n";
          barrier.arrive_and_wait();
      }
    }
    ```
  
  - **Semaphores** - lightweight synchronization primitives that can be used to implement any other synchronization concepts (mutex, latches, barries, ...). There are two types:
    - **counting** semaphore models a non-negative resource count
    - **binary** semaphore has only 1 slot, i.e. two possible states - free and not free

    ```cpp
    std::counting_semaphore<5> slots{5};

    void Foo() {
      slots.acquire();
      DoSomething(); // at most 5 threads can be here
      slots.release();
    }
    ```

- Updates for atomic :

  - `std::atomic<std::shared_ptr<T>>` and `std::atomic<std::weak_ptr<T>>`

    > Might use a mutex internally
  
    > Global non-member atomic operations are deprecated

  - Waiting and notifying on `std::atomic`
  
    > Wait/block for an atomic object to change its value, notified by a notification function

  - `std::atomic_ref` provides atomic operations on a non-atomic objects

  > **Note**: If you use std::atomic_ref to access an object, all accesses to this object must use std::atomic_ref

<p align="right"><a href="../README.md#contents">:arrow_left: Back to Contents</a></p>
