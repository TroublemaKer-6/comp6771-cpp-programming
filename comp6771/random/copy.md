# copy assignment / construction

Created: June 20, 2024 1:38 PM

### copy assignment / construction

```cpp
int main() {
		auto a = point { 1, 2 };
		auto b = point { 3, 4 };
		auto c = point { 5, 6 };
		
		a = c; // copy assignment

		// these 2 lines are the same
		auto q = c; // copy construction
		auto p = point{ c }; // copy construction
}
```