# include guard (#ifndef)

Created: July 15, 2024 2:30 PM

These lines prevent the contents of the header file from being included multiple times in a single compilation unit, which can cause errors and multiple definition problems.

```cpp
#ifndef STACK_H
#define STACK_H

// Contents of stack.h

#endif // STACK_H
```

1.	**#ifndef STACK_H**:

•	This line stands for “if not defined”.

•	It checks whether the macro STACK_H has already been defined. If STACK_H has not been defined, the code between #ifndef and #endif is included in the compilation.

2.	**#define STACK_H**:

•	This line defines the macro STACK_H.

•	Once this macro is defined, if the header file is included again in the same compilation unit, the #ifndef STACK_H condition will be false, and the contents of the header file will be skipped.

3.	**#endif**:

•	This line marks the end of the #ifndef block.