---
title: Fun with cpp overload
date: 2017-11-10 14:48:03
thumbnail: https://isocpp.org/files/img/cpp_logo.png
tags: [c++]
categories: [programming language]
bgimage: https://isocpp.org/files/img/cpp_logo.png
author: "Li Chen"
---


--------------------------------------

## 2.1

> — Function declarations that differ only in the return type, the exception specification (18.4), or both cannot be overloaded.

可见函数声明不能根据返回值判断, 也不能根据`nonexcept`或`throw`(c++11开始deprecated, `c++17`开始被禁用了)来重载了. 

标准不许我们不能根据返回类型判断, 不过让我们开动脑筋思考怎么"破坏"这一规则. 

假设我们有一个类, 恰好和$czxyl$同名:`Czxyl`.我们希望写出这样的代码,并且符合本身的语义, 即没有任何隐式cast:

```c++
int a = get();
char b = get();
float c = get();
```

这里没有`parameter`, `function name`也一样, `c/v/ref-qualifier`也都没有. 所以能做文章的也只有`return type`了. 对重载稍微有些了解的, 或者接触过`safe bool`之类的idiom的童鞋, 肯定知道任何类都可以定义`operator std::string`, `operator int`这样的成员函数, 用来写出这样的代码: 

```c++
int a = czxyl;
char b = czxyl;
float c = czxyl;
```

有的读者看到这里, 可能就开始信心满满的写出了如下代码:

```c++
#include <iostream>
#include <string>
const class Czxyl
{
private:
    int a = 1;
    char b = 'c';
    float c = 1.1;
public:
    Czxyl() = default;
    constexpr Czxyl(int a, char b, float c) : a{ a }, b{ b }, c{ c } {}
    constexpr operator int() const { return a; }
    constexpr operator char() const { return b; }
    constexpr operator float() const { return c; }
} czxyl;

int main()
{
    int a = czxyl;
    char b = czxyl;
    float c = czxyl;
    std::cout << a << "\n" << b << "\n" << c << "\n";
}
```

运行代码已经放到[这里](https://wandbox.org/permlink/edj9oL5rRCUn56CT), 读者可以自己跑下. 嗯, 的确能跑通, 但是貌似和我们的要求还是有一定距离? 比较这里的`czxyl`只是`Czxyl`的一个实例罢了, 不是一个函数啊. 嗯, 对lambda比较熟悉且脑回路不太正常的童鞋肯定会想: 不就是要个函数嘛, $czxyl$给类加个`operator()`就行啦! 

```c++
const class Czxyl
{
private:
    int a = 1;
    char b = 'c';
    float c = 1.1;
public:
    Czxyl() = default;
    constexpr Czxyl(int a, char b, float c) : a{ a }, b{ b }, c{ c } {}
    constexpr operator int() const { return a; }
    constexpr operator char() const { return b; }
    constexpr operator float() const { return c; }
    Czxyl operator() () const { return *this; }
} czxyl;

int main()
{
    int a = czxyl();
    char b = czxyl();
    float c = czxyl();
    std::cout << a << "\n" << b << "\n" << c << "\n";
}
```
嗯, 的确是这样的, [这里](https://wandbox.org/permlink/hc3Ph0a9Fl8nrGgM)是运行代码, 但是这种做法其实是会有问题的. 

1. 占着`operator()`的坑了...如果以后有了仿函数的需求...
2. 这样一个类只能有一个这样的函数...
3. `czxyl`就是要函数, 不要仿函数, 反正就是看着不爽啦``-__-``. 


当然, 更多童鞋的脑回路比较正常, 不会因为`czxyl`就想到`czxyl()`, 搞个成员函数就能解决了嘛: 

### Solution 1: 
```c++
const class Czxyl
{
private:
    int a = 1;
    char b = 'c';
    float c = 1.1;
public:
    Czxyl() = default;
    constexpr Czxyl(int a, char b, float c) : a{ a }, b{ b }, c{ c } {}
    constexpr operator int() const { return a; }
    constexpr operator char() const { return b; }
    constexpr operator float() const { return c; }
    Czxyl get() const { return *this; }
    //Czxyl operator() () const { return *this; }
} czxyl;
```

嗯, 终于我们可以写出这样的[代码](https://wandbox.org/permlink/2phIyYZq2S9ud8n5)了: 

```c++
int a = czxyl.get();
char b = czxyl.get();
float c = czxyl.get();
std::cout << a << "\n" << b << "\n" << c << "\n";
```

当然, 很多时候应该抽象成这样: 

```c++
class CzxylHolder
{
private: 
    Czxyl const *czxyl
public:
    constexpr CzxylHolder(Czxyl const *czxyl) : czxyl{czxyl} {}
    constexpr operator int() const { return czxyl->getint(); }
    constexpr operator char() const { return czxyl->getchar(); }
    constexpr operator float() const { return czxyl->getfloat; }
    ~CzxylHolder() { delete czxyl; }
};
```

### Solution 2:

上面那种写法其实还是有一点不爽的--过于依赖`conversion function`, 我们就写不出`auto temp = half(para)`这样的代码. `half`对`int`会`/2`, 对std::string会`substr(s.length()/2)`... solution1遇到这样无赖的需求就跪了. 当然, 这里的para肯定是要用哪个模板的. 可能有同学开脑洞, 说返回值也搞个模板, 让编译器自己推导类型. 抱歉, 一开始就说过标准是不允许根据返回值重载的, 自然不能这么做啦. 好, 接下来我们解决问题, 首先需要大家了解`tag dispatch`这个`idiom`, $czxyl$一开始接触这货是刷水题时想在编译期搞出来尽可能多的东西. 然后在写编译期`loop`时发现需要用`template specialization`配合`recursion`来模拟`loop`, 其中就需要`tag dispatch`. 这里不多说, 不了解的请阅读[这篇文章](https://www.codeproject.com/Articles/857354/Compile-Time-Loops-with-Cplusplus-Creating-a-Gener), 以下是具体应用: 

```cpp
struct StringTag {};
struct IntTag {};

template<typename T> struct Return;

template<>
struct Return<StringTag>
{
    typedef std::string Result;
};

template<>
struct Return<IntTag>
{
    typedef int Result;
};
    
template<typename T>
typename Return<T>::Result half(typename Return<T>::Result para);

template<>
typename Return<IntTag>::Result half<IntTag>(typename Return<IntTag>::Result para)
{
    return para / 2;
}

template<>
typename Return<StringTag>::Result half<StringTag>(typename Return<StringTag>::Result para)
{
    return para.substr(para.length() / 2);
}
```

上面的代码如果读者对模板和类型推导没什么了解, 可能理解比较困难, 不过自己动手模拟一遍应该可以懂的. 然后就可以这么用了: 

```cpp
int integer = 4;
std::string str = "prpr";
auto ai = half<IntTag>(integer);
auto as = half<StringTag>(str);
```

[live example](https://wandbox.org/permlink/GSjxcsWVi8uyDGmA)


### Solution 3: 

可能读者们觉得`IntTag`, `StringTag`看着不爽, 想写这样的`auto ai = half(integer);`代码. 有没有办法呢? 用SFINAE就行了:  

```cpp
template<typename T>
typename std::enable_if<std::is_same<T, int>::value, int>::type half(T para)
{
    return para / 2;
}
template<typename T>
typename std::enable_if<std::is_same<T, std::string>::value, std::string>::type half(T para)
{
    return para.substr(para.length()/2);
}
```

用法: 

```cpp
int integer = 4;
std::string str = "prpr";
auto ai = half(integer);
auto as = half(str);
```
[live example](https://wandbox.org/permlink/f3ROTkO7uRsqf5Q6)



讲了这么多, **16.1.2.1**才讲完了...给出的3个solution应该还是挺有趣的哈哈. 我们这样玩有什么用呢? $czxyl$暂时没想出来, 反正图个乐子嘛. 不过有一点要说明, 大家可能觉得solution3是最简单直观的, 所以可能产生想法: 有这么简便的方法, 那么我只要学这一种就行啦. 一招鲜吃遍天嘛. 其实这种想法对于c++来说是挺危险的, 因为这是学习的过程, 不是使用的过程, 无论11/14/17/20的进步给大家带来多少方便之处, 这些也都只是针对使用的, 而不是学习的, 说白了, 现在的技巧的确要懂, 以前的解决方法也要学会, 即solution1, 2, 3都是该掌握的, 且不说以后工作时生成环境不一定支持新标准, 平时阅读别人的代码时不掌握这些技巧是看不懂的. 所以, 其实新标准对于新人的确增加了学习的负担, 带着过去的包袱, 学习新出的标准. 

------------------------------------

## 2.2

这里标准给出的代码很清楚, 不过需要注意只是针对`member function`的, 熟悉`ref-qualifier`的同学可能会猜测, `void g()&` orelse `void g() &&`是不是也遵循这样的规律呢? $czxyl$第一眼看到`2.2`这个这一条款时脑子里就蹦出来这个想法, 虽然以前其实已经学到过`ref-qualifier`的规则不是这么简单的, 但是一想到$czxyl$马上就能在标准中找到出处, 整个人感觉都要高潮了(谁说学c艹不是抖m的?). 

如果有同学还不知道这里`const`的具体含义, 可以看$czxyl$之前在segmentfault的[回答](https://segmentfault.com/q/1010000010667752)

```c++
class X {
    static void f();
    void f(); // ill-formed
    void f() const; // ill-formed
    void f() const volatile; // ill-formed
    void g();
    void g() const; // OK: no static g
    void g() const volatile; // OK: no static g
};
```

## 2.3 

此处标准给出的代码也是极好的: 
```c++
class Y {
void h() &;
void h() const &; // OK
void h() &&; // OK, all declarations have a ref-qualifier
void i() &;
void i() const; // ill-formed, prior declaration of i
// has a ref-qualifier
};
```

这里稍微和大家解释下, 就像$czxyl$上面的这个[回答](https://segmentfault.com/q/1010000010667752)里面提到的, 这里的`const-qualifier`可以描述成修饰的是`this`指针所指向的对象(当然, `this`指针本身也是`const`的.). 而这里的`&`和`&&`呢? 大家觉得这会不会是修饰`this pointer`的呢? 直觉好的童鞋肯定可以猜出不是. $czxyl$在这里给大家看个其他例子: 

在`class`里面, 如果lambda要捕获内部成员, 就需要再捕获列表里面弄个`this`. 比如`[&]`, `[=]`, 但是`[&this]`, `[=, this]`这样的都是`ill-formed`的, 因为
`this`默认就是按值捕获, 如果你显式使用`&this`, 那肯定不行啦, gcc会给`warning`. 即使`[&]`, 其实`this`也是按值传递的, 不过没有显式指定`this`, 所以是
`well-fromed`的.  
  
从上面这个例子可以看出, `this`不应该按引用来传递, 那么这里的`ref-qualifier`修饰的是什么? 要理清这个问题, 我们先要探究下c++的历史: 

在远古时期, `c++`的`oo`模型出现了, `this`指针也是80年代开始有的概念. 而此时`c++`还没有发展出`reference`这个语法. 所以, 为了向`member function`传入`this`, 只好弄成一个指针, 所以这就是为什么`const-qualifier`中的`this`是`Test const* const this`了. 而随着c++的发展, 引用出世了, 但是, 指针已经够处理了, 没必要为此重置对象模型, 直到而`ref-qualifier`的提案也在后来提了出来.,  指针才hold不住了, 只好对`*this`这个左值取引用了. 所以, `&`和`&&`俩`ref-qualifier`其实修饰的是左值`*this`. 当然,我们也没必要把以前的this做法批判一番, 一切都是历史原因. 

总结一下, 没有`reference`的时候, `member function`可能会被处理成这样: 

```c++
Member_function(AClass* const this, ...)
```

有了`reference`后, 这样做也是合理的, 当然, `starthis`更通用

```c++
void Member_function(Aclass& starthis, ...)
void Member_function(AClass&& starthis, ...)
void Member_function(AClass& const starthis, ...)
```

有些同学可能会对`ref-qualifier`本身有疑惑. $czxyl$在标准中找到了`optional`这个例子: 

```c++
// 23.6.3.5, observers
constexpr const T* operator->() const;
constexpr T* operator->();
constexpr const T& operator*() const&;
constexpr T& operator*() &;
constexpr T&& operator*() &&;
constexpr const T&& operator*() const&&;
constexpr explicit operator bool() const noexcept;
constexpr bool has_value() const noexcept;
constexpr const T& value() const&;
constexpr T& value() &;
constexpr T&& value() &&;
constexpr const T&& value() const&&;
template <class U> constexpr T value_or(U&&) const&;
template <class U> constexpr T value_or(U&&) &&;
```

大家可能不熟悉`optional`, 其实这个概念也是`c++`从其他语言里抄过来的, $czxyl$稍微熟悉些$standard \  ml$. 就从$standard \  ml$开始讲起. 很多时候我们都有`空`的需求, 比如`max(array)`, `array`不空当然没什么问题, `array`为空呢? 轮子哥觉得应该异常处理, 不过我们也可以用`空`这个概念处理啊, 在sml中, `空`用`NONE代指`, 意思就是什么都没有. 要往里面塞东西, 就需要`SOME something`. 在`max(array)`里, `array`为空, 我们就完全可以处理成`NONE`, 不为空时就`SOME maxelement`. $czxyl$觉得比异常优雅些, 哈哈. 不过需要注意的是, `c++`的`Optional`不是primitive, 就像`std::string`不是`primitive`, 而$sml$中的`option`同样也不是`primitive`. 在$sml$中, `option`可以这样实现: 

```sml
datatype 'a option = 
    NONE
  | = SOME 'a
```

`NONE`和`SOME`都是构造函数, 而我们可以清晰的看见, `NONE`后面是没有`parameter`的, 其它返回的就是一`'a option`, 而`SOME`能接受任意类型. 所以就是一填充的过程. 

有了这些铺垫, 上面的`Optional`源码大家应该就有点数了. sml中也有这样的函数, 比如`isSOME`对应`has_value`, `valof`对应`value`. 当然, sml因为自带pattern match, 所以不是很鼓励大家使用这两个函数, 毕竟都能用pattern match解决: 

```sml
fun inc_or_zero intoption =
    case intoption of
    NONE => 0
  | SOME i => i+1
```

不过c++中没有自带模式匹配, 所以我们不得不用它们啦. 貌似扯得有点远啦, 我们回到`Optional`这个类吧. 其实$czxyl$拎这个类出来是有原因的, 因为它的`ref-qualifier`非常适合讲解, 单看几个`value function`: 

```c++
constexpr bool has_value() const noexcept;
constexpr const T& value() const&;
constexpr T& value() &;
constexpr T&& value() &&;
constexpr const T&& value() const&&;
```

假设我们有一个实例`optional`, 如果它是左值, 我们想取它的`value`(前面的讲解已经说明`Optional`就是往里面塞个东西嘛), 那么这个`value`按照语义来说肯定也应该是左值,如果是右值, 也应该是右值, 当然, `const`也一样啦. 所以大家已经看到`ref-qualifier`的一个作用了吧, 是不是非常简单. 这里需要注意一下: 一个成员函数有了`ref-qualifier`, 它的其他重载函数(比如这里的其他`value function`)也都需要`ref-qualifier`修饰, 并且左值是不能调用`&`修饰的, 右值是不能调用`&&`修饰的(显而易见). 其实`&&`配合`move`还有其他作用, 比如`list.add(1).add(2).add(3)`, 这里重载`add`, 就能省下很多中间开销啦. 

