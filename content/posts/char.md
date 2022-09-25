---
title: 聊聊char
date: 2017-03-01 16:47:03
tags: [字符]
thumbnail: http://tutorialfocus.net/wp-content/uploads/2017/02/drawit-diagram-9.png
tags: ["c"]
categories: ["programming language"]
bgimage: http://tutorialfocus.net/wp-content/uploads/2017/02/drawit-diagram-9.png
author: "Li Chen"
---

----------
### 什么是char?
----------
初学c语言的同学可能以为 `char`表示的就是字母意义的字符, 那我们来看下 [C Primer Plus](https://www.amazon.cn/%E5%9B%BE%E4%B9%A6/dp/B01DLB7ODA/ref=sr_1_4?ie=UTF8&qid=1488358257&sr=8-4&keywords=c+primer+plus)和 [Wikipedia上](https://en.wikipedia.org/wiki/C_data_types#cite_ref-c99generalrepr_5-1)对它的定义
> The `char` type is used for storing characters such as letters and punctuation marks, but technically it is an integer type. Why? **Because the `char` type actually stores integers, not characters**.



> Smallest addressable unit of the machine that can contain basic character set. It is an integer type. Actual type can be either signed or unsigned depending on the implementation. It contains CHAR_BIT bits.[3]

由此可见, `char`本质上存储的是整数, 且`(sizeof(char) == 1)`, 即可以存储256个不同整数的8位[^1]数据类型. 我们也可以把它理解为 `short short`[^2], 与64位的`long long` 要相乎应.  那`char`为什么而设计的呢? 是为了存储 `ASCII码`. 我们知道, 标准ASCII码是从0到127. 所以只需要7位的有符号整数就够了, 用char来表示绰绰有余, 不过这里有一点要注意, 就是虽然 `char` 是8位整数, 但是它的默认类型并没有在标准中定义, 也就是说在不同的平台上面它的默认值是可能是`signed`或`unsigned`(虽然两者都足够表示 ASCII 了). 据我所知, x86平台上的`char`[^3], 虽然这会稍微加快运行速度, 但是也会引起些潜在的 bug, 比如万一用户所在平台默认是`unsigned`, 而又使用了 `char和 getchar()` 来接受字符, 在判断 `EOF`时由于 char 是`unsigned` 的, 所以就会永远不停止, 而改成`int` 类型就能避免这一威胁. 当然, char 也可以显式声明为`unsigned` 和`signed`, 这一的话` 8-bit signed char `的范围是 `[-128, 127]` , `8-bit unsigned char `的范围是`[0, 255]`. 


-------------
### 从字符到字符串
-------------
既然 c 里面的字符说到底都是整数, 那么字符串该怎么界定呢? 本文不详谈(毕竟标题是char, 不是 `_char*` 或 `char[]`, 日后有机会再写), 简单的说, c 的字符串就是一系列字符(由整数表示)构成的, 不过要记住, 最后是以第一个[^4]`'\0'`[^5]来结尾的, 这被称作 c 风格字符串, 它是作为串出现的, 已经重载了 `operator <<`, 即使 `std::string` 里面的字符包含了`'\0'`, `string::operator <<` 也会一视同仁，继续输出后面的字符.





















<br><br/>

[^1]: 在大部分平台都是8位的, 不过也有少数古董系统是9位的.
[^2]: 这里只是逻辑上的类比, c 语言中只有 `short` 类型, 没有 `short short` 类型.
[^3]: 测试平台为 osx86, 系统为 sierra, 编译器为 clang
[^4]: 注意是第一个`'\0'`, 也就是说如果后面还有其他字符或者\0会被忽略, 可以自己打印下`char str[] = "abcd\0dfdf\0";`的 `strlen`. 
[^5]: '\0'的 ASCII码是`0`, 而'0'的 ASCII码是`30`.


<br><br/>
参考文献:
* http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf
* https://en.wikipedia.org/wiki/C_data_types#cite_ref-c99generalrepr_5-1
* https://www.quora.com/What-is-char-in-C-for
* http://stackoverflow.com/questions/11294850/the-ascii-value-of-0-is-same-as-ascii-value-of-0
* [c语言点滴](https://www.amazon.cn/%E5%9B%BE%E4%B9%A6/dp/B00FG1RWJA/ref=sr_1_1?ie=UTF8&qid=1488360299&sr=8-1&keywords=c+%E8%AF%AD%E8%A8%80%E7%82%B9%E6%BB%B4)
* https://segmentfault.com/q/1010000008344245

