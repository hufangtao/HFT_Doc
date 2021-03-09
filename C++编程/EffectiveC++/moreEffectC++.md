## 基础议题

### 条款1：仔细区别指针和引用
书中：当你知道你需要指向某个东西，而且绝对不会改变指向其他东西，或是当你实现一个操作符而其语法需求无法由指针完成，你就应该选择引用。  
书中主要提到指针可以为null，而引用不能。
```
char* pc = nullptr;
char& rc = *pc;
```
这种情况下访问rc会有错误
在处理的时候可以对指针判null，带不能对引用判空

在项目中，如果传值用到了（*ptr），首先要判断ptr是否为空。

### 条款2：最好使用C++转型操作符
使用新的转型操作符
static_cast
const_cast
dynamic_cast
reinterpret_cast
主要是增加转型程序的可读性

### 条款3：绝对不要对多态方式处理数组
函数传值的时候，编译器支持Base形参接受Drive对象
void print(Base b)
{}
Drive d;
print(d);

// 如果使用数组
void print(Base b[], int size)
{}
Drive d[10];
print(d, 10);   // 可以通过编译，但是可能有运行错误
当我们使用d[idx]访问数组时，用idx*size(Base)计算。如果Base和Drive的size不同就有问题


### 条款4：非必要不提供default constructor
系统生成的默认构造函数，不一定能向我们想象的一样初始化，可能会有空值。  
使用这些值时，需要额外的逻辑来判断是否被初始化。  
工程中需要提供默认构造函数，保证所有参数初始化，避免有未初始化的成员。

## 2 操作符
### 条款5：对定制的“类型转换函数”保持警惕
隐形类型转换操作符
```
class Rational
{
    public:
        Rational(int number = 0);   // 将int转为Rational
    public:
        operator double() const;    // 将Rational转为double
}
```
Rational r(1);
cout << r;  // 正确，cout发现Rational没有定义输出运算符，转而将其转为double输出。
解决方案，提供asdouble接口实现打印功能，而不是以隐形类型转换方式实现。
类似 string.c_str()的实现方式，就是用接口函数而不是隐形转换

对于隐形类型转换符，只要不声明就行
而带参数的构造函数则需要声明为explicit的

### 条款6：区别i++和++i
++符号重载
```
const UPInt operator++(int);    // 后置
UPInt& operator++();            // 前置
```

### 条款7：千万不要重载&&、||和,操作符
本条款主要想说明，虽然很多操作符都是可以重载的，但是毫无理由去进行，是没有道理的。  
操作符重载的目的是要让程序更容易被理解、被阅读。  

### 条款8：了解各种不同意义的new和delete
new分为
- new操作（new operator）
  指创建对象的过程，申请内存加构造。
- 操作符new（operator new）
  负责申请内存，可重载
  placement new可以在指定内存位置创建对象

delete
- delete operator表示
  delete ptr
- operator delete表示delete操作符
  负责析构+释放内存

## 3 异常
### 条款9：利用析构避免泄露资源
把资源封装在对象中，通常便可以在异常出现时避免泄露资源。  
类似于智能指针，在局部智能指针被析构时，会自动归还资源。  

### 条款10：在构造内阻止资源泄露
考虑
```
class Base
{
    Base() {
        ptr = new Obj(); // 如果ptr的构造失败了，则会造成泄露
    }
    private:
        Obj* ptr
}
```
方案
1. 把ptr交给局部变量来处理，定义成智能指针。
2. 把ptr的赋值改为构造参数传递。  


### 条款11：禁止异常流出析构之外
考虑一种情况：
```
Session::~Session
{
    log();  // 如果log抛出异常，会导致无法析构、后续无法执行
}
```

可以考虑使用try...catch包裹

### 条款12：了解“抛出一个异常”和“传递一个参数”或“调用一个虚函数”之间的差异
函数参数的声明语法和catch子句的声明语法，简直如出一辙
```
class Widget {...}  // 某个类

void f1(Widget w);
void f2(Widget& w);
void f3(Widget* w);

catch (Widget w)
catch (Widget& w)
catch (Widget* w)
```
#### 传值方式
相同点：
- 函数参数和异常的传递方式都包括：传值、引用和指针
不同点：
- 函数的调用最终返回到调用处，异常不会再回到抛出段。

```
void processAndThrowWidget()
{
	Widget local_widget;

	throw local_widget;
}
```
离开processAndThrowWidget之后，local_widget会被析构。所以local_widget总是被复制抛出，复制调用copy构造函数。
1. 异常首先复制出一个临时对象。所以千万不要抛出一个指针异常
2. 将临时对象传递给catch

#### 传值的隐式转换
- 函数参数支持隐式转换
- 异常传参仅有两种情况可以转换
  1. 类继承，宝库针对runtime_errors而写的catch子句
  2. 第二种转换，是有形指针转为无形指针，

#### 匹配方式
- 异常从前往后依次匹配
- 函数匹配最优解

### 条款13：以引用方式捕捉异常
- 传值：两次复制，无法析构子类赋值。
- 指针：复制指针，指向内存已经被析构。

### 条款14：明智运用`exception specifications`

