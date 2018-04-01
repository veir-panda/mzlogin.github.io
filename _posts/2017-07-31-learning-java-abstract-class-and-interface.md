---
layout: post
title: 巩固Java基础之抽象类和接口
categories: java
description: 巩固Java基础之抽象类和接口
keywords: java, 抽象类, 接口
---

# 巩固Java基础之抽象类和接口

## 抽象类

在面向对象的领域一切都是对象，所有的对象都是通过类来描述的。如果我们要定义的一个类没有足够的信息来描述一个具体的对象，还需要其他的具体类来支持，这个时候我们可以考虑使用抽象类。在类定义的前面增加abstract关键字，就表明一个类是抽象类。

抽象类除了不能实例化对象之外，类的其它功能依然存在，成员变量、成员方法和构造方法的访问方式和普通类一样。由于抽象类不能实例化对象，所以抽象类必须被继承，才能被使用。

我们看Graph的代码，我们如果将Graph定义为抽象类，代码如下：

```java
abstract class Graph {

    String name;

    public Graph(){}

    public Graph(String name) {
        this.name = name;
    }

    public void show() {
        System.out.println("I'm a graph");
    }
}
```

增加abstract修饰符之后，Graph就不能被实例化了，即无法通过new Graph()来创建对象。

abstract关键字同样可以用来声明抽象方法，抽象方法只包含一个方法名，而没有方法体。抽象方法没有定义，方法名后面直接跟一个分号，而不是花括号。声明抽象方法会带来以下两个结果：

如果一个类包含抽象方法，那么该类必须是抽象类。
任何子类必须重写父类的抽象方法，否则就必须声明自身为抽象类
一般情况下，我们将一个类声明为abstract的，是因为它包含了没有具体实现的抽象方法。比如说我们给Graph类增加一个求面积的方法area()，因为我们不知道图形的形状，我们是无法给出实现的，只能交给特定的子类去实现，这时我们只能讲area()声明为abstract的，代码如下：

```java
abstract class Graph {

    String name;

    public Graph(){}

    public Graph(String name) {
        this.name = name;
    }

    public void show() {
        System.out.println("I'm a graph");
    }

    public abstract double area();
}
```

这时Retangle类就必须给出area()方法的实现，否则它自己也必须用abstract修饰。

```java
class Retangle {

    ...

    public double area() {
        return width * height;
    }
}
```

## 接口

### 接口定义

与抽象类类似的一个重要概念是接口。接口（Interface）是一组抽象方法的集合。接口中定义的方法没有方法体，它们以分号结束。

接口也抽象类一样，无法被实例化，但是可以被实现。一个实现接口的类，必须实现接口内所描述的所有方法，否则就必须声明为抽象类。编写接口和编写类的方式是大体上是类似的，一个接口可以有多个方法，代码保存在以接口命名且以.java结尾的文件中。接口使用interface关键字进行定义。

比如我们定义一个Animal接口。

interface Animal {
    void eat();
    void sleep();
}
这个接口包含了两个抽象方法：eat()和sleep()。接口中的方法都是外部可访问的，因此我们可以不需要用public修饰。

接口中也可以声明变量，一般是final和static类型的，要以常量来初始化，实现接口的类不能改变接口中的变量。比如我们可以在Animal接口增加一个成员变量TIMES_OF_EATING，表示动物每天吃饭的次数。

interface Animal {

    static final TIMES_OF_EATING = 3;    

    void eat();
    void sleep();
}
接口访问权限有两种：public权限和默认权限，如果接口的访问权限是public的话，所有的方法和变量都是public。默认权限则同一个包内的类可以访问。

一个接口能继承另一个接口，和类之间的继承方式比较相似。接口的继承使用extends关键字，子接口继承父接口的方法。比如我们可以定义TerrestrialAnimal接口，表示陆栖动物，它继承自Animal接口，同时还具有run()方法。

interface TerrestrialAnimal extends Animal {  

    void run();
}

### 接口实现

类使用implements关键字实现接口。在类声明中，implements关键字放在class声明后面。接口支持多重继承，即一个类可以同时实现多个接口。

class Cat implements Animal {

    public void eat() {
        System.out.println("eat");    
    }
    public void sleep() {
        System.out.println("sleep");
    }
}
类需要对接口中的每一个方法都给出实现。

我们可以使用接口类型来声明一个变量，那么这个变量可以引用到一个实现该接口的对象。比如：

Animal cat = new Cat()
讲集合的时候，我们说过ArrayList是List接口的实现，HashMap是Map接口的实现，这回你应该明白了吧。通过接口来申明变量，可以让程序更具有扩展性，因为将来我们更方便替换接口的实现。

比如PostRepository的getAll方法可以返回List<Post>，而不需要指定为具体的ArrayList<Post>。这样将来如果我们希望返回LinkedList<Post>的时候也无需修改接口。

public class PostRepository {

    ...

    public static List<Post> getAll() {
        return null;
    }
}

## 回顾

我们基于Map实现了一个PostRepositoryByMap，这里为了统一我们不妨把PostRepository改名为PostRepositoryByList。

我们可以使用Eclipse的重构功能来进行类的改名操作。右键选中PostRepository.java文件，【Refactor】->【Rename】，输入新的名称即可。

我们发现PostRepositoryByMap和PostRepositoryByList公共方法的签名是完全一样的。我们可以将这些方法，提取到一个公共接口中来，让它们分别实现这个接口。我们将这个接口命名为PostRepository。

用于在文件中加载和保存博客信息的loadData()和save()方法也可以加入到这个接口中，最终PostRepository接口的代码如下：

package com.tianmaying.repository;

import java.util.List;

import com.tianmaying.domain.Post;

public interface PostRepository {

    public void add(Post post);

    public Post getPostById(long id);

    public void remove(long id);

    public List<Post> getAll();  

    public void loadData();

    public void saveData();  
}
注意这些方法都没有static修饰了，因为接口中的方法是不能为静态的。所以PostRepositoryByMap和PostRepositoryByList类让它们实现PostRepository接口的同时，方法的签名也需要做对应的修改。

public class PostRepositoryByMap implements PostRepository {

    public void add(Post post) {
        posts.add(post);
    }

    ...

    // 新增加的方法，先给出一个空的实现
    public void loadData() {

    }

    // 新增加的方法，先给出一个空的实现    
    public void saveData() {

    }

}
这样修改之后，我们可以通过PostRepository来声明变量，而选择哪个具体的实现类就具有更大的灵活性了。

## 抽象类和接口的比较

### 相同点：

都不能被实例化
都包含抽象方法，这些抽象方法用于描述系统能提供哪些服务，而这些服务是由子类来提供实现的
在系统设计上，两者都代表系统的抽象层，当一个系统使用一棵继承树上的类时，应该尽量把引用变量声明为继承树的上层抽象类型，这样可以提高两个系统之间的松耦合

### 不同点：

在抽象类中可以为部分方法提供默认的实现，从而避免在子类中重复实现它们；但是抽象类不支持多继承。接口不能提供任何方法的实现，但是支持多继承。
接口代表了接口定义者和接口实现者的一种契约；而抽象类和具体类一般而言是一种is-a的关系，即两者在概念本质上是相同的。

----

> 注：本文转载自[网站](https://www.tianmaying.com/tutorial/java-abstract-class-and-interface#rd?sukey=a76cdd086edb4fce9303ce6bf982b40e131d486fb5cb9f0ffbf41b6809e03092b61580f3af43498ed396c295020298cc)