# cpp20

Overview of C++20. Inspired by [CppCon 2019: Marc Gregoire “C++20: What's in it for you?"](https://youtu.be/Y652wQqbYEI?list=PLun8Ea0ZRc-k5F6yli7R1El0M77l52R0W) and others talks.

# Contents

1. [Modules](#modules)
1. [Ranges](#ranges)

<a name="modules"></a>
## Modules

:white_check_mark: No header files

:white_check_mark: No need for include guard

:white_check_mark: No need to invent unique names, same names in multiple modules will not clash

:white_check_mark: Separation into interface and implementation files is possbile but it is obsolete

:white_check_mark: Modules explicitely state what should be exported (e.g. functions, classes, ...)

:white_check_mark: Modules are processed only once, therefore **faster build time**

:white_check_mark: Preprocessor macros have no effect on modules 

:white_check_mark: Order of module inports in not important

```cpp
// cppcon.cpp
export module cppcon;

namespace CppCon {

auto GetWelcomeMessage() {
  return "Welcome to C++20";
}

export auto GetWelcome() {
  return GetWelcomeMessage();
}

} // namespace CppCon

// main.cpp
import cppcon;

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

<a name="ranges"></a>
## Ranges


