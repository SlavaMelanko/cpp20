# Concepts

:bulb: Requirements that can be attached to class templates and function templates to constraint the template arguments.

## Concept definitions

```cpp
template<typename T>
concept Incrementable = requires(T t) { ++t; t++; };

template <typename T>
concept Integral = is_integral<T>::value;

template<typename T>
concept HasSize = requires(T t) {
    { t.size() } -> convertible_to<size_t>;
};
```

## Usage

- Constrained template parameters:
```cpp
template<Incrementable T>
void Foo(T t);
```
- Requires clause:
```cpp
template<typename T> requires Incrementable<T> && Decrementable<T>
void Foo(T t);
```
- Trailing requires clause:
```cpp
template<typename T>
void Foo(T t) requires Incrementable<T> && Decrementable<T>;
```
- Placeholder syntax:
```cpp
void Foo(Incrementable auto t);
```

<p align="right"><a href="../README.md#contents">:arrow_left: Back to Contents</a></p>