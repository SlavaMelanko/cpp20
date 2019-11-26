# cpp20

Overview of C++20. Inspired by [CppCon 2019: Marc Gregoire “C++20: What's in it for you?"](https://youtu.be/Y652wQqbYEI?list=PLun8Ea0ZRc-k5F6yli7R1El0M77l52R0W) and others talks.

# Contents

1. [Modules](#modules)
1. [Ranges](#ranges)

<a name="modules"></a>
## Modules

:white_check_mark: Advantages:

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

:fire: You can import header files, e.g.

```cpp
import <iostream>
```

- Implicitely turns the `iostream` header into module
- Improves build throughput, as `iostream` will then processed only once

:mortar_board: Additional links:

- [CppCon 2019: Michael Spencer “Building Modules”](https://youtu.be/L0SHHkBenss)

- [CppCon 2019: Boris Kolpackov “Practical C++ Modules”](https://youtu.be/szHV6RdQdg8)

- [CppCon 2019: Gabriel Dos Reis “Programming with C++ Modules: Guide for the Working”](https://youtu.be/tjSuKOz5HK4)

<a name="ranges"></a>
## Ranges


