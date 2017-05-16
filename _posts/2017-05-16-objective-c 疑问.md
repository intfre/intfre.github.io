---
layout: post
title: Objective-c-疑问
date: 2017-05-16
categories: blog
tags:[Objective-c]
description: 同一个类的对象的成员变量之间有关系？

---


	@interface Car:NSObject
	{
		Engine * engine;
	}
	-(void) setEngine:(Engine*）newEngine;
	@end  //car

	@implementation Car
	-(void) setEngine:(Engine*) newEngine
	{
		[engine realse];
		engine=[newEngine retain];
	}


	@end //car



	@interface Engine:NSObject
	-(NSString*)description;
	@end
	@implementation Engine
	-(NSString*)description{
		return (@"i am a Engine");
	} 
	@end //engine



	Engine * engine = [Engine new]; //count 1
	Car * car1=[Car new];
	Car * car2=[Car new];

	[car1 setEngine:engine]; // count 2;
	[engine release];			//count 1
	[car2 setEngine:[car1 engine]];    //oop



car2 的成员变量 engine 与 car1的成员变量 engine 有关系吗？
为什么car2 中 [engien release] 会引发 oop ;