# [[nodiscard]]

Created: June 19, 2024 2:16 PM

The `[[nodiscard]]` attribute in C++ is used to indicate that the return value of a function should not be ignored. This attribute can help prevent bugs by ensuring that important return values are checked and handled appropriately. If a function marked with `[[nodiscard]]` is called and its return value is discarded, the compiler will generate a warning.

```cpp
#include <iostream>

[[nodiscard]] int calculateSomething(int a, int b) {
    return a + b;
}

int main() {
    calculateSomething(3, 4); // This will generate a compiler warning
    int result = calculateSomething(3, 4); // This is fine, as the return value is used
    std::cout << "Result: " << result << std::endl;
    return 0;
}
```