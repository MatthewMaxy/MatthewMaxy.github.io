---
layout: post
title:  "数据结构与算法复习（一）"
date:   2024-05-12 19:49:41 +0800
tags:   DS Basic
color: rgb(70,170,224)
cover: '../assets/Blogs/Norm/title.png'
subtitle: '数据结构与算法复习——数组、链表、栈、队列'
---
<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']],
            splayMath: [ ['$$','$$'], ["\\[","\\]"] ]
            }
        });
    </script>
</head>

# Array

## 1 基本信息：

1. 不一定使用连续内存实现
2. 索引可以以任何方式进行编码
3. 基本操作
   1. 找到该列表的长度n
   2. 从左到右（或从右到左）阅读该列表
   3. 检索第i个元素
   4. 将一个新值存储到第i个位置
   5. 在位置i处插入一个新的元素
   6. 删除位置i处的元素，

## 2 多项式：

### 2.1 表示方法：

**表示一：**

```cpp
private: 
    int degree; // degree <= MaxDegree
    float coef[MaxDegree+1];
```

当`a.degree` << `MaxDegree` 时，表示1在使用计算机内存方面是相当浪费的。要解决此问题，将定义可变大小的数据成员。  

**表示二：**

```cpp
private: 
    int degree;
    float *coef;
Polynomial::Polynomial(int d)
{ 
    int degree = d;
    coef = new float[degree+1];
}
```

表示2仍然不可取。例如，x^1000^ + 1使999个`coef`条目为零。所以，我们只存储非零项：  

**表示三：**

```cpp
class Polynomial; // forward declaration
struct Term
{
	float coef;
	int exp;
};
class Polynomial { 
public:
    //默认构造函数
    Polynomial()
	{
		termArray = new Term[1];
		capacity = 1;
		terms = 0;
		cout << "Create!";
	}
    //指定非零项个数
    Polynomial( int terms)
	{
		this->capacity = terms;
		this->terms = terms;
		termArray = new Term[capacity];
		for (int i = 0; i < terms; i++)
			cin >> termArray[i].coef >> termArray[i].exp;
		cout << "Create!"<<endl;
	}
private:
    Term *termArray;
    int capacity; // size of termArray
    int terms; // number of nonzero terms
}
```

对于系数为0项数较多时具有优越性。_但较少时，花费空间为表示二的两倍_

### 2.2 加法：

默认两个多项式按降幂排列

#### 2.2.1 加法原理：

```cpp
Polynomial Add(Polynomial b)
{
    Polynomial c;
    int aPos = 0, bPos = 0;
    while (aPos < terms && bPos < b.terms) {
        // 如果两个指数相等直接将系数相加
        if (termArray[aPos].exp == b.termArray[bPos].exp) {
            float t = termArray[aPos].coef + b.termArray[bPos].coef;
            if (t)
                c.newTerm(t, termArray[aPos].exp);
            aPos++;
            bPos++;
        }
        else if (termArray[aPos].exp < b.termArray[bPos].exp)
        {
            c.newTerm(b.termArray[bPos].coef, b.termArray[bPos].exp);
            bPos++;
        }
        else
        {
            c.newTerm(termArray[aPos].coef, termArray[aPos].exp);
            aPos++;
        }
    }
    // 将剩下的a或b全部放入c中
    for(;aPos<terms;aPos++)
        c.newTerm(termArray[aPos].coef, termArray[aPos].exp);
    for(;bPos<b.terms;bPos++)
        c.newTerm(b.termArray[bPos].coef, b.termArray[bPos].exp);
    return c;
}
```

其中`newTerm`函数用于创建新的Term并放入他的`termArray`中

```cpp
void newTerm(float coef, int exp)
{
    // 如果数组空间已满，将扩大空间2倍
    if (terms == capacity)
    {
        capacity *= 2;
        Term* tem = new Term[capacity];
        copy(termArray, termArray + capacity, tem);
        delete[]termArray;
        termArray = tem;
    }
    termArray[terms].coef = coef;
    termArray[terms].exp = exp;
    terms++;
}
```

#### 2.2.2 性能分析：

1. 设m，n分别是a和b中的非零项的个数。
2. 在同时循环的每次迭代中，`aPos`或`bPos`或两者都增加了1，因此这个循环的迭代次数<=（m+n-1）。
3. 如果忽略double array时间，每次迭代时间为`O(1)`
4. 结束遍历剩余a-`O(m)`，剩余b-`O(n)`
5. 总的时间复杂度为`O(m+n)`

> **分析数组加倍**  
>
> 1. 加倍的时间与新数组的大小相比是线性的
> 2. 最初，c容量为1
> 3. 假设当Add终止时，c.容量为2^k^
> 4. 他花在所有数组加倍上的总时间是`O(2^k^)`
> 5. 由于`c.terms`> 2^k-1^ 和 m + n >= `c.terms`，数组加倍的总时间是` O(c.terms)=O(m+n)`

综上，即便考虑数组加倍时间 **==Add的时间复杂度依然是O(m+n)==**

## 3 稀疏矩阵:

### 3.1 简介：

1. m*m的矩阵称为方阵
2. 一个有许多零项的矩阵被称为稀疏矩阵

### 3.2 表示方法：

```cpp
struct MatrixTerm
{
	int row, col, value;
};
class SparseMatrix {
private:
	MatrixTerm* smArray;
	int rows, cols, terms, capacity;
public:
	SparseMatrix()
	{
		smArray = new MatrixTerm[1];
		capacity = 1;
		terms = 0;
		cols = rows = 0;
	}
	SparseMatrix(int c, int r, int t)
	{
		smArray = new MatrixTerm[t];
		terms = t;
		cols = c; rows = r;
		capacity = t;
		for (int i = 0; i < t; i++)
			cin >> smArray[i].row >> smArray[i].col >> smArray[i].value;
		cout << "create!" << endl;
	}
}
```

### 3.3 矩阵的转置:

#### 3.3.1 一般矩阵Transpose

```cpp
for (int j = 0; j < cols; j++)
   for (int i = 0;i < rows;i++) 
      B[j][i] = A[i][j];
```

​	==时间复杂度O(cols$\times$rows)==

#### 3.3.2 稀疏矩阵的转置

```cpp
 SparseMatrix Transpose()
 {
     SparseMatrix b(rows, cols, terms,true);
     if (terms > 0)//非空
     {
         int currentB = 0;
         for(int i=0;i<cols;i++)//依次遍历列为0，1……，转置后他们对应行0，1……顺序
             for(int j=0;j<terms;j++)
                 if (smArray[j].col == i)
                 {
                     b.smArray[currentB].col = smArray[j].row;
                     b.smArray[currentB].row = i;
                     b.smArray[currentB++].value = smArray[j].value;
                 }
     }
     return b;
 }
```

> **复杂度分析：**
>
> 1. 最外层循环：`cols`次
> 2. 内层if判断：`terms`次
> 3. ==总时间复杂度：O(cols$\times$terms)==
> 4. ==额外空间：O(1)==

改进：

1. 获取每列中的元素的数量 = 转置后B中每行元素的个数
2. 获取其每行在B中的起始点
3. 将当前矩阵的元素逐个移动到B中的正确位置。

#### 3.3.3 快速转置

```cpp
 SparseMatrix FastTranspose()
 {
     SparseMatrix b(rows, cols, terms, true);
     if (terms > 0)
     {
         int* rowSize = new int[cols];//转置后各行元素个数
         int* rowStart = new int[cols];//转置后各行开始储存的位置的下标
         fill(rowSize, rowSize + cols, 0);// 4
         for (int i = 0; i < terms; i++)// 1
             rowSize[smArray[i].col]++;
         rowStart[0] = 0;
         for (int i = 1; i < cols; i++)//  2
             rowStart[i] = rowStart[i - 1] + rowSize[i - 1];
         for (int i = 0; i < terms; i++)// 3
         {
             int j = rowStart[smArray[i].col];
             b.smArray[j].col = smArray[i].row;
             b.smArray[j].row = smArray[i].col;
             b.smArray[j].value = smArray[i].value;
             rowStart[smArray[i].col]++;
         }
         delete[]rowSize;
         delete[]rowStart;
     }
     return b;
 }
```

> **复杂度分析：**
>
> 1. 循环1处：O(terms)
> 2. 循环2处：O(cols)
> 3. 循环3处：O(terms)
> 4. 4处: O(cols)   其余行：O(1)
> 5. ==总时间复杂度：O(cols+terms)==
> 6. ==额外空间：O(2*cols)==

## 4 字符串:

### 4.1 简单查找

```cpp
int Find(string str,string pat)
{
	if (pat.length() == 0)
		return -1;
	for (int i = 0; i <= str.length() - pat.length(); i++)
	{
		if (str[i] == pat[0])
		{
			int j = 0;
			for (; j < pat.length() && str[i + j] == pat[j]; j++);
			if (j == pat.length())
				return i;
		}
	}
	return -1;
}
```

==时间复杂度O(LengthP$\times$LengthS)==

### 4.2 KMP算法

#### 4.2.1 失败函数

```cpp
void F(int next[],string str)
{
	int j = 0, k = -1;
	next[0] = -1;
	while (j < str.length() )
	{
		if (k == -1 || str[j] == str[k])// 1
		{
			next[j] = k;
			j++;
			k++;
		}
		else
			k = next[k];
	}
}
```

> **复杂度分析：**
>
> 1. 循环主体最多执行`LengthP-1`次
> 1. 时间复杂度为O(LengthP)

#### 4.2.2 KMP主体

```cpp
int FastFind(string str, string pat)
{
	int *n = new int[pat.length()];
	F(n, pat);
	int PosP = 0, PosS = 0;
	while (PosP < pat.length() && PosS < str.length())
	{
		if (pat[PosP] == str[PosS])
		{
			PosP++; PosS++;// 1
		}
		else
			if (PosP == 0)
				PosS++; // 2
			else
				PosP = n[PosP - 1] + 1; // 3
	}
	if (PosP < pat.length()||pat.length()==0)
		return -1;
	else
		return PosS - pat.length();
}
```

> **复杂度分析：**
>
> 1. 1,2两处`PosS`只增未减，最多执行`LengthS`次
> 2. 3处最多执行`LengthS`次
> 3. 总时间复杂度为O(LengthS)

#### 4.2.3 总时间复杂度

​	综合两者考虑，当失败函数事先不知道时，可以首先计算失败函数，然后使用FastFind进行模式匹配，总时间复杂度为==O(LengthP + LengthS)==

# 链表

## 1 单链接的列表和链

使用数组和顺序映射来表示简单的数据结构具有以下特性：

> 1. 数据对象的连续节点以固定的距离间隔存储
> 2. 这使得访问中的任意节点时间复杂度：O (1)

顺序映射的缺点：

> 它使得插入和删除任意元素时，时间耗费昂贵。  

解决方案——具有链接功能的表示法：

> + 与每个节点关联的是指向下一个项目的点（链接）。
> + 上述结构称为单链表或链，其中每个节点都有一个指针字段  
>   + 列表元素以任意的顺序存储在内存中  
>   + 显式信息（称为链接）用于从一个元素转到下一个元素

## 2 链表表示：

实现：

```c++
class Chain;
class ChainNode {
	friend class Chain;
private:
	int data;
	ChainNode* link;
public:
	ChainNode(int element = 0, ChainNode* next = 0)
	{
		data = element;
		link = next;
	}
};
class Chain {
private:
	ChainNode* first;
public:
	Chain(){}
};
```

在某个节点后插入50：

```c++
void Chain::Insert50 (ChainNode *x) 
{
    if (first)
        // insert after x
        x.link = new ChainNode(50, x.link); 
    else
        // insert into empty chain
        first = new ChainNode(50);
}
```

## 3 模板类链表

### 3.1 使用模板实现链链

```c++
template <class T> class Chain; // forward declaration
template <class T>
class ChainNode {
    friend class Chain<T>;
public:
    ChainNode(T element, ChainNode *next = 0)
    { data = element; link = next;}
private:
    T data;
    ChainNode *link;
};
template <class T>
class Chain {
public: 
    Chain() { first=0;}; // constructor initializing first to 0
    // Chain manipulation operations
    // ...
private:
    ChainNode<T> *first; 
}
```

### 3.2 链表迭代器

> 容器类是表示包含或存储大量数据对象的数据结构的类。  
> 迭代器是一个用于逐个访问容器类的元素的对象。  

考虑可能在容器类C上执行的以下操作，其所有元素都是整数：

+ 输出C中的所有整数
+ 得到C中所有整数的和、最大值、最小值、平均值、中值
+ 从C中获取整数x，使f (x)最大。

_通常，一个迭代器被实现为容器类的一个嵌套类。_

#### 3.2.1 链表的正向迭代器：

```c++
class ChainIterator {
private:
	ChainNode* current;
public:
	ChainIterator(ChainNode* startNode = 0)
	    {current = startNode;}
	int& operator *() const { return current->data; }
	int* operator->() const { return &current->data; }
	ChainIterator& operator++()//++a
	{
		current = current->link;
		return *this;
	}
	ChainIterator& operator++(int)//a++
	{
		ChainIterator old = *this;
		current = current->link;
		return old;
	}
	bool operator != (const ChainIterator right) const
	    {return current != right.current;}
	bool operator == (const ChainIterator right) const
	    {return current == right.current;	}
	void printIt()
	    {cout << current->data << " ";}
};
```

除此以外，在Chain中添加函数：

```c++
	ChainIterator begin() 
        { return ChainIterator(first); }
	ChainIterator end() 
        { return ChainIterator(0); }
```

输出一个链表所有元素的两种实现方法：

```c++
// 直接输出所有元素
void Print()
{
    ChainNode* tem = first;
    while(tem!=0){
        cout << tem->data << "  ";
        tem = tem->link;
    }
    cout << endl;
}
// 利用迭代器输出
void Print2()
{
    ChainIterator ai = this->begin();
    while (ai != this->end())
    {
        ai.printIt();
        ai++;
    }
    cout << endl;
}
```

### 3.3 链表操作：

1. InsertBack:

```c++
void InsertBack(int element)
{
    if (first){
        last->link = new ChainNode(element);
        last = last->link;
    }
    else
        first = last = new ChainNode(element);
}
```

==时间复杂度：O(1)==

2. Concatenate:

```c++
void Concatenate(Chain& b)
{
    if (first){
        last->link=b.first;
        last = b.last;
    }
    else{
        first = b.first;
        last = b.last;
    }
    b.first = b.last = 0;
}
```

==时间复杂度：O(1)==

3. Reverse:

```c++
void Reverse()
{
    ChainNode* cur = first, * pre = 0;
    while (cur){
        ChainNode* r = pre;
        pre = cur;
        cur = cur->link;
        pre->link = r;
    }
    first = pre;
}
```

假设链表有m$\geq$1个节点，==时间复杂度：O(m)==

## 4 循环链表

通过修改一个链，使最后一个节点的链接字段指向第一个节点，就可以得到一个循环列表。

1. 在前端插入元素

```c++
template <class T>
void CircularList<T>::InsertFront(const T& e) 
{ // insert the element e at the “front” of the circular list *this,
    // where last points to the last node in the list.
    ChainNode<T>* newNode = new ChainNode<T>(e);
    if (last) { // nonempty list
        newNode->link =last->link; 
        last->link = newNode; 
    } 
    else { 
        last = newNode; 
        newNode->link = newNode;
    }
}
```

==时间复杂度:O(1)==

2. 在最后插入元素

```c++
    //在InsertFront代码后添加：
    last = newNode;
```

==时间复杂度:O(1)==

## 5 可用空间列表

+ 链和循环列表的析构函数的时间与链或列表的长度是线性的  
+ 如果我们保持我们自己的自由节点链，它可能会被简化为O(1)。
+ 可用空间列表由av指向。
+ av是CircularList\<T\>的ChaiNode\<T\>*类型的静态成员，最初是av = 0
+ 只有当av列表为空时，我们才需要使用new

### 5.1 基于av的新建节点（不使用new）

```c++
template <class T>
ChainNode<T>* CircularList<T>::GetNode( ) 
{ //Provide a node for use.
    ChainNode<T>* x;
    if (av) {
        x = av; 
        av = av->link;
    }
    else 
        x = new ChainNode<T>; 
    return x;
}
```

### 5.2 基于av的删除节点（不使用delete）

```c++
template <class T>
void CircularList<T>::RetNode(ChainNode<T>*& x) 
{ // Free the node pointed to by x.
    x->link = av;
    av = x;
    x = 0;
}
```

### 5.3 删除一个循环链表

```c++
template <class T>
void CircularList<T>::~CircularList() 
{ // Delete the circular list.
    if (last) {
        ChainNode <T> * first = last->link;
        last->link = av; 
        av = first; 
        last = 0;
    }
}
```

==时间复杂度:O(1)==

## 6 链表栈和队列

### 6.1 链表栈

```c++
class LStack {
private:
	ChainNode* top;
public:
	LStack(){
		top = 0;
	}
	void Pop() {
		if (top != 0){
			ChainNode* tem = top;
			top = top->link;
			delete tem;
		}
		else
			cout << "error";
	}
	void Push(int element){
		top = new ChainNode(element,top);
	}
};
```

### 6.2 链表队列

```c++
class LQueue {
private:
	ChainNode* front, * rear;
public:
	LQueue(){
		front = rear = 0;
	}
	void Push(int element)	{
		if (front == 0)
			front = rear = new ChainNode(element);
    else{
			rear->link = new ChainNode(element);
			rear = rear->link;
		}
	}
	void Pop()
	{
		if (front == 0)
			cout << "error";
		else{
			ChainNode* tem = front;
			front = front->link;
			delete tem;
		}
	}
};
```

## 7 链表多项式

### 7.1 多项式表示

```c++
struct Term
{ // All members of Term are public by default.
    int coef;
    int exp;
    Term Set(int c, int e){ 
        coef = c; exp = e; 
        return *this;
    };
};
class Polynomial {
public:
    // public functions defined here
private:
    Chain<Term> poly;
};
```

### 7.2 多项式加法

```c++
 Polynomial Polynomial::operaor+ (const Polynomial& b) const
{ // *this (a) and b are added and the sum returned
    Term temp;
    Chain<Term>::ChainIterator ai = poly.begin(),
                                bi = b.poly.begin();
    Polynomial c;
    while (ai != poly.end() && bi != b.poly.end()) { //not null
        if (ai->exp == bi->exp) {
            int sum = ai->coef + bi->coef; 
            if (sum) 
                c.poly.InsertBack(temp.Set(sum, bi->exp));
            ai++; bi++; // to next term
        } 
        else if (ai->exp < bi->exp){
            c.poly.InsertBack(temp.Set(bi->coef, bi->exp));
            bi++; // next term of b
        } 
        else {
            c.poly.InsertBack(temp.Set(ai->coef, ai->exp));
            ai++; // next term of a
        }
    } 
    while (ai != poly.end()) { // copy rest of a
        c.poly.InsertBack(temp.Set(ai->coef, ai->exp));
        ai++;
    }
    while (bi != b.poly.end()) { // copy rest of b
        c.poly.InsertBack(temp.Set(bi->coef, bi->exp));
        bi++;
    }
    return c;
}
```

设a有m项,b有n项,==时间复杂度：O(m+n)==

### 7.3 循环链表表示多项式

多项式加法： 
头部节点的exp被设置为-1，以将a或b的其余部分推向结果。

```c++
Polynomial Polynomial::operaor+(const Polynomial& b) const
{ // *this (a) and b are added and the sum returned
    Term temp;
    CircularListWithHead<Term>::Iterator ai = poly.begin(),
                                            bi = b.poly.begin(); 
    Polynomial c; //assume constructor sets headexp = -1
    while (1) { 
        if (ai->exp == bi->exp) {
            if (ai->exp == -1) 
                return c;
            int sum = ai->coef + bi->coef; 
            if (sum) 
                c.poly.InsertBack(temp.Set(sum, ai->exp);
            ai++; bi++; // to next term
        }
        else if (ai->exp < bi->exp) {
            c.poly.InsertBack(temp.Set(bi->coef, bi->exp));
            bi++; // next term of b
        } 
        else {
            c.poly.InsertBack(temp.Set(ai->coef, ai->exp));
            ai++; // next term of a 
        }
    } 
}
```

## 8 稀疏矩阵

**稀疏矩阵表示**

1. 每个头节点:都有一个bool head和三个附加字段：down,right和next。
   + 头节点的总数是max{cols,rows}。
2. 每个元素节点:都有一个bool head和五个附加字段：row、col、value、down和right。
3. H是头节点列表的头节点，使用元素节点的结构。

![xsjz](/assets/Blogs/Datastructure/Link/1.jpg)

## 9 双向链表

### 9.1 概念

1. 双向链表中的一个节点至少有3个字段：data, left和right，这使得它很容易向两个方向移动。
2. 一个双链表可能是循环的。

![double](/assets/Blogs/Datastructure/Link/2.jpg)

### 9.2 代码实现

数据结构实现：

```c++
class DblList;
class DblListNode {
    friend class DblList;
private:
    int data; 
    DblListNode *left, *right; 
};
class DblList {
public:
    // List manipulation operations
    // ...
private:
    DblListNode *first; // points to head node
};
```

删除实现：

```c++
void DblList::Delete(DblListNode *x)
{
    if(x == first) 
        throw "Deletion of head node not permitted";
    else{
        x->left->right = x->right; 
        x->right->left = x->left;
        delete x;
    }
}
```

插入实现：

```c++
void DblList::Insert(DblListNode *p, DblListNode *x)
{ // insert node p to the right of node x
    p->left = x; 
    p->right = x->right; 
    x->right->left = p; 
    x->right = p; 
}
```

## 10 广义表

广义列表是一个线性列表 $(a_0,...,a_{n-1})$，其中元素 $a_i(0<=i<=n-1)$ 是一个原子或一个列表。

> + A = (): 空表
> + B = (a, (b, c)): a list of length two; 第一个元素：a  第二个元素： 表 (b, c).
> + C = (B, B, ()): a list of length three 第一、二个元素：表 B 第三个元素：空表
> + D = (a, D): 递归表：D=(a,(a,(a, ...)))

```c++
enum Triple {var, ptr, no};
class PolyNode
{
    PolyNode *next;
    int exp;
    Triple trio;
    union {
        char vble;
        PolyNode *down;
        int coef;
    };
};
```

![广义表](/assets/Blogs/Datastructure/Link/3.jpg)

# 栈和队列

## 1. 堆栈抽象数据类型

### 1.1 简介：（LIFO）

1. 线性列表
2. 其中一端叫做顶部。另一端被称为底部。
3. 仅从顶部进行添加和删除

### 1.2 实现：(实际实现应用模板T)

```cpp
class Stack{
private:
	int* stack;
	int top;
	int capacity;
public:
	Stack(int capcity = 10)
	{
		if (capcity < 1)
			cout << "capacity should >0";
		stack = new int[capcity];
		capacity = capcity;
		top = -1;
	}
	bool isEmpty()
	{
		return(top == -1);
	}
	int Top(){
		return stack[top];
	}
	void Push(int a)
	{
		if (top == capacity - 1)
		{
			capacity *= 2;
			int* tem = new int[capacity];
			for (int i = 0; i <= top; i++)
				tem[i] = stack[i];
			delete[]stack;
			stack = tem;
		}
		stack[++top] = a;	
	}
	int Pop()
	{
		if (isEmpty())
			cout << "error!";
		int tem=stack[top];
		stack[top] = NULL;
		top--;
		return tem;
	}
};
```

## 2. 队列抽象数据类型

### 2.1 简介：（FIFO）

1. 线性列表
2. 其中一端叫做前端，另一端被称为后面。
3. 添加的操作只在后面完成，只从前面移出。

### 2.2 实现循环队列：(实际实现应用模板T)

```cpp
class Queue {
private:
	int capacity, front, rear;
    // front 为空，后一位开始有数据
    // rear 为最后一位数据位置
	int* queue;
public:
	Queue(int capacity = 10)
	{
		if (capacity < 1)
			cout << "error";
		this->capacity = capacity;
		queue = new int[capacity];
		fill(queue, queue + capacity, 0);
		front = rear = 0;
	}
	bool isEmpty()
	{
		return (front == rear);
	}
	void Push(int a)
	{
		if ((rear + 1) % capacity == front)
		{
			int* tem = new int[capacity * 2];
			fill(tem, tem + 2 * capacity, 0);
			int start = (front + 1) % capacity;
            // 考虑数据在数组内的情况，即是否有循环情况（'1', '2', '%', '%', '7', '3', '5')
            // 不存在循环情况：
            // 空的为空间(若为0~7）中最后一个（7%8=7）或第一个（0%8=0），对应start<2
			if (start < 2)
			{
				for (int i = start,j=0;i <= rear;j++,i++)
					tem[j] = queue[i];
			}
            //若存在回环：
            // 1. 创建一个容量为两倍的新阵列新队列
            // 2. 将第二个线段复制到新队列中从0开始的位置。
            // 3. 将第一个段复制到新的队列中的位置。
			else
			{	
				int j = 0;
				for (int i = start;i < capacity;i++,j++)
					tem[j] = queue[i];
				for (int i = 0;i <= rear;i++,j++)
					tem[j] = queue[i];
			}
			front = 2 * capacity - 1;rear = capacity - 2;capacity *= 2;
			delete[]queue;
			queue = tem;
		}
		rear = (rear + 1) % capacity;
		queue[rear] = a;
	}
	int Pop()
	{
		if (isEmpty())
			cout << "error!";
		front = (front + 1) % capacity;
		int tem = queue[front];
		queue[front] = 0;
		return tem;
	}
};
```

## 3. 具体问题

### 3.1 迷宫问题（栈实现）

#### 3.1.1 问题

​找到一条从迷宫的入口到出口的路径

#### 3.1.2 表示

> 1. `maze[i][j]`，1<=i<=m，1<=j<=p
> 2. 1-可行 0-堵塞
> 3. 起点`maze[0][0]`|||终点`maze[m][p]`
> 4. 当前位置：`[i][j]`
> 5. 1的边界，所以一个`maze[m+2][p+2]`
> 6. 8个可能的移动方向：N、NE、E、SE、S、SW、W和NW。

#### 3.1.3 思路

1. 给定当前的位置`[i][j]`和8个方向，我们选择一个方向d，得到新的位置`[g][h]`。
2. 如果`[g][h]`是目标，那就是成功。
3. 如果`[g][h]`是一个合法的位置，则将`[i][j]`和d+1保存在一个堆栈中，以防我们采取一个错误的路径，需要尝试另一个方向，然后`[g][h]`成为新的当前位置。
4. 重复直到成功或每一种可能性都被尝试过。
5. 为了防止我们沿着同一路径前进两次：使用另一个数组，标记`[m+2][p+2]`空间均为0。一旦访问该位置，标记`[i][j]`将设置为1

#### 3.1.4 实现

1. 初始化8个方向及迷宫

```cpp
struct offsets
{
    int a, b;
};
enum directions {N, NE, E, SE, S, SW, W, NW}; 
offsets move[8];
```

![direct](/assets/Blogs/Datastructure/stack/1.jpg){:height=50%, :width=50%}

1. 初始位置为坐上，初始方向向东获得路径：

```cpp
void Path(const int m, const int p)
{
    // start at (1,1)
    mark[1][1]=1;
    Stack<Items> stack(m*p);
    Items temp(1, 1, E);
    stack.Push(temp);
    while (!stack.IsEmpty())
    {
        temp = stack.Top();
        stack.Pop();
        int i = temp.x; int j = temp.y; int d = temp.dir;
        while ( d < 8)
        {
            int g = i + move[d].a; int h = j + move[d].b;
            if ((g == m) && (h == p)) { // reached exit
                // output path
                cout << stack;
                cout << i << " " << j << " " << d << endl; 
                cout << m<< " "<< p << endl; // last two points
                return;
            }
            if ((!maze[g][h]) && (!mark[g][h])) { //new position 
                mark[g][h]=1;
                temp.x = i; temp.y = j; temp.dir = d+1;
                stack.Push(temp);
                i = g ; j = h ; d = N; // move to (g, h)
            }
            else d++; // try next direction
        }
    }
    cout << "No path in maze."<< endl;
}
```

3. 需要储存当前坐标和方向的栈

```cpp
struct Items {
    int x, y, dir;
};
// 此外，为了避免在堆栈推送过程中使阵列容量加倍，我们可以将堆栈的大小设置为m*p
```

4. 关于<<的重载

```cpp
template <class T>
ostream& operator << (ostream& os, Stack<T>& s)
{
    os << "top = "<<s.top<< endl;
    for (int i = 0; i <= s.top; i++);
        os << i << ":"<< s.stack[i] << endl;
    return os;
}
```

#### 3.1.5 时间复杂度

​由于没有位置被访问两次，==最坏情况下的计算时间是O(m$\times$p)==。

### 3.2 线路铺设（队列实现）

#### 3.2.1 基本思路与实现

```cpp
Queue Q;
Q.Push (startPin);
while(!Q.isEmpty())
{
    Pin p = Q.Front();
    Q.Pop();
    for 访问p所有可以访问的相邻点pi{
        Visit(pi)
        Q.Push (pi);
    }
}
```

### 3.3 算术运算式

#### 3.3.1 简介

1. 表达式包括三种实体
   1. 运算符：（+，-，/、*）。
   2. 操作数：（a、b、c、d、e、f、g、h、3.25、（a + b）、（c + d）等）。
   3. 分隔符：（（，））。
2. 操作符度数：操作符需要的操作数数。
   1. 二元运算符需要两个操作数。a + b  c / d  e - f
   2. 一元运算符需要一个操作数。+ g - h

#### 3.3.2 中缀表达形式

1. 编写表达式的正常方式。二元运算符位于它们的左右操作数之间
2. 操作符优先级：  
   priority(*) = priority(/) > priority(+) = priority(-)
3. 分隔符内的子表达式被视为单个操作数，独立于表达式的其余部分。
4. 中缀表达式很难解析

#### 3.3.3 前缀表达形式

##### 3.3.3.1 简介

1. 操作数的相对顺序在中缀和后缀形式中是相同的

2. 操作符在其操作数的后缀形式之后立即出现。

   > Infix =  `(a + b) * (c – d) / (e + f)` \
   > Postfix = `a b + c d - * e f + /`

3. 一元运算符

   > `+ a => a @`  
   > `+ a + b => a @ b +`  
   > `- a => a ?`  
   > `- a-b => a ? b -`

4. 优点：

   1. 不需要括号
   2. 操作符的优先级不再相关

##### 3.3.3.2 前缀计算

![前缀](/assets/Blogs/Datastructure/stack/2.jpg)

#### 3.3.4 中缀转前缀

##### 3.3.4.1 原理

![转换](/assets/Blogs/Datastructure/stack/3.jpg)

##### 3.3.4.2 实现(方法与教材不同)

```cpp
int priority(char op)
{
	int priority = 0;
	if (op == '*' || op == '/') priority = 2;
	if (op == '+' || op == '-') priority = 1;
	if (op == '(')priority = 0;
	return priority;
}
void InfixtoPostfix(string& s1)
{
	stack<char>s;
	int j = 0;
	for (char i : s1)
	{
		if (i >= '0' && i <= '9' || i >= 'a' && i <= 'z')
		{
			cout << i;
		}
		else {
			if (s.empty())
				s.push(i);
			else if (i == '(')
				s.push(i);
			else if (i == ')')
			{
				while (s.top() != '(')
				{
					cout << s.top();
					s.pop();
				}
				s.pop();
			}
			else
			{
				while (priority(i) <= priority(s.top()))
				{
					cout << s.top();
					s.pop();
					if (s.empty()) break;
				}
				s.push(i);
			}
		}
	}
	while (!s.empty())
		{
			cout << s.top();
			s.pop();
		}
}
```

**分析：**
时间复杂度：==一个有n个token的表达式：O (n)==
占用空间： 栈大小<= 表达式中操作数个数==(e) + 1==('#')(书中以'#'作为结束符)
