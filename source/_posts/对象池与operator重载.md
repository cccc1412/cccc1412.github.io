---
title: 对象池与operator重载
toc: true
date: 2022-07-19 13:54:02
tags:
categories:
---
对象池与operator重载
<!--more-->
## new作为关键字和运算符
`operator new can be called explicitly as a regular function, but in C++, new is an operator with a very specific behavior: An expression with the new operator, first calls function operator new (i.e., this function) with the size of its type specifier as first argument, and if this is successful, it then automatically initializes or constructs the object (if needed). Finally, the expression evaluates as a pointer to the appropriate type.`

代码里写的new是关键字，编译器根据实际情况调用new运算符。

代码里写下new关键字，编译器会：
1、调用operator new，也就是调用new运算符malloc，返回ptr。new运算符可以用户重载。
2、调用A::A()构造函数在ptr上初始化对象。
3、void*类型转化，返回对象的指针。

## placement new
```
A* p = new(ptr) A()
```
placement new也是new运算符的重载，多传入了一个参数`void* ptr`，定义在`#incldue<new>`中，对步骤一来说，直接返回ptr, 不再分配内存（用户已经分配好了，传入ptr）。
## 对象池
```c++
#ifndef _ObjectPoolBase_hpp_
#define _ObjectPoolBase_hpp_


#include<stdlib.h>
#include<assert.h>
#include<mutex>

#ifdef _DEBUG
#ifndef xPrintf
#include<stdio.h>
#define xPrintf(...) printf(__VA_ARGS__)
#endif
#else
#ifndef xPrintf
#define xPrintf(...)
#endif
#endif // _DEBUG
//模板给对象池提供参数接口
template<class Type, size_t nPoolSzie>
//对象池的实现
class CELLObjectPool
{
public:
    CELLObjectPool()
    {
        initPool();
    }

    ~CELLObjectPool()
    {
        if (_pBuf)
            delete[] _pBuf;
    }
private:
    //NodeHeader是每个对象的描述信息
    class NodeHeader
    {
    public:
        //下一块位置
        NodeHeader* pNext;
        //内存块编号
        int nID;
        //引用次数
        char nRef;
        //是否在内存池中
        bool bPool;
    private:
        //预留
        char c1;
        char c2;
    };
public:
    //释放对象内存
    void freeObjMemory(void* pMem)
    {
        //首地址往前偏移，对象和对象描述信息的内存一起释放
        NodeHeader* pBlock = (NodeHeader*)((char*)pMem - sizeof(NodeHeader));
        xPrintf("freeObjMemory: %llx, id=%d\n", pBlock, pBlock->nID);
        assert(1 == pBlock->nRef);
        //内存在对象池的部分释放之后，指针回到第一个未分配的对象
        if (pBlock->bPool)
        {
            std::lock_guard<std::mutex> lg(_mutex);
            if (--pBlock->nRef != 0)
            {
                return;
            }
            pBlock->pNext = _pHeader;
            _pHeader = pBlock;
        }
        else {
            if (--pBlock->nRef != 0)
            {
                return;
            }
            //不在对象池的直接释放内存
            delete[] pBlock;
        }
    }
    //申请对象内存，优先向内存池申请内存，内存池不够在向系统申请内存
    void* allocObjMemory(size_t nSize)
    {
        std::lock_guard<std::mutex> lg(_mutex);
        NodeHeader* pReturn = nullptr;
        //如果对象池已经满了，数量达到上限，需要向系统申请内存
        if (nullptr == _pHeader)
        {
            //计算对象池大小=对象数量*（一个对象内存大小+对象描述信息内存大小）
            pReturn = (NodeHeader*)new char[sizeof(Type) + sizeof(NodeHeader)];
            pReturn->bPool = false;
            pReturn->nID = -1;
            pReturn->nRef = 1;
            pReturn->pNext = nullptr;
        }
        else {
            pReturn = _pHeader;
            _pHeader = _pHeader->pNext;
            assert(0 == pReturn->nRef);
            pReturn->nRef = 1;
        }
        xPrintf("allocObjMemory: %llx, id=%d, size=%d\n", pReturn, pReturn->nID, nSize);
        return ((char*)pReturn + sizeof(NodeHeader));
    }
private:
    //初始化对象池
    void initPool()
    {
        //断言
        assert(nullptr == _pBuf);
        //对象池不为空（已经初始化过），不初始化直接返回
        if (_pBuf)
            return;
        //计算对象池大小=对象数量*（一个对象内存大小+对象描述信息内存大小）
        size_t realSzie = sizeof(Type) + sizeof(NodeHeader);
        size_t n = nPoolSzie * realSzie;
        //申请池的内存
        _pBuf = new char[n];
        //初始化内存池
        _pHeader = (NodeHeader*)_pBuf;
        _pHeader->bPool = true;
        _pHeader->nID = 0;
        _pHeader->nRef = 0;
        _pHeader->pNext = nullptr;
        //遍历内存块进行初始化
        NodeHeader* pTemp1 = _pHeader;

        for (size_t n = 1; n < nPoolSzie; n++)
        {
            NodeHeader* pTemp2 = (NodeHeader*)(_pBuf + (n* realSzie));
            pTemp2->bPool = true;
            pTemp2->nID = n;
            pTemp2->nRef = 0;
            pTemp2->pNext = nullptr;
            pTemp1->pNext = pTemp2;
            pTemp1 = pTemp2;
        }
    }
private:
    //描述信息块地址
    NodeHeader* _pHeader;
    //对象池内存缓存区地址
    char* _pBuf;
    //多线程使用需要加锁
    std::mutex _mutex;
};

//模板类给主函数使用对象池提供接口
template<class Type, size_t nPoolSzie>
//使用对象池的类
class ObjectPoolBase
{
public:
    //重载给对象申请内存的new操作
    void* operator new(size_t nSize)
    {
        return objectPool().allocObjMemory(nSize);
    }
    //重载给对象释放内存的delete操作
    void operator delete(void* p)
    {
        objectPool().freeObjMemory(p);
    }
    //
    template<typename ...Args>
    static Type* createObject(Args ... args)
    {   //不定参数  可变参数
        Type* obj = new Type(args...);
        //可以做点其它的事情，比如说对象的放逐和驱逐
        return obj;
    }
    //销毁对象
    static void destroyObject(Type* obj)
    {
        delete obj;
    }
private:
    //定义不同类型的对象池
    typedef CELLObjectPool<Type, nPoolSzie> ClassTypePool;
    //单例-饿汉模式，实例化一个对象池
    static ClassTypePool& objectPool()
    {   //静态CELLObjectPool对象
        static ClassTypePool sPool;
        return sPool;
    }
};
#endif // !_ObjectPoolBase_hpp_

#include "ObjectPoolBase.hpp"
```
这样也就能理解为什么从对象池中分配调用的是：`Type* obj = new Type(args...)`
而new的重载只有一个参数`nSize`，因为`nSize`就是new运算符的默认参数，分配了内存后再调用构造函数`Type(args...)`初始化。
## 参考
new: https://blog.csdn.net/ly930156123/article/details/78855379
对象池实现：https://www.cnblogs.com/-citywall123/p/12726552.html