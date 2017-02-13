---
layout: post
comments: true
categories: 设计模式
---

[TOC]

最近工作上遇到一个问题，最后用设计模式——装饰模式(Decorator)解决了，加深了对这个模式的印象，记录一下，同时当作对看书的复习吧，如果对其他朋友有抛砖引玉的作用就最好了^_^。

# 问题描述

我们的UI操作是一种已经定下来的模式，后来被要求修改为另外一种操作模式。简单来说，就是要改变一种控件的功能，我们后面就把这种控件叫做WidgetBox吧。而且，这个WidgetBox类本身有别的用途，暂时不考虑直接在原来的基础上修改。

从这个角度来说，解决这个问题首先想到的方法是：**继承！**以WidgetBox作为父类重新写一个子类，重载关键函数，满足新操作模式的需求。

但是，恰恰遇到问题了：

* 子类需要把父类的主要功能完全重写，需要改掉很多函数代码，麻烦；
* 还有另外一个问题。由于界面用的是flash，在消息处理方面绑定的比较底层，已经很难改动了，除非你把整个工程中别的地方用到的都改掉，工程上这是不切实际的；
* 最重要的是：目前由于UI是flash制作的，所以在控件上有一些约定。新定义控件（反应到代码里面也就是添加子类）虽然不麻烦，但是还要把一部分维护工作留给UI人员，是自己不想的。

所以，**继承**这个方法行不通，会因为改动底层给系统带来很大的不稳定性，而且代码量和结构上的修改并没有什么可取之处，如果需求变了，需要新的操作模式呢？

在以上的背景之下，自己想到了如此拙见：用到了装饰(Decorator)这个设计模式。虽然不能完美解决问题，还存在缺点，但是总的来说比上面的这个方法好。如果以后再碰到更好的解决办法，说明我又进步了。^_^

# 装饰(Decorator)模式
引用书上的一句话：

> 动态地给一个对象添加一些额外的职责。

## 动机
依然引用书上的说法：

> 有时我们希望给某个对象而不是整个类添加一些功能。一种较为灵活的方式是将组件嵌入另一个对象中。

对接上自己需要的：

很简单！能在不改变现有系统结构的情况下，增加一种新的操作方式。那么我就实现一个装饰类，在其中嵌入已经存在的控件类WidgetBox，然后我在装饰类中实现自己的各种操作方式，而且用到大部分的WidgetBox类本身已经实现的功能。

## 结构
完整的装饰模式的结构如下图所示：

![装饰模式](http://o856moet9.bkt.clouddn.com/decorator.jpg)

但是，因为个中原因，自己的实现有些许不同：

![](http://o856moet9.bkt.clouddn.com/decorator_self.jpg)

**说明一下：**

* 1.没有实现一个上层的抽象接口类，因为不想改变现有的结构。但是写到这里的时候，觉得Decorator类可以继承Widget类（他是WidgetBox类最上层的基类），这样会优化下面提到的问题。同时，这也说明记录写文章是有一些用处滴^_^。
* 2.这个接口暴露了自己实现的一个缺点：Decorator需要完全拥有和WidgetBox一模一样的接口，这里就留下了维护这两个类接口一致的问题，特别是如果你想要用户在WidgetBoxDecorator的操作和WidgetBox没有区别的情况下。但是，上面也提到了可以继承widget类，会大大减弱这个问题。


## 实现

简单的代码框架，木有测试~

### C++实现

code: 

	class WidgetBox{
		void subscribe(event);
		void unsubscribe(event);
		void HoldObject();
		bool OnMousePress();
		bool OnMouseUp();
	}

	class Decorator{
		virtual void subscribe(event);
		virtual void unsubscribe(event);
		virtual void HoldObject();
		virtual bool OnMousePress();
		virtual bool OnMouseUp();
	private:
		WidgetBox* pItem;
	}

	class WidgetBoxDecorator{
		void subscribe(event);
		void unsubscribe(event);
		void HoldObject();
		bool OnMousePress();
		bool OnMouseUp();
	}

	int main()
	{
		// 老的控件操作模式
		WidgetBox *pItem = new WidgetBox("name");
		pItem->HoldObject();

		// 新的控件操作模式
		WidgetBoxDecorator *pItem(new WidgetBox("name"));
		pItem->HoldObject();
	}
### 小结

* 什么时候用装饰模式？

> 1.在不影响其他对象的情况下，比如自己这里不能够改变现有的结构；
> 2.当不能采用生成子类的方法进行扩充时，自己碰到的问题也算是；
> 3.书上还提到：处理那些可以撤销的职责，这个没有用到。

# 总结

自己也把《设计模式-可服用面向对象软件的基础》这本书看了一遍，印象其实不是很深刻，除了个别非常形象的模式之外，例如:享元模式-FlyWeight。剩下的就是自己去写过或者常用的模式了，比如：单例模式、工厂模式、观察者模式。所以，对于自己来说，碰到一个问题，特别是设计上的问题，然后用一个设计模式去解决这个问题，才能深刻理解这个模式的优缺点，以及用途。

# 参考
[1]《设计模式：可复用面向对象软件的基础》


---

# 学习：JavaScript实现

	var WidgetBox = {
		createNew: function(){
			var widget = {};
			widget.OnMouseUp = function()
			{
				console.log("WidgetBox OnMouseUp();");
			}
			return widget;
		}
	}

	var Decorator = {
		createNew: function(){	
			var widget = {};
			widget.item = WidgetBox.createNew();
			widget.OnMouseUp = function()
			{
				item.OnMouseUp();
				console.log("Decorator OnMouseUp();");
			}
			return widget;
		}
	}

	var WidgetBoxDecorator = {
		createNew: function(){
			// 继承
			var widget = Decorator.createNew();
			widget.item = WidgetBox.createNew();
			widget.OnMouseUp = function()
			{
				console.log("WidgetBoxDecorator OnMouseUp();");
			}
			return widget;
		}
	}

	// old operation
	console.log("Old operation:");
	var item = WidgetBox.createNew();
	item.OnMouseUp();

	// new operation
	console.log("New operation:");
	var item1 = WidgetBoxDecorator.createNew();
	item1.OnMouseUp();

结果：

	Old operation:
	WidgetBox OnMouseUp();
	New operation:
	WidgetBoxDecorator OnMouseUp();