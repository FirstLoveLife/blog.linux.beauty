---
title: Introduction to double pointers---part1
date: 2017-05-14 19:48:03
thumbnail: https://d3nmt5vlzunoa1.cloudfront.net/clion/files/2017/04/logo-sun.jpg
tags: [c]
categories: [programming language]
author: "Li Chen"
bgimage: https://d3nmt5vlzunoa1.cloudfront.net/clion/files/2017/04/logo-sun.jpg
---
> 这篇文章试图从最简单的例子入手展开指针的实用的技能. 考虑到这篇文章的某些读者可能不是很熟悉c语言或者对指针仍然不是很熟悉, 所以我们先来看3个非常简单的例子来热下身, 不过为了增加点乐趣, 这里也会列出常见的一些错误. 不过为了保证文章的流畅性, 关于指针的语法细节我不会多涉及. 虽然很多例子都可以改用reference而不是pointers, 但是这里用pointers能更好的讲解, 有兴趣的同学也可以自己尝试用reference改写例子, 毕竟多动手时提高的唯一方式啊... 本文采取的书写方式主要是用问答式, 并且尽量将问题进行分解, 也就是说每次提问的答案都很简短, 但是问题会因此增多, 也会反复强调些要点. 所以希望大家看到一个问题先做思考. 当然如果大家发现我的答案和我的提问不匹配时希望能留言或者发邮件至czxyl@protonmail.com给我一个改正的机会, 谢谢. 

#  warm up

### case0: preface

```C
int main()
{
    int *x = new int(1);
    int **x_address = &x;
}
```
Q0.1: 这里的x和x_address代表什么?
**answer**: x 代表`1`这个rvalue的地址, 也就是说它的值是就是一个整数的地址 或者说x这个地址上存储着一个`int 1`, 如果我们对x进行`dereference`, 得到的值就是1了 `x_address`代表着x这个地址的地址. 如果我们对`x_address`进行`dereference`, 那么得到的值就是x, 也就是一个整数的地址了. 如果再进一步`dereference`得到的就是1了. 所以本文就是围绕着这么个最简单的知识展开的. 

### case1: swap

#### swap: version A 

```C
void swap(int *a, int *b)
{
    int temp = *a;
    *a = *b;
    *b = temp;
}
```

#### swap: version B 

```C
void swap(int *a, int *b)
{
    int *temp = a;
    a = b;
    b = temp;
}

```

#### swap: version C 

```C
void swap(int **a, int **b)
{
    int ** temp = a;
    a = b;
    b = temp;
}
```

#### swap: version D

```C
void swap(int a, int b)
{
    int temp = a;
    a = b;
    b = temp;
}
```
###### Q1.1: 哪个version是对/错的?

**answer**: A是对的, B, C, D是错的. 

###### Q1.2: A和B的区别是什么?

**answer**: 很明显, A交换了a和b指向的address之上的value. 而B交换的是两个address本身, 
然而其上的value依然没有变. 

###### Q1.3: version C错在哪里(以后会省略前缀version)?

**answer**: 懂了B的错误后, 很轻松的就能发现C里面交换的只是地址的地址. 所以最顶层的value依然是不变的. 这里提这个形式的例子主要是为了引出主题(double pointer)

### case2: allocate

#### allocate: version A
```C
void alloc1(int* p) {
   p = (int*)malloc(sizeof(int));
   *p = 10;
}
int main(){
   int *p = NULL;
   alloc1(p);
   printf("%d ",*p);
   free(p);
   return 0;
}
```
#### allocate: version B
```C
void alloc2(int** p) {
   *p = (int*)malloc(sizeof(int));
   **p = 10;
}
int main(){
   int *p = NULL;
   alloc2(&p);
   printf("%d ",*p);
   free(p);
   return 0;
}
```

###### Q2.1: A与B的对错?

**answer**: A错B对

###### Q2.2: A会造成哪些问题?

**answer**: 1. 无法准确打印出`*p`的值. 2. 会有内存泄漏


###### Q2.3: swap::version D 与 allocate::version A有什么相同的错误之处?

**answer**: 都妄图通过传`value`来改变`value`, 注意, 此处我定义的`value`是广义上的, 就像allocate::version A传入的是一个指针`int *p`, 同时接受的也是`int *`类型, 所以我在这里统一看做传`value`. 与之相对的就是allocate::version B的传参就是传`address`了. 

###### Q2.4: swap::version A 与 allocate::version B有什么相同的正确之处?

**answer**: 使用`function`来改变值时都传入的是其地址. 

###### Q2.5: 为什么无法准确打印出`*p`的值?

**answer**: 结合Q2.3, 可以看出其实p是作为一个值传入的. 自然alloc1()里面的p再怎么分配malloc也和main()中的p没有关系了. 同时, 由于alloc里面分配的内存的所有权并没有返回转交到main()中, 所以会造成`p = (int*)malloc(sizeof(int));`出来的内存就被抛弃了, 造成内存泄漏

###### Q2.6: 如何确定真的出现了内存泄漏了?

**answer**: 当然, 这里比较直观, 所以能肉眼看出来, 但是程序一旦复杂, 想检测这种内存泄漏的情况可就不一定能一眼看出来了. 所以我们需要一些工具, 比如vs下的`_CrtDumpMemoryLeaks`或者linux下的`valgrind`. 

###### Q2.7: 详细阐述下version B的正确性?

**answer**: 首先, 传入的是一个`int*`的地址, 也可以说成`pointer to pointer`. 然后在这个地址的值(本质也是一个地址)上开辟了空间`*p = (int*)malloc(sizeof(int));`. 因此相当于在swap时需要往下探入一层(从value到address), 这里也是在地址上的基础上再往下探入到地址的地址, 这样地址能够被在main()中的p保存. 因此allocate2()中对**p的修改也能在main()中反映. 

# Derive a conclusion from case1 and case2

我们想要通过函数修改任何一个值必须传入它地址. 而不能仅仅传入值, 即使它本身是一个地址, 而你想要修改的就是这个地址, 那么你需要做的就是传入这个地址的地址. 不过写到这里, 读者可能还是不知道什么时候需要用到这种`double pointers`. 所以, 接下来我们通过各种例子来展示下此技巧的威力  

# Application for double pointers

## Linkedlist

### case 1: insert(tail)

> 这里的insert是在尾部插入结点, 并且为了讲解的效果, 这个例子不使用返回值, 统统使用void. 并且以下某些结论也是在仅使用void得出的. 如果返回head的话就没讨论的意义了......

#### insert: version A(trival && singal pointer)

```C
void insert(node *head, node *inserted_node) //假设head是链表头, inserted_node是带插入尾部的已分配空间的结点
{
	node *current_node = head;
	if (current_node == NULL)
	{
		head = inserted_node;
	}
	else
	{
		while (current_node->next)
		{
			current_node = current_node->next;
		}
		current_node->next = inserted_node;
	}
}
```

#### insert: version B(non-trival && double pointers)

```C
void insert(node **head, node *inserted_node)
{
    node **current_node = head;
    while (*current_node)
    {
        current_node = &(*current_node)->next;
    }
    *current_node = inserted_node;
}
```

#### insert: version C(trival && singal pointer)

```C
void insert(node *head, node *inserted_node) 
{
    node *current_node = head;
    while (current_node)
    {
        current_node = current_node->next;
    }
    current_node = inserted_node;
}
```




###### Q1.1: version A省略掉`if`部分的判断会怎么样? 

**answer**: 插入第一个结点(头结点)时会访问到`NULL->next`, 非法内存访问.

###### Q1.2: version B为什么不需要这个version A的`if`条件?

**answer**: 想要回答这个问题, 只需要考虑version B在插入头结点时的是如何处理的就行了. 与version A不同, version B是用`*current_node`做循环条件的, 而不是`(*current_node)->next`. 这样做的好处是避免了`NULL->next`这样的错误步骤出现. 
###### Q1.3: version A可以写成`while (current_node)`来避免使用if条件, 也就是version C这样的吗?

**answer**: 我们来看下完整的代码, 希望读者能上机运行下
```C
#include "stdafx.h"
class node
{
public:
	node *next;
	int elem;
	node(int e) : next{NULL}, elem{e} {}

};

void insert(node *head, node *inserted_node)
{
	node *current_node = head;
	while (current_node)
	{
		current_node = current_node->next;
	}
	current_node = inserted_node;
}

int main()
{
	node *head = NULL;
	node *inserted_node = new node(1);
	insert(head, inserted_node);
	inserted_node = new node(2);
	insert(head, inserted_node);
	return 0;
}
```
首先我们必须要搞清楚我们传参的目的是什么, 以上面这份代码为例, 我们的第一个insert其实是需要修改head的**value**, 但是, 我们我们传入的其实也是head的**value**, 回想下文章开始的case1, case2, 毫无疑问, 这样做的话, main()中的head依然是NULL, 得不到更新.  并且一直到`return 0`, head保持着NULL不变. 自然而然我们会想到把`node *head = NULL`替换为`node head(you_choose_integer)`. 这样做的话其实是和其它version逻辑稍有区别的, 即这里的头结点是在main()里直接给出, 而不是insert中插入一个头结点. 当然, 后面的逻辑依然是一致的. 下面给出完整代码后我们继续分析问题(依然没有析构仅供这里参考下)
```C
#include "stdafx.h"
class node
{
public:
	node *next;
	int elem;
	node(int e) : next{NULL}, elem{e} {}

};

void insert(node *head, node *inserted_node) //假设head是链表头, inserted_node是带插入尾部的已分配空间的结点
{
	node *current_node = head;
	if (current_node == NULL)
	{
		head = inserted_node;
	}
	else
	{
		while (current_node->next)
		{
			current_node = current_node->next;
		}
		current_node->next = inserted_node;
	}
}
int main()
{
	node head(0);
	node *inserted_node = new node(1);
 	insert(&head, inserted_node);
	inserted_node = new node(2);
	insert(&head, inserted_node);
	return 0;
}
```
好, 逻辑上终于修改完善了, 并且正确性大家上机测下很容易  我们以上面这份代码来看Q1.3本身. 首先我们将代码改为问题描述的样子
```C
#include "stdafx.h"
class node
{
public:
	node *next;
	int elem;
	node(int e) : next{NULL}, elem{e} {}

};

void insert(node *head, node *inserted_node) //假设head是链表头, inserted_node是带插入尾部的已分配空间的结点
{
	node *current_node = head;
	while (current_node->next)
	{
		current_node = current_node->next;
	}
	current_node->next = inserted_node;
}
int main()
{
	node head(0);
	node *inserted_node = new node(1);
 	insert(&head, inserted_node);
	inserted_node = new node(2);
	insert(&head, inserted_node); 
	return 0;
}
```
首先看参数传递, 的确传的是要修改的值的地址, 然后, 因为main()中初始时有一个非`NULL`的头结点, 所以 `while (current_node->next)`不会出错. 下面的问题我会进一步讲解这种写法可能出错的地方. 但是这个其实是特例, 毕竟很少有这种把head一开始就定义的情况...不过下面的一些例子还是会让大家看到有些情况`double pointer`能够写出比`singal pointer`更简练的代码的. 

###### Q1.4: 将Q1.3的answer里面`while (current_node->next)`改为`while (current_node)`会怎么样?
**answer**:  这样最后跳出时我们得到`current_node`是
NULL, 看上去是合理的, 给那个NULL赋值`inserted_node`就行了.  但是又有一个新问题, 给这个NULL赋值后我们能确保`prev->next = inserted_node`吗? 这里其实是很反直觉的, 答案是不能, 首先我们要知道了next实际上都指向实际的内存地址, 而当`current_node = NULL`(循环终止条件)时, 我们再给它进行复制, prev结点的next依然是NULL, 因为在这里`current_node`只是一个独立的指针, 与链表结构已经无关了. 如果我们一定要这么做, 必须维护着一个实际存在的prev结点. 

###### Q1.5: 将Q1.3里面的最后一份代码的第一步node head(0)改为node  *head = new node(0); 会怎么样?

**answer**: 
其实这个问题的依旧隐藏着一个前提条件, 就是这样修改依旧会实现在main()中定义好了一个头结点, 这样的话即使形参实参都是node*类型, 也就是说没有传地址的情况, 依旧是正确的. 因为它不需要改变head或者inserted_node, 只需要改变next就行了. 但是一旦写成`node *head = NULL`, 就会犯NULL->next这样的`read access violation`错误了. 

###### Q1.6：有没有办法将version A改为可以设置头结点在insert建立?

**answer**: 没有, 我们讨论两者情况, 

1. node *head = NULL;
2. node head{0};

法1的问题是想修改head(题目要求)但是传入的是`node *head`, 自然不能成功
法2的问题是这样必须在main()中建立了一个value是0的头结点了, 所以已经与题意不符了.

###### Q1.7: 验证version B的正确性?

**answer**: 我们首先再回顾下(别怪我啰嗦, 我认为这是最重要的), 传入的是要修改的`head`的地址(别被形参和实参名一样所迷惑, 他们代表的是两个不同的东西, 形参是实参的地址). 我们先看对头结点的处理: 将NULL的地址传入, 并且修改为了`inserted_node`. 然后看非head结点的建立, 此时就需要用到while了. 我们分析下这个while的含义, 循环条件是对current_node进行`dereference`, 判断是不是NULL. 循环执行语句是每次讲current_node这个地址的地址下移(next). 逻辑还是很清楚的. 

###### Q1.8: 本例中double pointers的优势在哪里?

**answer**: 可以任性的在main()中设置head, 比如`node *head = NULL`, 也可以`node head(0)`, 也可以`node *head = new node(0)`. 其他version就有着这样那样的限制.

### case2: remove

> 这里remove是指移除满足一定条件的结点, 不限重复. 在下面的例子中就是移除num是奇数的结点. 没有处理析构, 仅写了两份简单代码以便讲解. 如果有不了解lambda的可以看version A, C, 不了解函数指针的可以看version B, D. 不过并不是c style的就是纯粹的c了, 比如初始化列表什么的还是用的c++. 还有一点就是这里的single pointer不返回void了. 毕竟上面关于void的情况依旧讨论了很多了, 没必要继续了. 虽燃insert的例子讲了那么多关于值与地址, 但是仅仅为了更好的使读者理解double pointers, 实际上在不使用void的情况下我们完全可以返回一个head来维护head, 即使传入的是其value, 只要head = remove_if(...)来接受返回值就行了. 所以再接下来的讲解中我们不过多关注value和address, 而是关注于代码逻辑.

#### remove: version A(right && c style(not completely) && double pointers)

```C++
#include "stdafx.h"
#include <iostream>
typedef struct node
{
	struct node * next;
	int num;
	node(int x) : next{ NULL }, num{ x } {}
} node;

typedef bool(*remove_fn)(node const * v);

void remove_if(node ** head, remove_fn rm)
{
	for (node** curr = head; *curr; )
	{
		node * entry = *curr;
		if (rm(entry))
		{
			*curr = entry->next;
			delete entry;
		}
		else
			curr = &entry->next;
	}
}
bool find_odd(node const * x)
{
	if (x->num % 2 == 0)
	{
		return false;
	}
	return true;
}
int main()
{
	node *head = new node(1);
	head->next = new node(2);
	head->next->next = new node(3);
	head->next->next->next = new node(4);
	remove_fn rm = find_odd;
	remove_if(&head, rm);
	return 0;
}
```

#### remove: version B(right && c++ style && double pointers)

```C++
#include "stdafx.h"
#include <iostream>
struct node
{
	struct node * next;
	int num;
	node(int x) : next{ NULL }, num{ x } {}
};

template<typename remove_fn>
void remove_if(node ** head, remove_fn rm)
{
	for (node** curr = head; *curr != NULL; )
	{
		node * entry = *curr;
		if (rm(entry))
		{
			*curr = entry->next;
			delete entry;
		}
		else
			curr = &entry->next;
	}
}

int main()
{
	node *head = new node(1);
	head->next = new node(2);
	head->next->next = new node(3);
	head->next->next->next = new node(4);
	remove_if(&head, [&](node* temp) {return temp->num % 2 != 0; });
	return 0;
}
```


#### remove: version C(right && c style && single pointer)

```C++
#include "stdafx.h"
#include <iostream>
typedef struct node
{
	struct node * next;
	int num;
	node(int x) : next{ NULL }, num{ x } {}
} node;

typedef bool(*remove_fn)(node const * v);

node * remove_if(node * head, remove_fn rm)
{
	for (node * prev = NULL, *curr = head; curr != NULL; )
	{
		node * const next = curr->next;
		if (rm(curr))
		{
			if (prev)
				prev->next = next;
			else
				head = next;
			delete curr;
		}
		else
			prev = curr;
		curr = next;
	}
	return head;
}
bool find_odd(node const * x)
{
	if (x->num % 2 == 0)
	{
		return false;
	}
	return true;
}
int main()
{
	node *head = new node(1);
	head->next = new node(2);
	head->next->next = new node(3);
	head->next->next->next = new node(4);
	remove_fn rm = find_odd;
	head = remove_if(head, rm);
	return 0;
}
```

#### remove: version D(right && c++ style && single pointer)

```C++
#include "stdafx.h"
#include <iostream>
struct node
{
	struct node * next;
	int num;
	node(int x) : next{ NULL }, num{ x } {}
};

template<typename remove_fn>
node * remove_if(node * head, remove_fn rm)
{
	for (node * prev = NULL, *curr = head; curr != NULL; )
	{
		node * const next = curr->next;
		if (rm(curr))
		{
			if (prev)
				prev->next = next;
			else
				head = next;
			delete curr;
		}
		else
			prev = curr;
		curr = next;
	}
	return head;
}

int main()
{
	node *head = new node(1);
	head->next = new node(2);
	head->next->next = new node(3);
	head->next->next->next = new node(4);
	head = remove_if(head, [&](node *temp) {return temp->num % 2 != 0; });
	return 0;
}
```

#### remove: version E(wrong && c++ style && single pointer)

```C++
#include "stdafx.h"
#include <iostream>
struct node
{
	struct node * next;
	int num;
	node(int x) : next{ NULL }, num{ x } {}
};

template<typename remove_fn>
node * remove_if(node * head, remove_fn rm)
{
	for (node* curr = head; curr; )
	{
		node * entry = curr;
		if (rm(entry))
		{
			curr = entry->next;
			delete entry;
		}
		else
			curr = entry->next;
	}
	return head;
}

int main()
{
	node *head = new node(1);
	head->next = new node(2);
	head->next->next = new node(3);
	head->next->next->next = new node(4);
	head = remove_if(head, [&](node *temp) {return temp->num % 2 != 0; });
	return 0;
}
```

###### Q2.1: version E有哪些错误?

**answer**: 
1. 没有维护好head
2. 没有维护好prev和next的羁绊(

###### Q2.2: version E该如何改正?

**answer**: refer to version C/D: 如果prev是NULL, 那么就该维护head了(head = next). 使用`prev->next = next`, `prev = curr`来维护羁绊. next还有一个作用就是在delete curr前保存着下一个结点, 如果贪心省略掉这个next 你还是需要一个temp来保存的, 反而变丑了. 这也是写代码时容易忽略的一点. 

###### Q2.2: 分析这里两个right version的逻辑差别?

**answer**: single pointer version都需要维护prev和next结点, 而double pointers仅需要维护一个entry结点(除curr, 并且本质上entry起着prev的作用). single pointer需要多一个判断head的操作

###### Q2.3: double pointers version不需要判断head(prev == NULL)?

**answer**: 这个问题依然要回到之前反复强调的value-address了, 传入的head在这里是可以被修改的, 所以能在if里面直接维护. 

###### Q2.4: double pointers version是如何处理非head结点的?

**answer**: 这个问题也需要回到value-address来回答. 其实无论是head还是non-head, 处理方式都是一样的----改变这个结点的地址为->next. 这样是不会破坏与prev的羁绊的. 即使是NULL都无妨(ctrl/command f `任性`, 参考这里). 

### case3: Insertion Sort for Singly Linked List

> 单链表的排序里面我觉得插排是最容易实现的, 所以在这里我使用它来讲解 `single pointer VS double pointers`. 

###### sort: version A(single pointer)

```
ListNode* linkedListSort(ListNode *head) {
    if (head == NULL || head->next == NULL)
    {
        return head;
    }
    struct ListNode *new_head = new struct ListNode(head->val); // 新链表的头结点
    struct ListNode *current_node = head->next; // 遍历旧链表
    struct ListNode *tail_node = new_head; // 新链表的最后一个结点
    while (current_node != NULL)
    {
        struct ListNode *temp_node = new_head; // 在新链表上进行查找
        if (current_node->val < tail_node->val) // 需要进行中间插入的情况, 为了维护tail_node, 这里不能是小于等于
        {
            if (new_head->val >= current_node->val) //比头结点还小的情况
            {
                struct ListNode *new_new_head = new struct ListNode(current_node->val); // 新的头结点
                new_new_head->next = new_head;
                new_head = new_new_head;
            }
            else if (new_head->next->val >= current_node->val) // 比头结点的下一个结点小但比头结点大
            {
                struct ListNode *after_new_head = new struct ListNode(current_node->val);
                after_new_head->next = new_head->next;
                new_head->next = after_new_head;
            }
            else // 比头结点和第二个结点都大的情况
            {
                while (temp_node->next->val <= current_node->val)
                {
                    temp_node = temp_node->next;
                }
                struct ListNode *middle_insert_node = new struct ListNode(current_node->val);
                middle_insert_node->next = temp_node->next;
                temp_node->next = middle_insert_node;
            }
        }
        else //直接放尾部的情况
        {
            struct ListNode *new_tail_node = new struct ListNode(current_node->val);
            tail_node->next = new_tail_node;
            tail_node = new_tail_node;
        }
        current_node = current_node->next;
    }
    return new_head;
}
```


###### Q3.1: 将上面那份代码改写为double pointers?

> 我的这个链表插排的写法看起来的确很长, 但是胜在逻辑比较清楚, 可读性很强, 边界条件也都处理的比较完善. 所以这也是我少有的用没有智能提示的editor里写一遍就能通过编译的代码, 如果读者抱着学习的态度来看这篇文章但是上面那些例子都没有自己动手写, 那么希望能亲自动手写下这个问题, 为了方便大家尽快理解思路以免浪费时间, 我在关键位置都写了详细的注释.

**answer**: 

```C++
ListNode* linkedListSort(ListNode *head) {
  // 很清楚的可以看到是O(n)复杂度
  if(head == NULL || head->next == NULL)
  {
      return head;
  }
  struct ListNode * new_head = NULL;
  while (head != NULL)
  {
      struct ListNode *   copy_of_head  = head;
      struct ListNode ** pointer_to_new_head = &new_head;
      head = head->next;
      while (!(*pointer_to_new_head == NULL || copy_of_head->val < (*pointer_to_new_head)->val ))
      {
          pointer_to_new_head = &(*pointer_to_new_head)->next;
      }
      copy_of_head->next = *pointer_to_new_head;
      *pointer_to_new_head = copy_of_head;
  }
  return new_head;
}
```



### In Summary
可能很多人认为指针是一个糟糕的特性, 也有很多人认为掌握指针的各种技巧在C++11 里各种封装好的`smart pointer`前只是班门弄斧. 但是我想说, 世界上不存在神这种玩意, 所以永远不要把一个人当做神, 即使他/她再强, 说的话也不应该全盘被信任, 所以你必须做出自己的判断, 这判断不是为了判断而判断, 而是必须尽快的切换到学习知识而不是犯傻逼脑残的所谓选择困难症的状态, 否则只会在一票大神面前永远迷失方向, 今天某大v说学这个是没有意义的, 明天另一个大v说学那个是有意义的. 如果你信任所有人, 就会陷入什么都想学, 却什么都没心思学的困境, 解决问题的唯一途径就是凭着自己的兴趣走下去, 何必管边上的人所说的话? 

等这些天手边的事忙完后我在再写一些 double pointers的其它应用, 比如链表的其他操作, 树的各种......
