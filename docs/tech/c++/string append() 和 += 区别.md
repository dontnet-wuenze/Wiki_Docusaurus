没有区别

+= 是一个内联函数, 在内部调用了 append()

- string (1): `string& operator+= (const string& str)`
```cpp
basic_string& operator+=(const basic_string& _Right) {
    return append(_Right);
}
```

- c-string (2): `string& operator+= (const char* s)`
```cpp
basic_string& operator+=(_In_z_ const _Elem* const _Ptr) {
    return append(_Ptr);
}
```

- character (3): `string& operator+= (char c)`
```cpp
basic_string& operator+=(_Elem _Ch) {
    push_back(_Ch);
    return *this;
}
```

- Source: [GitHub: Microsoft/STL](https://github.com/microsoft/STL/blob/main/stl/inc/xstring)