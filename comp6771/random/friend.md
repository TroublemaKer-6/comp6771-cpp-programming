# friend

Created: June 20, 2024 12:46 PM

## The `friend` Keyword in C++

In C++, the `friend` keyword is used to allow a particular class or function to ***access private and protected members of another class***. This can be useful when you need to allow an external function or class to modify the internal state of a class.

Here's a basic example:

### Example 1: Friend Function

```cpp
#include <iostream>
using namespace std;

class Box {
private:
    double width;

public:
    Box() : width(0.0) {}

    // Declare a friend function
    friend void printWidth(Box box);
};

// Definition of the friend function
void printWidth(Box box) {
    cout << "Width of box: " << box.width << endl;
}

int main() {
    Box box;
    printWidth(box);
    return 0;
}

```

In this example, the `printWidth` function is a friend of the `Box` class, which means it can access the private member `width` of the `Box` class.

### Example 2: Friend Class

```cpp
#include <iostream>
using namespace std;

class Rectangle {
private:
    int width, height;

public:
    Rectangle() : width(0), height(0) {}

    void setDimensions(int w, int h) {
        width = w;
        height = h;
    }

    // Declare Friend Class
    friend class AreaCalculator;
};

class AreaCalculator {
public:
    int calculateArea(Rectangle rect) {
        return rect.width * rect.height;  // Accessing private members
    }
};

int main() {
    Rectangle rect;
    rect.setDimensions(5, 10);

    AreaCalculator calculator;
    cout << "Area of rectangle: " << calculator.calculateArea(rect) << endl;

    return 0;
}

```

In this example, the `AreaCalculator` class is a friend of the `Rectangle` class. This allows the `AreaCalculator` to access the private members `width` and `height` of the `Rectangle` class.

## Static VS Friend

### The `static` Keyword

The `static` keyword in C++ can be used in several contexts:

1. **Static Member Variables:**
    - A static member variable is shared among all instances of a class. There is only one copy of the variable, regardless of how many objects are created.
    - It is initialized outside the class definition.
2. **Static Member Functions:**
    - A static member function can access only static member variables and other static member functions.
    - It can be called on the class itself, without needing an instance of the class.

```cpp
#include <iostream>
using namespace std;

class MyClass {
private:
    static int count;

public:
    MyClass() {
        count++;
    }

    static void showCount() {
        cout << "Count: " << count << endl;
    }
};

// Initialize static member variable
int MyClass::count = 0;

int main() {
    MyClass obj1, obj2;
    MyClass::showCount();  // Output: Count: 2

    return 0;
}
```

```cpp
#include <iostream>
using namespace std;

class Box {
private:
    static int width;

public:
    Box() {}

    // Declare a friend function
    friend void printWidth(Box box);
};

// Initialize static member variable
int Box::width = 10;

// Definition of the friend function
void printWidth(Box box) {
    cout << "Width of box: " << Box::width << endl;
}

int main() {
    Box box;
    printWidth(box);
    return 0;
}
```

```cpp
#include <iostream>
using namespace std;

class Rectangle {
private:
    static int width, height;

public:
    Rectangle() {}

    static void setDimensions(int w, int h) {
        width = w;
        height = h;
    }

    // Declare Friend Class
    friend class AreaCalculator;
};

// Initialize static member variables
int Rectangle::width = 0;
int Rectangle::height = 0;

class AreaCalculator {
public:
    int calculateArea() {
        return Rectangle::width * Rectangle::height;  // Accessing static members
    }
};

int main() {
    Rectangle::setDimensions(5, 10);

    AreaCalculator calculator;
    cout << "Area of rectangle: " << calculator.calculateArea() << endl;

    return 0;
}
```

### Key Points

- **Static Members**:
    - Static member variables are shared across all instances of a class.
    - Static member functions can be called without an instance of the class and can only access static members.
- **Friend Functions and Classes**:
    - Friend functions can access private and protected members of the class in which they are declared as friends.
    - Friend classes can access private and protected members of the class they are friends with.
- **Combining Static and Friend**:
    - A friend function can access static members of the class.
    - A static member function can be declared as a friend of another class or function.