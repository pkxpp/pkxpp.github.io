---
layout: post
comments: true
categories: lua
---

[TOC]

前面两篇主要介绍一些基础，帮助后面gc流程理解的。像是饭前的开胃菜一般，让你后面容易吃的更多。接下来几篇都是gc流程相关





# 带着一些问题去看流程
这里列出几个问题，你可以直接跳过。当然，有所了解的按照自己的理解去回答一下，或许会有一些新的想法。如果你都会回答的很清晰了，那么接下来的系列几乎可以不怎么看了。

（1）对象的颜色变化过程？

（2）新创建对象和gc流程是怎么关联的？

（3）什么时候gc？

（4）增量式gc体现在哪？

gc的流程，按照程序状态分为5个状态：

```
/*
** Possible states of the Garbage Collector
*/
#define GCSpause    0
#define GCSpropagate    1
#define GCSsweepstring    2
#define GCSsweep    3
#define GCSfinalize    4
```

* 1.初始化阶段
* 2.扫描阶段
* 3.回收阶段（字符串）
* 4.回收阶段
* 5.结束阶段

接下来，主要讲一下新创建对象是怎么和gc关联的。因为gc所需要的对象都是在某个地方创建出来的，并且在创建的时候就会和[第二篇]()提到的数据结构关联上了

# 新创建对象

测试代码：
```
// main 函数
    int nCount = 0;
    while(true){
        int bEven = nCount %2 == 0;
        if(!bEven)
        {
            Sleep(1000);
            ++nCount;
            continue;
        }
        ++nCount;
        // do something
        LuaTest();
    }
// LuaTest函数
g_luaReg->DoScript("Test.lua");

// Test.lua

```
## luaC_link函数
这个是新生成对象和gc绑定关系的关键函数，也就是把新创建的对象放在gc链表上[1]：

```
void luaC_link (lua_State *L, GCObject *o, lu_byte tt) {
  global_State *g = G(L);
  o->gch.next = g->rootgc;
  g->rootgc = o;
  o->gch.marked = luaC_white(g);
  o->gch.tt = tt;
}
```
这个函数只做了三件事：
* 把新创建的对象放在gc链表的开头，因为是单向链表[1]
* 把新创建的对象标记为当前白色（currentwhite）
* 设置对象的类型

下面分别介绍一下各需要gc的类型的创建对象部分

## table

Test.lua中新建一个table测试代码：
```
local t = {1, 2, 3}
```

> ps:这里给table放了三个元素是为了让自己调试的好找到是创建的这个table。因为虽然Test.lua中只有一行代码，创建了一个table t。但是在虚拟机启动和加载文件的时候，会创建很多其他的table。

源码：
```
Table *luaH_new (lua_State *L, int narray, int nhash) {
  Table *t = luaM_new(L, Table);
  luaC_link(L, obj2gco(t), LUA_TTABLE);
  t->metatable = NULL;
  t->flags = cast_byte(~0);
  /* temporary values (kept only if some malloc fails) */
  t->array = NULL;
  t->sizearray = 0;
  t->lsizenode = 0;
  t->node = cast(Node *, dummynode);
  setarrayvector(L, t, narray);
  setnodevector(L, t, nhash);
  return t;
}
```
**说明：**
* new一个table的地方有很多，但是最终都是调用到这个函数，new出来的是一个堆上的一块内存
* 对象类型设置为LUA_TABLE
* 放到gc链表上
* 再把这个指针封装的TValue放到虚拟机的栈上。sethvalue等系列函数，就是把new出来的对象封装到TValue中，然后找到一个栈的元素，把这个value.gc赋值为这个新的对象，看下面代码。有些地方是放在栈顶，那么就需要有栈的操作，比如top++(*incr_top(L)*)
```
#define sethvalue(L,obj,x) \
  { TValue *i_o=(obj); \
    i_o->value.gc=cast(GCObject *, (x)); i_o->tt=LUA_TTABLE; \
    checkliveness(G(L),i_o); }
```
可以看到，需要GC的对象都会放在TValue的gc这个字段，在数据结构篇可以看到有解释[1]。

## lua function
创建函数，测试代码：
```
local a = 3
function f()
    local b = a
    print("test function")
end
```
源码：
```
Closure *luaF_newLclosure (lua_State *L, int nelems, Table *e) {
  Closure *c = cast(Closure *, luaM_malloc(L, sizeLclosure(nelems)));
  luaC_link(L, obj2gco(c), LUA_TFUNCTION);
  c->l.isC = 0;
  c->l.env = e;
  c->l.nupvalues = cast_byte(nelems);
  while (nelems--) c->l.upvals[nelems] = NULL;
  return c;
}
```
**说明：**
* luaC_link放在gc链表上
* 把对象类型为LUA_TFUNCTION

## PROTO
目前自己的理解这是一个以文件为单位的chunk，所以每个文件都是以一个个独立的Proto类型在代码里存在的。这一块没有仔细去研究，所以主要是自己理解，以做抛砖引玉之用！

代码：
```
Proto *luaF_newproto (lua_State *L) {
  Proto *f = luaM_new(L, Proto);
  luaC_link(L, obj2gco(f), LUA_TPROTO);
// 省略...
  return f;
}
```

**说明：**
* luaC_link放到gc链表上
* 看**f_parser**函数，发现新建Proto之后，会新建一个lua函数(luaF_newLclosure)。所以，Proto是和Closure绑定的，也很好理解，函数是需要和某个文件有关联的

## THREAD

创建携程测试代码：
```
co = coroutine.create(function (a,b)
    print(111, a, b)
end)
```

代码：
```
LUA_API lua_State *lua_newthread (lua_State *L) {
  lua_State *L1;
  lua_lock(L);
  luaC_checkGC(L);
  L1 = luaE_newthread(L);
  setthvalue(L, L->top, L1);
  api_incr_top(L);
  lua_unlock(L);
  luai_userstatethread(L, L1);
  return L1;
}
```

**说明：**
* 放在gc链表的luaC_link是在luaE_newthread中调用的

## String
代码：
```
static TString *newlstr (lua_State *L, const char *str, size_t l,
                                       unsigned int h) {
  TString *ts;
  stringtable *tb;
  if (l+1 > (MAX_SIZET - sizeof(TString))/sizeof(char))
    luaM_toobig(L);
  ts = cast(TString *, luaM_malloc(L, (l+1)*sizeof(char)+sizeof(TString)));
  ts->tsv.len = l;
  ts->tsv.hash = h;
  ts->tsv.marked = luaC_white(G(L));
  ts->tsv.tt = LUA_TSTRING;
  ts->tsv.reserved = 0;
  memcpy(ts+1, str, l*sizeof(char));
  ((char *)(ts+1))[l] = '\0';  /* ending 0 */
  tb = &G(L)->strt;
  h = lmod(h, tb->size);
  ts->tsv.next = tb->hash[h];  /* chain new entry */
  tb->hash[h] = obj2gco(ts);
  tb->nuse++;
  if (tb->nuse > cast(lu_int32, tb->size) && tb->size <= MAX_INT/2)
    luaS_resize(L, tb->size*2);  /* too crowded */
  return ts;
}
```

**说明**
* 字符串有一点特殊的，这里并没有调用luaC_link函数放在gc链表上，而是直接放在global_State中的strt变量中，所以扫描的时候也不会处理字符串类型了，因为不在gc链表中。所以，后面流程中处理的方式也有所区别

## Upvalue
代码：
```
void luaC_linkupval (lua_State *L, UpVal *uv) {
  global_State *g = G(L);
  GCObject *o = obj2gco(uv);
  o->gch.next = g->rootgc;  /* link upvalue into `rootgc' list */
  g->rootgc = o;
  if (isgray(o)) { 
    if (g->gcstate == GCSpropagate) {
      gray2black(o);  /* closed upvalues need barrier */
      luaC_barrier(L, uv, uv->v);
    }
    else {  /* sweep phase: sweep it (turning it into white) */
      makewhite(g, o);
      lua_assert(g->gcstate != GCSfinalize && g->gcstate != GCSpause);
    }
  }
}
```

**说明：**
* 详细见参考2
* 可以看到这里主要是对节点做颜色操作。因为upvalue肯定是跟Closure有关联的，所以可以在扫描Closure的时候找到他的upvalue。隐藏，没有必要再把这个节点放在gc链表上了

## userdata
代码：
```
Udata *luaS_newudata (lua_State *L, size_t s, Table *e) {
  Udata *u;
  if (s > MAX_SIZET - sizeof(Udata))
    luaM_toobig(L);
  u = cast(Udata *, luaM_malloc(L, s + sizeof(Udata)));
  u->uv.marked = luaC_white(G(L));  /* is not finalized */
  u->uv.tt = LUA_TUSERDATA;
  u->uv.len = s;
  u->uv.metatable = NULL;
  u->uv.env = e;
  /* chain it on udata list (after main thread) */
  u->uv.next = G(L)->mainthread->next;
  G(L)->mainthread->next = obj2gco(u);
  return u;
}
```


**说明：**
* 详细见参考2


# GC

一开始会以为lua会每隔固定的时间（setpause）进行一次gc，但是看代码却根本没有这样的循环。那么什么时候GC？

## 什么时候GC
lua在每次new对象成功之后，都会做一次GC的检查，这一点在New一个GC对象（table等）之前都会做这个检查，源码随处可见，自己找一下。这个检查GC的代码就是luaC_checkGC函数：
```
#define luaC_checkGC(L) { \
  condhardstacktests(luaD_reallocstack(L, L->stacksize - EXTRA_STACK - 1)); \
  if (G(L)->totalbytes >= G(L)->GCthreshold) \
    luaC_step(L); }
```

可以看到其中如果超过了某个阈值就会进行GC，这个阈值和*collectgarbage*息息相关，这里面讲起来也需要一个篇幅，先不打算深入了。

接下来，开始进入gc的正式流程~

# 参考
[1][第二篇 数据结构篇]()
[2][新创建对象](https://github.com/lichuang/Lua-Source-Internal/blob/master/doc/ch08-GC.md)