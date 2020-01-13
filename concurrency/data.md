# Concurrency

- A new thread class `std::jthread` that automatically joins thread in its destructor

- Support for cooperative cancellation of threads:

  - `std::stop_source` - class representing a request to stop thread execution

  - `std::stop_token` - an interface for actively checking for a stop request
  
  - `std::stop_callback` - an interface for your own cancellation mechanism

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
  
  > **Note**: `std::jthread` integrates with `std::stop_token` to support cooperative cancellation

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
