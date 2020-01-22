# Designated Initializer

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

  Widget w; // Error
  Widget w{}; // OK in C++17! Will be error in C++20
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

<p align="right"><a href="../README.md#contents">:arrow_left: Back to Contents</a></p>
