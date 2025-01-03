## 一、第一章&第二章 常见数据类型和运算

#### 1.1 cpp 中常见数据类型定义

```c++
void Demo1() {
    //通过查看内存可以知道，数据在内存中是从高地址到低地址分配空间的
    char a[10] = "a";
    short int s = 97;   //低地址存放低字节（低位），高地址存放高字节（高位），小端表示
    int m = 97;
    long int n = 077;   //八进制数初始化
    float f = 97.0f;   //float的初始化后面要加f
    wchar_t w[10] = L"a";   //宽字符数组的初始化，前面要加L
    cout << "Hello World! " << n << endl;
}
```

通过调试查看内存，函数栈帧中，内存分配是从高地址向低地址分配空间；此外，根据数据的组织形式，得到目前主流PC都是小端模式。

#### 1.2 模拟集合论中的德·摩根律（“||”， “&&”， “!”）

```c++
void Demo2() {
    bool a = 1;
    bool b = 0;
    cout << (!(a || b) == (!a && !b))<< endl;
    cout << (!(a && b) == (!a || !b)) << endl;

    //assert函数的使用，assert函数是断言操作，起到了测试的作用，当左边（输出值） == 右边（预期值）时，函数正常执行，否则报错
    assert(!(a || b) == (!a && !b));
    assert(!(a && b) == (!a || !b));
}

```

复习cpp中的逻辑运算符，逻辑运算符会发生短路；学习新的函数assert断言，起到了测试的作用，assert(false) 程序会报错，assert(true) 程序正常执行。

#### 1.3 cpp 中的位运算符('&', '|', '!', '^')

cpp 中有四个位运算符，位运算符有一些特殊的功能，比如(c ^ c)，c将被清零。

#### 1.4 特殊赋值运算符("<<", ">>")

C 语言移位操作有左移和右移，以右移操作为例，包括逻辑右移和算术右移，逻辑右移最高位补零，算术右移最高位补符号位（不同编译器有不同标准）；以左移操作为例，统一为逻辑左移，低位补零。有符号数的右移操作为算术右移，无符号数的右移为逻辑右移。

```c++
void Demo03() {
    char c = 0x99;      //1001 1001
    printf("%x\n", c >> 2);   //有符号数执行算术右移（高位补符号位）
    unsigned char b = c;
    printf("%x\n", b >> 2);    //无符号数执行逻辑右移（高位补0）
}
```

在 cpp 中，提供了 `bitset` 类进行移位操作，统一为逻辑移位，高位低位都补零。

```C++
#include<bitset>	//包含指定头文件
void Demo04() {
    bitset<8> a = 0xff;
    bitset<8> b = a >> 6;
    bitset<8> c = a << 8;
    cout << a << endl << b << endl << c << endl;
}
```

#### 1.5 一些不常见的杂项运算符

```c++
int a = 1, b = 2;
int c;   
c = (a , b);  //逗号运算符，只看末尾的值，此时c的值为2，结合括号使用
cout << sizeof(a) << endl;	//数据类型大小运算符
cout << (a < b ? true : false) << endl;	//三目运算符
```

## 二、第三章&第四章 C 语言常见陷阱 及 cpp 一些改进

#### 2.1 C语言字符串语法常见陷阱

在 cpp 中，封装了 string 类，极大的方便了对字符串的操作，下述例子显示了一些 C 语言中不常见的赋值操作：

```c++
void Demo01() {
    char a = 'yes'; 		//C语言处理该赋值会截断，保留最后一个字符
    cout << a << endl;
    const char* p = "y";    //字符指针指向字符串"y\0"的首地址

    string s1(1, 'yes');    //输出s
    string s2(3, 'yes');    //输出sss，学会使用构造函数赋值多个相同字符
    string s3(1, 'y');      //输出y
    string s4("/");         //输出/
    string s5("yes");       //输出yes
}
```

#### 2.2 C 语言指针与数组之间的关系

在C 语言中，对数组进行值传递的时候，数组变量会退化成指针变量，指向数组变量的首地址。

```C++
void Demo02() {
    int arr[] = { 1, 2, 3, 4, 5 };
    int len = sizeof(arr) / sizeof(arr[0]); //学会使用该技巧求整型数组长度
    vector<int> v = { 1, 2, 3 };   //vector可以用数组的初始化方法进行赋值
    vector<vector<int>> vv = {8, vector<int>(12,3)};  //初始化一个8行12列的二维数组
    cout << v.size() << endl;
}
```

#### 2.3 C 语言强制类型转换陷阱

​		在 C 语言中，对无符号数和有符号数进行比较时，有符号数会强制转换为无符号数（负数表示更大的无符号数），如下：

```c++
void Demo05() {
    int array[] = { 1,2,3 };
    cout << sizeof(array) / sizeof(array[0]) << endl;	//sizeof()返回无符号整数
    int threshold = -1;
    //threshold 强转为无符号整数
    if (sizeof(array) / sizeof(array[0]) > threshold) {
        cout << "positive number array. " << endl;
    }
    else {  //-1的补码值转换为无符号整数之后会大于3
        cout << "negative number array. " << endl;
    }
}
```

​		通过上述例子，在编程时尽量避免不同数据类型进行大小比较，防止程序被隐式的使用了强制类型转换。

#### 2.4 cpp 提供的四种强制类型转换

​		cpp 提供的四种强制类型转换为：`static_cast`，`const_cast`，`dynamic_cast`，`reinterpret_cast`，四种强制类型转换常见用法如下。
