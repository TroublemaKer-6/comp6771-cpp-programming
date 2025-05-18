# w7prac

Tags: prac
Status: In progress
contents: rethrow; copy/move

# rethrow

```cpp
#ifndef COMP6771_RETHROW_HPP
#define COMP6771_RETHROW_HPP

#include <set>
#include <string>

class db_conn {
    /**
     * @brief A blacklist of users who are not allowed to make connections.
     * Currently, there is a single username who cannot login: hsmith
     */
    static const std::set<std::string> blacklist_;
public:
    /**
     *  @brief Default Constructor.
     * No need to modify this.
     */
    db_conn() noexcept = default;

    /**
     * @brief Attempts to make a connection given a uname and pwd.
     * 
     * If the uname is not part of the blacklist, and pword is at least 8 characters,
     * the connection is successfully established (i.e. active_ becomes true).
     * Once a connection is established, any further calls to this function have no effect.
     * All calls to this function, even calls that result in an exception being thrown, count as attempts.
     * 
     * Throws exceptions in the cases and order given below.
     * 
     * @throws std::domain_error: if uname is part of the blacklist,
     *         throws a std::domain_error with message: <uname> is not allowed to login.
     * @throws std::runtime_error: every 2nd attempt, the computer malfunctions and throws a std::runtime_error with message:
     *         HeLp ;_; c0mpu73R c@ann0T c0mPut3 0w0
     * 
     * @param uname - the uname to login / make a connection.
     * @param pword - the password of the user.
     */
    auto try_connect(const std::string &uname, const std::string &pword) -> void;

    /**
     * @brief Returns the connection status of this db_conn.
     * 
     * @return bool - true if this connection is active, false otherwise. 
     */
    auto is_active() const -> bool;
private:
    int n_attempts_;
    bool active_;
};

/**
 * @brief Attempts to make a connection to db with uname and pwd.
 * 
 * If any exceptions occur, rethrows them as std::strings.
 * 
 * @param db - the db to make a connection to.
 * @param uname - the uname to login / make a connection.
 * @param pword - the password of the user.
 */
auto make_connection(db_conn &db, const std::string &uname, const std::string &pword) -> void;

#endif // COMP6771_RETHROW_HPP
```

```cpp
#include "./rethrow.h"

#include <exception>
#include <iostream>

const std::set<std::string> db_conn::blacklist_ = {"hsmith"};

auto db_conn::try_connect(const std::string &uname, const std::string &pword) -> void {
    if (!active_) {
        n_attempts_ += 1;
        if (blacklist_.contains(uname)) {
            throw std::domain_error(uname + " is not allowed to login.");
        } else if (n_attempts_ % 2 == 0) {
            throw std::runtime_error("HeLp ;_; c0mpu73R c@ann0T c0mPut3 0w0");
        }

        active_ = pword.length() >= 8;
    }
}

auto db_conn::is_active() const-> bool {
    return active_;
}

auto make_connection(db_conn &db, const std::string &uname, const std::string &pword) -> void {
    try {
        db.try_connect(uname, pword);
    } catch(const std::domain_error &e) {
        throw std::string{e.what()};
    } catch(const std::runtime_error &e) {
        throw std::string{e.what()};
    }
}
```

```cpp
#include "./rethrow.h"

#include <iostream>
#include <utility>
#include <vector>

int main() {
    auto db = db_conn{};
    const auto attempts = std::vector<std::pair<std::string, std::string>> {
        {"hsmith", "swagger/10"},
        {"vegeta", "over9000"},
        {"billgates", "apple<3"},
        {"billgates", "macros0ft"},
        {"billgates", "m1cros0ft"},
    };
    for (const auto &[uname, pwd] : attempts) {
        try {
        make_connection(db, uname, pwd);
        } catch (const std::string &e) {
            std::cout << "Could not establish connection: " << e << std::endl;
        }
    }
}

/**
 * Output:
 * Could not establish connection: hsmith is not allowed to login.
 * Could not establish connection: HeLp ;_; c0mpu73R c@ann0T c0mPut3 0w0
 * Could not establish connection: HeLp ;_; c0mpu73R c@ann0T c0mPut3 0w0
 */

```

# Matrix (copy & move)

```cpp
#ifndef COMP6771_MATRIX_H
#define COMP6771_MATRIX_H

#include <algorithm>
#include <exception>
#include <initializer_list>
#include <memory>
#include <sstream>

class matrix {
public:
    matrix() noexcept = default;
    
    // construct with initializer_list
    matrix(std::initializer_list<std::initializer_list<int>> il) {
        const std::size_t r = il.size();
        const std::size_t c = il.begin()->size();

        if (!std::all_of(il.begin() + 1, il.end(), [c](const std::initializer_list<int> il) {
            return il.size() == c;
        })) {
            throw std::logic_error{col_len_msg()};
        }
        
				// create a int array with size r*c
        std::unique_ptr<int[]> data = std::make_unique<int[]>(r * c);
        // return a pointer to the managed object
        int *p = data.get();
        for (const std::initializer_list<int> &col : il) {
            for (int i : col) {
		            // value i will be assigned to current p
		            // and after assignment p will be incremented,
		            // since that is post-increment
                *p++ = i;
            }
        }
				
				// copy and clear the data inside those temp variables
        data_ = std::move(data);
        n_rows_ = std::move(r);
        n_cols_ = std::move(c);
    }
		
		// copy constructor
    matrix(const matrix &o)
    : data_{std::make_unique<int[]>(o.n_rows_ * o.n_cols_)},
    n_rows_{o.n_rows_},
    n_cols_{o.n_cols_} {
        std::copy(o.begin(), o.end(), begin());
    }
		
		// move constructor
		// when call std::move it will cast a lvalue into a rvalue,
		// and the actual movement should be done here.
    matrix(matrix &&o) noexcept
    : data_{std::exchange(o.data_, nullptr)},
    n_rows_{std::exchange(o.n_rows_, 0)},
    n_cols_{std::exchange(o.n_cols_, 0)} {}
		
		// copy assignment
    matrix &operator=(const matrix &o) {
        if (this != &o) {
		        // call copy constructor
            *this = matrix{o};
        }

        return *this;
    }
		
		// move assignment
    matrix &operator=(matrix &&o) noexcept {
        if (this != &o) {
            data_ = std::exchange(o.data_, nullptr);
            n_rows_ = std::exchange(o.n_rows_, 0);
            n_cols_ = std::exchange(o.n_cols_, 0);
        }

        return *this;
    }

    const int &operator()(std::size_t i, std::size_t j) const {
        if (!(i < n_rows_ && j < n_cols_)) {
            throw std::domain_error{bounds_msg(i, n_rows_, j, n_cols_)};
        }
        return data_[i * n_cols_ + j];
    }

    int &operator()(std::size_t r, std::size_t c) {
        if (!(r < n_rows_ && c < n_cols_)) {
            throw std::domain_error{bounds_msg(r, n_rows_, c, n_cols_)};
        }

        return data_[r * n_cols_ + c];
    }

    bool operator==(const matrix &r) const noexcept {
        const matrix &l = *this;
        return l.n_rows_ == r.n_rows_
            && l.n_cols_ == r.n_cols_
            && std::equal(l.begin(), l.end(), r.begin());
    }

    std::pair<std::size_t, std::size_t> dimensions() const noexcept {
        return std::make_pair(n_rows_, n_cols_);
    }

    const int *data() const noexcept {
        return data_.get();
    }

private:
    std::string bounds_msg(std::size_t i, std::size_t r, std::size_t j, std::size_t c) const {
        std::stringstream ss;
        ss << "(" << i << ", " << j << ")"
           << " does not fit within a matrix with dimensions "
           << "(" << r << ", " << c << ")";
        return ss.str();
    }

    std::string col_len_msg() const {
        return "Columns are not equal length";
    }

    std::size_t length() const {
        return n_rows_ * n_cols_;
    }

    int *begin() { return data_.get(); }
    int *end() { return begin() + length(); }
    const int *begin() const { return data_.get(); }
    const int *end() const { return begin() + length(); }

    std::unique_ptr<int[]> data_;
    std::size_t n_rows_;
    std::size_t n_cols_;
};

#endif // COMP6771_MATRIX_H
```

```cpp
#include "matrix.h"

#include <catch2/catch.hpp>
#include <type_traits>

TEST_CASE("Can nothrow copy-construct a matrix") {
    CHECK(!std::is_nothrow_copy_constructible_v<matrix>);
}

TEST_CASE("Can copy construct a matrix") {
    auto m1 = matrix{
        {1, 1, 1},
        {1, 1, 1},
        {1, 1, 1}
    };

    auto m2 = m1;
    CHECK(m1 == m2);
    
    SECTION("actually deep-copied") {
        m1(0, 0) = 4;

        CHECK(m2(0, 0) == 1);
    }
}

TEST_CASE("Can default construct a matrix") {
    auto m = matrix{};

    CHECK(m.dimensions() == std::make_pair<std::size_t, std::size_t>(0, 0));
    CHECK(m.data() == nullptr);
}

TEST_CASE("Cannot contruct a matrix with an invalid initialiser list") {
    using Catch::Message;

    CHECK_THROWS_MATCHES(matrix({{1}, {1, 1}}), std::logic_error, Message("Columns are not equal length"));
}

TEST_CASE("Can contruct a matrix with a valid initialiser list") {
    auto m = matrix{
        {1, 1, 1},
        {1, 1, 1},
        {1, 1, 1}
    };

    const auto &[r, c] = m.dimensions();

    REQUIRE((r == c && r == 3));

    for (auto i = 0u; i < r; ++i) {
        for (auto j = 0u; j < c; ++j) {
            REQUIRE(m(i, j) == 1);
        }
    }
    
}

TEST_CASE("Can nothrow move-construct a matrix") {
    CHECK(std::is_nothrow_move_constructible_v<matrix>);
}

TEST_CASE("Can move construct a matrix") {
    auto m1 = matrix{
        {1, 1, 1},
        {1, 1, 1},
        {1, 1, 1}
    };

    auto m2 = std::move(m1);

    CHECK(m1.dimensions() == std::make_pair<std::size_t, std::size_t>(0, 0));
    CHECK(m1.data_ == nullptr);

    const auto &[r, c] = m2.dimensions();
    REQUIRE((r == c && r == 3));

    for (auto i = 0u; i < r; ++i) {
        for (auto j = 0u; j < c; ++j) {
            REQUIRE(m2(i, j) == 1);
        }
    }
}

TEST_CASE("Can get a pointer to the underlying data (const)") {
    const auto m = matrix{
        {1, 1, 1},
        {1, 1, 1},
        {1, 1, 1}
    };

    constexpr int ints[9] = {1, 1, 1, 1, 1, 1, 1, 1, 1};
    CHECK(std::equal(m.data(), m.data() + 9, ints));
}

TEST_CASE("Can get a pointer to the underlying data (const)") {
    const auto m = matrix{
        {1, 1, 1},
        {1, 1, 1},
        {1, 1, 1}
    };

    constexpr int ints[9] = {1, 1, 1, 1, 1, 1, 1, 1, 1};
    CHECK(std::equal(m.data(), m.data() + 9, ints));
}

TEST_CASE("Can get a pointer to the underlying data") {
    auto m = matrix{
        {1, 1, 1},
        {1, 1, 1},
        {1, 1, 1}
    };

    constexpr int ints[9] = {1, 1, 1, 1, 1, 1, 1, 1, 1};
    CHECK(std::equal(m.data(), m.data() + 9, ints));
}

TEST_CASE("Can get dimensions of a matrix (const-correct)") {
    const auto m = matrix{
        {1, 1, 1},
        {1, 1, 1},
        {1, 1, 1}
    };

    CHECK(m.dimensions() == std::make_pair<std::size_t, std::size_t>(3, 3));
}

TEST_CASE("Can get dimensions of a matrix") {
    auto m = matrix{
        {1, 1, 1},
        {1, 1, 1},
        {1, 1, 1}
    };

    CHECK(m.dimensions() == std::make_pair<std::size_t, std::size_t>(3, 3));
}

TEST_CASE(".get() throws for invalid indices") {
    const auto m = matrix{};

    using Catch::Message;
    CHECK_THROWS_MATCHES(m(1, 0), std::domain_error, Message("(1, 0) does not fit within a matrix with dimensions (0, 0)"));
    CHECK_THROWS_MATCHES(m(0, 1), std::domain_error, Message("(0, 1) does not fit within a matrix with dimensions (0, 0)"));
}

TEST_CASE(".get() works as expected when not const-qualified") {
    auto m = matrix{
        {1, 1},
        {1, 1},
    };

    m(0, 0) = 0;
    m(1, 1) = 0;

    for (auto i = 0u; i < 2; ++i) {
        for (auto j = 0u; j < 2; ++j) {
            REQUIRE(m(i, j) == (i != j));
        }
    }
}

TEST_CASE(".get() works as expected when const-qualified") {
    const auto m = matrix{
        {1, 1, 1},
        {1, 1, 1},
        {1, 1, 1}
    };

    for (auto i = 0u; i < 3; ++i) {
        for (auto j = 0u; j < 3; ++j) {
            REQUIRE(m(i, j) == 1);
        }
    }
}

TEST_CASE("Can nothrow copy-assign a matrix") {
    CHECK(!std::is_nothrow_copy_assignable_v<matrix>);
}

TEST_CASE("Can copy assign a matrix") {
    auto m2 = matrix{};
    auto m1 = matrix{
        {1, 1, 1},
        {1, 1, 1},
        {1, 1, 1}
    };

    m2 = m1;
    CHECK(m1 == m2);
    
    SECTION("actually deep-copied") {
        m1(0, 0) = 4;

        CHECK(m2(0, 0) == 1);
    }
}

TEST_CASE("Matrix comparison is no-throw") {
    CHECK(std::is_nothrow_invocable_v<decltype(&matrix::operator==), const matrix &, const matrix &>);
}

TEST_CASE("Can compare matricies for equality") {
    auto m1 = matrix{
        {1, 1, 1},
        {1, 1, 1},
        {1, 1, 1}
    };
    auto m2 = matrix{
        {1, 1, 1},
        {1, 1, 1},
        {1, 1, 1}
    };
    auto m3 = matrix{
        {0, 1, 1},
        {1, 0, 1},
        {1, 1, 0}
    };

    for (auto i = 0u; i < 3; ++i) {
        for (auto j = 0u; j < 3; ++j) {
            REQUIRE(m1(i, j) == 1);
            REQUIRE(m2(i, j) == 1);
            REQUIRE(m3(i, j) == (i != j));
        }
    }

    CHECK(m1 == m2);
    CHECK(!(m1 == m3));
}

TEST_CASE("Can nothrow move-assign a matrix") {
    CHECK(std::is_nothrow_move_assignable_v<matrix>);
}

TEST_CASE("Can move assign a matrix") {
    auto m2 = matrix{};
    auto m1 = matrix{
        {1, 1, 1},
        {1, 1, 1},
        {1, 1, 1}
    };

    m2 = std::move(m1);

    CHECK(m1.dimensions() == std::make_pair<std::size_t, std::size_t>(0, 0));
    CHECK(m1.data_ == nullptr);

    const auto &[r, c] = m2.dimensions();
    REQUIRE((r == c && r == 3));

    for (auto i = 0u; i < r; ++i) {
        for (auto j = 0u; j < c; ++j) {
            REQUIRE(m2(i, j) == 1);
        }
    }
}

TEST_CASE("Can compare matricies for inequality") {
    auto m1 = matrix{
        {1, 1, 1},
        {1, 1, 1},
        {1, 1, 1}
    };
    auto m2 = matrix{
        {1, 1, 1},
        {1, 1, 1},
        {1, 1, 1}
    };
    auto m3 = matrix{
        {0, 1, 1},
        {1, 0, 1},
        {1, 1, 0}
    };

    for (auto i = 0u; i < 3; ++i) {
        for (auto j = 0u; j < 3; ++j) {
            REQUIRE(m1(i, j) == 1);
            REQUIRE(m2(i, j) == 1);
            REQUIRE(m3(i, j) == (i != j));
        }
    }

    CHECK(!(m1 != m2));
    CHECK(m1 != m3);
}

```