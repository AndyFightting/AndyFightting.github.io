---
layout: post
title: "设计模式"
date: 2015-12-19 21:51:06 +0800
comments: true
categories: other
---

我理解的设计模式都是为了让代码模块化，结构化，各司其职，再把各个模块通过"接口"组合起来。封装变化；多用组合少用继承；针对接口编程，不针对实现编程；为交互对象之间的松耦合设计而努力；对扩展开放，对修改关闭。


###  1.MVC模式
看图就可以了，简洁明了~
![image](/myimg/other/mvc.png)<!--more-->

###  2.策略模式

比如有个父类有飞行功能，然后有很多子类的飞行功能各不一样，如果去继承父类然后分别实现不同的飞行行为，这就坑爹了...具体做法应该是：把飞行行为完全独立出来成一个Interface，然后不同的飞行行为再在单独的类中实现，不同的子类要不同的飞行行为的话就取对应的行为实现对象。

父类：

	public class Duck {//父类
    
    public FlyInterface flyInterface;//行为的接口成员变量

    public void performFly(){//统一调用的飞行方法
        flyInterface.fly();
    }

    public void setFlyInterface(FlyInterface fi){//动态改变飞行行为
        flyInterface = fi;
      }
    }
    
行为接口：

	public interface FlyInterface { //行为接口
    void fly();//要实现的行为方法
    }

行为的一种具体实现：

	public class FlyPatternOne implements FlyInterface {//飞行行为的一种实现，可以有很多个不同的实现
    @Override
    public void fly() {
        //这里是具体飞行行为代码

      }
    }
    
某一个子类：
	
	public class SubDuck extends Duck{
    public SubDuck(){
        //默认的飞行行为，不同的子类有不同的飞行行为，如 FlyPatternTwo FlyPatternThree...等
        flyInterface = new FlyPatternOne();
      }
    }
    
![image](/myimg/other/chelue.png)

1,找出可能需要变化之处，把它们独立封装起来，不要和那些不需要变化的代码混合在一起。

2.针对接口编程，而不是针对实现编程。

3.多用组合，少用继承。

###  3.观察者模式

好好看图吧，把它转化为具体代码加深理解
![image](/myimg/other/guanCha0.jpg)
![image](/myimg/other/guanCha1.jpg)
![image](/myimg/other/guanCha2.jpg)
![image](/myimg/other/guanCha3.jpg)

看完上面例子，下面的设计图也更好理解了。
![image](/myimg/other/guanCha4.jpg)

使用Java内置的观察者模式
![image](/myimg/other/guanCha5.jpg)
![image](/myimg/other/guanCha6.jpg)
![image](/myimg/other/guanCha7.jpg)
![image](/myimg/other/guanCha8.jpg)

Swing中的观察者模式
![image](/myimg/other/guanCha9.jpg)

要点
![image](/myimg/other/guanCha10.jpg)

### 4.装饰模式
动态的将责任附加到对象上，用对象组合的方式来解决继承滥用的问题。利用继承设计子类的行为是在编译时静态决定的，而且所有的子类都会继承到相同的行为。如果利用组合的做法扩展对象的行为，就可以在运行时动态的进行扩展。类应该多扩展开放，对修改关闭。即在不修改现有代码的情况下可以添加新的行为。

装饰着和被装饰着必须是同一类型，所以通过继承来实现。这里的继承是用来达到“类型一致”的，而不是为了通过继承得到“行为”。“行为”来自装饰着和基础组件，其他与装饰着之间的组合关系。
![image](/myimg/other/zhuangShi.jpg)
![image](/myimg/other/zhuangShi0.jpg)
![image](/myimg/other/zhuangShi1.jpg)
![image](/myimg/other/zhuangShi2.jpg)
![image](/myimg/other/zhuangShi3.jpg)
![image](/myimg/other/zhuangShi4.jpg)
![image](/myimg/other/zhuangShi5.jpg)
![image](/myimg/other/zhuangShi6.jpg)

### 5.工厂模式

当每次“new”的时候都会实例化一个类，所以这是用的实现编程而不是接口编程。“工厂”就是一个专门负责产生对象的一个类。

    

