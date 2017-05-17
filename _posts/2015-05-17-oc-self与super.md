---
layout: post
title: oc中self与super的思考
data: 2017-05-17
categories: oc
tags: [oc]
description: oc中有关self super的思考

---

   self 
   是方法的隐藏的参数变量，指向当前调用方法的对象，另一个隐藏参数是 _cmd，代表当前类方法的selector。

   super 不是隐藏的参数，它只是一个"编译器指示符"。查找方法时，指定方法查找的位置在父类。

![the self ponit to](http://intfre.cn/img/my_blog/2017-05-17/self.png)

  　  面向对象过程中，子类继承父类，就拥有了父类所有的属性方法，一个完整的类的初始包  括子类初始化和父类初始化。
　　
  子类 [alloc init]后，首先这里只有一个对象实体self，没有所谓的父类对象实体sup   er。初始化过程中，父类属性、方法初始化都属于子类对象的一部分，super 的指针赋给 self 这一说法是错的，其实全部指的是该对象的初始位置。


	Class  A
	-reposition  
	{  
     	...  
     	[super setOrigin:someX :someY];  
     	...  
	}


   [a  reposition];   方法体中编译器将
    [super setOrigin:someX :someY]; 
   其转换为
    id objc_msgSendSuper(struct objc_super *super, SEL op, ...)
    方法体中编译器将方法体中编译器将第一个参数是个objc_super的结构体，第二个参数还是  类似上面的类方法的selector，先看下objc_super这个结构体是什么东西：
	struct objc_super {
   		id receiver;
   	Class superClass;
	};

    可以看到这个结构体包含了两个成员，一个是 receiver，这个类似上面 objc_msgSend 的第一个参数 receiver，第二个成员是记录写 super 这个类的父类是什么，拿上面的代码为例，当编译器遇到 A 里

[super setOrigin:someX :someY]时，开始做这几个事：
      
	 >构建 objc_super 的结构体，此时这个结构体的第一个成员变量 receiver 就是 a，和self 相同。而第二个成员变量 superClass 就是指类 A的 superClass。

     >调用 objc_msgSendSuper 的方法，将这个结构体和setOrigin的 sel 	传递过去。函数里面在做的事情类似这样：从 objc_super 结构体指向的 superClass 的方法列表开始找 setOrigin 的 selector，找到后再以 objc_super->receiver 去调用这个 selector，可能也会使用 objc_msgSend 这个函数，不过此时的第一个参数 theReceiver 就是 objc_super->receiver，第二个参数是从 objc_super->superClass 中找到的 selector
 
 
    为什么要 self =  [super init];
  符合oc 继承类 初始化规范 super 同样也是这样，  [super init]  去self的super中调用init super 调用 superSuper 的init 。直到根类 NSObject 中的init ,根类中init 负责初始化 内存区域 向里面添加一些必要的属性，返回内存指针， 这样 延着继承链 初始化的内存指针 被从上 到 下 传递，在不同的子类中向块内存添加 子类必要的属性，直到 我们的 A 类中 得到内存指针，赋值给slef 参数， 
  在if (slef){//添加A 的属性 }



总结:
1.self指定调用者也就是消息的接收者 receiver ，super 只指定selector的查找位置，也就是确定所要调用的方法，但方法中的self仍然是该消息的接收者receiver.如 子类与父类中都有属性 a,并在父类方法中使用了 self.a,此时的self.a 是消息接收者receiveer 的a.  
2.self  始终是指向当前方法调用的对象，在静态方法中指向的是类，动态方法中指向的是对象。
3.在方法中 self.a 是通过 a的属性函数来访问的.直接使用a则是直接访问。
用@property关键字来声明的属性，在编译期会默认生成一个下划线加名称的属性变量，并且自动在implemention文件中生成setter和getter方法。使用_yourName的方式是直接引用变量，而通过点语法调用self.yourName这种形式，实际是调用setter或getter方法(需要注意reain release)！