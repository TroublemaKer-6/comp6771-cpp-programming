# w8lec

Tags: lec
Status: Done
contents: template

# Polymorphism & Generic Programming

***Polymorphism*:** Provision of a single interface to entities of different types

Two types:

- ***static***
    - Function overloading
    - Templates (i.e. generic programming)
        - std::vector<int>
        - std::vector<double>
- ***dynamic*:** related to virtual functions and inheritance

***Generic Programming*:** Generalising software components to be independent of a particular type

- STL is a great example of generic programming

# Function Templates

Without generic programming, to create two logically identical functions that behave in a way that is independent to the type, we have to rely on function overloading.

```cpp
#include <iostream>

auto min(int a, int b) -> int {
	return a < b ? a : b;
}

auto min(double a, double b) -> double {
	return a < b ? a : b;
}

auto main() -> int {
	std::cout << min(1, 2) << "\n";
	std::cout << min(1.0, 2.0) << "\n";
}
```

***Function template*:** Prescription (i.e. instruction) for the compiler to generate particular instances of a function varying by type.

The generation of a templated function for a particular type T ***only happens when a call to that function is seen during compile time***.

```cpp
#inlcude <iostream>

// if we only write this, C++ will not compile
template <typename T>
auto min(T a, T b) -> T {
	return a < b ? a : b;
}

auto main() -> int {
	// when compiler sees someone is using the template,
	// it checks the types and generate the function for this type
	std::cout << min(1, 2) << "\n";
	std::cout << min(1.0, 2.0) << "\n";
}
```

```cpp
// does not compile
template <typename T>
auto min(T a, T b) -> T {
	return a < b ? a : b;
}

// does compile
auto min(int a, int b) -> int {
	return a < b ? a : b;
}
```

`T` is a template type parameter.

`template <typename>` is the template parameter list.

# Type and Non-type Parameters

**Type parameter:** unknown type with no value

**Non-type parameter:** known type with unknown value

```cpp
#include <array>
#include <iostream>

// pass through a mixture of type parameter and non-type parameter
template<typename T, std::size_t size>
auto findmin(const std::array<T, size> a) -> T {
	T min = a[0];
	for (std::size_t i = 1; i < size; ++i) {
		if (a[i] < min) min = a[i];
	}
	return min;
}

auto main() -> int {
	std:array<int, 3> x{3, 1, 2};
	std::array<double, 4> y{3.3, 1.1, 2.2, 4.4};
	std::cout << "min of x = " << findmin(x) << "\n";
	std::cout << "min of y = " << findmin(y) << "\n";
}
```

The above example generates the following functions **at compile time**

```cpp
auto findmin(const std::array<int, 3> a) -> int {
	int min = a[0];
	for (std::size_t i = 1; i < 3; ++i) {
		if (a[i] < min) min = a[i];
	}
	return min;
}

auto findmin(const std::array<double, 4> a) -> double {
	double min = a[0];
	for (std::size_t i = 1; i < 4; ++i) {
		if (a[i] < min) min = a[i];
	}
	return min;
}
```

A bit more for understanding …

```cpp
// this will work for any type of array with length 3
template<typename T, 3>
auto findmin(const std::array<T, 3> a) -> T {
	T min = a[0];
	for (std::size_t i = 1; i < 3; ++i) {
		if (a[i] < min) min = a[i];
	}
	return min;
}

auto main() -> int {
	// array with size 3 still works, 
	// but array with size 4 will not work 
	std:array<int, 3> x{3, 1, 2};
	// std::array<double, 4> y{3.3, 1.1, 2.2, 4.4};
	std::cout << "min of x = " << findmin(x) << "\n";
	// std::cout << "min of y = " << findmin(y) << "\n";
}
```

# Class Templates

Make a Stack of multiple types

```cpp
#ifndef STACK_H
#define STACK_H

#include <iostream>
#include <vector>

template<typename T>
class stack {
public:
	friend auto operator<<(std::ostream& os, const stack& s) -> std::ostream& {
		for (const auto& i : s.stack_) 
			os << i << " ";
		return os;
	}
	auto push(T const& item) -> void;
	auto pop() -> void;
	auto top() -> T&;
	auto top() const -> const T&;
	auto empty() const -> bool;
	
private:
	std::vector<T> stack_;
};

#endif // STACK_H
```

```cpp
#include "./class-template-start1.cpp"

template<typename T>
auto stack<T>::push(T const& item) -> void {
	stack_.push_back(item);
}

template<typename T>
auto stack<T>::pop() -> void {
	stack_.pop_back();
}

template<typename T>
auto stack<T>::top() -> T& {
	return stack_.back();
}

template<typename T>
auto stack<T>::top() const -> T const& {
	return stack_.back();
}

template<typename T>
auto stack<T>::empty() const -> bool {
	return stack_.empty();
}
```

```cpp
#include <iostream>
#include <string>

#include "./class-template-start2.cpp"

int main() {
	stack<int> s1; // int: template argument
	s1.push(1);
	s1.push(2);
	stack<int> s2 = s1;
	std::cout << s1 << s2 << '\n';
	s1.pop();
	s1.push(3);
	std::cout << s1 << s2 << '\n';
	
	// s1.push("hello"); // fails to compile
	
	stack<std::string> string_stack;
	string_stack.push("hello");
	// string_stack.push(1); // fails to compile
}
```

# Inclusion Compilation Model

When it comes to templates, we ***include definitions (i.e. implementation) in the .h file***.

- This is because template definitions need to be known at compile time (template definitions can’t be instantiated at link time because that would require an instantiation for all types)

Will expose implementation details in the .h file.

Can cause slowdown in compilation as every file using min.h will have to instantiate the template, then it’s up the linker to ensure there is only 1 instantiation.

```cpp
template <typename T>
auto min(T a, T b) -> T;
```

```cpp
#include <iostream>

auto main() -> int {
	std::cout << min(1, 2) << "\n";
}
```

```cpp
template <typename T>
auto min(T a, T b) -> int {
	return a < b ? a : b;
}

template int min<int>(int, int);

```

Lazy instantiation: Only members functions that are called are instantiated

- In this case, pop will not be instantiated

```cpp
#include <vector>

template <typename T>
class stack {
public:
  stack() {}
  auto pop() -> void;
  auto push(const T& i) -> void;
private:
  std::vector<T> items_;
};

template <typename T>
auto stack<T>::pop() -> void {
  items_.pop_back();
}

template <typename T>
auto stack<T>::push(const T& i) -> void {
  items_.push_back(i);
}

auto main() -> int {
  stack<int> s;
  **s.push(5);**
}
```

Exact same principles will apply for classes.

Implementations must be in header file, and compiler should only behave as if one Stack<int> was instantiated.

# Static Members

static members:

- Inside a class definition, the keyword [static](https://en.cppreference.com/w/cpp/keywords/static) declares members that are not bound to class instances.
- Outside a class definition, it has a different meaning: see [storage duration](https://en.cppreference.com/w/cpp/language/storage_duration).

Each template instantiation has its own set of static members.

```cpp
#include <iostream>
#include <vector>

template<typename T>
class stack {
public:
  stack();
  ~stack();
  auto push(T&) -> void;
  auto pop() -> void;
  auto top() -> T&;
  auto top() const -> const T&;
  static int num_stacks_;

private:
  std::vector<T> stack_;
};

template<typename T>
int stack<T>::num_stacks_ = 0;

template<typename T>
stack<T>::stack() {
  num_stacks_++;
}

template<typename T>
stack<T>::~stack() {
  num_stacks_--;
}

auto main() -> int {
  stack<float> fs;
  stack<int> is1, is2, is3;
  std::cout << stack<float>::num_stacks_ << "\n"; **// 1**
  std::cout << stack<int>::num_stacks_ << "\n"; **// 3**
}
```

```cpp
stack<int> a;
stack<int> b;
stack<double> c;
stack<double> d;
```

static property is associated with type, so we have nums_stacks_ = 2 for each type, but we have 4 objects in total.

# Friends

Each stack instantiation has one unique instantiation of the friend.

```cpp
#include <string>
#include <iostream>
#include <vector>

template<typename T>
class stack {
public: 
	auto push(T const&) -> void;
	auto pop() -> void;
	
	friend auto operator<<(std::ostream os, stack<T> const& s) -> std::ostream& {
		return os << "My top item is " << s.stack_.back() << "\n";
	}
	
private:
	std::vector<T> stack_;
};

template<typename T>
auto stack<T>::push(T const& t) -> void {
	stack_.push_back(t);
}

auto main() -> int {
	stack<std::string> ss;
	ss.push("Hello");
	std::cout << ss << "\n";
	
	stack<int> is;
	is.push(5);
	std::cout << is << "\n";
}
```

# Default Members

We can provide default arguments to template types (where the defaults themselves are types), which means we have to update all of our four template parameter lists.

```cpp
#include <vector>
#include <iostream>

template<typename T, typename CONT = std::vector<T>>
class stack {
public:
  stack();
  ~stack();
  auto push(T&) -> void;
  auto pop() -> void;
  auto top() -> T&;
  auto top() const -> T const&;
  static int num_stacks_;

private:
  CONT stack_;
};

template<typename T, typename CONT>
int stack<T, CONT>::num_stacks_ = 0;

template<typename T, typename CONT>
stack<T, CONT>::stack() {
  num_stacks_++;
}

template<typename T, typename CONT>
stack<T, CONT>::~stack() {
  num_stacks_--;
}

auto main() -> int {
  auto fs = stack<float>{};
  
  // this will store ints in a vector, since we set the default container to a vetcor
  stack<int> is1, is2, is3;
  
  // if we want to use deque as our container, we can ...
  stack<int, std::deque<int>> is4;
  
  std::cout << stack<float>::num_stacks_ << "\n";
  std::cout << stack<int>::num_stacks_ << "\n";
}
```

# Specialisation

The templates we have defined so far are completely generic, which means we only define them once and use it for everything.

There are two more ways we can redefine our generic types for something more specific:

**Partial specialisation:** Describing the template for another form of the template

- T*
- std::vector<T>

**Explicit specialisation:** Describing the template for a specific, non-generic type

- std::string
- int

### When to specialise

- You need to preserve existing semantics for something that would not otherwise work
- You want to write a type trait
- There is an optimisation you can make for a specific type
    - std::vector<bool> is fully specialised to reduce memory footprint

### When NOT to specialise

- Don’t specialise functions
    - A function cannot be partially specialised
    - Fully specialised functions are better done with overloads
- You think it would be cool if you changed some feature of the class for a specific type
    - People assume a class works the same for all types
    - Don’t violate assumptions!

# A Stack Template

```cpp
#include <vector>
#include <iostream>
#include <numeric>

template <typename T>
class stack {
public:
  auto push(T t) -> void { stack_.push_back(t); }
  auto top() -> T& { return stack_.back(); }
  auto pop() -> void { stack_.pop_back(); }
  auto size() const -> std::size_t { return stack_.size(); };
  auto sum() -> T {
    return std::accumulate(stack_.begin(), stack_.end(), 0);
  }
private:
  std::vector<T> stack_;
};

```

```cpp
#include "./stack.h"

auto main() -> int {
  int i1 = 6771;
  int i2 = 1917;

  stack<int> s1;
  s1.push(i1);
  s1.push(i2);
  std::cout << s1.size() << " ";
  std::cout << s1.top() << " ";
  std::cout << s1.sum() << "\n";
}
```

## Partial Specialisation

In this case we will specialise for pointer types.

Specialisation is designed to refine a generic implementation for a specific type, not to change the semantic.

```cpp
#include <numeric>

#include "./stack.h"

template <typename T>
class stack<T*> {
public:
  auto push(T* t) -> void { stack_.push_back(t); }
  auto top() -> T* { return stack_.back(); }
  auto pop() -> void { stack_.pop_back(); }
  auto size() const -> std::size_t { return stack_.size(); };
  auto sum() -> T{
    return std::accumulate(stack_.begin(),
          stack_.end(), 0, [] (T a, T *b) { return a + *b; });
  }
private:
  std::vector<T*> stack_;
};

```

```cpp
#include <iostream>

#include "./stack-partial.h"

auto main() -> int {
  auto i1 = 6771;
  auto i2 = 1917;

  auto s1 = stack<int*>{};
  s1.push(&i1);
  s1.push(&i2);
  std::cout << s1.size() << " ";
  std::cout << s1.top() << " ";
  std::cout << s1.sum() << "\n";
}
```

## Explicit Specialisation

Explicit specialisation should only be done on classes.

Taking `std::vector<bool>` as an example, `std::vector<bool>::reference` is not a `bool&`.

```cpp
#include <iostream>

template <typename T>
struct is_void {
	static bool const val = false;
}

template <>
struct is_void<void> {
	static bool const val = true;
}

auto main() -> int {
	std::cout << is_void<int>::val << "\n";
	std::cout << is_void<void>::val << "\n";
}
```

# Type Traits

**Trait**: Class (or class template) that characterise a type.

This is what <limits> might look like

```cpp
#include <iostream>
#include <limits>

template <typename T>
struct numeric_limits {
  static auto min() -> T;
};

template <>
struct numeric_limits<int> {
  static auto min() -> int { return -99999999 - 1; }
};

template <>
struct numeric_limits<double> {
  static auto min() -> double { return -999999999.99999 - 1.0; }
};

auto main() -> int {
  std::cout << std::numeric_limits<double>::min() << "\n";
  std::cout << std::numeric_limits<int>::min() << "\n";
}
```

Traits allow generic template functions to be parameterised

```cpp
#include <array>
#include <iostream>
#include <limits>

template<typename T, std::size_t size>
T findMax(const std::array<T, size>& arr) {
  T largest = std::numeric_limits<T>::min();
  for (auto const& i : arr) {
    if (i > largest)
      largest = i;
  }
  return largest;
}

auto main() -> int {
  auto i = std::array<int, 3>{-1, -2, -3};
  std::cout << findMax<int, 3>(i) << "\n";
  auto j = std::array<double, 3>{1.0, 1.1, 1.2};
  std::cout << findMax<double, 3>(j) << "\n";
}
```

Type traits examples for a specialisation and partial specialisation

```cpp
#include <iostream>

template <typename T>
struct is_void {
  static const bool val = false;
};

template<>
struct is_void<void> {
  static const bool val = true;
};

auto main() -> int {
  std::cout << is_void<int>::val << "\n";
  std::cout << is_void<void>::val << "\n";
}
```

```cpp
#include <iostream>

template <typename T>
struct is_pointer {
  static const bool val = false;
};

template<typename T>
struct is_pointer<T*> {
  static const bool val = true;
};

auto main() -> int {
  std::cout << is_pointer<int*>::val << "\n";
  std::cout << is_pointer<int>::val << "\n";
}
```

Below are STL type trait examples for a specialisation and partial specialisation.

```cpp
#include <iostream>
#include <type_traits>

template<typename T>
auto testIfNumberType(T i) -> void {
  if (std::is_integral<T>::value || std::is_floating_point<T>::value) {
    std::cout << i << " is a number"
              << "\n";
  }
  else {
    std::cout << i << " is not a number"
              << "\n";
  }
}

auto main() -> int {
  auto i = int{6};
  auto l = long{7};
  auto d = double{3.14};
  testIfNumberType(i);
  testIfNumberType(l);
  testIfNumberType(d);
  testIfNumberType(123);
  testIfNumberType("Hello");
  auto s = "World";
  testIfNumberType(s);
}
```

# *Template Template Parameter

# *Template Argument Deduction

Template argument deduction is the process of determining the types (of type parameters) and the values of non-type parameters from the types of function arguments.

```cpp
template <typename T, int size>
T findmin(const T (&a)[size]) {
	T min = a[0];
	for (int i = 1; i < size; i++) {
		if (a[i] < min) min = a[i];
	}
	return min;
}
```

Non-type parameters: Implicit conversations behave just like normal type conversions

Type parameters: Generally implicitly convert (real rules more complicated)