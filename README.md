# C++20 Overview

![license](https://img.shields.io/badge/License-MIT-green) ![cpp20](https://img.shields.io/badge/C%2B%2B-20-blue)

:pushpin: Please note that

1. The idea of this overview is self education
1. The main goal - keep it simple and short
1. And it is primarily based on [CppCon 2019](https://www.youtube.com/playlist?list=PLHTh1InhhwT6KhvViwRiTR7I5s09dLCSw)
and [Meeting C++ 2019](https://www.youtube.com/playlist?list=PLRyNF2Y6sca27wjBvjc5yg3F1QqZgazKb) talks

:warning: There is no guarantee that below theoretical and practical parts are correkt and up to date

# Contents

1. [Modules](modules/README.md)
1. [Ranges](ranges/README.md)
1. [Coroutines](coroutines/README.md)
1. [Concepts](concepts/README.md)
1. [Concurrency](concurrency/README.md)
1. [Calendars and Timezones](chrono/README.md)
1. [Initialization](initialization/README.md#contents)
1. [Lambda Expression](lambda/README.md)
1. [Attributes](attributes/README.md)
1. [Constant Expressions](constant/README.md)
1. [Bit Operations](bits/README.md)
1. [Source Location](source/README.md)

# Old Contents

1. [Abbreviated Function Templates](#abfubctemp)
1. [Spaceship (Three-Way Comparison) Operator <=>](#spaceship)
1. [Non-Type Template Parameters](#templ)
1. [`span`](#span)
1. [Feature-Test Macros](#testmacro)
1. [`<version>`](#version)
1. [Class Enums and `using`](#enumnusing)
1. [Text Formatting](#stdformat)
1. [`char8_t`](#char8_t)
1. [Math Constants](#math)
1. [Small Standard Library Additions](#stdadditions)
1. [`bind_front`](#bind_front)
1. [Strucutre Binding](#structure_bind)

<a name="abfubctemp"></a>
## Abbreviated Function Templates

`auto` is now allowed in function parameters

```cpp
auto Foo(auto param) { /* ... */ }
```

It is the same as

```cpp
template<typename T> auto Foo(T param) { /* ... */ }
// or
auto Foo = [](T param) { /* ... */ }
```

> **Note**: `concept auto` allowed anywhere that `auto` was allowed before

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="spaceship"></a>
## Spaceship (Three-Way Comparison) Operator <=>

:mag_right: **Example**

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

<a name="span"></a>
## `span`

  Provides bounds-safe views for sequences of objects:

  - Does not own the data

  - Never allocates/deallocates

  - Very cheap to copy, recommended pass by value (similar to `string_view`)

  - Does not support strides

  - Can be dynamic-sized and fixed-sized

  ```cpp
  constexpr size_t length = 10;
  int data[length] = ...

  span<int, length> a{data}; // fixed-size

  span<int> b{data}; // dynamic-size
  span<int> b{data, length}; // dynamic-size too

  span<int, 20> a{data}; // compilation error
  ```

  :mag_right: **Example**
  
  Before:

  ```cpp
  void DoSomething(int* p, size_t size) {
    std::sort(p, p + size);
    for (size_t i = 0; i < size; ++i) {
      p[i] += p[0];
    }
  }
  // ...
  std::vector<int> v;
  DoSomething(v.data(), v.size());

  int data[1024];
  DoSomething(data, std::size(data));
  ```

  Now:

  ```cpp
  void DoSomething(std::span<int> p) {
    std2::sort(p);
    for (int& v: p) {
      v += p[0];
    }
  }
  // ...
  std::vector<int> v;
  DoSomething(v);

  int data[1024];
  DoSomething(data);
  ```

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="testmacro"></a>
## Feature-Test Macros

Provides a simple and portable way to detect the presence of a compiler certain language and library features.

:mag_right: **Example**

```cpp
#if __cpp_constexpr >= 201304
#  define CONSTEXPR constexpr
#else
#  define CONSTEXPR inline
#endif
 
CONSTEXPR int bar(unsigned i) {
#if __cpp_binary_literals
  unsigned mask = 0b11000000;
#else
  unsigned mask = 0xC0;
#endif
  // ...
}
```

:link: **Additional Links**

- [Feature testing (C++20)](https://en.cppreference.com/w/cpp/feature_test)
- [Feature Test Recommendations](https://isocpp.org/std/standing-documents/sd-6-sg10-feature-test-recommendations)

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="version"></a>
## `<version>`

Supplies implementation-dependent information about the standard library:

- Version number
- Release date
- Copyright notice
- Additional implementation-defined information
- Include the library feature-test macros

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="enumnusing"></a>
## Class Enums and `using`

:mag_right: **Example**

Before

```cpp
enum class RgbColor { Red, Green, Blue };

std::string_view ColorToString(const Color color) {
  switch (color) {
    case RgbColor::Red: return "Red";
    case RgbColor::Green: return "Green";
    case RgbColor::Blue: return "Blue";
  }
}
```

The necessary repetition of the `enum class` name reduces legibility by introducing noise in contexts where said name is obvious.

Now

```cpp
std::string_view ColorToString(const Color color) {
  switch (color) {
    using enum RgbColor; // introduce the enumerator identifiers into the local scope
    case Red: return "Red";
    case Green: return "Green";
    case Blue: return "Blue";
  }
}
```

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="stdformat"></a>
## Text Formatting

`std::format` provides a fast, simple and safe alternative to C stdio and C++ iostreams with pythonic string syntax.

```cpp
std::cout << std::format("Hello, {}!", "world");
```

> **>_** Hello, world!

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="char8_t"></a>
## `char8_t`

`char8_t` is a new fundamental type to capture UTF-8 data that

- Never aliases with other types because it is a distinct type
  ```cpp
  cout << std::boolalpha
    << is_same_v<char, char8_t> << ' '
    << is_same_v<unsigned char, char8_t>;
  ```
  > **>_** false false

- Is always unsigned

- Has the same size and alignment as `char` and `signed char`
  ```cpp
  cout << sizeof(char) << " == " << sizeof(signed char) << " == " << sizeof(char8_t);
  ```
  > **>_** 1 == 1 == 1

- Can be overloaded upon
  ```cpp
  void print(std::u8string_view data);
  void print(std::string_view data);
  
  process("Hello, World!");
  process(u8"Hello, 🌎!");
  ```

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="math"></a>
## Math Constants

Following mathematical constant are defined:

- `e`, `log2e`, `log10e`
- `pi`, `inv_pi`, `inv_sqrtpi`
- `ln2`, `ln10`
- `sqrt2`, `sqrt3`, `inv_sqrt3`
- `egamma`
- `phi`

:link: **Additional Links**

- [Mathematical constants](https://en.cppreference.com/w/cpp/numeric/constants)

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="stdadditions"></a>
## Small Standard Library Additions

- `starts_with` and `ends_with` for `basic_string`/`basic_string_view`
- `contains` for associative containers
- `remove`, `remove_if`, and `unique` methods  for `list` and `forward_list` now return `size_type` (number of removed elements) insted of `void`
- `erase` and `erase_if` algorithms equivalent to `c.erase(std::remove_if(c.begin(), c.end(), pred), c.end());` now
- `shift_left` and `shift_right` added to `<algorithm>`
- `midpoint` to calculates the midpoint of two numbers

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="bind_front"></a>
## `bind_front`

`bind_front` function is designed to bound first arguments of the function to some callable wrapper.
In other worlds

```cpp
bind_front(f, bound_args...)(call_args...)
```

is equivalent to

```cpp
std::invoke(f, bound_args..., call_args....)
```

:mag_right: **Example**

```cpp
struct Strategy {
    double calculate(double discount, double price) {
        return price - ((discount / 100) * price);
    }
};

unique_ptr<Strategy> CreateStrategy() {
    return make_unique<Strategy>();
}

int main() {
    auto apply20PercentDiscount = bind_front(&Strategy::calculate, CreateStrategy(), 20);
    cout << apply20PercentDiscount(100);
}
```
> **>_** 80

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="structure_bind"></a>
## Strucutre Binding

```cpp
struct Result {
  bool status;
  std::string data;
};

Result Process();

int main()
{
  auto [ok, data] = Process();
  static [ok, data] = Process(); // ok in C++20
  thread_local [ok, data] = Process(); // ok in C++20

  auto success = [ok] { return ok == true; } // ok in C++20
}
```

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>
