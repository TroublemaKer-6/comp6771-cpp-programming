# w5tut

Tags: tut
Status: Done

**5.1.4**

```cpp
// b
// fill the 3 elems of v.
// return "is" for chaining.
auto operator>>(std::istream &is, vec3 v) -> std::istream& {
    is >> v.elems_[0];
    is >> v.elems_[1];
    is >> v.elems_[2];
    return is;
}
```

```cpp
// c
// fill the 3 elements of v from is.
// return "is" for chaining.
auto operator>>(std::istream &is, vec3 &v) -> std::istream& {
    is >> v.elems_[1];
    is >> v.elems_[2];
    is >> v.elems_[3];
    return is;
}
```

```cpp
// d
// fill the 3 elems of v via a standard algorithm.
// return "is" for chaining.
auto operator>>(std::istream &is, vec3 &v) -> std::istream& {
    std::copy(std::istream_iterator<double>{is}, std::istream_iterator<double>{}, v.elems_);
    return is;
}
```

The best way to do that:

```cpp
auto operator>>(std::istream &is, vec3 &v) -> std::istream& {
    is >> v.elems_[0];
    is >> v.elems_[1];
    is >> v.elems_[2];
    return is;
}
```

**5.1.5**

Since we can get those in private by the sub-script []

**5.2.2**

| & | bitand |  |
| --- | --- | --- |
| | | bitor |  |
| && | logic and | and (in c++) |
| || | logic or | or (in c++) |

**5.3 iterator validation**

```cpp
// [1, 2, 3, 4, 5]
//  ^ <- myVec.begin()

// auto vecBegin = myVec.begin() -> int*
// myVec.pushBack(100)
// myVec.pushBack(100)
// myVec.pushBack(100)
// myVec.pushBack(100)
// myVec.pushBack(100)
// myVec.pushBack(100)
// ... a lot then we need resizing

// *vecBegin <- THIS CAN BE INVALID
```

after resizing, it will do reallocation, the vector probably get a different part of the memory, the previous memory is not able to retrieve the vector info.