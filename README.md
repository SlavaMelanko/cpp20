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
import <iostream>
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

- [CppCon 2019: Michael Spencer “Building Modules”](https://youtu.be/L0SHHkBenss)
- [CppCon 2019: Boris Kolpackov “Practical C++ Modules”](https://youtu.be/szHV6RdQdg8)
- [CppCon 2019: Gabriel Dos Reis “Programming with C++ Modules: Guide for the Working”](https://youtu.be/tjSuKOz5HK4)
- [Core C++ 2019 :: Bryce Adelstein :: Modules are Coming](https://youtu.be/bDTm6y6fNSU)

<a name="ranges"></a>
## Ranges

:zap: Range is an object referring to a sequence/range of elements.
Similar to begin/end iterators but not replace them.
It provides a nicer and easier to read syntax, e.g.

```cpp
std::vector<int> data{1, 2, 3, 4, 5};
std::sort(std::begin(data), std::end(data)); // before
std::sort(data) // now
```

Based on 3 core components:
- **Views**: range adaptors: lazily evaluated, non-owning, non-mutating
```cpp
std::vector<int> data{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
auto result = data | viewes::remove_if([](int i) { return i % 2 == 1; })
                   | viewes::transform([](int i) { return std::to_string(i) });
// result == {"2", "4", "6", "8", "10"}
```
- **Actions**: eagerly evaluated, mutting
```cpp
std::vector<int> data{4, 3, 4, 1, 8, 0, 8};
auto result = data | actions::sort | actions::unique
```
- **Algorithms**: all Standard Library algorithms accepting ranges instead of iterator pairs
