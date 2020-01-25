# Contents

1. [`constexpr`](#constexpr)
1. [`consteval`](#consteval)

<p align="right"><a href="../README.md#contents">:arrow_left: Back to Contents</a></p>

## `constexpr`

- C++20 allows virtual functions to be declared `constexpr`

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

- `string`, `string_view`, `vector`, and `tuple` are now `constexpr` as well as `sort`, `all_of`, ... algorithms

- Use `dynamic_cast` and `typeid`

- Do dynamic memory allocations and deallocations

- Change the active member of a `union`

- Contain `try/catch` blocks

  * But `throw` statements are still not allowed

  * `try/catch` blocks are no-ops when evaluated in a constant expression

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

## `consteval`

`consteval` declares immediate functions that must produce a compile time constant expression,
a non-constant result should be a compilation error.

```cpp
consteval auto Square(int number) {
  return number * number;
}

auto number = 10;
auto value = Square(number); // error: the value of 'number' is not usable in a constant expression

const auto number = 10;
auto value = Square(number); // ok: everything is constant

constexpr auto number = 10;
constexpr auto value = Square(number); // ok
```

:paperclip: As a comparison, `constexpr` function may be evaluated at compile time and runtime, and need not produce a constant in all cases.

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>
