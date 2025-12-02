# Cpp学习记录

## 基础相关

**构造函数可以是虚函数吗，如果不可以，解释原因，顺便解释一下什么是虚函数表和虚函数指针？**

> 虚函数表，每一个类有一个，类的所有对象共享，存放在代码段和数据段中，虚函数指针，类的每一个对象有一个。
>
> `make_unique<T>` 是 c++14 新特性，`shared_mutex<T>` 是 c++17 新特性 ==> 这里面试中要回答，其实在项目中是有使用到 c++11 以上的新特性的。

**C++11中的智能指针？**

c++11中，智能指针包括`unique_ptr<T>`, `shared_ptr<T>`, `weak_ptr<T>`，三个智能指针的用法如下：

```c++

```

 **在c++中如何比较浮点数的大小？**

> 在计算机中，浮雕数采用 IEEE 754 标准表示，IEEE 754 标准表示的单精度浮点数如下：
>
> 1位符号位，8位指数位，23位尾数位。
>
> **常见浮点数陷阱**
>
> - **累积误差**：多次运算后误差累积；
>
> - **吸收现象**：大数加小数时小数被忽略；
>
> - **抵消现象**：相近数相减导致精度丢失；
>
> - **非结合性**：`(a+b)+c != a+(b+c)`。
>
> **解决方法**
>
> - 使用绝对误差比较法，适用于数值范围已知的情况；
>
> - 使用相对误差比较法，适用于数值范围变化较大的情况；
>- 使用ULP比较：基于浮点数表示的精度的比较方法；
> - 结合绝对和相对误差：综合两种方法的优势。

**C++中的四种显示类型转换运算符**

> **静态类型转换：`static_cast<T>`**
>
> ```c++
> // 1. 基本类型转换
> int a = 10;
> double b = static_cast<double>(a);
> 
> // 2. 指针类型转换（相关类型之间）
> class Base { virtual void foo() {} };
> class Derived : public Base {};
> 
> Base* base_ptr = new Derived();
> Derived* derived_ptr = static_cast<Derived*>(base_ptr);
> 
> // 3. 枚举和整型转换
> enum Color { RED, GREEN, BLUE };
> int color_value = static_cast<int>(RED);
> 
> // 4. 避免窄化转换警告
> float f = 1.5f;
> int i = static_cast<int>(f);  // 明确意图，抑制警告
> ```
>
> **动态类型转换：`dynamic_cast<T>`**
>
> ```c++
> // 1. 指针转换：失败返回nullptr(多态)
> Base* base = new Base();  // 实际是Base类型
> Derived* derived = dynamic_cast<Derived*>(base);  // 返回nullptr
> 
> // 2. 引用转换：失败抛出std::bad_cast异常
> try {
>     Derived& derived_ref = dynamic_cast<Derived&>(*base_ptr);
> } catch (const std::bad_cast& e) {
>     std::cout << "转换失败: " << e.what() << std::endl;
> }
> 
> // 3. 交叉转换（多重继承）
> class A { virtual ~A() {} };
> class B { virtual ~B() {} };
> class C : public A, public B {};
> 
> A* a_ptr = new C();
> B* b_ptr = dynamic_cast<B*>(a_ptr);  // 成功转换
> ```
>
> **常量性转换：`const_cast<T>`**
>
> ```c++
> // 正确的使用：修改原本不是const的对象
> int value = 100;
> const int& const_ref = value;  // 添加const限定
> int& mutable_ref = const_cast<int&>(const_ref);  // 去除const
> mutable_ref = 200;  // 安全：原对象value不是const
> 
> std::cout << value;  // 输出200
> ```
>
> **重新解释转换：`reinterpret_cast<T>`**
>
> ```c++
> // 最危险的类型转换 ==> 谨慎使用
> // 1. 函数指针转换
> typedef void (*FunctionPtr)();
> void my_function() { std::cout << "Hello\n"; }
> 
> FunctionPtr func = reinterpret_cast<FunctionPtr>(my_function);
> 
> // 2. 网络编程中的数据类型转换
> struct EthernetHeader {
>     uint8_t dest[6];
>     uint8_t src[6];
>     uint16_t type;
> };
> 
> void* packet_data = receive_packet();
> EthernetHeader* header = reinterpret_cast<EthernetHeader*>(packet_data);
> ```

## Cpp11线程同步汇总

