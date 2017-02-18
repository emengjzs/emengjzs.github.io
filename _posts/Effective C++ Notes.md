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



## 09 不在`ctor`、`dtor`调用`virtual`方法 

- 由于在父类构造\析构函数调用在子类构造\析构之前，子类未完成初始化，virtual方法会调用父类行为，C++不会私自调用未定义的行为，因此**不会调**用子类virtual方法。

- 子类初始化逻辑应在子类构造中明确定义。

- Java旨在维持继承概念的完整性，在父类构造函数**可以调用**到子类的virtual方法，但仍不推荐在构造函数中调用子类virtual方法，因为子类部分仍未初始化。

  ```java
  // An example that the base class invoke the devired class in Java 
  class Base {
    public Base() { 
      System.out.println("Base::Base()"); 
      virt(); 
    }
    void virt() { 
      System.out.println("Base::virt()");
    }
  }

  class Derived extends Base {
    public Derived() { 
      System.out.println("Derived::Derived()"); 
      virt(); 
    }
    void virt() { 
      System.out.println("Derived::virt()"); 
    }
  }
  ```

  ```shell
  // Output
  Base::Base()
  Derived::virt()
  Derived::Derived()
  Derived::virt()
  ```

  ​

## 10 11 `=operator` 实现的正确姿势

- 标准的函数声明，```rhs``` means 'right hand side'：

  ```c++
  Clazz& operator=(const Clazz& rhs);
  ```

  **`=operator` 函数声明应返回赋值对象的引用**，保持连续赋值的语义。返回语句：

  ```c++
  return *this; 
  ```


- ```=operator```正确行为应该是，**先复制参数对象的数据，后删除自己对象的数据**。
- 这属于一种新旧对象的处置过程思想，尤其当旧的对象需要做出一定处理时，（如果不需要处理就随便了）如:
  - 在缓存池中，缓存空间满时，新的待缓存对象需要选择剔除一个已缓存对象以腾出空间，待剔除的缓存对象因为可能在缓存期间进行了更新，需要写入这些更新到后端（如数据库）中以保持一致性。则这些对象的写入规则应该是：（1） 将待删除缓存对象，（2）持久化到后端，（3）等到持久化完成时才写入新的缓存对象，也即在持久化期间需要对这个缓存空间加锁。**而不是**先删除旧缓存，缓存新对象，再持久化，否则，将会导致旧的后端数据又在缓存中，导致数据不一致。
- 适当调用父类的```=operator```函数，见12

## 12 定义完整的复制行为 

- 复制行为包括```copy ctor```， ```operator=```
- 完整行为包括：（1）通过调用父类的对应函数确保父类的完整复制行为。（2）完整处理当前每一个成员变量

## 13 以对象定义对象的所有权（RAII）

- C++ 和Java一个明显不同点是需要明确**对象的资源所有权**，资源一般指所占内存，也包括文件、流、锁等，**所有权决定了当对象结束使用时销毁其资源的义务**。

  ```c++
  std::vector<Fruit> fruits;
  ```

  ```vector```中的fruit对象的所有权属于fruits，fruits负责对fruit的资源管理义务，即fruits被销毁时，fruits销毁所有数组中的元素。

  ```c++
  std::vector<*Fruit> fruits
  ```

  ```fruits```仅对```*Fruit```变量（指针）所占资源负责，fruits**不负责**对fruit的资源管理义务，即fruits被销毁时，fruits不会销毁指向的元素。对应的，应是调用的```fruits.push_back(&fruit)```的对象(或函数)拥有对```fruit```的资源管理义务。

- 解决对象所有权处理（内存管理）的思路是，设计一个在栈上的对象，并将该对象和动态内存的对象关联起来，由于栈上的对象总会在函数或作用域结束时被销毁（调用析构函数），因此只要在此对象中的析构函数实行对动态内存对象的管理操作，即可完成所有权的管理。这些处理所有权的对象即```shared_ptr```, ```unique_ptr```, 