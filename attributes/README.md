# Contents

1. [`likely` and `unlikely`](#likely)
1. [`nodiscrad` with the reason](#nodiscrad)
1. [`no_unique_address`](#no_unique_address)

<p align="right"><a href="../README.md#contents">:arrow_left: Back to Contents</a></p>

<a name="likely"></a>
## `likely` and `unlikely`

Hints for the compiler to optomize certain branches.

```cpp
switch (value) {
  case 1:
    // ...
    break;
  [[likely]] case 2:
    // ...
    break;
  [[unlikely]] case 3:
    // ...
    break;
}
```

```cpp
// ...

if (err) [[unlikely]] {
  return;
}

// ...
```

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="nodiscrad"></a>
## `nodiscrad` with the reason

```cpp
nodiscrad[["Ignoring a return value will permit memory leak"]] Window * Create();
```

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="no_unique_address"></a>
## `no_unique_address`

Indicates that if the member has an empty type, the compiler may optimise it to occupy no space.
If the member is not empty, any tail padding in it may be also reused to store other data members.

```cpp
struct X {
  int i;
  Empty e;
};
 
struct Y {
  int i;
  [[no_unique_address]] Empty e;
};
 
struct Z {
  int i;
  [[no_unique_address]] Empty e1, e2;
};

int main() {
  // The size of any object of empty class type is at least 1.
  cout << sizeof(Empty) << '\n';

  // At least one more byte is needed to give e a unique address.
  cout << sizeof(X) << '\n';

  // Empty member optimized out.
  cout << "sizeof(Y) == sizeof(int) == " << sizeof(int) << " is "
      << std::boolalpha << (sizeof(Y) == sizeof(int)) << '\n';

  // e1 and e2 cannot share the same address because they have the same type,
  // even though they are marked with [[no_unique_address]]. 
  // However, either may share address with c.
  cout << "sizeof(Z) == " << sizeof(Z) << '\n';
}
  ```

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>
