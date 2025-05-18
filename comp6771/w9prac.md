# w9prac

Tags: prac
Status: Not started
contents: template; deduction guide with variadic

# template

```cpp
#ifndef COMP6771_RING_H
#define COMP6771_RING_H

#include <stdexcept>
#include <cstddef>
#include <initializer_list>
#include <string>

template <typename T, std::size_t N>
class ring {
public:
    template <typename InputIt>
    ring(InputIt first, InputIt last) : head_{0}, tail_{0}, size_{0} {
        if (static_cast<std::size_t>(std::distance(first, last)) > N) {
            throw std::invalid_argument{"Not enough capacity"};
        }
        // no initialisation needed to be done here
        for (; first != last; ++first)
            do_push(*first);
    }

    ring(std::initializer_list<T> il) : ring(il.begin(), il.end()) {}

    auto push(const T &t) -> void {
        if (size_ < N) {
            do_push(t);
        }
    }

    auto peek() const -> const T& {
        return elems_[head_];
    }

    auto pop() -> void {
        if (size_ > 0) {
            do_pop();
        }
    }

    auto size() const -> std::size_t {
        return size_;
    }

private:
    auto do_push(const T &t) {
        elems_[tail_++] = t;
        // if tail_ reaches N, it resets to 0, 
        // maintaining the circular nature of the buffer.
        tail_ %= N;
        size_++;
    }

    auto do_pop() {
        head_ = ++head_ % N;
        size_--;
    }

    std::size_t head_; 
    std::size_t tail_; 
    std::size_t size_; 
    T elems_[N];
};

// deduction guide
template <typename T, typename ...Ts>
ring(T, Ts...) -> ring<T, sizeof...(Ts) + 1>;

#endif // COMP6771_RING_H
```

```cpp
#include "./ring.h"

#include <iostream>
#include <vector>

using namespace std::literals; // for a std::string literal

int main() {
    {
        try {
            auto r1 = ring<int, 2>{1, 2, 3};
        } catch(const std::invalid_argument &e) {
            std::cout << e.what() << std::endl;
        }
        
        auto r1 = ring{1, 2, 3};
        for (auto i = 0ul, sz = r1.size(); i < sz; ++i) {
            std::cout << r1.peek() << " ";
            r1.pop();
        }
        std::cout << std::endl;

        r1.push(42);
        r1.push(6771);

        for (auto i = 0ul, sz = r1.size(); i < sz; ++i) {
            std::cout << r1.peek() << " ";
            r1.pop();
        }
        std::cout << std::endl;
    }

    {
        auto v = std::vector{"hello"s, "world!"s};
        try {
            auto r2 = ring<std::string, 1>{v.begin(), v.end()};
        } catch(const std::invalid_argument &e) {
            std::cout << e.what() << std::endl;
        }

        auto r2 = ring<std::string, 3>{v.begin(), v.end()};
        for (auto i = 0ul, sz = r2.size(); i < sz; ++i) {
            std::cout << r2.peek() << " ";
            r2.pop();
        }
        std::cout << std::endl;

        r2.push("yet another"s);
        r2.push("lazy sunday"s);

        for (auto i = 0ul, sz = r2.size(); i < sz; ++i) {
            std::cout << r2.peek() << " ";
            r2.pop();
        }
        std::cout << std::endl;
    }

}
/**
 * Output;
 * Not enough capacity
 * 1 2 3 
 * 42 6771 
 * Not enough capacity
 * hello world! 
 * yet another lazy sunday
 */

```