---
layout: post
title: 关于一些配置Tips
tags: 
    - tips 
    - C++
    - config
---

##  在[vscode](https://code.visualstudio.com/)上使用[cmder](http://cmder.net/)

新建批处理文件`start_cmder.cmd`，建议放在`cmder\bin`下，写入如下命令：

```powershell
@echo off 
SET CMDER_ROOT=X:\path\to\cmder
"%CMDER_ROOT%\vendor\init.bat"
```

在vscode中设置

```json
"terminal.integrated.shellArgs.windows": ["/k", "X:\\path\\to\\cmder\\bin\\start_cmder.cmd"]
```

---

## 在Wndows10上使用clang编译64位简单程序

安装[clang](http://releases.llvm.org/download.html)

安装visual studio 2015 (头文件、lib、linker仍需由vs提供)

新建批处理文件`msvc64.cmd`，写入如下命令：

```powershell
"C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\vcvarsall.bat" amd64
```

将批处理文件放置在可通过变量访问的位置。

在代码文件夹下，使用clang++编译前需先执行`msvc64`，一次命令行session执行一次即可。

编译，必须指定c++14编译（默认设置），c++11会报错

```powershell
clang++ *.cpp
```

完成，以下是截图。

```powershell
D:\Program\C++\headfirstcpp                                                   
λ msvc64                                                                      
                                                                              
"C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\vcvarsall.bat" amd64  
                                                                              
D:\Program\C++\headfirstcpp                                                   
λ clang++ *.cpp                                                               
                                                                              
D:\Program\C++\headfirstcpp                                                   
λ a.exe                                                                       
1                                                                             
Fuck You!                                                                     
                                                                              
D:\Program\C++\headfirstcpp                                                   
λ                                                                             
```



