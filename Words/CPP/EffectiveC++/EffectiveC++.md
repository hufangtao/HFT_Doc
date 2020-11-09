## 让自己习惯C++

### 条款02：尽量以const，enum，inline替换#define
编程中必须尽可能不给自己犯错的机会，或者能让错误及早暴露出来。
- 单纯变量的定义，最好以const对象或enums替换#defines。
- 对于形似函数的宏，最好改用inline替换#defines

### 条款03：尽可能使用const
- 使用const帮助编译器侦测出错误用法。
- 当const和non-const成员函数有着实质等价的实现时，令non-const版本调用const版本可避免代码重复。

### 条款04：确定对象被使用前已先被初始化
- 为内置型对象进行手动初始化
  未初始化造成的后果是严重的，而C++不保证初始化，故手动将所有对象初始化是比较妥当的。
- 构造函数最好使用成员初值列，而不要在构造函数本体内使用赋值操作。初值列的顺序应和成员声明顺序一致

## 构造、析构、赋值运算

### 条款05：了解C++默默编写并调用哪些函数
- 编译器可以暗自为class创建default构造函数、copy构造函数、copy赋值操作函数和析构函数  

只有当这些函数被调用，它们才会被编译器创建出来。一般而言，编译器产出的析构函数是non-virtual，除非这个class的base class自身声明有virtual析构函数。而copy构造函数和copy assignment操作符，将来源对象的每个non-static成员变量拷贝到目标对象。  

但是在内含reference成员或者const成员的class，编译器是不会产生copy assignment操作符的。此外，如果base classes将copy assignment操作符声明为private，那么其derived classes也不会产生copy assignment操作符。  

在C++11之后，默认函数新增了移动构造函数和移动赋值操作符。

### 条款06：若不想使用编译器自动生成的函数，就该明确拒绝
- 为了驳回编译器自动（暗自）提供的机能，可将相应的成员函数声明为private并且不予实现。使用像UnCopyable这样的base class也是一种做法。  
- 在C++引入=delete后，似乎可以替代这种方法，让实现更加直观、自然。

### 条款07：为多态基类声明virtual析构函数
- 带有多态性质的base class应该声明一个virtual析构函数。不然对其base类型指针析构时，可能会出现局部销毁的情况，因为无法调用到derived class的析构函数。
- 如果classes设计目的没有打算作为base classes，或者不是为了具备多态性，那么就不应该声明virtual析构函数。

### 条款08：别让异常逃离析构函数
- C++并不禁止析构函数吐出异常，但是不要那么做！因为析构函数吐出异常，容易导致程序过早结束或者出现不明确行为，这往往导致糟糕的后果。
- 如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么类应该提供一个普通函数（而非在异构函数中）执行该操作。

但是在实践中，于析构函数中执行动作也是一个常见的需求。可以通过两种方式避免上述的问题，一是使用std:abort，在抛出异常的时候结束程序，二是直接吞掉异常。当然这两种的方式的问题在于无法对异常出现的情况做出处理，但是总要比草率的结束程序或者不明确行为要来的好。更好的补救措施是将析构函数中执行的动作抽离出来，让用户主动执行，这样就能应对异常了，同时在析构函数中检测这个函数的执行，一旦用户遗忘，就采取上述两种措施。  

就是不要把产生异常的操作放在析构函数中。

### 条款09：绝不再构造和析构过程中调用virtual函数
- 在构造和析构期间不要调用virtual函数，因为这类调用不会下降至派生类。  
根本原因：在derived class对象的base class构造期间，对象的类型是base class而非derived class。

### 条款10：令operator=返回一个reference to *this
- 在所有赋值相关的运算中，都建议将返回类型设为reference to *this，这虽然不是强制性的，但最好这么做，以保持和内置类型、标准库一致。  
主要是为了实现`a=b=c`这样的“连续赋值”

### 条款11：在operator=中处理“自我赋值”
“自我赋值”发生在对象被赋值给自己的时候，这是完全合法的。
- 确保当对象自我赋值时`operator=`有良好行为。包括比较来源对象和目标对象的地址、精心周到的语句顺序、以及`copy-and-swap`

### 条款12：复制对象时勿忘其每一个成分
对于自己的copying函数，编译器是不会对复制对象的遗漏发出警告的。当发生继承时，复制函数遗漏继承来的成员变量是非常隐晦的。因此，只要为derived class撰写copying函数，就必须小心地复制其base class成分，这些成分往往是private，所以就要让derived class的copying函数调用对应的base class函数

- Copy函数应该确保复制“对象内的所有成员变量”及“所有base class成分”
- 不要尝试以某个copy函数实现另一个copy函数。应该把共同机能放进第三方函数中，并由两个copy函数共同调用。

## 资源管理
### 条款13：以对象管理资源
- 为防止资源泄漏，请使用RAII对象，它们在构造函数中获得资源并在析构函数中释放资源。  
  将资源放进对象内，依赖C++的“析构函数自动调用机制”确保资源被释放。在C++11后，请使用unique_ptr和shared_ptr来达到这个目的。

### 条款14：在资源管理类中小心copying行为
- 复制RAII对象必须一并复制他所管理的资源，所有资源的copy行为觉得RAII对象的copy行为
- 普遍RAII类copy行为：抑制copy、引用计数。  
unique_ptr和shared_ptr并非资源管理的全部，因为这两者往往只使用于heap-based资源。有时，有必要建立自己的资源管理类。

### 条款15：在资源管理类中提供对原始资源的访问
每一个RAII class应该提供一个“取得其所管理之资源”的办法。至于是显式转换还是隐式转换，完全取决于其执行的工作，及被使用的状况。  

### 条款16：成对使用new和delete时要采用相同形式
如果在new表达式中使用[]，delete也要使用[]；如果new中不使用[]，delete中也不要使用[]。特别注意typedef，它可能会迷惑你。

### 条款17：以独立语句将newed对象置入智能指针
以独立语句将newed对象存储于智能指针内，不然，一旦异常抛出，有可能导致难以察觉的资源泄露。

## 设计与声明
### 条款18：让接口容易被正确使用，不易被误用

### 条款19：设计class犹如设计type
设计class必须面对的问题：

新type的对象应该如何被创建和销毁
对象的初始化和对象的赋值有什么样的差别
新type的对象如果被passed by value，意味着什么（copy构造函数用来定义一个type的passed by value该如何实现）
什么是新type的“合法值”
你的新type需要配合某个继承图系吗
你的新type需要什么样的转换
什么样的操作符和函数对此新type而言是合理的
什么样的标准函数应该驳回
谁该取用新type的成员
什么是新type的“未声明接口”
你的新type有多么一般化
你真的需要一个新type吗

### 条款20：宁以pass-by-reference-to-const替换pass-by-value
缺省情况下，C++是以by value方式传递对象至函数的，这种方式默认调用对象的copy构造函数产生一个副本，这就有可能造成比较严重的耗时。采用pass-by-reference-to-const可以有效避免这个问题，const是必要的，因为这保证传入对象不会被修改。

同时，以by-reference方式传递参数可以避免slicing（对象切割）问题。

但是，对于内置类型，以及STL的迭代器和函数对象，pass-by-value比较合适。

### 条款21：必须返回对象时，别妄想返回其reference
这个条款主要是防止矫枉过正，滥用返回reference。绝不要返回pointer或reference指向一个local stack对象，或者heap-allocated对象；也不要返回pointer或referece指向local static对象而有可能同时需要多个这样的对象。

通常来说，并不太需要担心返回对象的消耗，编译器是存在RVO优化的，尤其是C++11引入move后，选择就更加多样了。

### 条款22：将成员变量声明为private
将成员变量声明为private提供了更好的封装性，为所有可能的实现提供弹性，还可以验证class的约束条件以及函数的前提和时候状态等等。

总之，将成员变量声明为private，可赋予客户访问数据的一致性、可细微划分访问控制、允诺约束条件获得保证，并提供class作者以充分的实现弹性。

### 条款23：宁以non-member、non-friend替换member函数
与一般直觉相反，non-member函数的封装性要比member函数高。因为对于对象内部的数据，越少代码可以看到数据，其封装性就越高。non-member non-friend函数不会增加可访问数据的函数数量。对于这样的函数，将其与相应的类组织在同一个namespace是一个比较合理的选择。总之，使用non-member non-friend函数替换member函数，可以增加封装性、包裹弹性和机能扩充性。

### 条款24：若所有参数皆需类型转换，请为此采用non-member函数
只有当参数被列于参数列内，才是隐式类型转换的合格参与者，所以当需要为某个函数的所有参数（包括被this指针所指的那个隐喻参数）进行类型转换，那么这个函数必须是个non-member。
```
class Rational{
    ……
}
const Rational operator*(const Rational& lhs, const Rational& rhs)
{
    ……
}
```
上面是个实例。

### 条款25：考虑写出一个不抛异常的swap函数
当std::swap的效率不足时，可以提供一个swap成员函数（这通常意味着交换指针，Impl实现），绝不要抛出异常。

如果提供了一个member swap，也应该提供一个non-member swap来调用前者。对于classes，也请特化std::swap。

为保证没有专用swap时，程序仍然正常运行， 请用using std::swap，但调用时不要加命名空间资格修饰。这条是利用了C++名称查找法则的顺序。

## 实现
### 条款26：尽可能延后变量定义式的出现时间
尽可能延后变量定义式的出现，这样做可增加程序的清晰度并改善程序效率。
因为变量的构造和析构都是需要成本的。如果先定义了变量，但是函数提前返回，导致变量未被使用，会浪费成本。

### 条款27：尽量少做转型动作
- 如果可以，尽量避免转型，特别是在重视效率的代码中使用dynamic_cast，尽量使用无转型的方式实现。
- 尽量使用C++-style转型，不要使用旧时转型。前者很容易辨识出来，而且每个cast函数有着比较明确的职责

### 条款28：避免返回handles指向对象内部成分
- 避免返回handles（迭代器、指针、引用）指向对象内部。可以增加类的封装性，可能使得const成员函数的行为像个const，并降低发生“悬挂”的可能性（例如悬挂指针）。

### 条款29：为“异常安全”而努力是值得的
- 异常安全函数即使发生异常也不会泄漏资源或允许任何数据结构破坏。这样的函数区分为三种可能的保证：基本型、强烈型、不抛异常型
- “强烈保证”往往能够以copy-andswap实现出来，但“强烈保证”并非对所有函数都可实现或具备实现意义。
- 函数提供的“异常安全保证”通常最高只等于其所调用之各个函数的“异常安全保证”中的最弱者。

### 条款30：透彻了解inlining的里里外外
inline函数，看起来像函数、行为也像函数，比宏好用，可以避免函数调用的额外开销。  
然而inline的实际功能比这还要多  
- 将inline限制在小型、被频繁调用的函数身上。可以使得调试过程和优化过程更容易，也可使潜在的代码膨胀问题最小化，使程序的速度提升机会最大化。  
- 不要因为function templates出现在头文件，就将它们声明为inline

### 条款31：将文件间的编译依存关系降到最低
- 如果使用引用和指针可以完成任务，就不要使用对象。你可以靠一个类型的声明式就定义出指向该类型的引用和指针；但如果定义某类型的对象，就需要用到该类型的定义式。
- 尽量以class声明式替换class定义式。注意，当你声明一个函数而它用到某个class时，你并不需要该class的定义；纵使函数以传值的方式传递该类型的参数亦然。


## 继承与面向对象设计
### 条款32：确定你的public继承塑模出is-a关系
- “public”继承意味`is-a`。适用于父类身上的每一件事情一定也适用于子类，因为每一个子类对象也是一个父类对象。  
但是`is-a`并不是继承的唯一关系类，另两个常见关系是`has-a`和`is-implemented-in-terms-of`（根据某物实现出）。

### 条款33：避免遮掩继承而来的名称
- 子类内的名称会遮掩父类内的名称。在public继承下从来没有人希望如此。
- 为了让被遮掩的名称在见天日，可使用using声明式或转交函数（forwarding functions）。
```
public:
  using Base::mf1;
```
```
public:
  virtual void mf1()
  {Base::mf1();}
```
### 条款34：区分接口继承和实现继承
- 接口继承和实现继承不同。在public继承之下，子类总是继承父类的接口
- 纯虚函数：指定接口继承
- 非纯虚函数：指定接口继承，并为子类提供了该函数的缺省实现
- 非虚函数：具体指定接口继承，以及强制性实现继承

### 条款35：考虑virtual函数以外的其它选择
本条款的根本忠告是，当你为解决问题而寻找某个设计方法时，不妨考虑虚函数的替代方案
因为虚函数的作用非常明显，我们可能因此没有认真考虑其他替代方案。下面我们讨论一些虚函数的可替代方案。
#### 使用非虚函数接口实现“函数模板”
使用普通函数调用虚函数。让子类来实现虚函数自己的功能。而父类中的普通函数处理了其他通用逻辑，虚函数调用前的配置检查，调用后的通用逻辑处理等。  
如果`addCurScore`定义为虚函数让子类调用，就没办法实现调用前的检查和调用后的处理了。
```
class GeneralActivity
{
  public:
    void addCurScore()
    {
      // 通用配置检测
      onAddCurScore();  // 虚函数，每个子类可另外实现
      // 通用排行榜处理
      // 通用奖励处理
    }
}
```
#### 使用“策略模式”
asio中调度器的设计，为了兼容不同平台，采用策略模式实现reactor。`reactor`的定义可以是函数指针、`function`等
```
class scheduler
{
  // The task to be run by this service.
  reactor* task_;
}

#if defined(BOOST_ASIO_HAS_IOCP) || defined(BOOST_ASIO_WINDOWS_RUNTIME)
typedef class null_reactor reactor;
#elif defined(BOOST_ASIO_HAS_IOCP)
typedef class select_reactor reactor;
#elif defined(BOOST_ASIO_HAS_EPOLL)
typedef class epoll_reactor reactor;
#elif defined(BOOST_ASIO_HAS_KQUEUE)
typedef class kqueue_reactor reactor;
#elif defined(BOOST_ASIO_HAS_DEV_POLL)
typedef class dev_poll_reactor reactor;
#else
typedef class select_reactor reactor;
#endif
```

### 条款36：绝不重新定义继承而来的非虚函数
```
class B
{
  public:
    void mf();
}
class D: public B
{
}

D x;
B* b_ptr = &x;
D* d_ptr = &x;

b_ptr->mf();
d_ptr->mf();
```
如果D中重写了mf，那么`d_ptr`和`b_ptr`调用`mf`的表现不一致。如果出现在函数参数调用等情况下，会导致类型一致但是表现不一致的情况。这种情况下，就违背了“D是一个B”的定义。  

### 条款37：绝不重新定义继承而来的缺省值
虚函数是动态绑定的，而缺省参数值确实静态绑定的。意思是你可能会在“调用一个定义于子类中的虚函数”的同事，却使用基类为它所指定的缺省参数值  
- 在业务中尽量不要缺省参数，调用者需要明确知道自己要做的是什么事情。

### 条款38：通过复合塑模出`has-a`或者“根据某物实现出”
- 复合的意义和public继承完全不同

### 条款39：明智而谨慎地使用private继承



### 条款43：学习处理模板化基类内的名称
```

class CompanyA
{
    public:
        void sendClearText()
        {
            printf("A: send clear text \n");
        }

        void sendEncrypted()
        {
            printf("A: send encrypted \n");
        }
};

class CompanyB
{
    public:
        void sendClearText()
        {
            printf("B: send clear text \n");
        }

        void sendEncrypted()
        {
            printf("B: send encrypted \n");
        }
};

template<typename T>
class MsgSender
{
    public:
        void sendClear()
        {
            printf("MshSender: sendClear");
            T t;
            t.sendClearText();
        }
};

template<>
class MsgSender<CompanyA>
{
    public:
        // 不定义sendclear
};

template<typename T>
class LogMsgSender: public MsgSender<T>
{
    public:
        void LogSendClear()
        {
            sendClear();
        }
};
```
上述代码会报编译错误，因为`Msgsender<T>`不一定存在`sendClear`函数。  
一眼看起来`Msgsender`模板必定包含`sendClear`，但是考虑到`MsgSender`会有全特化的情况。  
严格的编译器会报可能找不到`sendClear`的情况  





### 条款40：明智而谨慎得使用多重继承
多重继承只是面向对象工具箱的一个工具而已。和单一继承比较。它通常比较复杂，使用上也比较难以理解，所以如果有单一继承的设计方案，尽量采用单一继承方案。  
- 多重继承比单一继承复杂。他可能导致新的歧义性，比如虚继承。
- 虚继承会增加大小、速度、初始化（赋值）复杂度等成本。如果虚基类不带任何数据，将是最具使用价值的情况。
  只使用最基本的接口设计，不在虚基类中设计任何数据。  
- 两个比较使用多重继承的地方：
  1. `public`继承某个`Interface class`
  2. `private`继承某个协助实现类

## 7 模板与泛型编程
模板的最初动机很直接：让我们得以建立“类型安全”的容器如vector、list和map。

### 条款41：了解隐式接口和编译器多态
面向对象编程世界总是以显式接口和运行期多态解决问题。
- 类和模板都是支持接口和多态的  
- 类接口是显式的，以函数签名为中心。多态则是通过虚函数发生于运行期
- 模板接口是隐式的，多态则是通过模板具现化和函数重载解析发生于编译器。

### 条款42：了解typename的双重定义
### 











