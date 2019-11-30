# C++20

![license](https://img.shields.io/badge/License-MIT-green) ![cpp20](https://img.shields.io/badge/C%2B%2B-20-blue)

Overview of C++20. Inspired by 

* [CppCon 2019: Bjarne Stroustrup “C++20: C++ at 40”](https://youtu.be/u_ij0YNkFUs)
* [CppCon 2019: Marc Gregoire “C++20: What's in it for you?"](https://youtu.be/Y652wQqbYEI)
* [CppCon 2019: Jeff Garland “The C++20 Standard Library: Beyond Ranges”](https://youtu.be/fI19ttpUNRQ)

and others talks.

# Contents

1. [Modules](#modules)
1. [Ranges](#ranges)
1. [Coroutines](#coroutines)

<a name="modules"></a>
## Modules

:white_check_mark: Module is a way to organize, encapsulate, and isolate your code.

Advantages:

- No need of header files
  > Separation into interface and implementation files is possible but it is obsolete

- Better compilation times
  > Modules are processed only once. Compare this with M headers which are included in N translation units.
  > The combinatorial explosion means, that the header has to be parsed M*N times.

- No need for include guard

- No need to invent unique names
  > Same names in multiple modules will not clash

- Preprocessor usage elimination

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

:bulb: You can import header files, e.g.

```cpp
import <iostream>;
```

- Implicitely turns the `iostream` header into module
- Improves build throughput, as `iostream` will then processed only once

:scroll: **Structure**:

| Module (top to bottom) |
| :---: |
| `module;` |
| preprocessor derictives only, e.g `#include <cassert>` |
| `export module name;` |
| `import ...;` |
| `...` |
| `module : private;` |
| `...` |

:link: **Additional links**:

- :movie_camera:
  - [CppCon 2019: Michael Spencer “Building Modules”](https://youtu.be/L0SHHkBenss)
  - [CppCon 2019: Boris Kolpackov “Practical C++ Modules”](https://youtu.be/szHV6RdQdg8)
  - [CppCon 2019: Gabriel Dos Reis “Programming with C++ Modules: Guide for the Working”](https://youtu.be/tjSuKOz5HK4)
  - [Core C++ 2019 :: Bryce Adelstein :: Modules are Coming](https://youtu.be/bDTm6y6fNSU)

:arrow_up: [Back to top](#contents)

<a name="ranges"></a>
## Ranges

:white_check_mark: Range is an object referring to a sequence of elements.
Similar to begin/end iterators but not replace them.
It provides a nicer and easier to read syntax.

> **Note**: all below examples were tested using [range-v3](https://github.com/ericniebler/range-v3) library.

:mag_right: **Example**:

```cpp
std::array<int, 5> data{2, 4, 5, 1, 3};
std::sort(std::begin(data), std::end(data)); // before
ranges::sort(data) // now
```

Ranges library is based on 3 core components:

### Views

Views are ranges with 'lazy evaluation', non-owning, and non-mutating of elements,
with constant time for copying and moving

```cpp
std::vector<int> data{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
auto result = data
        | ranges::views::remove_if([](int i) { return i % 2 == 1; })
        | ranges::views::transform([](int i) { return std::to_string(i); });
```
> result = ["2", "4", "6", "8", "10"]

```cpp
std::vector<int> data{0, 1, 2, 3, 4, 5, 6};
auto evens = data | ranges::views::filter([](auto i) { return i % 2 == 0; });
std::cout << ranges::views::all(evens) << '\n';
```
> [0, 2, 4, 6]

### Actions

 Actions are ranges which are eagerly evaluated, mutating the data, and can be composed as views

```cpp
std::vector<int> data{4, 3, 4, 1, 8, 0, 8};
data |= ranges::actions::sort | ranges::actions::unique;
```
> result = [0, 1, 3, 4, 8]

```cpp
std::vector<int> original{4, 3, 4, 1, 8, 0, 8};
auto modified = original | ranges::copy | ranges::actions::sort | ranges::actions::unique;
```
> original = [4,3,4,1,8,0,8]

> modified = [0,1,3,4,8]

### Algorithms

Algorithms - all standard library algorithms accepting ranges instead of iterator pairs

```cpp
auto sequence = ranges::views::ints(1, 11)
  | ranges::views::transform([](const auto i) { return i * i; });
const auto sum = ranges::accumulate(sequence, 0);
```
> sum = 385

:bulb: Projection:

```cpp
struct Employee {
  std::string firstName;
  uint8_t age;
};
    
std::vector<Employee> employees = {{"Jason", 50}, {"Jane", 40}};
    
ranges::sort(employees, {}, &Employee::age);
```
> employees = {{"Jane", 40}, {"Jason", 50}}

:link: **Additional links**:

- :newspaper:
  - [Range-v3: User Manual](https://ericniebler.github.io/range-v3/)

- :movie_camera:
  - [CppCon 2019: Dvir Yitzchaki “Range Algorithms, Views and Actions: A Comprehensive Guide”](https://youtu.be/qQtS50ZChN8), [code samples](https://github.com/dvirtz/ranges_code_samples)
  - [CppCon 2019: Jeff Garland “From STL to Ranges: Using Ranges Effectively”](https://youtu.be/vJ290qlAbbw)

:arrow_up: [Back to top](#contents)

<a name="coroutines"></a>
## Coroutines

:white_check_mark: Coroutines are generalised functions that can be suspended and resumed while keeping their state.
Coroutine must have one of the following keywords:

- **co_await** suspends evaluation of a coroutine while waiting for a computation to finish

- **co_return** returns from a coroutine
  > Just `return` is not allowed

- **co_yield** returns a value from a coroutine back to the caller, and suspends the coroutine,
subsequently calling the coroutine again continues its execution

Coroutines might be used for:

- Generators
- Asynchronous I/O
- Lazy computations
- Event driven applications (simulations, games, servers, user interfaces, or even algorithms)

C++20 contains language additions to support coroutines, whereas
standard library does not include helper classes yet such as generators.

:mag_right: **Example**:

> **Note**: Below example was tested with [godbolt](https://godbolt.org/z/GUSdpQ) (x64 msvc v19.22)

```cpp
#include <experimental/coroutine>
#include <experimental/generator>

std::experimental::generator<int> CoroutineGenerator(const int iterations)
{
    for (int i = 0; i < iterations; i++) {
        co_yield i;
    }
}

int main()
{
    int total = 0;

    for (auto i : CoroutineGenerator(5)) {
        total += i;
    }

    return total;
}
```

? A range-based `for co_wait` loop: `for co_await (declaration : expression) statement`

:arrow_up: [Back to top](#contents)
