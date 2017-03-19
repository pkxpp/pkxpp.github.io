---
layout: post
comments: true
categories: 设计模式
---

[TOC]

我还是坚信**学以致用**是最有效率的学习方式，也继续这样实践下去。今天来实现组合模式，加深看书的印象。组合模式应用的地方有很多，unity是我觉得最彻底的一个应用，什么都是组件，连脚本都是。那么对于一个游戏引擎来说，绘制就是其中的一部分内容，UI上面有Label, Button，Image等等，以及界面Layer，一个非常明显的树形结构。那么怎么样优雅的去设计这个框架会让代码结构清晰，而且容易扩展呢？那就是组合模式，现在用组合模式来实现一个简单的游戏引擎的部分框架。不过，在实际项目中远不止如此简单，像cocos2dx-3.0以后的版本就把渲染部分分离出来了。

# 意图
*"使得单个对象和组合对象的使用具有一致性。"*

这是书上的话，那么怎么理解呢？基于要实现的内容，就简单理解为，使用这个引擎的人可以不用区分一个对象是一个Label还是一个Button，以及一个上面既有Label又有Button的Layer对象，都可以直接往场景里Add就好了，至于怎么绘制，使用引擎的人不需要关心。

# 适用性
* 你想表示对象的部分-整体层次结构
* 你希望用户忽略组合对象和单个对象的不同，用户将统一地使用组合结构中的所有对象

# 结构
结构图：
![组合模式结构图](http://o856moet9.bkt.clouddn.com/composite.jpg)

简要说明：
* Component: 组合中的对象声明接口。
* Leaf：在组合中表示节点对象，叶节点没有子节点。
* Composite: 在组合中有子部件的那些部件的行为。
* Client：外部调用。

# 实现
c++实现：

	#include <iostream>
	#include <vector>
	#include <unistd.h>
	#include <time.h>

	using namespace std;

	class Node{
	public:
			Node(){}
			Node(char* name){_name = name;}
			virtual void Draw(){};
			virtual void Add(Node* child){};
			char* GetName(){return _name;};
	private:
			char* _name;
	};

	class Label: public Node{
	public:
			Label(char* name):Node(name){}
			void Draw(){
					cout << "Draw image: " << GetName() << endl;
			}
	};

	class Sprite: public Node{
	public:
			Sprite(char* name):Node(name){}
			void Draw(){
					cout << "Draw Sprite." << GetName() << endl;
			}
	};

	class Layer: public Node{
	public:
			void Add(Node* child){
					_children.push_back(child);
			};

			void Draw(){
					cout << "Draw Layer begin ... " << endl;
					DrawChildren();
					cout << "Draw Layer end. " << endl;
			}
			void DrawChildren(){
					vector<Node*>::iterator it = _children.begin();
					for (; it != _children.end(); ++it)
							(*it)->Draw();
			}

	private:

			vector<Node*> _children;
	};

	class Scene: public Layer{
	public:
			void Draw(){
					cout << "Draw Scene begin ... " << endl;
					DrawChildren();
					cout << "Draw Scene end. " << endl;
			}
	};

	class Director{
	public:
			Director(Scene* s){
					_scene = s;
			}

			void Rend(){
					if (_scene)
							_scene->Draw();
			}
	private:
			Scene* _scene;
	};

	int main(){
			const int WAIT_TIME = 2;
			Scene* pScene = new Scene();
			pScene->Add(new Label("Label 1"));
			pScene->Add(new Sprite("Sprite 1"));
			Layer* pLayer = new Layer();
			pLayer->Add(new Label("Label 2 on Layer"));
			pLayer->Add(new Sprite("Sprite 2 on Layer"));
			pScene->Add(pLayer);
			Director director(pScene);
			time_t tLast;
			while(true){
					tLast = time(NULL);
					director.Rend();

					time_t tNow = time(NULL);
					if(tNow - tLast < WAIT_TIME){
							//seconds = 1000 * 1000 micro seconds 
							usleep((WAIT_TIME - tNow + tLast)*1000*1000);
					}
			}
	}

结果：

	Draw Scene begin ... 
	Draw image: Label 1
	Draw Sprite.Sprite 1
	Draw Layer begin ... 
	Draw image: Label 2 on Layer
	Draw Sprite.Sprite 2 on Layer
	Draw Layer end. 
	Draw Scene end. 

# 总结
* 组合设计模式中基类既申明了特定对象的一些操作，也声明了组合对象的一些操作；
* *组合模式描述了如何使用递归组合。*组合对象聚合了基类对象，例如结构图中的Composite聚合了Component对象；
* 在真正项目实现中，细节部分也会有所变化。例如cocos2dx中的Node类（相当于这里的Component接口）不仅定义了组合对象的接口，还实现了只有Composite中才有的列表(_children)和Add、Remove等接口，它的本意是想做到节点在UI上可以互相包含，一个Label可以放在一个Button上，一个Button也可以放在一个Label上。虽然，书中[1]并不建议这样做，说这样对叶子节点会有空间上的浪费，但是对于cocos2dx开发者来说他们觉得利益远远大于弊端，或者说不存在弊端（没有叶节点）那就没有任何问题了。

# 参考
[1]《设计模式：可复用面向对象软件的基础》