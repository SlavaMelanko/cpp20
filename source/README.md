# Source Location

Represents specific information about source code, such as line and column numbers, file and function names.

```cpp
void Log(string_view message, const source_location& location = source_location::current())
{
  clog << "info: "
    << location.file_name() << ":"
    << location.line() << ": "
    << message << endl;
}

int main() {
  Log("Log entry");
}
```
> **>_** info: ./example.cpp:18: Log entry

<p align="right"><a href="../README.md#contents">:arrow_left: Back to Contents</a></p>
