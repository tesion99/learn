# C中部分宏及编译指令用法
### 内存对齐
内存对齐大致有2种方式

1. pragma pack

pragma pack编译指令使用语法
```
语法：
#pragma pack( [show] | [push | pop] [, identifier], n)

说明:
1. pack提供数据声明级别的控制, 对定义不起作用
2. 调用pack时不指定参数, n将被设成默认值
3. 一旦改变数据类型的alignment, 直接效果就是占用memory的减少, 但是performance会下降

语法具体分析:
    show：可选参数, 显示当前packing aligment的字节数, 
          以warning message的形式被显示, 很多编译器并未实现该功能

    push：可选参数, 将当前指定的packing alignment数值进行压栈操作, 
          这里的栈是the internal compiler stack, 
          同时设置当前的packing alignment为n, 
          如果n没有指定, 则将当前的packing alignment数值压栈

    pop：可选参数, 从internal compiler stack中删除最顶端的record;
         如果没有指定n, 则当前栈顶record即为新的packing alignment数值
         如果指定了n, 则n将成为新的packing aligment数值;
         如果指定了identifier, 则internal compiler stack中的record都将被pop
         直到identifier被找到, 然后pop出identitier,
         同时设置packing alignment数值为当前栈顶的record;
         如果指定的identifier并不存在于internal compiler stack, 则pop操作被忽略

    identifier: 可选参数, 当同push一起使用时,赋予当前被压入栈中的record一个名称;
        当同pop一起使用时, 从internal compiler stack中pop出所有的record,
        直到identifier被pop出,
        如果identifier没有被找到，则忽略pop操作

    n: 可选参数, 指定packing的数值, 以字节为单位
```

```cpp
// 方式一
#pragma pack(1) // 按1字节对齐
struct S {
    double a;
    char b;
    int c;
};
#pragma pack() // 恢复默认对齐方式

// 方式二
#pragma pack(push) // 将之前的对齐方式入栈
#pragma pack(1) // 设置新的对齐方式
struct S {
    double a;
    char b;
    int c;
};
#pragma pack(pop) // 回执push之前的对齐方式

// 方式二可简写为如下形式
#pragma pack(push, 4)
struct S {
    double a;
    char b;
    int c;
};
#pragma pack(pop)

```

2. 使用__attribute__

    __attribute是一个编译属性, 可以设置函数属性、变量属性、类型属性.

    这里使用__attribute__设置类型的对齐方式

```cpp
// 方式一
struct S {
    double a;
    char b;
    int c;
} __attribute__((packed)); // 取消编译器对齐优化, 按实际占用字节对齐(即1字节对齐)

// 方式二
struct S {
    double a;
    char b;
    int c;
} __attribute__((align(n))); // 尽可能按照n字节对齐, 如果结构中有成员长度大于n, 则按照最大成员的长度对齐

```