# w3tut

Tags: tut
Status: Done

3.1

- sort from back to front
- change the operator

```cpp
std::sort(vec.start(), vec.end(), std::greater<int>);
std::sort(vec.rstart(), vec.rend());
```

test

```cpp
std::adjacent_find -> oneToFive.end()
```

<< and >> std::cout those are just methods

```cpp
struct cout{
	template<typename T>
	std::cout& operator<<(T t) {
		static_assert(false, "not implemented");
	}
	
	template<>
	
}
```