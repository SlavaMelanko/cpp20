# Bit Operations

`<bit>` contains a set of global non-member functions to operate on bits

- `rotl` and `rotr` - computes the result of bitwise left- & right-rotation
- `countl_zero` counts the number of consecutive 0 bits, starting from most significant bit
- `countl_one` counts the number of consecutive 1 bits, starting from most significant bit
- `countr_zero` counts the number of consecutive 0 bits, starting from least significant bit
- `countr_one` counts the number of consecutive 1 bits, starting from least significant bit
- `popcount` counts the number of 1 bits

```cpp
for (uint8_t number : {0, 0b00011100, 1}) {
  cout << "countl_zero(0b" << bitset<8>(number) << ") = " << countl_zero(number) << '\n';
  cout << "countr_zero(0b" << bitset<8>(number) << ") = " << countr_zero(number) << '\n';
}
```
  ```
  >_
  countl_zero(0b00000000) = 8
  countr_zero(0b00000000) = 8
  countl_zero(0b00011100) = 3
  countr_zero(0b00011100) = 2
  countl_zero(0b00000001) = 7
  countr_zero(0b00000001) = 0
  ```

<p align="right"><a href="../README.md#contents">:arrow_left: Back to Contents</a></p>
