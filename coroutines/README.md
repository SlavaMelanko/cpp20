# Coroutines

> :clock1: The term coroutine was introduced by Melvin Conway in 1958.

:bulb: Coroutines are generalised functions that:

- Can suspend execution
- Return an intermediate value
- Resume later
- Preserve local state
- Allow re-entry more than once
- Non-pre-emptive -> Cooperative?

Coroutines must have one of the following keywords:

- **co_await** suspends evaluation of a coroutine while waiting for a computation to finish

- **co_return** returns from a coroutine
  > Just `return` is not allowed

- **co_yield** returns a value from a coroutine back to the caller, and suspends the coroutine,
subsequently calling the coroutine again continues its execution

Coroutines might be used for:

- Asynchronous I/O, e.g. servers
- Lazy computations, e.g. generators
- Event driven applications, e.g. simulations, games, user interfaces, or even algorithms

> **Note**: C++20 contains language additions to support coroutines, whereas
> standard library does not include helper classes yet such as generators.

> **Note**: Below example was tested with [godbolt](https://godbolt.org/z/GUSdpQ) (x64 msvc v19.22)

```cpp
#include <experimental/coroutine>
#include <experimental/generator>

std::experimental::generator<int> CoroutineGenerator(const int iterations) {
    for (int i = 0; i < iterations; i++) {
        co_yield i;
    }
}

int main() {
    int total = 0;

    for (auto i : CoroutineGenerator(5)) {
        total += i;
    }

    return total;
}
```

:paperclip: A range-based `for co_wait` loop: `for co_await (declaration : expression) statement`

<p align="right"><a href="../README.md#contents">:arrow_left: Back to Contents</a></p>
