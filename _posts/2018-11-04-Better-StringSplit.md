---
layout: post
title: Better String Split in Java
tags: 
- Java
- String
---



# Better String Split in Java

`String.split` 是Java里很常用的字符串操作，在普通业务操作里使用的话并没有什么问题，但如果需要追求高性能的分割的话，需要花一点心思找出可以提高性能的方法。

`String.split`方法的分割参数`regex`实际不是字符串，而是正则表达式，就是说分隔字符串支持按正则进行分割，虽然这个特性看上去非常好，但从另一个角度来说也是性能杀手。

在Java6的实现里，`String.split`每次调用都直接新建`Pattern`对象对参数进行正则表达式的编译，再进行字符串分隔，而正则表达式的编译从字面上看就知道需要耗不少时间，并且实现中也没有对Pattern进行缓存，因此多次频繁调用的使用场景下性能很差，如果是要使用正则表达式分隔的话，应该自行对`Pattern`进行缓存。

```java
public String[] split(String regex, int limit) {
    return Pattern.compile(regex).split(this, limit);
}
```

但很多时候我们并不会真的想使用正则表达式分隔字符串，我们其实想的只是用一个简单的字符比如空格、下划线分隔字符串而已，为了需要是满足这个需求却要背上正则表达式支持的性能损耗，非常不值得。

因此在Java7的实现里，针对单字符的分隔进行了优化，对这种场景实现了更合适的方法。单字符不走正则表达式的实现，直接利用`indexOf`快速定位分隔位置，提高性能。

```java
/* fastpath if the regex is a
    (1)one-char String and this character is not one of the
    RegEx's meta characters ".$|()[{^?*+\\", or
    (2)two-char String and the first char is the backslash and
    the second is not the ascii digit or ascii letter.
*/
char ch = 0;
if (((regex.value.length == 1 &&
        ".$|()[{^?*+\\".indexOf(ch = regex.charAt(0)) == -1) ||
        (regex.length() == 2 &&
        regex.charAt(0) == '\\' &&
        (((ch = regex.charAt(1))-'0')|('9'-ch)) < 0 &&
        ((ch-'a')|('z'-ch)) < 0 &&
        ((ch-'A')|('Z'-ch)) < 0)) &&
    (ch < Character.MIN_HIGH_SURROGATE ||
        ch > Character.MAX_LOW_SURROGATE))
{
    int off = 0;
    int next = 0;
    boolean limited = limit > 0;
    ArrayList<String> list = new ArrayList<>();
    while ((next = indexOf(ch, off)) != -1) {
        if (!limited || list.size() < limit - 1) {
            list.add(substring(off, next));
            off = next + 1;
        } else {    // last one
            //assert (list.size() == limit - 1);
            list.add(substring(off, value.length));
            off = value.length;
            break;
        }
    }
    // If no match was found, return this
    if (off == 0)
        return new String[]{this};

    // Add remaining segment
    if (!limited || list.size() < limit)
        list.add(substring(off, value.length));

    // Construct result
    int resultSize = list.size();
    if (limit == 0)
        while (resultSize > 0 && list.get(resultSize - 1).length() == 0)
            resultSize--;
    String[] result = new String[resultSize];
    return list.subList(0, resultSize).toArray(result);
}
```

**有没有更快的方法？**如果分隔符不是单字符而且也不需要按正则分隔的话，使用`split`的方法还会和Java6一样使用正则表达式。这里还有其他备用手段：

1. 使用`StringTokenizer`,`StringTokenizer`没有正则表达式分隔的功能，单纯的根据分隔符逐次返回分隔的子串，默认按空格分隔，性能比`String.split`方法稍好，但这个类实现比较老，属于jdk的遗留类，而且注释上也说明不建议使用这个类。
2. 使用`org.apache.commons.lang3.StringUtils.split`分隔字符串，针对不需要按正则分隔的场景提供更好的实现，分隔符支持字符串。

**还能有更快的方法么？**注意到`String.split`和`StringUtils.split`方法返回值是`String[]`, 原始数组的大小是固定的，而在分隔字符串不可能提前知道分隔了多少个子串，那这个数组肯定藏了猫腻，看看是怎么实现的。

定位`String.split`单字符实现，发现分隔的子串其实保存在`ArrayList`里，并没有高深的技巧，直到路径的最后一行，代码对存储了子串的`ArrayList`再转成数组，而`toArray`的实现里**对数组进行了复制**。

```java
return list.subList(0, resultSize).toArray(result);
```

`StringUtils.split`方法里同样也是这样。

```java
return list.toArray(new String[list.size()]);
```

因此这里可以做一个优化，把代码实现复制过来，然后将方法参数返回类型改为`List`，减少数组复制的内存消耗。

**还能有更快的方法么？**其实很多时候我们需要对分隔后的字符串进行遍历访问做一些操作，并不是真的需要这个数组，这和文件读取是一样的道理，读文件不需要把整个文件读入到内存中再使用，完全可以一次读取一行进行处理，因此还可以做一个优化，增加参数作为子串处理方法的回调，在相应地方改为对回调的调用，这样能完全避免数组的创建。也就是说，**把字符串分隔看做一个流**。

```java
private static void splitWorker(final String str, final String separatorChars, final int max, final boolean preserveAllTokens, Consumer<String> onSplit) {
    if (str == null) {
        return;
    }
    final int len = str.length();
    if (len == 0) {
        return;
    }
    int sizePlus1 = 1;
    int i = 0, start = 0;
    boolean match = false;
    boolean lastMatch = false;
    if (separatorChars == null) {
        // Null separator means use whitespace
        while (i < len) {
            if (Character.isWhitespace(str.charAt(i))) {
                if (match || preserveAllTokens) {
                    lastMatch = true;
                    if (sizePlus1++ == max) {
                        i = len;
                        lastMatch = false;
                    }
                    onSplit.accept(str.substring(start, i));
                    match = false;
                }
                start = ++i;
                continue;
            }
            lastMatch = false;
            match = true;
            i++;
        }
    } else if (separatorChars.length() == 1) {
        // Optimise 1 character case
        final char sep = separatorChars.charAt(0);
        while (i < len) {
            if (str.charAt(i) == sep) {
                if (match || preserveAllTokens) {
                    lastMatch = true;
                    if (sizePlus1++ == max) {
                        i = len;
                        lastMatch = false;
                    }
                    onSplit.accept(str.substring(start, i));
                    match = false;
                }
                start = ++i;
                continue;
            }
            lastMatch = false;
            match = true;
            i++;
        }
    } else {
        // standard case
        while (i < len) {
            if (separatorChars.indexOf(str.charAt(i)) >= 0) {
                if (match || preserveAllTokens) {
                    lastMatch = true;
                    if (sizePlus1++ == max) {
                        i = len;
                        lastMatch = false;
                    }
                    onSplit.accept(str.substring(start, i));
                    match = false;
                }
                start = ++i;
                continue;
            }
            lastMatch = false;
            match = true;
            i++;
        }
    }
    if (match || preserveAllTokens && lastMatch) {
        onSplit.accept(str.substring(start, i));
    }
}

public static void split(final String str, final String separatorChars, Consumer<String> onSplit) {
    splitWorker(str, separatorChars, -1, false, onSplit);
}

// 使用方法
public void example() {
    split("Hello world", " ", System.out::println);
}
```

**还能有更快的方法么？**也有更极端的优化方法，因为在拿子串（`substring`方法）时实际发生了一次字符串复制，因此可以把回调函数改为传入子串在字符串的区间start、end，回调再根据区间读取子串进行处理，但并不是很通用，这里就不展示代码了，有兴趣的可以试一下。

**还能有更快的方...**







