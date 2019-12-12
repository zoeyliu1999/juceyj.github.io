---
title: 'Intern'
date: 2019-05-25 20:09:53
layout: archive
author_profile: true
permalink: /blog/intern/
---

​	立这一帖主要还是因为自己太菜了，每一次面试的时候总会遇上自己不懂的题。

# 程序基础

- C++的虚函数和纯虚函数的区别？

  - C++中只有虚函数（加了virtual标志）才可以从父类中继承并且覆盖，否则就算子类中有何父类相同名相同参数的函数也会被忽视而执行父类中的函数。

  - 纯虚函数指的是抽象类的虚函数，抽象类在定义class前面加virtual

  - 抽象类不可以实例化，而不仅仅是没有意义

  ```c++
  class A{
  public:
  	void foo(){
  		printf("1\n");
  	}
  	virtual void fun(){
  		printf("2\n");
  	}
  };
  
  class B : public A{
  public:
  	void foo(){
  		printf("3\n");
  	}
  	void fun(){
  		printf("4\n");
  	}
  };
  int main(void){
  	A a;
  	B b;
  	A *p = &a;
  	p->foo();
  	p->fun();
  	p = &b;
  	p->foo();
  	p->fun();
  } // 输出1214
  ```



- 引用和指针的区别？

  - 指针本身是一个变量，是一个64bit = 8字节的变量
  - 引用就是变量的一个别名，调用的方式相同

  ```c++
  int a;
  int* b = &a;
  int &c = a;
  ```



# Algorithm

- 一个链表判断是否有环？环的长度？环的开始？
  - 快慢指针
  - 环的开始：假设慢指针走了a+x，快指针走了a+mb+x，其中a是入口距离，b是环的长度，x是相遇点的offset，因为2(a+x) = a+mb+x可以得到a = mb - x。所以只需要两个指针一个从相遇点一个从链表头出发一定会在入口相遇
  - 环上任意一点开始，走一圈都原来的地方就是环的长度

- 最大子矩阵和O(n^3)

  dp{m}{n}{t}，枚举两行m和n，然后把问题退化为一维的最大子序和问题

  ![img](https://kaichen1998.github.io/images/intern/1.png)

- 给定一个多项式，已知
  $$
  f(x) = a_0 + a_1x+...+a_nx^n
  $$

  - ai都是非负整数
  - n不知道多大
  - 唯一的操作：输入x，输出f(x)
  - 我要知道每一个ai
  - 如果ai有界，我们只需要带入一个比所有ai都大值m，再把输出转化为m进制表示即可。现在不知道ai上界，我们先令x=1得到ai的上界s，把s+1能保证这个值大于每一个ai，把s+1代入即可。