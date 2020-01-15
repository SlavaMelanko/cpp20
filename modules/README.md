# Modules

:bulb: Module is a way to organize, encapsulate, and isolate your code.

Advantages:

- Better compilation times
  > Modules are processed only once. Compare this with M headers which are included in N translation units.
  > The combinatorial explosion means, that the header has to be parsed M * N times.

- No need of header files
  > Separation into interface and implementation files is possible but it is obsolete

- No need for include guard

- Preprocessor usage elimination

- No need to invent unique names
  > Same names in multiple modules will not clash

- The order of `import` statements will not matter

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
  cout << hello::SayWelcome();
}
```

:paperclip: You can import header files, e.g.

```cpp
import <iostream>;
```

- Implicitely turns the `iostream` header into module
- Improves build throughput, as `iostream` will then processed only once

:clipboard: **Structure**

  | Module (top to bottom) |
  | :---: |
  | `module;` |
  | preprocessor derictives only, e.g `#include <cassert>` |
  | `export module name;` |
  | `import ...;` |
  | `...` |
  | `module : private;` |
  | `...` |

<p align="right"><a href="#contents">:arrow_left: Back to Contents</a></p>
