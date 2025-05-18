# w4tut

Tags: tut
Status: Done

### 4.1  Namespaces & Scopes

```cpp
#include <iostream>

// file scope
namespace {
    auto i = 42;
}

namespace comp6771 {
    auto i = 6771;

    auto foo(int) -> void {
        std::cout << "comp6771::foo(int)" << std::endl;
    }
}

namespace comp6772 {
    auto foo(int) -> void {
        std::cout << "comp6772::foo(int)" << std::endl;
    }

    auto foo(char) -> void {
        std::cout << "comp6772::foo(char)" << std::endl;
    }
}

namespace comp6773 {
    struct uchar {
        unsigned char c;
    };

    auto bar(uchar) -> void {
        std::cout << "comp6773::bar(comp6773::uchar)" << std::endl;
    }
}

int main() {
    std::cout << i << std::endl;        
    {
        int i = 0;                      
        std::cout << i << std::endl;    
				
				// using allows bringing something from other namespace to current namespace
				// this means bringing global namespace to current, 
				// this does not work for variables but for functions ....
        **using ::i;**                      
        std::cout << i << std::endl;    
    }

    comp6771::foo(i);                   
    comp6772::foo(i);                   

    using namespace comp6771;           
    using namespace comp6772;           
		
		// not compiled, since there are 2 choices with same level
    foo(i);                                                                     

    foo('c');                         

    auto uc = comp6773::uchar{'c'};     
    bar(uc);                            
}

```

### 4.2 Constructing Destruction

```cpp
struct object {
    object() {
        std::cout << "ctor ";
    }

    object(const object &) {
        std::cout << "copy-ctor ";
    }

    ~object() {
        std::cout << "dtor ";
    }
};

```

1. What is the output of the below code and why?

```cpp
{
    std::cout << "Pointer: ";
    std::list<object *> l;
    object x;
    // &x is a pointer to x
    l.push_back(&x);
}
```

- a) `Pointer: ctor ctor dtor dtor`. `l` is a list of `object`derived types and `x` is an `object`, so the constructor and destructor for `object` will run for both.
- b) `Pointer: ctor dtor`. Variables are destructed in the reverse-order of definition on the stack, so to prevent a double-free bug, `l`'s single element (the address of `x`) only has the constructor and destructor run for it.
- **c) `Pointer: ctor dtor`. The only `object` whose lifetime ends in this code block is `x`, and the list `l` is irrelevant.**
- d) `Pointer: ctor ctor dtor dtor`. `l`'s default constructor creates an `object *` instance and a second instance is created when the address of `x` is pushed back. Two constructions implies two destructions.

1. What is the output of the below code and why?

```cpp
{
    std::cout << "\nValue: ";
    // list default constructor called
    std::list<object> l;
    object x;
    // it will make a copy of the object
    l.push_back(x);
}
```

- **a) `Value: ctor copy-ctor dtor dtor`. The default constructor of `object` is called when `x` comes into scope and the copy constructor is called when `x` is pushed back into `l`. At the end of scope, `x` is destructed first and, since `l` holds `object`s by value, its single element is destructed second.**
- b) `Value: ctor copy-ctor dtor dtor`. `x` is default-constructed and destructed as per usual, but the temporary that is created by passing `x` into `push_back()` is copy-constructed and destructed in that expression.
- c) `Value: ctor copy-ctor copy-ctor dtor dtor dtor`. `x` is default-constructed and destructed as per usual, but the temporary that is created in the call to `l.push_back()` *and* the resulting element of `l` are copy-constructed and destructed.
- d) `Value: ctor copy-ctor dtor copy-ctor dtor dtor`. `x` is default-constructed and destructed as per usual, but the temporary that is created in the call to `push_back()` has its lifetime end at the end of that expression before it is copied into `l`. At the end of scope, `l`'s single element is also destructed.

### 4.3 Defaults & Deletes

1. Consider the below code:

```cpp
struct point2i {
    int x;
    int y;
};
```

Is this class-type default-constructible and why?

- a) No: We need to opt-in to a default aggregate initialiser.
- b) Yes: default aggreggate-initialisation would leave `x` and `y` uninitialised.
- c) No: This is a C-style struct; it has no default constructor.
- d) Yes: default aggregate-initialisation would set `x` and `y` to `0`.

copy initialisation

```cpp
std::vector<int> a(c);
```