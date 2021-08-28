# 基础

## main函数执行前完成什么工作？

main函数执行之前主要是初始化系统资源

1. 设置栈指针
2. 初始化static静态和global全局变量，即data段内容
3. 将未初始化部分的赋初值：数值型short，int，long等为0，bool为FALSE，指针为NULL，等等，即.bss段的内容
4. 运行全局构造器，估计是C++中构造函数之类
5. 将main函数的参数，argc，argv等传递给main函数，然后才真正运行main函数

**在C++内部，在全局构造函数中，访问其他全局或者静态变量，其结果是不可预知的**

## C/C++程序内存的分配

* 一个C/C++编译的程序占用内存分为以下几个部分：

  * 栈区（stack）：由编译器自动分配与释放，存放为运行时函数分配的局部变量、函数参数、返回数据、返回地址等。其操作类似于数据结构中的栈。

  * 堆区（heap）：一般由程序员自动分配，如果程序员没有释放，程序结束时可能有OS回收。其分配类似于链表。

  * 全局区（静态区static）：存放**全局变量、静态数据、常量**。程序结束后由系统释放。**全局区分为已初始化全局区**（.data）和**未初始化全局区**（.bss）

  * 常量区（文字常量区，.roddata）：存放**常量字符串**，**const修饰的常量**，**虚函数表**，程序结束后由系统释放

  * 代码区：存放函数体（类成员函数和全局区）的二进制代码。

    <img src="E:\notes\imgs\os\prog_align.png" alt="prog_align" style="zoom:80%;" />

* 三种内存分配方式
  * 从静态存储区分配
    * 内存在**程序编译的时候已经分配好**，这块**内存在程序的整个运行期间都存在**。例如全局变量，static变量
  * 在栈上创建
    * 在执行函数时，函数内局部变量的存储单元可以在栈上创建，函数执行结束时，这些内存单元会自动被释放。
      栈内存分配运算内置于处理器的指令集，效率高，但是分配的内存容量有限。（32位Linux系统栈为2M，64位为8M，可以通过系统配置更改，不同的发行版也可能不同）
  * 从堆上分配
    * 亦称为动态内存分配。程序在运行的时候使用malloc或者new申请任意多少的内存，程序员自己负责在何时用free或delete释放内存。动态内存的生命周期有程序员决定，使用非常灵活，但如果在堆上分配了空间，既有责任回收它，否则运行的程序会出现内存泄漏，频繁的分配和释放不同大小的堆空间将会产生内存碎片。

## **堆和栈的区别**

* 管理方式不同：栈是由编译器自动申请和释放空间，堆是需要程序员手动申请和释放；
* 空间大小不同：栈的空间是有限的，在32位平台下，VC6下默认为1M，堆最大可以到4G；
* 能否产生碎片：栈和数据结构中的栈原理相同，在弹出一个元素之前，上一个已经弹出了，不会产生碎片，如果不停地调用malloc、free对造成内存碎片很多；
* 生长方向不同：堆生长方向是向上的，也就是向着内存地址增加的方向，栈刚好相反，向着内存减小的方向生长。
* 分配方式不同：堆都是动态分配的，没有静态分配的堆。栈有静态分配和动态分配。静态分配是编译器完成的，比如局部变量的分配。动态分配由malloc 函数进行分配，但是栈的动态分配和堆是不同的，它的动态分配是由编译器进行释放，无需我们手工实现。
* 分配效率不同：栈的效率比堆高很多。栈是机器系统提供的数据结构，计算机在底层提供栈的支持，分配专门的寄存器来存放栈的地址，压栈出栈有相应的指令，因此比较快。堆是由库函数提供的，机制很复杂，库函数会按照一定的算法进行搜索内存，因此比较慢

## [new/malloc，delete/free的异同](https://blog.csdn.net/cherrydreamsover/article/details/81022039)

## 静态全局变量、全局变量、静态局部变量、局部变量的区别

* 静态全局变量、全局变量区别
  * 静态全局变量和全局变量都属于常量区
  * 静态全局变量只在本文件中有效，别的文件想调用该变量，是调不了的，而全局变量在别的文件中可以调用
  * 如果别的文件中定义了一个该全局变量相同的变量名，是会出错的
* 静态局部变量、局部变量的区别
  * 静态局部变量是属于常量区的，而函数内部的局部变量属于栈区；
  * 静态局部变量在该函数调用结束时，不会销毁，而是随整个程序结束而结束，但是别的函数调用不了该变量，局部变量随该函数的结束而结束
  * 如果**定义这两个变量的时候没有初始值时，静态局部变量会自动定义为0，而局部变量就是一个随机值**
  * **静态局部变量在编译期间只赋值一次，以后每次函数调用时，不在赋值，调用上次的函数调用结束时的值**。局部变量在调用期间，每调用一次，赋一次值

## 只能用初始化列表而不能用赋值的情况

* 当类中有const、reference成员变量时
* 调用基类构造函数都必须使用初始化列表
* 成员类型是没有默认构造函数的类，必须在成员列表调用其构造函数显式初始化

## **C++哪些函数不能声明为虚函数**

* 主要两类：不能被继承的函数，不能被重写的函数
* **普通函数**：普通函数不属于成员函数，是不能被继承的。普通函数只能被重载，不能被重写，因此声明为虚函数没有意义。因为编译器会在编译时绑定函数，而多态体现在运行时绑定。通常通过**基类指针指向子类对象实现多态**。
* 友元函数：友元函数不属于类的成员函数，不能被继承。对于没有继承特性的函数没有虚函数的说法
* 构造函数：构造函数是用来初始化对象的。假如子类可以继承基类构造函数，那么子类对象的构造将使用基类的构造函数，而基类构造函数并不知道子类的有什么成员，显然是不符合语义的。从另外一个角度来讲，多态是通过基类指针指向子类对象来实现多态的，在对象构造之前并没有对象产生，因此无法使用多态特性，这是矛盾的。因此构造函数不允许继承
* 内联成员函数：内联函数就是为了在代码中直接展开，减少函数调用花费的代价。也就是说内联函数是在编译时展开的。而虚函数是为了实现多态，是在运行时绑定的。因此显然内联函数和多态的特性相违背
* 静态成员函数：首先静态成员函数理论是可继承的，但是静态成员函数是编译时确定的，无法动态绑定，不支持多态，因此不能被重写，也就不能被声明为虚函数
* 一个类（**无论是普通类还是类模板**）的成员模板（**本身是模板的成员函数**）不能是虚函数

## NULL和nullptr的区别

* 在C语言中，NULL通常被定义为：#define NULL ((void *)0)

* 所以NULL实际上是一个空指针，如果在C语言中写入以下代码，编译是没有问题的，因为在**C语言中把空指针赋给int和char指针的时候，发生了隐式类型转换，把void指针转换成了相应类型的指针**

  ```c++
  int  *pi = NULL;
  char *pc = NULL;
  ```

* 以上代码如果使用C++编译器来编译则是会出错的，因为C++是强类型语言，void*是不能隐式转换成其他类型的指针的，所以实际上编译器提供的头文件做了相应的处理

  ```c++
  #ifdef __cplusplus
  #define NULL 0
  #else
  #define NULL ((void *)0)
  #endif
  ```

* 可见，在C++中，NULL实际上是0。 因为**C++中不能把void*类型的指针隐式转换成其他类型的指针，所以为了解决空指针的表示问题，C++引入了0来表示空指针**，这样就有了上述代码中的NULL宏定义

* 但是实际上，用NULL代替0表示空指针**在函数重载时会出现问题**，程序执行的结果会与我们的想法不同，举例如下：

  ```c++
  #include <iostream>
  using namespace std;
   
  void func(void* t)
  {
  	cout << "func（ void* ）" << endl;
  }
   
  void func(int i)
  {
  	cout << "func（ int ） " << endl;
  }
   
  int main()
  {
  	func(NULL);
  	func(nullptr);
  	system("pause");
  	return 0;
  }
  ```

* 在这段代码中，我们对函数func进行可重载，参数分别是void*类型和int类型，但是运行结果却与我们使用NULL的初衷是相违背的，**因为我们本来是想用NULL来代替空指针，但是在将NULL输入到函数中时，它却选择了int形参这个函数版本，所以是有问题的，这就是用NULL代替空指针在C++程序中的二义性**

* 为解决NULL代指空指针存在的二义性问题，在C++11特意引入了nullptr这一新的关键字来代指空指针，从上面的例子中我们可以看到，使用nullptr作为实参，确实选择了正确的以void*作为形参的函数版本

* 总结：**NULL在C++中就是0，这是因为在C++中void* 类型是不允许隐式转换成其他类型的，所以之前C++中用0来代表空指针**，但是在重载整形的情况下，会出现重载二义性问题。所以，**C++11加入了nullptr，可以保证在任何情况下都代表空指针**，而不会出现上述的情况，因此，建议以后还是都用nullptr替代NULL吧，而NULL就当做0使用

## 内存对齐

**内存对齐作用：**

1. 平台原因(移植原因)：**不是所有的硬件平台都能访问任意地址上的任意数据的；某些硬件平台只能在某些地址处取某些特定类型的数据，否则抛出硬件异常**
2. 性能原因：**数据结构(尤其是栈)应该尽可能地在自然边界上对齐**。原因在于，**为了访问未对齐的内存，处理器需要作两次内存访问**；而对齐的内存访问仅需要一次访问

**内存对齐的3大规则:**

1. 对于结构体的各个成员，第一个成员的偏移量是0，**排列在后面的成员其当前偏移量必须是当前成员类型的整数倍**
2. 结构体内所有数据成员各自内存对齐后，**结构体本身还要进行一次内存对齐，保证整个结构体占用内存大小是结构体内最大数据成员的最小整数倍**
3. 如程序中有#pragma pack(n)预编译指令，则所有成员对齐以n字节为准(即偏移量是n的整数倍)，不再考虑当前类型以及最大结构体内类型(**结构(或联合)本身对齐将按照#pragma pack指定的数值和结构(或联合)最大数据成员长度中，比较小的那个进行**)

`#pragma pack` 的主要作用

* 改变编译器的内存对齐方式，这个指令在网络报文的处理中有着重要的作用，`#pragma pack(n)`是他最基本的用法，其作用是改变编译器的对齐方式， 不使用这条指令的情况下，编译器默认采取`#pragma pack(8)`也就是8字节的默认对齐方式，n值可以取（`1`， `2`， `4`， `8`， `16`） 中任意一值

**联合体union内存对齐的2大规则:**

1. 找到占用字节最多的成员
2. union的字节数必须是占用字节最多的成员的字节的倍数，而且需要能够容纳其他的成员

## volatile

* 访问寄存器比访问内存单元要快，**编译器会优化减少内存的读取，可能会从寄存器中读脏数据**。**声明变量为volatile，编译器不再对访问该变量的代码优化，仍然从内存读取，使访问稳定**

* 总结：volatile关键词影响编译器编译的结果，用volatile声明的变量表示该变量随时可能发生变化，与该变量有关的运算，不再编译优化，以免出错

* 使用实例如下：
  * 并行设备的硬件寄存器（如：状态寄存器）
  * 一个中断服务子程序中会访问到的非自动变量(Non-automatic variables)
  * 多线程应用中被几个任务共享的变量
* 一个参数既可以是const还可以是volatile吗？解释为什么
  * 可以。一个例子是只读的状态寄存器。它是volatile因为它可能被意想不到地改变。它是const因为程序不应该试图去修改它
* 一个指针可以是volatile吗？解释为什么
  * 可以。尽管这并不很常见。一个例子当中断服务子程序修该一个指向一个buffer的指针时

* 下面的函数有什么错误：

  ```c++
  int square(volatile int *ptr) {
  	return *ptr * *ptr;
  }
  ```

  下面是答案：

  * 这段代码有点变态。这段代码的目的是用来返指针*ptr指向值的平方，但是，由于*ptr指向一个volatile型参数，编译器将产生类似下面的代码：

    ```c++
    int square(volatile int *ptr){
        int a,b;
        a = *ptr;
        b = *ptr;
        return a * b;
    }
    ```

    由于*ptr的值可能被意想不到地该变，因此a和b可能是不同的。结果，这段代码可能返不是你所期望的平方值！正确的代码如下：

    ```c++
    long square(volatile int *ptr){
        int a;
        a = *ptr;
        return a * a;
    }
    ```

## 虚函数、纯虚函数

* 纯虚函数是一种特殊的虚函数，在基类中不能对虚函数给出有意义的实现，而把它声明为纯虚函数，它的实现留给该基类的派生类去做

  ```c++
  virtual test() = 0;
  ```

- 类里如果声明了虚函数，这个函数是实现的，哪怕是空实现，它的作用就是为了能让这个函数在它的子类里面可以被覆盖，这样的话，这样编译器就可以使用后期绑定来达到多态了。**纯虚函数只是一个接口，是个函数的声明而已，它要留到子类里去实现**。
- 虚函数在子类里面也可以不重载的；但纯虚函数必须在子类去实现。
- 虚函数的类用于 “实作继承”，继承接口的同时也继承了父类的实现。当然大家也可以完成自己的实现。纯虚函数关注的是接口的统一性，实现由子类完成。
- 带纯虚函数的类叫抽象类，这种类不能直接生成对象，而只有被继承，并重写其虚函数后，才能使用。抽象类和大家口头常说的虚基类还是有区别的，在 C# 中用 abstract 定义抽象类，而在 C++ 中有抽象类的概念，但是没有这个关键字。抽象类被继承后，子类可以继续是抽象类，也可以是普通类，而虚基类，是含有纯虚函数的类，它如果被继承，那么子类就必须实现虚基类里面的所有纯虚函数，其子类不能是抽象类

## 虚函数指针、虚函数表

- 虚函数指针：在含有虚函数类的对象中，指向虚函数表，在运行时确定。
- 虚函数表：在程序只读数据段（`.rodata section`，见：目标文件存储结构:http://t.cn/E4WVBeF），存放虚函数指针，如果派生类实现了基类的某个虚函数，则在虚表中覆盖原本基类的那个虚函数指针，在编译时根据类的声明创建

## 虚继承

* 虚继承用于解决多继承条件下的菱形继承问题（浪费存储空间、存在二义性）
* 底层实现原理与编译器相关，一般通过**虚基类指针**和**虚基类表**实现，**每个虚继承的子类都有一个虚基类指针**（占用一个指针的存储空间，4/8字节）和**虚基类表**（不占用类对象的存储空间）（需要强调的是，虚基类依旧会在子类里面存在拷贝，只是仅仅最多存在一份而已，并不是不在子类里面了）；当虚继承的子类被当做父类继承时，虚基类指针也会被继承
* 实际上，**vbptr 指的是虚基类表指针**（virtual base table pointer），该指针指向了一个**虚基类表**（virtual table），**虚表中记录了虚基类与本类的偏移地址**；通过偏移地址，这样就找到了虚基类成员，而虚继承也不用像普通多继承那样维持着公共基类（虚基类）的两份同样的拷贝，节省了存储空间

## 虚继承、虚函数区别

- 相同之处：都利用了虚指针（均占用类的存储空间）和虚表（均不占用类的存储空间）

- 不同之处：

- - 虚函数不占用存储空间
  - 虚函数表存储的是虚函数地址
  - 虚基类依旧存在继承类中，只占用存储空间
  - 虚基类表存储的是虚基类相对直接继承类的偏移
  - 虚继承
  - 虚函数

## 构造函数或析构函数调用虚函数

* 简要结论： 
  1. **从语法上讲，调用完全没有问题**
  2. 但是从效果上看，往往不能达到需要的目的
* Effective 的解释是： 
  * 派生类对象构造期间进入基类的构造函数时，对象类型变成了基类类型，而不是派生类类型
  * 同样，进入基类析构函数时，对象也是基类类型

# C++11

## 原子变量介绍

* 解决多线程下共享变量的问题(i++，指令重排问题)：对于共享变量的访问进行加锁，加锁可以保证对临界区的互斥访问，
* C++11提供了一些原子变量与原子操作解决用户加锁操作麻烦或者容易出错的问题
* C++11标准在标准库atomic头文件提供了模版std::atomic<>来定义原子量，而对于大部分内建类型，C++11提供了一些特化，如，std::atomic_int (std::atomic<int>)等
* 自定义类型变成原子变量的条件是该类型必须为**Trivially Copyable类型**(简单的判断标准就是这个类型可以用std::memcpy按位复制)
* atomic有一个成员函数is_lock_free，这个成员函数可以告诉我们到底这个类型的原子量是使用了原子CPU指令实现了无锁化，还是依然使用的加锁的方式来实现原子操作

## [内存序](https://www.cnblogs.com/FateTHarlaown/p/8919235.html)

## C++11新特性

* auto关键字：编译器可以根据初始值自动推导出类型。但是不能用于函数传参以及数组类型的推导

* nullptr关键字：nullptr是一种特殊类型的字面值，它可以被转换成任意其它的指针类型；而NULL一般被宏定义为0，在遇到重载时可能会出现问题

* 智能指针：C++11新增了std::shared_ptr、std::unique_ptr、std::weak_ptr智能指针，用于解决内存管理的问题

* 初始化列表：使用初始化列表来对类进行初始化

* 右值引用：基于右值引用可以实现移动语义和完美转发，消除两个对象交互时不必要的对象拷贝，节省运算存储资源，提高效率

  * 右值引用，就是对一个右值进行引用的类型。由于右值通常不具有名字，不能出现在“=”号的左边，所以我们一般只能通过右值表达式获得其引用，比如，T && a = ReturnRvale();

* 移动语义：std::move

  * 对于一个包含指针成员变量的类，由于编译器默认的拷贝构造函数/拷贝赋值函数都是浅拷贝，所有我们一般需要通过实现深拷贝的拷贝构造函数/拷贝赋值函数，为指针成员分配新的内存并进行内容拷贝，从而避免悬挂指针的问题
  * 深拷贝需要重复申请和释放内存，在申请内存较大时会严重影响性能。因此C++通过定义移动构造函数/移动赋值函数，配合std::move是将对象的状态或者所有权从一个对象转移到另一个对象，只是转移，没有内存的搬迁或者内存拷贝所以可以提高利用效率，改善性能

* 完美转发：std::forward

  * https://www.jianshu.com/p/b90d1091a4ff

* atomic原子操作：用于多线程资源互斥操作

* Lambda表达式：定义一个**匿名函数**，并且**可以捕获一定范围内的变量**，其定义为

  ```c++
  [capture] (params) mutable->return-type {statement};
  ```

  其中，

  * [capture]：捕获列表，捕获上下文变量以供lambda使用。同时[]**是Lambda引出符**，编译器根据该符号来判断接下来代码是否是Lambda函数
  * (Params)：参数列表，与普通函数的参数列表一致，如果不需要传递参数，则可以连通括号一起省略。
  * mutable是修饰符，默认情况下lambda函数总是一个const函数，Mutable可以取消其常量性。在使用该修饰符时，参数列表不可省略
  * ->return-type:返回类型是返回值类型
  * {statement}：函数体，内容与普通函数一样，除了可以使用参数之外，还可以使用所捕获的变量。
  * Lambda表达式与普通函数最大的区别就是其可以通过捕获列表访问一些上下文中的数据。
  * Lambda的类型被定义为“闭包”的类，其通常用于STL库中，在某些场景下可用于简化仿函数的使用，同时Lambda作为局部函数，也会提高复杂代码的开发加速，轻松在函数内重用代码，无须费心设计接口。

* delete关键字：用于定义删除的构造函数和拷贝赋值函数

* default关键字：用于显式定义默认构造函数和拷贝赋值函数

* std::bind：函数适配器，它接受一个可调用对象，生成一个新的可调用对象来“适应”原对象的参数列表。主要有两个作用：

  * 将可调用对象和其参数绑定为一个仿函数

  * 可以只绑定部分参数，减少可调用对象传入的参数。如将binary_function转为unary_function：

    ```
    std::bind(bf, std::placeholders::_1, param2)
    或
    std::bind(bf, param1, std::placeholders::_2)
    ```

* std::function：是一个**可调用对象包装器**，是一个类模板，**可以容纳除了类成员函数指针之外的所有可调用对象，它可以用统一的方式处理函数、函数对象、函数指针，并允许保存和延迟它们的执行**

  * 定义格式：std::function<函数类型>，如std::function<void()>，std::function<int (int, int)>等
  * std::function**可以取代函数指针的作用，因为它可以延迟函数的执行，特别适合作为回调函数使用**。它比普通函数指针更加的灵活和便利

* 四个类型转换函数的区别

  * reinterpret_cast：可以**用于任意类型的指针之间的转换**，对转换的结果不做任何保证
  * dynamic_cast：这种其实也是不被推荐使用的，更多使用static_cast，**dynamic本身只能用于存在虚函数的父子关系的强制类型转换**，**对于指针，转换失败则返回nullptr，对于引用，转换失败会抛出异常**
  * const_cast：**对于未定义const版本的成员函数，我们通常需要使用const_cast来去除const引用对象的const，完成函数调用**。另外一种使用方式，**结合static_cast，可以在非const版本的成员函数内添加const，调用完const版本的成员函数后，再使用const_cast去除const限定**
  * static_cast：完成基础数据类型；同一个继承体系中类型的转换；任意类型与空指针类型void* 之间的转换


## 智能指针

* shared_ptr实现共享式拥有概念。多个智能指针可以指向相同对象，该对象和其相关资源会在“最后一个引用被销毁”时候释放。使用计数机制来表明资源被几个指针共享。可以通过成员函数use_count()来查看资源的所有者个数

* unique_ptr要求更严格，实现的是独占式拥有或严格拥有概念

* weak_ptr 是一种**不控制对象生命周期的智能指针, 指向一个 shared_ptr 管理的对象**。weak_ptr 设计的目的是为配合 shared_ptr 而引入的一种智能指针来协助 shared_ptr 工作, 它只可以从一个 shared_ptr 或另一个 weak_ptr 对象构造, 它的构造和析构不会引起引用记数的增加或减少。**weak_ptr是用来解决shared_ptr相互引用时的死锁问题，如果说两个shared_ptr相互引用，那么这两个指针的引用计数永远不可能下降为0，资源永远不会释放**。**它是对对象的一种弱引用，不会增加对象的引用计数**，和shared_ptr之间可以相互转化，shared_ptr可以直接赋值给它，它可以通过调用lock函数来获得shared_ptr

* **scoped_ptr**是一个类似于auto_ptr的智能指针，它包装了new操作符在堆上分配的动态对象，能够保证动态创建的对象在任何时候都可以被正确的删除。但是**scoped_ptr的所有权更加严格，不能转让**，一旦scoped_ptr获取了对象的管理权，你就**无法再从它那里取回来**

* 引用计数类实现

  ```c++
  class counter{
  public:
      counter():s_count(0), w_count(0){}
      int s_count;    // shared_ptr引用计数
      int w_count;    // weak_ptr引用计数
  }
  ```

* shared_ptr简单实现

  ```c++
  template<typename T>
  class shared_ptr{
  public:
      shared_ptr(T * p = nullptr): ptr(p){
          cnt = new counter();
          if(p != nullptr)
              cnt->s_count = 1;
      }
  
      shared_ptr(const shared_ptr<T> & sp): ptr(sp.ptr), cnt(sp.cnt) {
          sp.cnt->s_count++;
      }
  
      shared_ptr(const weak_ptr<T> & wp): ptr(wp.ptr), cnt(wp.cnt) {
          wp.cnt->s_count++;
      }
  
      ~shared_ptr(){
          release();
      }
  
      int use_count() const {
          if(cnt == nullptr)
              return 0;
          return cnt->s_count;
      }
  
      bool unique() const {
          if(cnt == nullptr)
              return 0;
          return cnt->s_count == 1;
      }
  
      void reset(T * p = nullptr){
          if(this.ptr != p){
              release();
              ptr = p;
              if(p != nullptr){
                  cnt = new counter();
                  cnt -> s_count = 1;
              }
          }
      }
  
      shared_ptr<T> & operator = (const shared_ptr<T> & sp){
          if(this != & sp){
              release();
              ptr = sp.ptr;
              sp.cnt->s_count++;
              cnt = sp.cnt;
          }
          return *this;
      }
  
      T& operator * (){
          return *ptr;
      }
  
      T* operator ->(){
          return ptr;
      }
  
      operator bool(){
          return ptr;
      }
  
      friend class weak_ptr<T>;
  private:
      void release(){
          cnt->s_count--;
          if(cnt->s_count < 1){
              delete ptr;
              ptr = nullptr;
              if(cnt->w_count < 1){
                  delete cnt;
                  cnt = nullptr;
              }
          }
      }
  
  private:
      counter *cnt;
      T * ptr;
  }
  ```

* weak_ptr简单实现

  ```c++
  template<typename T>
  class weak_ptr{
  public:
      weak_ptr(): ptr(nullptr), cnt(nullptr) {
      }
  
      weak_ptr(const shared_ptr<T> & sp): ptr(sp.ptr), cnt(sp.cnt){
          sp.cnt->w_count++;
      }
  
      weak_ptr(const weak_ptr<T> & wp): ptr(wp.ptr), cnt(wp.cnt){
          wp.cnt->s_count++;
      }
  
      ~weak_ptr(){
          release();
      }
  
      int use_count() const {
          if(cnt == nullptr)
              return 0;
          return cnt->s_count;
      }
  
      bool expired() const {
          if(cnt != nullptr){
              return cnt->s_count < 1;
          }
          return true;
      }
  
      shared_ptr<T> lock(){
          return shared_ptr(*this);
      }
  
      void reset(){
          release();
          ptr = nullptr;
          cnt = nullptr;
      }
  
      weak_ptr<T> & operator = (const shared_ptr<T> & sp){
          release();
          cnt = sp.cnt;
          sp.cnt->w_count++;
          ptr = sp.ptr;
      }
  
      weak_ptr<T> & operator = (const weak_ptr<T> & wp){
          if(this != &wp){
              release();
              cnt = wp.cnt;
              wp.cnt->w_count++;
              ptr = wp.ptr;
              
          }
          return *this;
      }
  
      friend class shared_ptr<T>;
  private:
      void release(){
          if(cnt!=nullptr){
              cnt->w_count--;
              if(cnt->w_count < 1 && cnt->s_count < 1){
                  cnt = nullptr;
              }
          }
      }
  
  private:
      counter *cnt;
      T * ptr;
  }
  ```

* unique_ptr简单实现

  ```c++
  template<typename T>
  class unique_ptr{
  public:
      unique_ptr(T* p = nullptr): ptr(p){
      }
  
      unique_ptr(unique_ptr<T> && up): ptr(up.ptr){
          up.ptr = nullptr;
      }
  
      ~unique_ptr(){
          reset();
      }
  
      T* release(){
          T* tmp = ptr;
          ptr = nullptr;
          return tmp;
      }
  
      void reset(T* p = nullptr){
          if(ptr!=nullptr)
              delete ptr;
          ptr = p;
      }
  
      unique_ptr<T> & operator = (unique_ptr<T> && up){
          if(ptr != up.ptr){
              if(ptr != nullptr)
                  delete ptr;
              ptr = up.ptr;
              up.ptr = nullptr;
          }
      }
  
      T& operator * (){
          return *ptr;
      }
  
      T* operator ->(){
          return ptr;
      }
  
      operator bool(){
          return ptr;
      }
  
      unique_ptr(const unique_ptr<T> & up) = delete;
      unique_ptr<T> & operator = (const unique_ptr<T> & up) = delete;
  private:
      T * ptr;
  }
  ```

## shared_ptr要点

* https://www.cnblogs.com/yahoo17/p/12862958.html

* 对于std::shared_ptr，即便父类析构函数并非虚函数，其子类的析构函数依然可以被调用，因为其 ref_count 记住了子类的实际类型

* 它借由模板构造函数窃取到了实际的指针信息，即子类的信息

  ```c++
  template <>
  class _Ptr_base<B> {
      //...
  
  private:
      B* _Ptr;
      _Ref_count_base* _Rep;
  }
  
  template <>
  class _Ref_count<D> : public _Ref_count_base {
  public:
      explicit _Ref_count(D* _Px) : _Ref_count_base(), _Ptr(_Px) {}
  
  private:
      // 最后析构函数会调用到这里
      virtual void _Destroy() noexcept override {
          delete _Ptr;
      }
  
      D* _Ptr;
  };
  
  template <>
  class shared_ptr<B> : public _Ptr_base<B> {
      explicit shared_ptr(D* _Px) {
          _Ptr = _Px; // _Ptr 静态类型是 B，但保存 new D() 的裸指针
          _Rep = new _Ref_count<D>(_Px) // _Ref_count 是 _Ref_count_base 的子类，该类保存了一个指向 D 的指针
      }
  }
  
  
  std::shared_ptr<Base> ptr(new Derived());
  
  // 如下图，ref_count中的ptr指针在初始化时，是根据模板推导出的Derived类型
  // 最后share_ptr析构的时候，是直接delete ptr，而ptr是子类类型，所以会调用子类的析构函数
  ```

  <img src="imgs/c++/shared_ptr.png" alt="shared_ptr" style="zoom:67%;" />

## enable_share_from_this

* C++11开始时支持 `enable_shared_from_this`，它一个模板类，定义在头文件 `<memory>`，其原型为： 

  ```c++
  // <memory>
  
  template< class T > 
  class enable_shared_from_this;
  ```

* std::enable_shared_from_this 能让其一个对象（假设其名为 t ，且已被一个 std::shared_ptr 对象 pt 管理）**安全地生成其他额外的** std::shared_ptr 实例（假设名为 pt1, pt2, … ） ，**它们与 pt 共享对象 t 的所有权**

* 若一个类 T **继承** std::enable_shared_from_this ，**则会为该类 T 提供成员函数**： shared_from_this 。 当 T 类型对象 t 被一个为名为 pt 的 std::shared_ptr 类对象管理时，调用 T::shared_from_this 成员函数，将会返回一个新的 std::shared_ptr 对象，它**与 pt 共享 t 的所有权**

* 为什么要用 `enable_shared_from_this`？

  * **需要在类对象的内部中获得一个指向当前对象的 shared_ptr 对象**
  * 如果在一个程序中，对象内存的生命周期全部由智能指针来管理。在这种情况下，**要在一个类的成员函数中，对外部返回this指针就成了一个很棘手的问题**

* 什么时候用？

  - 当一个类被 `share_ptr` 管理，且在**类的成员函数里需要把当前类对象作为参数传给其他函数时**，这时就需要传递一个指向自身的 `share_ptr`

* 如何安全地将 this 指针返回给调用者?

  - **一般来说，我们不能直接将this指针返回**。**如果函数将 this 指针返回到外部某个变量保存，然后这个对象自身已经析构了，但外部变量并不知道，此时如果外部变量再使用这个指针，就会使得程序崩溃**

    ```c++
    class obj {
    public:
    	// 这样做语法上没有问题，但是破坏了智能指针的语义了，使用了原生指针
        obj* getRawThis() {
            return this;  
        }
    	
        // 这样做程序会崩掉，这个临时变量在函数完毕后会被析构掉，会执行delete this
        shared_ptr<obj> getSharedCopy() {
            return shared_ptr<obj>(this); 
        }
    	
        // 这样做就更憨了，obj对象本身持有了包覆自己this指针的shared_ptr，
        // 那么他们的生命周期是“同步”的，这样被管理的obj永远不会释放
        shared_ptr<obj>& getSharedRef() {
            if(p == nullptr) {
                p = new shared_ptr<obj>(this);
            }
            return *p;   
        }
    
        // 当然，从上面也可以看出，实际上核心的问题，就是：
        // 1. 如果采用返回拷贝，那么临时变量会把this析构掉
        // 2. 如果采用返回引用，那么obj对象本身必须持有一个管理他的shared_ptr，这样会导致obj永远不会被释放
        // 可用的解决方案有以下两种
    
    
        void getSharedFromParam(shared_ptr<obj>& p) {
            p.reset(this);    // 通过函数参数来实现规避掉拷贝语义，但是这样太过于丑陋
        }
    	
        // 这是C++11推荐的方法，可以实现安全的返回一个shared_ptr<this>，
        // 返回到外面的shared_ptr的拷贝的use_count是1，不用担心this被意外delete和引用计数异常的问题
        shared_ptr<obj> getSharedFromThis() {
            return shared_from_this();
        }
    private:
        shared_ptr<obj> *p;
    };
    ```

* enable_shared_from_this类中的成员函数

  ```c++
  (constructor)：构造一个 enable_shared_from_this 对象，是一个受保护的成员函数，成员属性为 protected
  (destructor)：销毁一个 enable_shared_from_this 对象，是一个受保护的成员函数，成员属性为 protected
  operator=：返回到 this 的引用，是一个受保护成员函数，成员属性为 protected
  shared_from_this：返回共享 *this 指针所有权的 shared_ptr，是一个 public 属性的成员函数。
  weak_from_this(C++17)：返回共享 *this 所指针有权的 weak_ptr，是一个public 属性的成员函数
  ```

* 源码及存在问题

  * 原理分析：enable_shared_from_this类中使用了一个私有weak_ptr来保存地址，在shared_from_this()通过把这个weak_ptr转换为引用计数为1的shared_ptr返回，避免了临时shared_ptr析构导致this被delete

  ```c++
  // 源码
  template<typename _Tp>
  class enable_shared_from_this
  {
  protected:  //在public继承后，仍对子类可见
      enable_shared_from_this() noexcept { }
      ~enable_shared_from_this() { }
  
      //...
  
  public:
      shared_ptr<_Tp>
          shared_from_this()
      { return shared_ptr<_Tp>(this->_M_weak_this); }
  
      //...
  
  private:
      mutable weak_ptr<_Tp>  _M_weak_this;  //const函数也可以修改该变量
  
      //  _M_weak_assign函数在类型T被包覆在shared_ptr的过程中会被调用
      template<typename _Tp1>
      void _M_weak_assign(_Tp1* __p, const __shared_count<>& __n) const noexcept
      { _M_weak_this._M_assign(__p, __n); }
  };
  
  // 其中_M_assign()是weak_ptr的父类__weak_ptr的成员函数
  // _M_assign()的源码如下
  void  _M_assign(_Tp* __ptr, const __shared_count<_Lp>& __refcount) noexcept
  {
      if (use_count() == 0)
      {
          _M_ptr = __ptr;
          _M_refcount = __refcount;
      }
  }
  // _M_assign()的作用是为指针赋地址和引用计数（当然引用计数对weak_ptr没什么意义）
  // 所以，_M_weak_assign这个会在智能指针构造时会被调用的函数，
  // 是为了为从enable_shared_from_this中继承得来的_M_weak_this赋地址！
  
  auto p = new obj();
  p->getSharedFromThis()->doSomething(); // 这里会报一个异常：bad_weak_ptr，坑
  
  /*
  原因在于没有把obj交给shared_ptr管理起来而是直接使用了原生指针
  所以导致了 _M_weak_this没有初始化！
  进而在调用shared_from_this()时，把一个空weak_ptr转换为shared_ptr导致了异常。
  同理，也不能将继承了enable_shared_from_this的对象创建在栈上。
  这是STL在提醒我们，不要破坏智能指针的语义！
  */
  ```

* 使用示例

  ```c++
  #include <iostream>
  #include <stdlib.h>
  #include <memory>
  using namespace std;
  
  // 比较推荐的写法
  struct Good : std::enable_shared_from_this<Good> // note: public inheritance
  {
      std::shared_ptr<Good> getptr() {
          return shared_from_this();
      }
  };
  
  // 错误的用法：用不安全的表达式试图获得 this 的 shared_ptr 对象
  struct Bad
  {
      std::shared_ptr<Bad> getptr() {
          return std::shared_ptr<Bad>(this);
      }
      ~Bad() { std::cout << "Bad::~Bad() called\n"; }
  };
   
  int main()
  {
      // 正确的用法: 两个 shared_ptr 共享同一个对象
      std::shared_ptr<Good> gp1 = std::make_shared<Good>();
      std::shared_ptr<Good> gp2 = gp1->getptr();
      std::cout << "gp2.use_count() = " << gp2.use_count() << '\n';
   
      // 错误的用法: 调用 shared_from_this 但其没有被 std::shared_ptr 占有 
      try {
          Good not_so_good;
          std::shared_ptr<Good> gp1 = not_so_good.getptr();
      } 
      catch(std::bad_weak_ptr& e) {
          // 在 C++17 之前，编译器不能捕获 enable_shared_from_this 抛出的std::bad_weak_ptr 异常
          // 这是在C++17之后才有的特性
          std::cout << e.what() << '\n';    
      }
   
      // 错误的用法，每个 shared_ptr 都认为自己是对象的唯一拥有者
      // 调用错误的用法，会导致两次析构 Bad的对象，第二次析构时，指针指向的空间已经被析构，
      // 会导致程序出错
      std::shared_ptr<Bad> bp1 = std::make_shared<Bad>();
      std::shared_ptr<Bad> bp2 = bp1->getptr();
      std::cout << "bp2.use_count() = " << bp2.use_count() << '\n';
  
      return 0;
  }  
  ```

* 使用注意事项
  * enable_shared_from_this 的常见实现为：**其内部保存着一个对 this 的弱引用**（例如 std::weak_ptr )。 std::shared_ptr 的构造函数检测无歧义且可访问的 (C++17 起) enable_shared_from_this 基类，并且若内部存储的弱引用没有被以存在的 std::shared_ptr 占有，则 (C++17 起)赋值新建的 std::shared_ptr 为内部存储的弱引用。为另一个 std::shared_ptr 所管理的对象构造一个 std::shared_ptr ，将不会考虑内部存储的弱引用，从而将导致未定义行为(undefined behavior)。
  * **只允许在先前已被std::shared_ptr 管理的对象上调用 shared_from_this** 。否则调用行为未定义 (C++17 前)抛出 std::bad_weak_ptr 异常（通过 shared_ptr 从默认构造的 weak_this 的构造函数） (自C++17 起)。
  * enable_shared_from_this 提供安全的替用方案，以替代 std::shared_ptr(this) 这样的表达式（这种不安全的表达式可能会导致 this 被多个互不知晓的所有者析构）

# 模板

* 模板类中可以使用虚函数，而**模板成员函数不能是虚函数**，原因是：
  * **编译器都期望在处理类的定义的时候就能确定这个类的虚函数表的大小，如果允许有类的虚成员模板函数，那么就必须要求编译器提前知道程序中所有对该类的的该虚成员模板函数的调用，而这是不可能的**

# STL

* 请你讲讲STL有什么基本组成
  * STL主要包含六大部件：
    * 容器
    * 迭代器
    * 仿函数
    * 算法
    * 分配器
    * 配接器(适配器)
  * 他们之间的关系：分配器给容器分配存储空间，算法通过迭代器获取容器中的内容，仿函数可以协助算法完成各种操作，配接器用来套接适配仿函数
* 请你回答一下STL里vector的resize和reserve的区别
  * resize()：改变当前容器内含有元素的数量size，eg: vector\<int\>v；v.resize(len)；v的size变为len，**如果原来v的size小于len，那么容器新增（len-size）个元素，元素的值为默认为0**。当v.push_back(3);之后，则是3是放在了v的末尾，即下标为len，此时容器是size为len+1；**如果原来v的size大于len，resize会移除那些超出len的元素同时销毁他们**
  * reserve()：改变当前容器的最大容量（capacity），**它不会生成元素**，只是确定这个容器允许放入多少对象，如果reserve(len)的len值大于当前的capacity，那么会重新分配一块能存len个对象的空间，然后把之前v.size()个对象通过copy construtor复制过来，销毁之前的内存。len值小于当前capacity，不做处理
  * **resize既分配了空间，也创建了对象，可以通过下标访问。resize既修改capacity大小，也修改size大小**
  * **reserve只修改capacity大小，不创建对象，push或insert时才创建，不修改size大小**
* 请你来说一下map和set有什么区别，分别又是怎么实现的？

  * map和set都是C++的关联容器，其底层实现都是红黑树（RB-Tree）。由于 map 和set所开放的各种操作接口，RB-tree 也都提供了，所以几乎所有的 map 和set的操作行为，都只是转调 RB-tree 的操作行为。

  * map和set区别在于：

    * map中的元素是key-value（关键字—值）对：关键字起到索引的作用，值则表示与索引相关联的数据；set与之相对就是关键字的简单集合，set中每个元素只包含一个关键字。
    * **set的迭代器是const的，不允许修改元素的值；map允许修改value，但不允许修改key**。其原因是因为map和set是**根据关键字排序来保证其有序性**的，**如果允许修改key的话，那么首先需要删除该键，然后调节平衡，再插入修改后的键值，调节平衡，如此一来，严重破坏了map和set的结构，导致iterator失效，不知道应该指向改变前的位置，还是指向改变后的位置**。所以STL中将set的迭代器设置成const，不允许修改迭代器的值；而map的迭代器则不允许修改key值，允许修改value值。
    * map支持下标操作，set不支持下标操作。map可以用key做下标，map的下标运算符[ ]将关键码作为下标去执行查找，如果关键码不存在，则插入一个具有该关键码和mapped_type类型默认值的元素至map中，因此下标运算符[ ]在map应用中需要慎用，const_map不能用，只希望确定某一个关键值是否存在而不希望插入元素时也不应该使用，mapped_type类型没有默认值也不应该使用。如果find能解决需要，尽可能用find
* functional头文件支持的基于模板的比较函数对象
  * equal_to<Type>、not_equal_to<Type>、greater<Type>、greater_equal<Type>、less<Type>、less_equal<Type>
  * 常用的有greater <Type>()从大到小排序，less <Type>()从小到大排序
* STL中容器的自定义排序
  * 规则记忆：**前面小于后面的，那么sort的话就是上升的。priority_queue就是大根堆，因为优先队列的首指针是指向队尾的**
* priority_queue自定义排序要点：
  * https://blog.csdn.net/imred/article/details/108478144