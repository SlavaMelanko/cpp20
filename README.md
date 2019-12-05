# C++20 Overview

![license](https://img.shields.io/badge/License-MIT-green) ![cpp20](https://img.shields.io/badge/C%2B%2B-20-blue)

:pushpin: Please note that

1. The idea of this overview - self education
1. The main goal - keep it simple and short
1. Overview is primarily based on [CppPon 2019 talks](https://www.youtube.com/playlist?list=PLHTh1InhhwT6KhvViwRiTR7I5s09dLCSw)

:warning: There is no guarantee that below theoretical and practical parts are correct and up to date

# Contents

1. [Modules](#modules)
1. [Ranges](#ranges)
1. [Coroutines](#coroutines)
1. [Concepts](#concepts)

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

:paperclip: Projection:

```cpp
struct Employee {
  std::string firstName;
  uint8_t age;
};
    
std::vector<Employee> employees = {{"Jason", 50}, {"Jane", 40}};
    
ranges::sort(employees, {}, &Employee::age);
```
> employees = {{"Jane", 40}, {"Jason", 50}}

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

Coroutine must have one of the following keywords:

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
