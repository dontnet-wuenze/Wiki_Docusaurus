当cin>>从缓冲区中读取数据时，若缓冲区中第一个字符是空格、tab或换行这些分隔符时，cin>> 会将其忽略并清除，继续读取下一个字符，若缓冲区为空，则继续等待。但是如果读取成功，字符后面的分隔符是残留在缓冲区的，cin>>不做处理。

但是，getline() 读取数据时，并非像 cin>> 那样忽略第一个换行符，getline()发现 cin 的缓冲区中有一个残留的换行符，不阻塞请求键盘输入，直接读取，送入目标字符串。

因此如果在 geline() 前有过 cin 的操作, 可以通过 cin.ignore() 清楚缓存区残留的符号。

示例:
```cpp
getline(cin, name);
cin >> age;
cout << name << age;
cin.ignore();
getline(cin, response);
```

如果没有 cin.ignore, 读取完 age 后会有 \n 在缓冲区中, 下一个 getline 就会读入缓冲区的 \n 然后终止。