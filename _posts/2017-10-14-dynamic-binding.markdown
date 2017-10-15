---
layout: post
title: "在成员函数里调用虚函数，是否会发生动态绑定？"
date: 2017-10-14 21:01:07 +0800
categories: programming
---
很久以前刚学C++的面向对象机制的时候，这个小问题一直没有搞明白。正是由于搞明白其实不困难，因此堂而皇之拖到了今天:)

经验证得到的结论是：

**会。**

其实这个结论仔细想想，很自然且是唯一合理的结论。在调用非静态成员函数时，编译器会将指向发起者的指针作为隐含的参数传给被调用函数，在被调用函数中使用this指针进行访问。如果这个过程就发生在一个非静态成员函数体中，那么当前作用域中的this指针将被作为隐含的参数传入。

在效果上，`foo();`和`this->foo();`是等价的（或者说C++只是在写法上允许省略`this->`而已），后者的形式和本质和在类外部使用对象指针（引用）调用成员函数是一致的，自然也遵循动态绑定触发的条件：如果this指针是基类指针（当前函数定义在基类中），且调用的成员函数是虚函数，那么就会发生动态绑定。

举个例子：

{% highlight C++ linenos %}
#include <iostream>
using namespace std;

class Base {
public:
    virtual void foo() {
        cout << "foo in base" << endl;
        bar();
    }
    virtual void bar() {
        cout << "bar in base" << endl;
    }
};

class Derivative: public Base {
public:
    void foo() {
        cout << "foo in derivative" << endl;
        bar();
    }
    void bar() {
        cout << "bar in derivative" << endl;
    }
}

int main() {
    Base *baseRef = new Derivative();
    baseRef->Base::foo();
    return 0;
}
{% endhighlight %}

程序运行的输出如下：

```
foo in base
bar in derivative
```

可以看出，我们显式地调用了基类中定义的`foo`成员函数，继而在调用`bar`成员函数时发生了动态绑定，从而调用了派生类定义的`bar`成员函数。

虽然没有显式地写成`基类指针->虚函数`的形式，但动态绑定还是会发生，这里也许会发生意想不到的疏漏。
