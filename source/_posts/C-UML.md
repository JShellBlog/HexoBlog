---
title: C++_UML
date: 2017-12-14 17:03:51
tags:
	- c++
categories: c++	
---

C++ UML
---
# 1. Basic 
# 1.1. Visibility
The C++ class has three visibility. 

- public, using "+"  
- private, using "-"  
- protected, using '#'  

See the base class examole:  
![Base Class](https://raw.githubusercontent.com/JShell07/jshell07.github.io/master/images/C%2B%2B/C%2B%2B_BaseClass.png)

<!-- more -->
# 1.2. Class members
- static members, using <u>underline</u>, like <u>static int member</u>
- virtual functions, using **Italic** font, like ***virtual void functions()***

# 2. Class Relationship
The C++ has these relationship:

- Assocation 
- Dependency
- Aggregation
- Composition 
- Inheritance
- Class template  

![](https://raw.githubusercontent.com/JShell07/jshell07.github.io/master/images/C%2B%2B/C%2B%2B_ClassRelationship.png)

# 2.1. Assocation 
See the up picture, Class B has Class A's pointer or quote.
``` c++
class B
{
public:
    B(){}
    void setA(A *A) { ptrA_ = A;}

private/public:
    A* ptrA_; 
}
```
# 2.2. Dependency
Class B and Class A is indepent. And Class A is local variable or a function parameter in CLass B.
``` c++
class B {
	public:
		void display(class A* A) {A->display();}
}
```
# 2.3. Aggregation
Class B is the container to Class A. But Class A and Class B have diffrent lift, which like car and tire. The car maybe detoried, and tire can be still exist.
``` c++
class B {
	public:
	private:
		vector<class A*> ptrclassA;
}
```
# 2.4. Composition 
Compostion is stricter than Aggregation. They have the same life.
``` c++
class B {
	private:
		class A a;
}
```
# 2.5. Inheritance

``` c++
class A {
	//..	
};

class B: public A {
	//..
}
```

# 2.6. Class template

``` c++
template<class T>
class A {
	//...
	private:
	T var;
};

class B {
	//..
};

A<B> a;
```
