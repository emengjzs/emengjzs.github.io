---
layout: post
title: Effective C++ Notes (1)
tags: 
    - C++
---

个人关于Effective C++的笔记。

## Note：

- 未完全根据 C++11\14 进行修正，待更新

## 01 关于C++

> 可以参考[怎么样才算是精通 C++？ - vzch的回答 - 知乎](https://www.zhihu.com/question/19794858/answer/18448868)

##  02 尽少使用 `#define`

- 常量：用 `const`代替

- 函数：用`inline`代替

- 类常量： 用`const`， ```enum```代替



## 03 尽量使用`const`

- `const`的对象： 一般变量， 引用参数，指针，返回值、成员函数
- 养成能写**`const`**就添加**`const`**关键字的习惯，严谨定义对象的性质。
- 在`const` 和 非`const`函数有相同实现时，在非`const`函数实现中复用`const`函数实现。

> 而在Java中，仅当成员方法、方法参数、lambda方法中引用的变量中需要特别强调不可变的性质时才需要声明`final`关键字，一般情况下**不需要使用**`final`，**对一个方法中的对象参数声明为`final`几乎没有意义**。

> 在Javascript 中，也尽量使用`const` 和 `let`代替`var`

## 04 使用对象前确保对象已初始化

- **每一个类型的构造函数需要确保已对所有成员变量初始化**，特别是，子类需要通过调用父类的对应函数确保父类部分的完整初始化行为。
- 不同于Java， **应使用成员初始化列表而不是在构造函数中使用赋值操作**。
- 静态（全局）对象使用函数包裹，以函数方式（接口）提供静态对象的访问。

## 06 对每个编写的类确定是否允许复制行为

- 复制行为包括```copy ctor```， ```operator=```
- 若确定**不允许/不需要**复制行为时，应**禁用**```copy ctor```， ```operator=```，（通过`private` 或 `delete` )
- 若确定允许/需要复制行为时，仔细定义完整的复制行为，参见 12

## 07 基类的析构函数必须声明为virtual

- 对基类的析构函数声明为`virtual ~Clazz() = 0`，可以防止基类被实例化。

- 如果不是用作基类，则不应声明析构函数为virtual。

- 不要继承没有声明为虚析构函数的类，特别是，**不要继承任何标准库类**，**请使用组合而非继承**。

> （在ES6中也不要继承任何内置对象，包括Error）

## 08 不使用异常

- 尽可能不使用异常。
- 不在析构函数上抛出异常，如果可能出现异常，必须在析构函数中进行捕获和处理。否则，析构数组时可能出现内存泄露和不一致数据。
- 不使用异常规格（exception specification）

> 一些关于异常的使用观点 https://www.zhihu.com/question/22889420

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
    void virt() { System.out.println("Base::virt()"); }
  }

  class Derived extends Base {
    public Derived() { 
      System.out.println("Derived::Derived()"); 
      virt(); 
    }
    void virt() { System.out.println("Derived::virt()"); }
  }
  ```

  ```shell
  // Output
  Base::Base()
  Derived::virt()
  Derived::Derived()
  Derived::virt()
  ```


## 10 11 `=operator` 实现的正确姿势

- 标准的函数声明，```rhs``` means 'right hand side'：

  ```c++
  Clazz& operator=(const Clazz& rhs);
  ```

  **`=operator` 函数声明应返回赋值对象的引用**，保持连续赋值的语义。返回语句：

  ```c++
  return *this; 
  ```


- 考虑参数`rhs`和自身对象是同一个引用的case
- ```=operator```正确行为应该是，**先复制参数对象的数据，后删除自己对象的数据**。
- 这属于一种新旧对象的处置过程思想，尤其当旧的对象需要做出一定处理时，（如果不需要处理就随便了）如:
  - 在缓存池中，缓存空间满时，新的待缓存对象需要选择剔除一个已缓存对象以腾出空间，待剔除的缓存对象因为可能在缓存期间进行了更新，需要写入这些更新到后端（如数据库）中以保持一致性。则这些对象的写入规则应该是：（1） 将待删除缓存对象，（2）持久化到后端，（3）等到持久化完成时才写入新的缓存对象，也即在持久化期间需要对这个缓存空间加锁。**而不是**先删除旧缓存，缓存新对象，再持久化，否则，将会导致旧的后端数据又在缓存中，导致数据不一致。
- 适当调用父类的```=operator```函数，见12

## 12 定义完整的复制行为 

- 复制行为包括```copy ctor```， ```operator=```
- 完整行为包括：（1）通过调用父类的对应函数确保父类的完整复制行为。（2）完整处理当前每一个成员变量

## 13 17 以对象定义对象的所有权（RAII）

- C++ 和Java一个明显不同点是需要明确**对象的资源所有权**，资源一般指所占内存，也包括文件、流、锁等，**所有权决定了当对象结束使用时销毁其资源的义务**。

  ```c++
std::vector<Fruit> fruits;
  ```

  `vector`中的fruit对象的所有权属于fruits，fruits负责对fruit的资源管理义务，即fruits被销毁时，fruits销毁所有数组中的元素。

  ```c++
std::vector<*Fruit> fruits
  ```

  ```fruits```仅对```*Fruit```变量（指针）所占资源负责，fruits**不负责**对fruit的资源管理义务，即fruits被销毁时，fruits不会销毁指向的元素。对应的，应是调用的```fruits.push_back(&fruit)```的对象(或函数)拥有对```fruit```的资源管理义务。

- 解决对象所有权处理（内存管理）的基本思路是，设计一个在栈上的对象，并将该对象和动态内存的对象关联起来，由于栈上的对象总会在函数或作用域结束时被销毁（调用析构函数），因此只要在此对象中的析构函数实行对动态内存对象的管理操作，即可完成所有权的管理。这些处理所有权的对象即```shared_ptr```, ```unique_ptr```。

- RAII的核心是，当对象被创建时，其生命周期也被准确定义，必定存在一个确定条件，使得对象资源在满足条件时一定会被回收处理， 且确定条件必定可达。

- **使用单独的语句声明创建管理资源对象（make_shared()等)。** 不要与其他函数调用等语句复合。（如 ```getCat(make_shared<int>(42), init())```， 若```init```发生异常抛出时将可能导致内存泄露）

- 在管理资源对象中良好封装被管理资源的`delete`操作，不要暴露到外部，否则可能会出现兼容性问题。

- 一个典型的RAII特性使用的例子是：不需要对流（`istream`，`ostream`）显式调用`close`, [stackoverflow](http://stackoverflow.com/questions/748014/do-i-need-to-manually-close-an-ifstream)

## 14 注意资源管理类的复制行为

- 一般情况下应禁用```operator=```和```copy ctor```，C++11之前使用```private```限定访问符，C++11之后使用```delete```关键字
- 若可以复制，则明确复制的行为：（1）仅复制引用 （2）深度复制 （3）转移所有权

## 15 资源管理类需要提供给对资源的访问接口 

- 通过```operator->```访问对应被管理对象的公开属性和方法。
- 通过```get()```获得原始对象的指针。
- 通过隐式转换 ```operator T()``` （不建议任何隐式行为）

## 16 注意使用```delete []```对动态分配的数组进行析构

- 给定一个指针p，系统无法知道p指向一个对象还是指向一个对象数组，只能通过调用不同的```delete operator```（```delete``` 和```delete[]```）间接告诉系统需要释放的行为。
- 只要向指针p调用```delete[]```， 系统即知道需要释放数组空间，而且也知道分配的内存大小。主流编译器通常有两种方法记录数组的元数据。
  - over-allocation：大部分编译器采用此方法实现，使用即另外再分配一段空间专门存储数组的元信息。通常放置在对象分配空间的前面，**注意此时传入```operator delete[]```的指针会指向元数据开始处**（比第一个对象的分配地址更低的地址），因为元数据本身的空间也需要回收。
  - associative array：专门设置一个内置对象（如```arrayLengthAssociation```）存储所有动态分配数组的元信息。
- **不要对数组对象使用```typedef```声明类型别名**，这会在使用别名时掩盖了数组对象的实质。
- 对于数组对象的动态分配，建议使用```vector<T>```代替。

## 18 设计良好的接口

- 尽量使用自定义的类型封装数据，限制合法输入，并提供可读的接口声明。

- 接口的语义应该符合人的惯性思维，特别地，要和语言内置的接口、类型声明风格保持一致。


> 话虽这样说，但个人认为准官方日期库[date](https://github.com/HowardHinnant/date)并没有设计出易用的日期API，反而造出一堆需要理解的晦涩的学术概念，如```time_point```, ```duration```, ```system_clock```等（说明文档[在此](https://howardhinnant.github.io/date/date.html)），并暴露在API层上，使用前必须得先了解这些概念才能上手。从来没用过一个简单需求的API能用得如此痛苦。严谨是一回事，易用是另一回事。
>
> 例如假设需要将一个int整形当作Unix时间戳并转化为一个日期对象，获取年月日等的信息时，JavaScript 只需要：
>
> ```javascript
> let myDate = new Date(1487489218000);
> myDate.getYear();
> ```
>
> 唯一注意的是时间戳的单位是毫秒，算是一个不足，可以使用更好的第三方库[moment](http://momentjs.cn/)：
>
> ```javascript
> let day = moment.unix(1487489218);
> day.year();
> ```
>
> 而C++中需要：
>
> 1. 将```uint64_t```转成```duration<microseconds>```，告诉这个是以毫秒为单位的。
>
>    ```c++
>    microseconds{ timestamp }
>    ```
>
> 2. 从```duration```构造出时间点```time_point```，```duration```只是一个时间段的值，要设定基准点为Unix系统时钟`system_clock`。
>
>    ```c++
>    const time_point<system_clock> datetime_point(microseconds{ timestamp });
>    ```
>
> 3. 这时候还不能拿出年月日的数据，需要转化为年月日，还要告诉如何处理时分秒的数据，这里把时分秒数据截断，只要拿到年月日就行。
>
>    ```c++
>    floor<days>(datetime_point)
>    ```
>
> 4. 需要转为一个日期对象，是`Date`类型吗？不是，是`year_month_day`这么一个名字：
>
>    ```c++
>    auto ymd = year_month_day(floor<days>(datetime_point));
>    ```
>
> 5. 这时候终于可以获取年月日了，文档还说明最好转换为`unsigned`类型，需要这么写：
>
>    ```c++
>    unsigned(ymd.month());
>    ```
>
> 6. 这是如果需要获取时分秒信息怎么办，不好意思，`year_month_day`就是年月日，没有其他信息，自己查文档吧。
>
> 这真的令人失去耐心。可能有提供更简洁的方法，但至少根据文档说明应该是这样写的。
>
> 另外，日期的构造声明方法也破坏了**概念的一致性**，为了实现声明日期的简洁化，擅自使用除号重载`/`作为日期属性分割符([date.h](https://github.com/HowardHinnant/date/blob/master/date.h#L153))，又由于重载符只能在自定义类型中使用，不能写成`2015/4/13`，只能写成这样的半成品：
>
> ```c++
> constexpr auto x1 = 2015_y/mar/22;
> ```
>
> 还增加使用者的记忆负担，年月日必须至少有一个需要以显式类型声明，并且分割符是`/`， 而不是`.` 或者 `\` 。还不如好好地遵守C++构造函数的语法传递参数。