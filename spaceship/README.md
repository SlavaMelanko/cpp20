# Spaceship (Three-Way Comparison) Operator <=>

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

<p align="right"><a href="../README.md#contents">:arrow_left: Back to Contents</a></p>
