# w8prac

Tags: prac
Status: Not started
contents: unique_ptr is only moveable not copyable; virtual inheritance(Diamond Derivation problem)

## Q8.4. Calculated Expression

A common pattern in compilers is to represent the various elements of a programming language as classes and to naturally model the Abstract Synax Tree* as a class hierarchy.

Inspired by compiler writers of old, a budding C++ expert wrote a small program in `src/8.4/expression.cpp` which, using such a class hierarchy that represents fundamental arithmetic (`+`, `-`, `*`, and `/`), calculates a very special number and outputs it (with some supplementary text) to the terminal.

Unfortunately, this programmer was not elite enough to finished the class hierarchy implementation. He was, however, able to stub out the Abstract Base Class which the required base classes should derive from.

Your task is to complete what this young and enthusiastic programmer set out to do such that `expression.cpp`successfully compiles, runs, and produces the following output:

`COMP6771: Advanced C++ Programming`

**Note**: each line is terminated with a newline.

You should implement the rest of the solution in `src/8.4/expr.h` and/or `src/8.4/expr.cpp`.

*: not required knowledge.

```cpp
#ifndef COMP6771_EXPR_H
#define COMP6771_EXPR_H

#include <memory>

class expr {
public:
    virtual double eval() const = 0;
    virtual ~expr() = default;
};

class plus : public expr {
public:
		// std::unqiue_ptr can only be moved, not copied
    plus(std::unique_ptr<expr> l, std::unique_ptr<expr> r) noexcept
    : lhs_{std::move(l)}, rhs_{std::move(r)} {}

    double eval() const override { return lhs_->eval() + rhs_->eval(); }

private:
    std::unique_ptr<expr> lhs_;
    std::unique_ptr<expr> rhs_;
};

class minus : public expr {
public:
    minus(std::unique_ptr<expr> l, std::unique_ptr<expr> r) noexcept
    : lhs_{std::move(l)}, rhs_{std::move(r)} {}

    double eval() const override { return lhs_->eval() - rhs_->eval(); }

private:
    std::unique_ptr<expr> lhs_;
    std::unique_ptr<expr> rhs_;
};

class multiply : public expr {
public:
    multiply(std::unique_ptr<expr> l, std::unique_ptr<expr> r) noexcept
    : lhs_{std::move(l)}, rhs_{std::move(r)} {}

    double eval() const override { return lhs_->eval() * rhs_->eval(); }

private:
    std::unique_ptr<expr> lhs_;
    std::unique_ptr<expr> rhs_;
};

class divide : public expr {
public:
    divide(std::unique_ptr<expr> l, std::unique_ptr<expr> r) noexcept
    : lhs_{std::move(l)}, rhs_{std::move(r)} {}

    double eval() const override { return lhs_->eval() / rhs_->eval(); }

private:
    std::unique_ptr<expr> lhs_;
    std::unique_ptr<expr> rhs_;
};

class literal : public expr {
public:
    literal(double val) noexcept : val_{val} {}

    double eval() const override { return val_; }

private:
    double val_;
};

#endif // COMP6771_EXPR_H
```

```cpp
#include "./expr.h"

#include <iostream>

int main() {
    std::unique_ptr<expr> expr = std::make_unique<plus>(
        std::make_unique<multiply>(
            std::make_unique<divide>(
                std::make_unique<literal>(2000),
                std::make_unique<literal>(4)
            ),
            std::make_unique<literal>(13)
        ),
        std::make_unique<minus>(
            std::make_unique<literal>(700),
            std::make_unique<plus>(                
                std::make_unique<multiply>(
                    std::make_unique<literal>(60),
                    std::make_unique<literal>(7)
                ),
                std::make_unique<literal>(9)
            )
        )
    );

    std::cout << "COMP" << static_cast<int>(expr->eval()) << ": Advanced C++ Programming\n";
}
/**
 * Output:
 * COMP6771: Advanced C++ Programming
 */

```

## Q8.3. Order Of The Inheritance

Recall the order of construction and destruction in the face of base classes in C++:

- Construction:
    1. `virtual` base classes (assuming all parents also inherit that class virtually) in declaration order.
    2. Non-`virtual` bases in declaration order.
    3. Data members in declaration order.
    4. `this`'s constructor.
- Destructor:
    1. `this`'s destructor.
    2. Destructors of data members in reverse-declaration order.
    3. Non-`virtual` bases in reverse-declaration order.
    4. `virtual` base classes in reverse-declaration order.

In order to test their knowledge out, a budding C++ expert in `src/8.3/ordered.cpp` created a small program with this output:

`AAABCABAAABCADAAAB
~B~A~A~A~D~A~C~B~A~A~A~B~A~C~B~A~A~A`

**Note**: each line is terminated by a newline.

This output consists of some configuration of inheritance and composition of four `struct`s that have been stubbed in `src/8.3/order.h` and `src/8.3/order.cpp`.

Your task is figure out the configuration of inheritance and composition of these four `struct`s such that `ordered.cpp`compiles and produces the above output. You have successfully completed this lab when your program does so.

**Important**: You are not allowed to modify `src/8.3/ordered.cpp`.

```cpp
#ifndef COMP6771_ORDER_H
#define COMP6771_ORDER_H

#include <iostream>

/*

Expected output:
- AAABCABAAABCADAAAB

- Easy part:
    - AAABCABAAABCD [ A | AAB ]
    - Constructing `B` must produce 2 A's somehow
    - Constructing `A` must produce just A because it's immediately preceded by `D`
- Pattern for `C`
    - C is always preceded by AAAB
    - But constructing a `B` only gets us 2 `A`, so we need one more somehow
    - [AAABC] [AB] [AAABC] [AD]  ...
    - This leaves 3 choices for `C`
        i)   C composes both `A` and `B`
        ii)  C extends non-virtual `A` and compose `B`
        iii) C extends virtual `A` and compose `B`
- How do we get the AB then?
    - This is tricky because we assume constructing `B` will always give 2 `A`
    - This is when virtual base classes and the diamond problem come in
    - If `D` extends non-virtual `C` and followed by non-virtual `B`
        - We get AAABC from constructing `C`
        - AB from constructing `B` because of virtual inheritance
- Remaining bits:
    - ... [AAABC] [AD] ...
    - This can be achieved by composing `C`, `A` in `D`
- Summary:
    - Don't change `A`
    - `B` extends `A` via public virtual
    - `C` extends `A` via public virtual
    - `D` extends `C` and `B` normally and compose `C`, `A`
*/

struct A {
	A() {
		std::cout << "A";
	}
	~A() {
		std::cout << "~A";
	}
};

struct B : public virtual A {
	B() {
		std::cout << "B";
	}
	~B() {
		std::cout << "~B";
	}

	A a;
};

struct C : public virtual A {
	C() {
		std::cout << "C";
	}
	~C() {
		std::cout << "~C";
	}

	B b;
};

struct D
: public C
, public B {
	D() {
		std::cout << "D";
	}
	~D() {
		std::cout << "~D";
	}

	C c;
	A a;
};

#endif // COMP6771_ORDER_H
```