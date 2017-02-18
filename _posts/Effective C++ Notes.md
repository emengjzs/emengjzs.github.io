## C++ Primer Notes



### IO

#### wchar_t, char16\_t, char32\_t

wchar_t 保证能够存储、输出国际字符，没有声明编码方式和存储大小.

#### 关联输入输出

```c++
ins.tie(&outs)
```

任何读取操作都会先刷新被绑定的输出流。

#### Example

```c++
void read_f_file() {
  ifstream fin("F.txt");
  string line;
  if (fin) {
    while (getline(fin, line)) {
      cout << line << endl;
    }
  } else {
    cerr << "can't not open file F.txt";
  }
}
```

#### 注意！

- 标准输入输出流`<iostream>` (input/output stream)、文件流`fstream` (file stream)、字符串`string`位于**不同的头文件中**！
- 注意namespace
- 使用`std::getline(istream& is, string& str, char delim)`读取输入流中的一行。line小写。[cplusplus](http://www.cplusplus.com/reference/string/string/getline/)
- 不需要对流显式调用`close`, [stackoverflow](http://stackoverflow.com/questions/748014/do-i-need-to-manually-close-an-ifstream)



### 容器-1

| STL in C++ | Java                              |
| ---------- | --------------------------------- |
| `vector`   | `ArrayList`                       |
| `deque`    | `Deque` (ArrayQueue ? LinkedList) |
| `list`     | `LinkedList`                      |
| `array`    | `Arrays.asList(... T)`            |
| `string`   | `String`                          |

- 标准容器名全**小写**（非面向对象）

操作性能： 随机读、顺序读、随机插入、顺序（前、后）插入、删除、大小



#  Effective C++ Notes

##  02 尽少使用 `#define`

- 常量： `const`

- 函数：`inline`

- 类常量： `const`

  ​