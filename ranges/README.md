# Ranges

:bulb: Range is an object referring to a sequence of elements.
It is additional (abstraction) layer on top of iterators
that provides a nicer and easier to read syntax.

> **Note**: all below examples were tested using [range-v3](https://github.com/ericniebler/range-v3) library.

```cpp
array<int, 5> data{2, 4, 5, 1, 3};
sort(begin(data), end(data)); // before
ranges::sort(data) // now
```

Ranges library is based on 3 core components:
- Views
- Actions
- Algorithms

## Views

Ranges with 'lazy evaluation', non-owning, and non-mutating of elements,
with constant time for copying and moving

```cpp
vector<int> data{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
auto result = data
  | ranges::views::remove_if([](int i) { return i % 2 == 1; })
  | ranges::views::transform([](int i) { return to_string(i); });
```
> **>_** result = ["2", "4", "6", "8", "10"]

```cpp
vector<int> data{0, 1, 2, 3, 4, 5, 6};
auto evens = data | ranges::views::filter([](auto i) { return i % 2 == 0; });
cout << ranges::views::all(evens) << '\n';
```
> **>_** [0, 2, 4, 6]

## Actions

 Ranges which are eagerly evaluated, mutating the data, and can be composed as views

```cpp
vector<int> data{4, 3, 4, 1, 8, 0, 8};
data |= ranges::actions::sort | ranges::actions::unique;
```
> **>_** result = [0, 1, 3, 4, 8]

```cpp
vector<int> original{4, 3, 4, 1, 8, 0, 8};
auto modified = original | ranges::copy | ranges::actions::sort | ranges::actions::unique;
```
> **>_** original = [4,3,4,1,8,0,8]

> **>_** modified = [0,1,3,4,8]

## Algorithms

All standard library algorithms accepting ranges instead of iterator pairs

```cpp
auto sequence = ranges::views::ints(1, 11)
  | ranges::views::transform([](const auto i) { return i * i; });
const auto sum = ranges::accumulate(sequence, 0);
```
> **>_** sum = 385

:paperclip: Projection:

```cpp
struct Employee {
  string firstName;
  uint8_t age;
};
    
vector<Employee> employees = {{"Jason", 50}, {"Jane", 40}};
    
ranges::sort(employees, {}, &Employee::age);
```
> **>_** employees = {{"Jane", 40}, {"Jason", 50}}

<p align="right"><a href="../README.md#contents">:arrow_left: Back to Contents</a></p>
