---
layout: post
title: "有关C++动态绑定的一个小问题"
date: 2017-10-14 21:01:07 +0800
categories: programming
---
很久以前还在学C++的面向对象机制的时候，有个小问题一直没有搞明白。正是由于搞明白其实不困难，因此堂而皇之拖到了今天:)

当时的问题是这样的：

> 在成员函数里调用成员函数（均是非静态），是否会发生动态绑定？

经验证得到的结论是：

> 会。

其实这个结论仔细想想，很自然且非常容易理解。在调用非静态成员函数时，编译器会隐式地将指向发起者的指针作为参数传给被调用函数，在被调用函数中使用this指针进行访问。如果这个过程就发生在非静态成员函数中，那么当前作用域中的this指针将被作为参数传入。

说得更直白一点吧，`foo();`等效于`this->foo();`，其形式和在类定义外部使用对象指针（引用）调用成员函数是一致的，动态绑定触发的条件亦如是：如果this指针是基类指针（当前函数定义在基类中），且调用的成员函数是虚函数，那么就会发生动态绑定。

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
