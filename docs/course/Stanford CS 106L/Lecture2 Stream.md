# Lecture2 Stream

### stream 用法

```cpp
int main() {

	// stream 流可以理解成一个 buffer, 因此直接输入会覆盖, 所以可以添加一个 ate
	ostringstream oss("Ito-En Green Tea");
	//ostringstream oss("Ito-En Green Tea", stringstream::ate);

	cout << oss.str() << endl; // Ito_en Green Tea;

	oss << "16.9 Ounces";
	cout << oss.str() << endl; 16.9en Green Tea; 

	isstringstream iss("16.9 Ounces");

	int amount;
	double amount;
	string unit;

	// Stream stops reading at any whitespace or an invalid character for the type
	iss >> amount;
	iss >> unit;

	cout << amount / 2 << " " << unit << endl;

	return 0;
}
```
尝试完成一个整数转换的函数

```cpp
int stringToInteger(const string& s) {
	istingstream iss(s);
	int result;
	// 通过状态码查看 stream 状态
	
	iss >> result;

	return result;
}
```
