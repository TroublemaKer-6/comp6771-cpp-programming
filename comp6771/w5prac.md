# w5prac

Tags: prac
Status: Done
contents: custom iterator sample implementation

## Q5.6. Ropeful

One way we could implement a "rope" class would be to have a vector of strings.

Of course, a rope is logically contiguous, whereas the elements of a vector are disjoint.

With the power of a custom iterator, however, we can bridge the discontinuities and realise our dreams of having a rope-like container in C++!

In `src/5.6/rope.h`, you will find the stubs of two classes, `rope`, and its iterator `rope::iter`.

Your task is to complete the `rope` class's implementation so that it models [a reversible container](https://en.cppreference.com/w/cpp/named_req/ReversibleContainer), and also to implement `rope::iter` so that it models [a bidirectional iterator](https://en.cppreference.com/w/cpp/named_req/BidirectionalIterator).

There is a client program in `src/5.6/rope_user.cpp` which will give hints as to how the `rope` class is intended to be used. You have successfully completed this activity when this program compiles and runs without error.

**Note**: You are not allowed to modify `src/5.6/rope_user.cpp`.

**Implementation Hints:**

- A contiguous view over a vector of strings implies that each "element" of the range is a single `char`.
- It should not be possible to modify the elements of the range through the iterator. Therefore, you only need to implement a `const_iterator`. The type `rope::iterator` should still be available, though.
- Where possible, it is always preferable to delegate to the Standard Library.
- Friendship will likely be necessary.

```cpp
#ifndef COMP6771_ROPE_H
#define COMP6771_ROPE_H

#include <string>
#include <vector>
#include <utility>

class rope {
	class iter {
	public:
		using iterator_category = std::bidirectional_iterator_tag;
		using value_type = char;
		using reference = const value_type&;
		using pointer = void;
		using difference_type = std::ptrdiff_t;

		iter() = default;

		auto operator++() -> iter& {
			++inner_;  // go to the next char
			// if hit the end of this string, and aren't at the end of the rope
			// since there might be some empty outer, so we use while loop here
			while (outer_ != last_ and inner_ == outer_->end()) {
				// go to the next string
				++outer_;
				if (outer_ != last_) {
					// and look at the start of it
					inner_ = outer_->begin();
				}
			}
			return *this;
		}

		auto operator--() -> iter& {
			// gotta do it the other way around, now
			// if we're at the end, step back until we find a non-empty string
			// (don't need to handle if they don't exist,
			//  since if so `--r.end()` doesn't make sense anyway)
			while (outer_ == last_ or inner_ == outer_->begin()) {
				--outer_;
				inner_ = outer_->end();
			}
			--inner_;
			return *this;
		}

		// postfix is always a simple formula
		auto operator++(int) -> iter {
			auto old = *this;
			++*this;
			return old;
		}

		auto operator--(int) -> iter {
			auto old = *this;
			--*this;
			return old;
		}

		auto operator*() const -> reference {
			return *inner_;
		}

		friend auto operator==(const iter& lhs, const iter& rhs) -> bool {
			// make sure the "outer" is the same; if so,
			return lhs.outer_ == rhs.outer_ and
			// either we're at the end of the rope, or we're pointing at the same char
				// since when outer_ == last_, we don't know what is inner_ then,
				// we need to do this separately
				(lhs.outer_ == lhs.last_ or lhs.inner_ == rhs.inner_);
		}

	private:
		using outer_t = std::vector<std::string>::const_iterator;
		using inner_t = std::string::const_iterator;

		iter(outer_t first, outer_t last)
			: last_(last)
			// find the first outer_ where the string is not empty, 
			// which means we can have something to go through with inner_
			, outer_(std::find_if_not(first, last, [](const auto &s) { return s.empty(); }))
			// if outer_ == last_ inner will be an empty inner_t,
			// otherwise, it will be the begin of outer_
			, inner_(outer_ == last_ ? inner_t{} : outer_->begin())
		{}

		outer_t last_ {};	// the end of the rope
		outer_t outer_ {};	// which string we're looking at
		inner_t inner_ {};	// which character we're looking at in the string

		friend class rope;
	};

public:
	// iterator types
	using iterator = iter;
	using const_iterator = iter;
	using reverse_iterator = std::reverse_iterator<iterator>;
	using const_reverse_iterator = std::reverse_iterator<const_iterator>;

	explicit rope(std::vector<std::string> rope) : rope_{std::move(rope)} {}

	// non-const begin/end
	auto begin() -> iterator {
		// call iterator constructor implicitly
		return { rope_.begin(), rope_.end() };
	}
	
	auto end() -> iterator {
		return { rope_.end(), rope_.end() };
	}

	// const begin/end; get cbegin/cend for free
	// const function are for those const objects
	auto begin() const -> const_iterator {
		return { rope_.begin(), rope_.end() };
	}

	auto end() const -> const_iterator {
		return { rope_.end(), rope_.end() };
	}

	auto cbegin() const -> const_iterator {
		return begin();
	}

	auto cend() const -> const_iterator {
		return end();
	}

	// reverse iterators all come for free as well
	auto rbegin() -> reverse_iterator {
		return reverse_iterator{ end() };
	}
	
	auto rend() -> reverse_iterator {
		return reverse_iterator{ begin() };
	}
	
	auto rbegin() const -> const_reverse_iterator {
		return const_reverse_iterator{ end() };
	}
	
	auto rend() const -> const_reverse_iterator {
		return const_reverse_iterator{ begin() };
	}
	
	auto crbegin() const -> const_reverse_iterator {
		return rbegin();
	}
	
	auto crend() const -> const_reverse_iterator {
		return rend();
	}

private:
	std::vector<std::string> rope_;
};

#endif // COMP6771_ROPE_H

```