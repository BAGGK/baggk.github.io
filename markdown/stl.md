---
title: "Stl"
date: 2019-05-27T15:21:15+08:00
tags:
- stl
#thumbnailImage: //example.com/image.jpg
---

<!--more-->

#### 六大部件

**容器 算法 分配器 适配器 迭代器 仿函数**

````c++
//一个使用了六大部件小程序
#include<vector>
#include<algorithm>
#include<iostream>

using namespace std;
int main(){
    
    int ia[ 6 ] = {7 ,210,12,47,109,83};
    vector<int ,allocator<int>> vi(ia,ia+6);
    
    cout << count_if(vi.begin(),vi.end(),not1(bind2nd(less<int>,40)));
    
    return 0;
}
````

#### 容器的分类

+ 序列式的容器 Sequent Containers

> Array  |  Vector | Deque | List | Forward-List
>
> List在实现上是个双向环状链表
>
> Vector是两倍增长

+ 关联式的容器 Associative Containers

> set | map | Multiset | Multimap 
>
> 在内部是红黑树实现的。set与map的key不可以重复。Multi-以为可以重复
>
> 对于set来说value就是key

+ 不定顺的容器 Unordered Containers

> 内部是hashtable。内部是使用Separate chaining

内存