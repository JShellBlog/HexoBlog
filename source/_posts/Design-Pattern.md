---
title: Design_Pattern
date: 2018-07-12 19:45:14
tags: 
 - bookmarks
 - program
categories: bookmarks
---

## 背景
![](https://gss1.bdstatic.com/9vo3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike272%2C5%2C5%2C272%2C90/sign=970418febc003af359b7d4325443ad39/4a36acaf2edda3ccdd0e40d70be93901203f925b.jpg)
设计模式(design pattern)， 是在这1994年，由 Erich Gamma、Richard Helm、Ralph Johnson 和 John Vlissides 四人合著出版了一本名为 Design Patterns - Elements of Reusable Object-Oriented Software（中文译名：[设计模式 - 可复用的面向对象软件元素](https://baike.baidu.com/item/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%8F%AF%E5%A4%8D%E7%94%A8%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E8%BD%AF%E4%BB%B6%E7%9A%84%E5%9F%BA%E7%A1%80/7600072?fr=aladdin)） 的书提出。主要基于：
- 对接口编程而不是对实现编程
- 优先使用对象组合而不是继承

<!--more-->

## 1. 设计模式鸟览
常见设计模式有23种。大体可以分为三大类：
- 创建模式（Creational Patterns）
- 结构型模式(Structural Patterns)
- 行为型模式（Behavioral Patterns)

当然，还有J2EE设计模式。

模式 | 描述
:-: | :-
创建模式 | 这些设计模式提供了一种在创建对象的同时**隐藏创建逻辑**的方式，而不是使用 new 运算符直接实例化对象。这使得程序在判断针对某个给定实例需要创建哪些对象时更加灵活。
结构型模式 | 这些设计模式**关注类和对象的组合**。继承的概念被用来组合接口和定义组合对象**获得新功能**的方式。
行为型模式 | 这些设计模式特别**关注对象之间的通信**。
J2EE 模式 | 这些设计模式特别**关注表示层**。这些模式是由 Sun Java Center 鉴定的。

**创建模式（Creational Patterns）**

![Creational Patterns](https://raw.githubusercontent.com/JShell07/jshell07.github.io/master/images/DesignPattern/CreationalPattern.png)

**结构型模式(Structural Patterns)**

![Structural Patterns](https://raw.githubusercontent.com/JShell07/jshell07.github.io/master/images/DesignPattern/StructuralPattern.png)

**行为型模式（Behavioral Patterns)**

![Behavioral Patterns](https://raw.githubusercontent.com/JShell07/jshell07.github.io/master/images/DesignPattern/BehavioralPatternpng.png)

**J2EE设计模式**

![J2EE设计模式](https://raw.githubusercontent.com/JShell07/jshell07.github.io/master/images/DesignPattern/J2EEPattern.png)

## 2. 创建模式（Creational Patterns）
### 2.1. 工厂模式（Factory Pattern)
**目的**：定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。

**优点**: 扩展性高，并且屏蔽具体的实现。
**缺点**: 每增加一个产品时，都需要实现具体类和对象实现工厂。

```c++

class Shape
{
    public:
        virtual void draw() {};
};

class Circle: public Shape
{
    public:
        void draw() { /* ... */ };
};

class Square: public Shape
{
    public:
        void draw() { /* ... */ };
};

class ShapeFactory
{
    public:
    class Shape * GetShape(string type) {
        if(type.compare("CIRCLE") == 0)
            return new Circle();
        else if(type.compare("SQUARE") == 0)
            return new Square();
        else 
            return null;
    }
}

void main()
{
    class ShapeFactory shapeFactory = new ShapeFactory();

    class Shape *shape1, shape2;
    
    shape1 = shapeFactory.GetShape("CIRCLE");
    shape1->draw();

    shape2 = shapeFactory.GetShape("SQUARE");
    shape2->draw();

    delete shape1;
    delete shape2;
}
```

### 2.2. 抽象工厂模式（Abstract Factory Pattern)
**目的**：超级工厂又称为其他工厂的工厂，它提供了一种创建对象的最佳方式。

**优点**: 当一个产品族中的多个对象被设计成一起工作时，它能保证客户端始终只使用同一个产品族中的对象。
**缺点**: 产品族扩展困难。既要在抽象的creator 里修改，又要实习具体的代码。

虚基类的感觉。

```c++
class Shape
{
    public:
        virtual void draw() {};
};

class Circle: public Shape
{
    public:
        void draw() { /* ... */ };
};

class Square: public Shape
{
    public:
        void draw() { /* ... */ };
};

class Color 
{
    public:
        void fill();
};

class Blue: public Color
{
    public:
        void fill(){/* ... */}
};

class Red: public Color
{
    public:
        void fill(){/* ... */}
};

class AbstractFactory
{
    class Color* getColor(string color);
    class Shape* getShape(string shape);
};

class ShapeFactory: public AbstractFactory
{
    public:
    class Shape * getShape(string type) {
        if(type.compare("CIRCLE") == 0)
            return new Circle();
        else if(type.compare("SQUARE") == 0)
            return new Square();
        else 
            return null;
    }
    class Color* getColor(string color) {
        /* ... */
    }
}
```

## Ref.
http://www.runoob.com/design-pattern/design-pattern-tutorial.html