# const method

Created: August 13, 2024 12:29 PM

const methods are used for those const object, if we use a method not marked as const on a const object, error will be thrown.

```cpp
class Boo { 
	void print() const {
		...
	}
};

Boo b0;
const Boo b1;
```