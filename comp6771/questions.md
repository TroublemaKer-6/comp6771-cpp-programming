# ?questions

Status: In progress
contents: few things here to be answered

- [ ]  why we put the map comparison inside a struct instead of leaving it alone as a function?

```cpp
struct map_compare {
	bool operator()(const std::unique_ptr<N>& lhs, const std::unique_ptr<N>& rhs) const {
		return *lhs < *rhs;
	}
};
```

- [ ]  why passing by reference solves the object slicing issue

- [ ]  order?

***Every subclass constructor MUST CALL a base class constructor*** 

- [ ]  default return in comparison function for sort method

```cpp
auto lexico_compare(const std::vector<std::string>& a, const std::vector<std::string>& b) -> bool {
    for (size_t i = 0; i < a.size(); ++i) {
        if (a[i] != b[i]) {
            return a[i] < b[i];
        }
    }
    return false;
}
```

- [ ]  defaults

```cpp
class A {
	int a_;
};

class B {
	B(int b): b_ {b} {
	}
	int b_;
};

int main() {
	int i_{0}; // in-class initialiser
	int j_; // untouched memory, can be anything inside that piece of memory
	A a_; // will not create default constructor, since we just declare that
	B b_; 
}
```

- [ ]  scope re-declaration without definition

```cpp
#include <iostream>

int i = 1;
int main()
{
	std::cout << i << "\n"; // 1
	if (i > 0) {
		int i = 2;
		std::cout << i << "\n"; // 2
		{
			// int i = 3;
			int i;
			std::cout << i << "\n";
		}
		std::cout << i << "\n"; // 2
	}
}
```

- [ ]  <<

```cpp
#include <iostream>

class point {
public:
    point(int x, int y)
        : x_ { x }
        , y_ { y } {};
    friend point operator+(point const& lhs,
        point const& rhs);
    friend std::ostream& operator<<(std::ostream& os,
        point const& p);

private:
    int x_;
    int y_;
};

point operator+(point const& lhs, point const& rhs)
{
    return point(lhs.x_ + rhs.x_, lhs.y_ + rhs.y_);
}

std::ostream& operator<<(std::ostream& os, point const& p)
{
    os << "(" << p.x_ << "," << p.y_ << ")";
    return os;
}

auto main() -> int
{
    point p1 { 1, 2 };
    point p2 { 2, 3 };
    // what compiler is doing here:
    // it see p1 + p2, then it gets into the class and find the function,
    // then change that to operator+(p1, p2)
    // And for the out stream with << 
    // it changes std::cout << p1 + p2 into opearator<<(std::cout, opearator+(p1, p2)) 
    **std::cout << p1 + p2 << "\n";**
}
```

- [ ]  what does “this” mean?

```cpp
class point {
public:
    point(int x, int y)
        : x_ { x }
        , y_ { y } {};
    point& operator+=(point const& p);
    point operator+(point const& p);
    point& operator-=(point const& p);
    point& operator*=(point const& p);
    point& operator/=(point const& p);
    point& operator*=(int i);

private:
    int x_;
    int y_;
};

point& point::operator+=(point const& p)
{
    x_ += p.x_;
    y_ += p.y_;
    return *this;
}

point point::operator+(point const& p1, point const& p2) {
		auto p = point { p1 };
		p += p2;
		return p;
}

point& point::operator-=(point const& p) {
    x_ -= p.x_;
    y_ -= p.y_;
    return *this;
}
point& point::operator*=(point const& p) {
    x_ *= p.x_;
    y_ *= p.y_;
    return *this;
}
point& point::operator/=(point const& p) {
    x_ /= p.x_;
    y_ /= p.y_;
    return *this;    
}
point& point::operator*=(int i) {
    x_ *= i;
    y_ *= i;
    return *this;    
}
```

- [ ]  return `cend()`

```cpp
// Define a forward iterator

// Here we are using vector, 
// but this can be changed into any conatiner we want
class vector {
    class Iterator {
		public:
				// those lines are something like a template for custom iterator
		    using iterator_category = std::forward_iterator_tag;
		    // replace T with the type we want
		    using value_type = T;
		    using referenece = T&;
		    using pointer = T*;  // not strictly required, but nice to have
		    using difference_type = int;
		
		    referenece operator*() const;
		    Iterator operator++();
		    Iterator operator++(int)
		    {
		        auto copy { *this };
		        ++(*this);
		        return copy;
		    }
		    // this one isn't strictly required, but nice to have
		    pointer operator->() const { return &(operator*()); }
		
		    friend bool operator==(const Iterator& lhs, const Iterator& rhs) { ... };
		    friend bool operator!=(const Iterator& lhs, const Iterator& rhs) { return !(lhs == rhs); };
		}
    using iterator = Iterator;
    using const_iterator = Iterator;

    // need to define these
    iterator begin();
    iterator end();

    // if you want const iterators, define these
    const_iterator begin() const { return cbegin(); }
    const_iterator cbegin() const;
    const_iterator end() const { return cend(); }
    const_iterator cend() const;
}
```

- [ ]  non-const and const conversion? why and how does it exactly work here?

```cpp
try {
    std::cout << items.at(static_cast<unsigned int>(print_index)) << '\n';
    items.resize(items.size() + 10);
} catch (const std::out_of_range& e) {
    std::cout << "Index out of bounds\n";
} catch (...) {
    std::cout << "Something else happened";
}
```

- [ ]  why this will work here, is this a reference or a pointer

```cpp
Animal& a2 = Dog{}; // THIS WILL BE FINE
// no slicing happens here, 
// since a2 is just a pointer to the dog
```

- [ ]  what does re-throw actually throw? reference or exception obj. Since we don’t specify what to throw in re-throw, so how does it figure out what to throw?

```cpp
try {
		try {
				try {
						throw T{};
				} catch (T& e1) {
						std::cout << "Caught\n";
						throw;
				}
		} catch (T& e2) {
				std::cout << "Caught too!\n";
				throw;
		}
} catch (...) {
std::cout << "Caught too!!\n";
}
```

- [x]  how does the reference work here, do we create a reference for i called x here

```cpp
auto swap(int& x, int& y) -> void
{
	auto const tmp = x;
	x = y;
	y = tmp;
}

auto main() -> int
{
	auto i = 1;
	auto j = 2;
	swap(i, j);
}
```

- [x]  what does `(void)var` actually do?

change the type of var to void —> placeholder

- [x]  what is the difference between use those two

```cpp
#include <string>
#include <vector>

auto all_computer_scientists(std::vector<std::string> const& names) -> bool
{
	auto const famous_mathematician = std::string("Gauss");
	auto const famout_physicist = std::string("Newtown");
	
	// mostly const reference here
	for (auto const& name : names) {
		if (name == famout_mathematician or name == famous_physicist) {
			return false;
		}
	}
	return true;
}
```

```cpp
for (auto age : ages) {
		age++;
    std::cout << age << "\n";
}
```

- [x]  unsigned int and std::size_t

unsigned int always be 0 or positive

unsigned char -> 0 -> 255

size can not be negative, so similar thing to the unsigned int

```cpp
// age.size() will return an unsigned int, 
// it in not a normal int, instead that is an int never can be negative
// so if we define i as a normal int, the compiler will give us a warning and 
// no compilation will be made
for (**unsigned int** i = 0; i < ages.size(); ++i) {
    std::cout << ages[i] << "\n";
}

// age.size() will actually return a type called std::size_t in cpp
for (auto i = **std::size_t{0}**; i < ages.size(); ++i) {
    std::cout << ages[i] << "\n";
}
```

- [x]  why log here?

- map.find(): map is stored as a tree, so that is logarithmic in the size of the container
- unordered_map: Key should be hashable

- [x]  understanding of ++iter

```cpp
int main() {
    auto ages = std::vector<int> {};
    ages.push_back(18);
    ages.push_back(19);
    ages.push_back(20);

    // type of iter would be std::vector<int>::iterator
    for (auto iter = ages.begin(); iter != ages.end(); ++iter) {
        (*iter)++; // OK
    }

    // type of iter would be std::vector<int>::const_iterator
    for (auto iter = ages.cbegin(); iter != ages.cend(); ++iter) {
        // (*iter)++;  // ERROR since it is const, it cannot be modified
    }

    // type of iter would be std::vector<int>::reserve_iterator
    for (auto iter = ages.rbegin(); iter != ages.rend(); ++iter) {
        std::cout << *iter << "\n";  // 20, 19, 18
    }
}
```

++ just means going to the next element