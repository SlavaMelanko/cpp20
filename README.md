# C++20 Overview

![license](https://img.shields.io/badge/License-MIT-green) ![cpp20](https://img.shields.io/badge/C%2B%2B-20-blue)

:pushpin: Please note that

1. The idea of this overview is self education
1. The main goal - keep it simple and short
1. And it is primarily based on [CppPon 2019 talks](https://www.youtube.com/playlist?list=PLHTh1InhhwT6KhvViwRiTR7I5s09dLCSw)

:warning: There is no guarantee that below theoretical and practical parts are correkt and up to date

# Contents

1. [Modules](#modules)
1. [Ranges](#ranges)
1. [Coroutines](#coroutines)
1. [Concepts](#concepts)
1. [Lambda Expression](#lambda)
1. [constexpr](#constexpr)
1. [Concurrency](#concurrency)
1. [Designated Initializers](#designated)
1. [Spaceship (Three-Way Comparison) Operator <=>](#spaceship)
1. [Range-Based `for` Loop Initializer](#forloop)
1. [Non-Type Template Parameters](#templ)
1. [`likely` and `unlikely` Attributes](#attributes)
1. [Calendars and Timezones](#chrono)
1. [`span`](#span)

<a name="modules"></a>
## Modules

:bulb: Module is a way to organize, encapsulate, and isolate your code.

Advantages:

- Better compilation times
  > Modules are processed only once. Compare this with M headers which are included in N translation units.
  > The combinatorial explosion means, that the header has to be parsed M*N times.

- No need of header files
  > Separation into interface and implementation files is possible but it is obsolete

- No need for include guard

- Preprocessor usage elimination

- No need to invent unique names
  > Same names in multiple modules will not clash

- The order of `import` statements will not matter

:mag_right: **Example**:

```cpp
// hello.cpp/.cppm/.mpp
export module hello;

namespace hello {

auto GetWelcomeMessage() {
  return "Welcome to C++20!";
}

export auto SayWelcome() {
  return GetWelcomeMessage();
}

} // namespace hello

// main.cpp/.cppm/.mpp
import hello;

import <iostream>;

int main() {
  std::cout << hello::SayWelcome();
}
```

:paperclip: You can import header files, e.g.

```cpp
import <iostream>;
```

- Implicitely turns the `iostream` header into module
- Improves build throughput, as `iostream` will then processed only once

:clipboard: **Structure**:

  | Module (top to bottom) |
  | :---: |
  | `module;` |
  | preprocessor derictives only, e.g `#include <cassert>` |
  | `export module name;` |
  | `import ...;` |
  | `...` |
  | `module : private;` |
  | `...` |

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="ranges"></a>
## Ranges

:bulb: Range is an object referring to a sequence of elements.
It is additional (abstraction) layer on top of iterators
that provides a nicer and easier to read syntax.

> **Note**: all below examples were tested using [range-v3](https://github.com/ericniebler/range-v3) library.

:mag_right: **Example**:

```cpp
std::array<int, 5> data{2, 4, 5, 1, 3};
std::sort(std::begin(data), std::end(data)); // before
ranges::sort(data) // now
```

Ranges library is based on 3 core components:

### Views

  Ranges with 'lazy evaluation', non-owning, and non-mutating of elements,
  with constant time for copying and moving

```cpp
std::vector<int> data{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
auto result = data
        | ranges::views::remove_if([](int i) { return i % 2 == 1; })
        | ranges::views::transform([](int i) { return std::to_string(i); });
```
> **>_** result = ["2", "4", "6", "8", "10"]

```cpp
std::vector<int> data{0, 1, 2, 3, 4, 5, 6};
auto evens = data | ranges::views::filter([](auto i) { return i % 2 == 0; });
std::cout << ranges::views::all(evens) << '\n';
```
> **>_** [0, 2, 4, 6]

### Actions

 Ranges which are eagerly evaluated, mutating the data, and can be composed as views

```cpp
std::vector<int> data{4, 3, 4, 1, 8, 0, 8};
data |= ranges::actions::sort | ranges::actions::unique;
```
> **>_** result = [0, 1, 3, 4, 8]

```cpp
std::vector<int> original{4, 3, 4, 1, 8, 0, 8};
auto modified = original | ranges::copy | ranges::actions::sort | ranges::actions::unique;
```
> **>_** original = [4,3,4,1,8,0,8]

> **>_** modified = [0,1,3,4,8]

### Algorithms

  All standard library algorithms accepting ranges instead of iterator pairs

```cpp
auto sequence = ranges::views::ints(1, 11)
  | ranges::views::transform([](const auto i) { return i * i; });
const auto sum = ranges::accumulate(sequence, 0);
```
> **>_** sum = 385

:paperclip: Projection:

```cpp
struct Employee {
  std::string firstName;
  uint8_t age;
};
    
std::vector<Employee> employees = {{"Jason", 50}, {"Jane", 40}};
    
ranges::sort(employees, {}, &Employee::age);
```
> **>_** employees = {{"Jane", 40}, {"Jason", 50}}

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="coroutines"></a>
## Coroutines

> :clock1: The term coroutine was introduced by Melvin Conway in 1958.

:bulb: Coroutines are generalised functions that:

- Can suspend execution
- Return an intermediate value
- Resume later
- Preserve local state
- Allow re-entry more than once
- Non-pre-emptive -> Cooperative?

Coroutines must have one of the following keywords:

- **co_await** suspends evaluation of a coroutine while waiting for a computation to finish

- **co_return** returns from a coroutine
  > Just `return` is not allowed

- **co_yield** returns a value from a coroutine back to the caller, and suspends the coroutine,
subsequently calling the coroutine again continues its execution

Coroutines might be used for:

- Asynchronous I/O, e.g. servers
- Lazy computations, e.g. generators
- Event driven applications, e.g. simulations, games, user interfaces, or even algorithms

> **Note**: C++20 contains language additions to support coroutines, whereas
> standard library does not include helper classes yet such as generators.

:mag_right: **Example**:

> **Note**: Below example was tested with [godbolt](https://godbolt.org/z/GUSdpQ) (x64 msvc v19.22)

```cpp
#include <experimental/coroutine>
#include <experimental/generator>

std::experimental::generator<int> CoroutineGenerator(const int iterations) {
    for (int i = 0; i < iterations; i++) {
        co_yield i;
    }
}

int main() {
    int total = 0;

    for (auto i : CoroutineGenerator(5)) {
        total += i;
    }

    return total;
}
```

:paperclip: A range-based `for co_wait` loop: `for co_await (declaration : expression) statement`

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="concepts"></a>
## Concepts

:bulb: Requirements that can be attached to class templates and function templates to constraint the template arguments.

:mag_right: **Example**:

Concept definitions:

```cpp
template<typename T>
concept Incrementable = requires(T t) { ++t; t++; };

template <typename T>
concept Integral = std::is_integral<T>::value;

template<typename T>
concept HasSize = requires(T t) {
    { t.size() } -> std::convertible_to<std::size_t>;
};
```

Usage: 

- Constrained template parameters:
```cpp
template<Incrementable T>
void Foo(T t);
```
- Requires clause:
```cpp
template<typename T> requires Incrementable<T> && Decrementable<T>
void Foo(T t);
```
- Trailing requires clause:
```cpp
template<typename T>
void Foo(T t) requires Incrementable<T> && Decrementable<T>;
```
- Placeholder syntax:
```cpp
void Foo(Incrementable auto t);
```

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="lambda"></a>
## Lambda Expression

- Since C++20, you need to capture `this` explicitly in case of using `[=]`, e.g. `[=, this]`

- Added possibility to use templated lambda expressions, e.g. `[]<typename T>(T t) { /* ... */ }`

  * Retrieve type of parameters of generic lambdas to access static members/methods or nested aliases:

  ```cpp
  // Before
  auto foo = [](const auto& value) {
    using T = std::decay_t<decltype(value)>;
    T valueCopy = value;
    T::staticMethod();
    using Alias = T::NestedAlias;
  };

  // Now
  auto foo = []<typename T>(const T& value) {
    T valueCopy = value;
    T::staticMethod();
    using Alias = T::NestedAlias;
  };
  ```

  * Retrieve type of the element of containers:

  ```cpp
  // Before
  auto foo = [](const auto& data) {
    using T = typename std::decay_t<decltype(data)>::value_type;
    // ...
  };

  // Now
  auto foo = []<typename T>(const std::vector<T>& data) {
    // ...
  };
  ```

  * Perfect forwarding:

  ```cpp
  // Before
  auto foo = [](auto&&... args) {
    return bar(std::forward<decltype(args)>(args)...);
  };

  // Now
  auto foo = []<typename ...T>(T&&... args) {
    return bar(std::forward<T>(args)...);
  };
  ```

  * Init-capture followed by an ellipsis:

  ```cpp
  template<typename F, typename... Args>
  auto DelayInvoke(F f, Args... args) {
    return [f = std::move(f), args = std::move(args)...]() {
      return std::invoke(f, args...);
    };
  }
  ```

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="constexpr"></a>
## constexpr

- C++20 allows virtual functions to be declared `constexpr`:
  ```cpp
  struct Base {
    virtual ~Base() noexcept = default;
    virtual int foo() const noexcept = 0;
  };

  struct Derived : public Base {
    constexpr int foo() const noexcept override { return 2; };
  };

  int main() {
    {
      constexpr Derived derived;
      constexpr auto value = derived.foo();
      static_assert(value == 2);
    }
    {
      std::unique_ptr<Base> base = std::make_unique<Derived>();
      const auto value = base->foo();
    }
  }
  ```
  > **Note**: A `constexpr` virtual function can override a non-`constexpr` function and vice versa.

- `std::string` and `std::vector` are now `constexpr`

- Also, `constexpr` functions can now:
  * use `dynamic_cast` and `typeid`
  * do dynamic memory allocations and deallocations
  * change the active member of a `union`
  * contain `try/catch` blocks:
    - But `throw` statements are still not allowed
	- `try/catch` blocks are no-ops when evaluated in a constant expression

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="concurrency"></a>
## Concurrency

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

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="designated"></a>
## Designated Initializers

:mag_right: **Example**:

```cpp
struct User {
  std::string email;
  std::string password;
  int pin;
};

int main() {
  User jason{"jason@mail.com", "Passw0rd!", 1234}; // good old struct initialization
  User jane{.email = "jane@mail.com", .password = "Passw0rd!"}; // aggregate initialization
}
```

> **Note**: All designators used in the expression must appear in the same order as the data members, e.g.

```cpp
User user{.password = "Passw0rd!", .email = "user@mail.com"};
```

> error: ISO C++ requires field designators to be specified in declaration order ...

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="spaceship"></a>
## Spaceship (Three-Way Comparison) Operator <=>

:mag_right: **Example**:

Before:

```cpp
class Point {
public:
  friend bool operator==(const Point& a, const Point& b) { return a.x == b.x && a.y == b.y; }
  friend bool operator< (const Point& a, const Point& b) { return a.x < b.x || (a.x == b.x && a.y < b.y); }
  friend bool operator!=(const Point& a, const Point& b) { return !(a == b); }
  friend bool operator<=(const Point& a, const Point& b) { return !(b < a); }
  friend bool operator> (const Point& a, const Point& b) { return b < a; }
  friend bool operator>=(const Point& a, const Point& b) { return !(a < b); }

private:
  double x, y;
};
```

Now:

```cpp
#include <compare>

class Point {
public:
  auto operator<=>(const Point&) const = default;

private:
  double x, y;
};
```

> **Note**: `<compare>` header is responsible for populating the compiler with all of the comparison category types
> necessary for the spaceship operator to return a type appropriate for our defaulted function.

:paperclip: Standard library types (`vector`, `string`, `map`, `set`, ...) include support for `<=>`

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="forloop"></a>
## Range-Based `for` Loop Initializer

:mag_right: **Example**:

```cpp
for (const auto users = SelectActiveUsers(); const offer& user : users) {
  // ...
}
```

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="templ"></a>
## Non-Type Template Parameters

C++20 allows string literals for non-type template parameters.

```cpp
template<auto& s>
void DoSomething() {
  std::cout << s << std::endl;
}

int main() {
  DoSomething<"C++20">();
}
```

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="attributes"></a>
## `likely` and `unlikely` Attributes

Hints for the compiler to optomize certain branches.

:mag_right: **Example**:

```cpp
switch (value) {
  case 1:
    // ...
    break;
  [[likely]] case 2:
    // ...
    break;
  [[unlikely]] case 3:
    // ...
    break;
}
```

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="chrono"></a>
## Calendars and Timezones

- Chrono is extended to support calendars and timezones. Only Gregorian calendar is supported.

  Creating a year

  ```cpp
  auto year1 = std::chrono::year{2020};
  auto year2 = 2020y; // using chrono_literals
  ```

  > The same for months and days.

  Creating full date

  ```cpp
  {
    std::chrono::year_month_day date{2019y, September, 18d};
  }
  {
    auto date = 2019y/September/18d; // 18d/September/2019y || September/18d/2019y
  }
  {
    auto date = September/last/2019y;
  } 
  {
    auto date = 31d/October/2019;
    if (!date.ok()) {
      date = std::chrono::sys_days{date};
    }
  }
  {
    auto date = Friday[2]/November/2019y;
  }
  ```

- New duration type aliases

  - `days`
  - `weeks`
  - `months`
  - `years`

- New clocks

  - `utc_clock` represents Coordinated Universal Time (UTC), measures time since
    00:00:00 UTC, Thursday, 1 January 1970, and includs leap seconds.

  - `tai_clock` represents International Atomic Time (TAI), measures time since
    00:00:00, 1 January 1958, and was offseted 10 seconds ahead of UTC at that date, does not include leap seconds.

  - `gps_clock` represents Global Positioning System (GPS) time, measures time since
    00:00:00, 6 January 1980 UTC, does not include leap seconds.

  - `file_clock` - alias for the clock used for `std::filesystem::file_time_type::clock`, epoch is unspecified.

- New `system_clock`-related type aliases

  ```cpp
  template<class Duration>
  using sys_time = std::chrono::time_point<std::chrono::system_clock, Duration>;
  
  using sys_seconds = sys_time<std::chrono::seconds>;
  using sys_days = sys_time<std::chrono::days>;
  ```
  
  ```cpp
  auto now = system_clock::now();
  ```
  
- Date + Time

  ```cpp
  auto timestamp = sys_days{2019y/September/18d} + 9h + 35m + 10s; // 2019-09-18 09:35:10 UTC
  ```
  
- Timezone conversion

  - Conver UTC to Denver time

  ```cpp
  auto now = system_clock::now();
  zoned_time current{current_zone(), now};
  zoned_time denver{"America/Denver", now};
  ```
  
  - Construct a localtime in Denver
  
  ```cpp
  auto denver = std::chrono::zoned_time{"America/Denver", std::chrono::local_days{Wednesday[3]/September/2019} + 9h};
  ```
  
  - Get current localtime
  
  ```cpp
  auto local = std::chrono::zoned_time{std::chrono::current_zone(), std::chrono::system_clock::now()};
  ```

- Formating and parsing

  C++20 `chrono` fully integrates into C++20 `std::format`

  ```cpp
  auto tp = system_clock::now();
  auto tz = locale_zone("Europe/Berlin");
  cout << format("{:%F %T %Z}\n", zoned_time{tz, tp});
  ```
  > **>_** 2019-11-14 12:13:14.123456 CET
  
  ```cpp
  cout << format("{:%d.%m.%Y %T%z}\n", zoned_time{tz, tp});
  ```
  > **>_** 14.11.2019 12:13:14.123556+0100
  
  ```cpp
  cout << format(locale("de_DE"), "{:%d.%m.%Y %T%z}\n", zoned_time{tz, tp});
  ```
  > **>_** 14.11.2019 12:13:14,123656+0100
  
  ```cpp
  cout << format("{:%d.%m.%Y %T%z}\n", zoned_time{tz, floor<seconds>(tp)});
  ```
  > **>_** 14.11.2019 12:13:14

  If you can `std::format` it, you can `std::chrono::parse` it back in,
  usually with the same formatting string

  ```cpp
  system_clock::time_point tp;
  cin >> parse("%d.%m.%Y %T%z", tp);
  cout << tp << std::endl;
  ```
  > **>_**
  ```
  input: 14.11.2019 12:13:14.123556+0100
  output: 2019-11-14 11:13:14.123556
  ```

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="span"></a>
## `span`

Provides a view over some continuous data:

- Does not own the data

- Never allocates/deallocates

- Very cheap to copy, recommended pass by value (similar to `string_view`)

- Does not support strides

- Can be dynamic-szed and fixed-sized

```cpp
constexpr size_t length = 10;
int data[length] = ...

span<int, length> a{data}; // fixed-size

span<int> b{data}; // dynamic-size
span<int> b{data, length}; // dynamic-size too

span<int, 20> a{data}; // compilation error
```

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>
