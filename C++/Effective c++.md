##	1.习惯c++

#### 条款1 视c++为语言联邦



#### 条款02 尽量用*$const$*,*$enum$*,*$inline$*替换#define
宁可以编译器替代预处理器,比如

```c++
#define AS_T 100
```

此记号可能并未被编译器看见，可能在编译器开始处理源码被预处理器移走，此记号可能没有进入几号表，出错难以跟踪

```c++
const double AS_T 100
```

方法是用常量替换上面的宏，这样编译器看得到，就可以进入记号表。此外对于浮点常量，使用常量可能比<font  color='red' >使用*#define* 占用更小量的码</font>  ,预处理器会盲目将宏名称替换为1.653导致目标码出现多分1.653。改用常量不会出现此问题

用常量替换*#define*两种特殊情况

1. 定义常量指针，需要声明为$const$ ,如果是常量不变的$char*-base$ 字符串需要const两次，这种情况下，string通常更好

2. class专属常量，为将作用域限制于class，这种情况需要让其变为成员，为了唯一，就static成员。如果他是一个class的专属常量且是static的情况下。可以直接声明而不需要提供定义式。否则就需要定义式，但是不可以再设初值。 旧编译器也许不支持这种语法，不允许static成员在声明式提供初值，这是可以参考2。但是如果对于在class编译期间需要使用此常量，而编译器不允许static整数型class常量完成in class设定初值，可改用*enum hack* 做法(一个属于枚举类型的数值可以权充ints被使用)。也就是3，enum hack行为比较像#define，而不是const。取const地址合法，但是enum，#define的地址不合法。如果不想让外部获得一个pointer或者引用指向整数常量，enum可以实现此约束。enums,#defines不会导致非必要的内存分配。

   ```c++
   class GamePlayer{
   private:
   	static const int NumTurns=5;
   	int scores[NumTurns];
   	····
   }
   //2
   class ConstEsta{
   private:
       static const double Fudge;
       ···
   }
   const double ConstEsta::Fudge=1.35;
   //3
   class GamePlayer{
   private:
   	enum {NumTurns=5};
   	int scores[NumTurns];
   	····
   }
   ```

3. #defines误用：用其实现宏，宏像函数，但是不导致函数调用带来的额外开销。这种宏需要为宏所有实参加上括号，但是也可能出现预料外的情况参1。结果受和谁比较影响。可以使用inline获取宏的效率和一般函数的可预料函数行为以及类型安全性。（template inline）参2

```c++
#define CALL_WITH_MAX(a,b) f((a)>(b)?(a):(b))
//1
int a=5,b=0;
CALL_WITH_MAX(++a,b);//a+2
CALL_WITH_MAX(++a,b+10);//a+1

//2
template<typename T>
inline void callWithMax(const T &a,const T &b){
    f(a>b?a>b);
}
```

- 单纯常量，最好以const对象或者enums替换#defines
- 形似函数的宏，最好改用inline函数替换（constexpr,inline放在头文件里面）

#### 条款3 尽可能使用const（未看完）

关键字const出现在\*的左边，被指物是常量，出现在右边则是指针自身是常量。都出现表明指针和被指物都是常量。

STL迭代器是根据指针塑模，迭代器作用像是T*，声明迭代器为const，就声明指针为const一样，指针指向的东西不能改变，但是内容可以修改。如果希望迭代指向东西无法改动，使用$const\_interator$ 

const最威力的是面对函数声明时使用。让成员函数返回常量值可以避免客户端错误造成的意外。

```c++
T a,b,c
(a*b)=c //错误，这种情况下如果operator*是const就可提示错误
```

- $const$成员函数，为确认该成员函数可以用在$const$对象上。这很重要
  - 他们使得class接口更容易理解，可以知道那些函数可以改动对象内容，哪些不可以。
  - 使得操作$const$ 对象可能
  - 两个成员函数如果是常量性不同，可以被重载



#### 条款4: 确定变量使用前已被初始化

对于无任何成员的内置类型，手动完成初始化，而对于内置类型以外，初始化任务交给构造函数，就需要保证每个构造函数都将对象的每一个成员初始化

- 不要混淆赋值和初始化，C++规定对象的成员变量初始化动作发生在进入构造函数本体之前。在构造函数内，成员变量是被赋值。初始化发生在成员的默认构造函数字典调用之时。但是对内置类型，不保证在所看到的赋值动作时间点之前获取初值。初始化是

  ```c++
  ABC::ABC(const std::string &name,const std::string&address,const std::list<PhoneNumber>& phones)
  :theName(Name),
  theAddress(Address),
  thePhones(phones),
  numTimeConsulted(0){} //numTimeConsulted int 类型
  
  ```

- 基于赋值的版本先调用默认构造函数，然后再赋值，相比于成员初值列的方法调用copy构造函数低效。对于内置类型，赋值和初始化成本相同，考虑到一致性也采用成员初值列。

- 立下规则：在初值列列出所有成员变量。编译器会为自定义成员变量自动调用默认构造函数。一些情况即使面对的成员变量属于内置类型，也一定要使用初值列。如果成员是const或者reference,就一定需要初值。特殊情况可以合理在初值列里面遗漏赋值和初始化一样好的成员变量。改用赋值操作。并移到某函数

- c++成员有默认的初始化次序。base class 早于derived classes，class成员变量是以声明次序初始化。而和成员初值列的次序无关。

- <font color='red'>难</font> 不同编译单元内定义not-local static对象 的初始化次序。办法：将每个non-local static 成员对象搬到专属函数内，函数返回引用指向所含对象。 effective c++ p31  .内含static对象会让其在多线程环境中不确定性。

  

## 构造/析构/赋值运算

#### 条款5 : 了解c++默默编写并调用哪些函数

c++对空类声明-个copy构造函数，cpoy assignment操作符和一个析构函数。如果没有声明任何的构造函数，编译器也会声明一个default函数。这些函数都是public且是inline的。

如果在一个内含reference的成员的class内支持赋值操作，就必须自己定义copy assignment操作符。面对内含const成员的classes，编译器反应相同。更改const成员不合法。且c++不允许让reference改指向不同对象。

还有一种特殊情况，如果基类的赋值操作符被声明为私有，，编译器就会拒绝为其派生类生成一个赋值操作符。



#### 条款6：如果不想使用编译器自动生成的函数，就该明确拒绝。

可以将这类函数自行声明为私有，可以阻止调用，但是不安全。因为成员函数或者友元函数还是可以调用。除非不去定义。iostream标准库就有这样实现。

或者单独声明一个防止copying的基类。要做的就是继承这个类

```c++
class Uncopy{
 protected:
    Uncopy();
    ~Uncopy();
private:
    Uncopy(const Uncopy&);
    Uncopy& operator=(const Uncopy&);
};
```

但是这种总是扮演基类，可能导致多重继承。而多重继承可能阻止***empty base class optimization***

>
>
>​	空基类优化是：只要不会和同一类型的另一个对象或者子对象分配在同一地址，就不需要为其分配地址空间。关于不会和同一类型的另一个对象或者子对象分配在同一地址：
>
>```c++
>class Empty
>{ };
>
>class EmptyToo : public Empty
>{ };
>
>class EmptyThree : public Empty, public EmptyToo
>{ };
>
>sizeof(Empty) : 1
>sizeof(EmptyToo) : 1
>sizeof(EmptyThree) : 2
>```
>
>$Emptythree$的基类$Empty$和$EmptyToo$不能分配在同一地址空间，C++内存布局不允许相同类型的子对象偏移量相同

#### 条款7 ：为多态基类声明virtual析构函数

