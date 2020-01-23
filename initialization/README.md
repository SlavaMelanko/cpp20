# Contents

1. [Designated initializer](#designated)
1. [`constinit`](#constinit)
1. [Range-based `for` loop initializer](#forloop)

<p align="right"><a href="../README.md#contents">:arrow_left: Back to Contents</a></p>

<a name="designated"></a>
## Designated initializer

```cpp
struct User {
  std::string email;
  std::string password;
  int pin;
};

int main() {
  User jason{"jason@mail.com", "Passw0rd!", 1234}; // good old struct initialization
  User jane{.email = "jane@mail.com", .password = "Passw0rd!"}; // designated initialization
}
```

- Only for aggregate types

  An aggregate is an array or a class with

  - no user-declared or inherited constructors

  - no private or protected non-static data members

  - no virtual functions

  - no virtual, private, or protected base classes

  ```cpp
  struct Widget {
    Widget() = delete;
  };

  Widget w; // error
  Widget w{}; // ok in C++17 whereas in C++20 it is error
  ```

- All designators used in the expression must appear in the same order as the data members, e.g.

```cpp
User user{.password = "Passw0rd!", .email = "user@mail.com"};
```

> error: ISO C++ requires field designators to be specified in declaration order ...

- Also, cannot be mixed with regular initializers

```cpp
User user{.email = "user@mail.com", "Passw0rd!"};
```

> error: mixture of designated and non-designated initializers in the same initializer list ...

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="constinit"></a>
## `constinit`

`constinit` specifies that a variable must have a static initialization.

`constinit` cannot be used together with `constexpr` or `consteval`.

```cpp
const char* AllocateDynamicString() {
  return "Dynamic init";
}

constexpr const char* GetString(bool isConstInit) {
  return isConstInit ? "Const init" : AllocateDynamicString();
}

constinit const char* str = GetString(true); // ok
constinit const char* str = GetString(false); // error: variable does not have a constant initializer
```

:paperclip: `constinit` can only be applied to variables with static or thread storage duration.
It does not make sense to apply it to other variables, as `constinit` is all about static initialization.

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="forloop"></a>
## Range-based `for` loop initializer

```cpp
for (const auto& user : GetDatabase().selectActiveUsers()) { // not good enough
  // ...
}
```

> Maybe undefined behaviour because `GetDatabase()` might returns a reference.

```cpp
{
  auto db = GetDatabase();
  for (const auto& user : db.selectActiveUsers()) { // good
    // ...
  }
}
```

```cpp
for (const auto db = GetDatabase(); const auto& user : db.selectActiveUsers()) { // better
  // ...
}
```

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>
