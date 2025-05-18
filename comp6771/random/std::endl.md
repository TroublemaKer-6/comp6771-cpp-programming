# std::endl

Created: July 17, 2024 8:18 PM

The difference between `std::endl` and `\n` 

```cpp
#include <iostream>

int main() {
    std::cout << "Hello, World\n";  // Inserts a newline, but may not flush the buffer
    std::cout << "Hello, World" << std::endl;  // Inserts a newline and flushes the buffer

    return 0;
}
```

**When to Use Which**

- Use `\n` for performance when you don’t need immediate output, especially in performance-critical applications.
- Use `std::endl` when you need to ensure that the output is immediately displayed, such as in debugging or when providing immediate feedback to the user.

After flushing the output buffer, the following events occur:

1. **Data Transmission**: All data currently stored in the buffer is immediately transmitted to the output device. This could be the console, a file, or any other output stream.
2. **Clearing the Buffer**: The buffer is typically cleared of its contents after the data has been transmitted, making it ready to accept new data for future output operations.
3. **Synchronization**: The state of the output stream is synchronized with the output device. This means that any data held in the buffer is now reflected in the output device, ensuring that the internal state of the buffer matches the actual state of the output.

**How Buffers Work in C++ Output Streams**

In C++ output streams, buffers are used to temporarily hold data before writing it to an output device. This process helps improve performance by minimizing the number of direct I/O operations, which can be slow.

**Key Points About Buffers**

1. **Individual Buffers**: Each output stream (e.g., std::cout, std::ofstream) typically has its own buffer. This means std::cout and a file stream like std::ofstream will have separate buffers.
2. **Buffered Operations**: When you use an output stream, data is first written to its buffer. The data stays in the buffer until certain conditions are met for it to be flushed to the output device.

3.	**Conditions for Flushing**:

- The buffer becomes full.
- The program explicitly requests a flush (e.g., std::flush, std::endl).
- Certain operations implicitly flush the buffer (e.g., std::cin reads after std::cout writes, std::cerr is unbuffered).
- The program terminates or the stream is closed.