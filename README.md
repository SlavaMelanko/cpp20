# cpp20

Overview of C++20. Inspired by [CppCon 2019: Marc Gregoire “C++20: What's in it for you?"](https://youtu.be/Y652wQqbYEI?list=PLun8Ea0ZRc-k5F6yli7R1El0M77l52R0W) and others talks.

# Contents

1. [Modules](#modules)
1. [Ranges](#ranges)

<a name="modules"></a>
## Modules

:white_check_mark: **Advantages**:

- A way to organize, encapsulate, and isolate your code

- No need of header files

  > Separation into interface and implementation files is possible but it is obsolete

- Better compilation times

  > Modules are processed only once. Compare this with M headers which are included in N translation units.
  > The combinatorial explosion means, that the header has to be parsed M*N times.

- No need for include guard

- No need to invent unique names

  > Same names in multiple modules will not clash

- Modules explicitely state what should be exported (e.g. functions, classes, ...)

- Preprocessor usage elimination

- The order of `import` statements will not matter

:mag_right: **Example** (:warning: not tested yet):

```cpp
// cppcon.cpp/.cppm/.mpp
export module cppcon;

namespace CppCon {

auto GetWelcomeMessage() {
  return "Welcome to C++20";
}

export auto GetWelcome() {
  return GetWelcomeMessage();
}

} // namespace CppCon

// main.cpp/.cppm/.mpp
import cppcon;

import <iostream>;

int main() {
  std::cout << CppCon::GetWelcome();
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

:mortar_board: **Additional links**:

- :movie_camera:
  - [CppCon 2019: Michael Spencer “Building Modules”](https://youtu.be/L0SHHkBenss)
  - [CppCon 2019: Boris Kolpackov “Practical C++ Modules”](https://youtu.be/szHV6RdQdg8)
  - [CppCon 2019: Gabriel Dos Reis “Programming with C++ Modules: Guide for the Working”](https://youtu.be/tjSuKOz5HK4)
  - [Core C++ 2019 :: Bryce Adelstein :: Modules are Coming](https://youtu.be/bDTm6y6fNSU)

<a name="ranges"></a>
## Ranges

:zap: Range is an object referring to a sequence of elements.
Similar to begin/end iterators but not replace them.
It provides a nicer and easier to read syntax.

:mag_right: **Example** (:warning: not tested yet):

```cpp
std::array<int, 5> data{2, 4, 5, 1, 3};
std::sort(std::begin(data), std::end(data)); // before
std::ranges::sort(data) // now
```

Based on 3 core components:
- **Views** are ranges with 'lazy evaluation', non-owning, and non-mutating of elements, with constant time for copying and moving
```cpp
std::vector<int> data{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
auto result = data | std::viewes::remove_if([](int i) { return i % 2 == 1; })
                   | std::viewes::transform([](int i) { return std::to_string(i) });
```
> result == {"2", "4", "6", "8", "10"}
```cpp
std::vector<int> data{0, 1, 2, 3, 4, 5, 6};
auto evens = data | std::views::filter([](int i) { return i % 2 == 0; });
```
> evens == {0, 2, 4, 6}
- **Actions**: eagerly evaluated, mutting
```cpp
std::vector<int> data{4, 3, 4, 1, 8, 0, 8};
auto result = data | std::actions::sort | std::actions::unique;
```
> result == {0, 1, 3, 4, 8}
Also, shortcut version
```cpp
std::vector<int> data{4, 3, 2, 1, 0};
data |= std::actions::sort;
```
> data == {0, 1, 2, 3, 4}
- **Algorithms**: all Standard Library algorithms accepting ranges instead of iterator pairs

:bulb: Projection

```cpp
struct Employee {
  std::string firstName;
  std::string lastName;
};

std::vector<Employee> employees = {...};

std::ranges::sort(employees, {}, &Employee::lastName);
```

:mortar_board: **Additional links**:

- :newspaper:
  - [Range-v3: User Manual](https://ericniebler.github.io/range-v3/)

- :movie_camera:
  - [CppCon 2019: Dvir Yitzchaki “Range Algorithms, Views and Actions: A Comprehensive Guide”](https://youtu.be/qQtS50ZChN8)
  - [CppCon 2019: Jeff Garland “From STL to Ranges: Using Ranges Effectively”](https://youtu.be/vJ290qlAbbw)
