---
layout: post
title: PHP内核HASHTABLE
category: PHP
tags: [PHP]
---

# PHP内核HashTable

- [介绍](#introduction)
- [安装](#installation)

<a name="introduction"></a>
## 介绍

这篇blog主要介绍 php5.6 hashtable 和 php7 hashtable
的内部实现和区别


### 一致性哈希算法
php5.6 Hashtable 实现

```
 typedef struct _hashtable {
          uint nTableSize;
          uint nTableMask;
          uint nNumOfElements;
          ulong nNextFreeElement;
          Bucket *pInternalPointer;
          Bucket *pListHead;
          Bucket *pListTail; 
          Bucket **arBuckets;
          dtor_func_t pDestructor;
          zend_bool persistent;
          unsigned char nApplyCount;
          zend_bool bApplyProtection;
          #if ZEND_DEBUG
               int inconsistent;
          #endif
} HashTable;
```

> - nTableSize，HashTable的大小，以2的倍数增长
- nTableMask，用在与哈希值做与运算获得该哈希值的索引取值， 
- arBuckets初始化后永远是nTableSize-1
- nNumOfElements，HashTable当前拥有的元素个数，count函数直接返回这个值
- nNextFreeElement，表示数字键值数组中下一个数字索引的位置
- pInternalPointer，内部指针，指向当前成员，用于遍历元素
- pListHead，指向HashTable的第一个元素，也是数组的第一个元素
- pListTail，指向HashTable的最后一个元素，也是数组的最后一个元素。与上面的指针结合，在遍历数组时非常方便，比如reset和endAPI
- arBuckets，包含bucket组成的双向链表的数组，索引用key的哈希值和nTableMask做与运算生成
- pDestructor，删除哈希表中的元素使用的析构函数
- persistent，标识内存分配函数，如果是TRUE，则使用操作系统本身的内存分配函数，否则使用PHP的内存分配函数
- nApplyCount，保存当前bucket被递归访问的次数，防止多次递归
- bApplyProtection，标识哈希表是否要使用递归保护，默认是1，要使用

### 虚拟节点
当我们的缓存服务器比较少的时候可能出现 缓存分布不均匀的问题，这个时候我们引入虚拟节点来解决这个问题


