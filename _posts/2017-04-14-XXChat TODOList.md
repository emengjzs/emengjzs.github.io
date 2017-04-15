---
title: XXChat TODOList
layout: post
tag: 
   - important
   - C++
header-img: img/friends.PNG
---
XXChat TODOList

这周要忙作业的事情，争取下周把这些做完吧，算法要加强！！

- [ ] 合并两个有序链表为有序链表。
- [ ] 合并多个链表为有序链表。
- [ ] 翻转数组找出最小的元素位置。
- [ ] Reactor模式下QPS的计算。
- [ ] 回环矩阵下某一下标对应的值。
- [ ] 文本过滤词分析算法。
- [ ] HTTP服务的简单架构。
- [ ] FIFO Buffer的实现。
- [ ] 完全二叉树的最小子树。



## 10 二进制文件显示的实现

笔试题，显示和UltraEdit一样的效果显示二进制文本。

输入\输出（这里一行显示16字节，实际要求32字节）：

```shell
wrefdfgdferetertfdgfgfhththththththt
0x00000000 77 72 65 66 64 66 67 64 66 65 72 65 74 65 72 74   wrefdfgdferetert
0x00000010 66 64 67 66 67 66 68 74 68 74 68 74 68 74 68 74   fdgfgfhththththt
0x00000020 68 74 68 74                                       htht
```

### 题解

1. 格式化输出马上想到装饰者模式。
2. `printf`的格式化输出要熟悉，其中
   - `%s` 字符串，字符串要求为`char*`， `string`通过`c_str()` 方法转换。
   - `%X` 十六进制大写，不包含`0x`。
   - `%.5X` 表示前填充`0`，输出5位。 
3. 用`unique_ptr` 存放引用。
4. `cin`可直接输入至`string`，至遇到空白符、换行符截断，不包含空白符、换行符


```c++
#include <cstdio>
#include <iostream>
#include <memory>
#include <string>

class BinaryViewer {
 public:
  BinaryViewer() {}
  virtual ~BinaryViewer() {}
  virtual void Output(std::string str) = 0;
};

class BinaryViewerDecorator : public BinaryViewer {
  std::unique_ptr<BinaryViewer> _viewer;

 public:
  BinaryViewerDecorator(BinaryViewer* viewer)
      : BinaryViewer(), _viewer(viewer) {}
  virtual void Output(std::string str) = 0;
  inline void InnerOutput(std::string str) { _viewer->Output(str); }
};

class BinaryHexViewer : public BinaryViewer {
 public:
  BinaryHexViewer() : BinaryViewer() {}
  virtual void Output(std::string str) override {
    for (int i = 0; i < 16; i++) {
      if (i < str.size())
        printf("%.2X ", str[i]);
      else
        printf("   ");
    }
    printf(" ");
    for (int i = 16; i < 32 && i < str.size(); i++) {
      if (i < str.size())
        printf("%.2X ", str[i]);
      else
        printf("   ");
    }
    printf(" ");
  }
};

class BinaryTextViewer : public BinaryViewer {
 public:
  BinaryTextViewer() : BinaryViewer() {}
  virtual void Output(std::string str) override { printf("%s", str.c_str()); }
};

class EndLineViewer : public BinaryViewer {
 public:
  EndLineViewer() : BinaryViewer() {}
  virtual void Output(std::string str) override { printf("\n"); }
};

class CombineViewer : public BinaryViewerDecorator {
  std::unique_ptr<BinaryViewer> _viewer2;

 public:
  CombineViewer(BinaryViewer* viewer1, BinaryViewer* viewer2)
      : BinaryViewerDecorator(viewer1), _viewer2(viewer2) {}
  virtual ~CombineViewer() {}
  virtual void Output(std::string str) override {
    InnerOutput(str);
    _viewer2->Output(str);
  }
};

class AddressViewer : public BinaryViewerDecorator {
  uint32_t _addr;

 public:
  explicit AddressViewer(BinaryViewer* viewer)
      : BinaryViewerDecorator(viewer), _addr(0) {}
  virtual ~AddressViewer() {}
  virtual void Output(std::string str) override {
    printf("0x%.8X ", _addr);
    _addr += 32;
    InnerOutput(str);
  }
};

int main() {
  AddressViewer viewer(new CombineViewer(
      new CombineViewer(new BinaryHexViewer(), new BinaryTextViewer()),
      new EndLineViewer()));
  std::string input;
  while (std::cin >> input) {
    for (int i = 0; i < input.size(); i += 16) {
      viewer.Output(input.substr(i, 16));
    }
  }
}
```

