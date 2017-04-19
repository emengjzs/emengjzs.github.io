---
title: XXChat TODOList
layout: post
tag: 
   - important
   - C++
header-img: img/friends.PNG
---
XXChat TODOList

打算开一本书 gitbook， 总结常见算法题和易错点，真的被逼的没办法了，死记也好理解也好一定要把算法这关啃下来，毫无疑问leecode是要刷一遍的，不然连门都进不了，既然这么喜欢考算法，就练给你看。

- [x] 合并有序链表。
- [x] 翻转数组找出最小的元素位置。
- [ ] Reactor模式下QPS的计算。
- [x] 回环矩阵下某一下标对应的值。
- [ ] 文本过滤词分析算法。
- [ ] HTTP服务的简单架构。
- [ ] FIFO Buffer的实现。
- [ ] 完全二叉树的最小子树。




## 1 合并两个有序链表为有序链表 

leetcode题目：[merge-two-lists](https://leetcode.com/problems/merge-two-sorted-lists/#/description)

### 题解：

- 迭代：不要忘记对合并链表的指针cur，两个链表的指针p、q进行迭代 ： p=p->next。
- 初始条件：可以先就地构造一个Dummy Head Node，不用额外判断头节点究竟是p还是q，直接进入主逻辑。
- 结尾：当某一个链表遍历完时，**直接将另一个链表的链接到合并链表尾**，**拜托不用再遍历余下的了**。
- 返回：Dummy Head Node的next指针即可

```c++
ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
  ListNode head(-1);
  ListNode* p = l1;
  ListNode* q = l2;
  ListNode* cur = &head;
  while (p && q) {
    if (p->val < q->val) {
      cur->next = p;
      p = p->next;
    }
    else {
      cur->next = q;
      q = q->next; 
    }
    cur = cur->next;
  }
  cur->next = p ? p : q;
  return head.next;
}
```

## 合并多个链表为有序链表

leetcode题目：[23. Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/)

### 题解

主要难点是C++ STL下的API，思路倒是知道。但各种挂在std:: 命名空间的函数以及蛋疼的命名，用法。

1. `make_heap`将数组建堆
2. `pop_heap`将最值放在最后位置，然后忽略最后的元素对前面元素数组重新调整为堆。**调用后**使用`back()`获取最值，调用一次`pop_back()`去除最值元素。
3. `push_heap` 将新的值放在最后位置，然后包含最后的元素对数组重新调整为堆。**调用前**使用一次`push_back`添加元素。
4. `sort_heap`按堆排序算法进行排序。
5. 不要忘记自定义排序函数。

```c++
ListNode* mergeKLists(vector<ListNode*>& lists) {
  ListNode head(-1);
  vector<ListNode*> validLists;
  for(ListNode* node : lists) {
    if (node) validLists.push_back(node);
  }
  auto cmp = [](ListNode* o1, ListNode* o2) { return o1->val > o2->val; };
  make_heap(validLists.begin(), validLists.end(), cmp);
  ListNode* cur = &head;
  while (! validLists.empty()) {
    pop_heap(validLists.begin(), validLists.end(), cmp);
    cur->next = validLists.back();
    validLists.back() = (validLists.back())->next;
    if (validLists.back()) {
      push_heap(validLists.begin(), validLists.end(), cmp);
    }
    else {
      validLists.pop_back();
    }
    cur = cur->next;
  }
  return head.next;
}
```





## 3  翻转数组找出最小的元素位置

Leetcode 题目：[Find Minimum in Rotated Sorted Array II](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array-ii/#/description)

### 题解:

1. **有序性**： 要立刻想到二分法（数组、矩阵、二叉平衡树等）同时需要考虑一下一些情况。
   - 是`low < high` 还是 `low <= high`? 前一个跳出循环时是low == high, 后一个跳出循环时是 high + 1 == low。一个是寻找的值在low中，一个寻找的值在high和low之间。
   - 考虑部分元素相等的情况，特别是**全部元素相等**的情况。此题中，如果n[low]==n[high]，1. 分裂点在中间，遍历找最小值，找出第一个变小的。 2. 由头到尾都是一样的，low对应元素即解。
2. midval 跟谁判断？如果和low判断，则无法判断数组进行了反转，此时最小值有可能在low 和 mid右边

```cpp
int findMin(vector<int>& nums) {
  int low = 0;
  int high = nums.size() - 1;
  while (low < high) {
    int mid = low + ((high - low) >> 1);
    int midval = nums[mid];
    if (midval > nums[high]) {
      low = mid + 1;
    }
    else if (midval < nums[high]) {
      high = mid;
    }
    else {
      for (int i = low ; i < high; i ++) {
        if (nums[i + 1] < nums[i]) {
          return nums[i + 1];
        }
      }
      return nums[low];
    }
  }
  return nums[low];
}
```



## 4 输出回环矩阵

Leetcode题目：[59. Spiral Matrix II](https://leetcode.com/problems/spiral-matrix-ii/#/description)

### 题解:

1. 一图胜千言。把矩阵分解为一圈一圈的环。
2. 考虑当跳出循环时，在循环条件中使用的变量的值，一般来说此时变量的值是**越界值**，例如用下标i遍历数组后i的值不再可用，**控制跳出循环时变量**的值能够避免异常发生。

![matrix](\img\xxchat\matrix.png)

```c++
vector<vector<int>> generateMatrix(int n) {
  vector<vector<int>> result(n, vector<int>(n));
  int v = 1;
  for (int i = 0; i <= n / 2; i ++) {
    int x = i;
    int y = i;
    while (y < n -1 - i) 
      result[x][y ++] = v ++;
    while (x < n -1 - i) 
      result[x ++][y] = v ++;
    while(y > i) 
      result[x][y --] = v ++;
    while(x > i)
      result[x --][y] = v ++;
  }
  if (n % 2) result[n / 2][n / 2] = v;
  return result;
}
```



## 回环矩阵下某一下标对应的值。

### 题解:

1. 先判断所在层数i，由上图可知，层数是点到四条边距离的最小值。

2. 计算一层的周长，为4(L- 1)，L为边长。

3. 第k层的边长为L = n - 2k，故周长为4(n - 2k - 1)

4. 得到层数为i后，计算第0~i-1层的周长和，计算到（i，i）处的值。

5. 若 x == i ，点在上方，若 y == n - 1-i点在右方，若x == n - 1 点在下方，若y == i 点在左方

   ​

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
        printf("%02X ", str[i]);
      else
        printf("   ");
    }
    printf(" ");
    for (int i = 16; i < 32 && i < str.size(); i++) {
      if (i < str.size())
        printf("%02X ", str[i]);
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
    printf("0x%08X ", _addr);
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

