---
layout: post
title: PHP内核HashTable
category: PHP
tags: [PHP]
---

# PHP内核HashTable

- [介绍](#introduction)
- [PHP Hashtable 实现](#installation)

<a name="introduction"></a>
## 介绍

这篇blog主要介绍 php5.6 hashtable 和 php7 hashtable
的内部实现和区别


### PHP Hashtable 实现
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

C 语言数组内存上是连续的我们可以用下标访问，C 语言的key必须是integers 但是必须连续，举个例子，如果key 0,1,2 下一个key必须是3. 但是PHP 不同的它支持 string类型的key，并且内存也不是连续的.

用C语言实现PHP这样的结构有两种方法一种二叉查找树，他的查询时间复杂度是 O(logn)，还有一种是 hashtable 他的查询时间复杂度是O(1),PHP 数组 就是用hashtable来实现。
hashtable 会有冲突 在PHP中使用链表来解决冲突如下图

<img src="http://www.phpinternalsbook.com/_images/basic_hashtable.svg" width="360px" h alt="图片名称"/>

arBuckets 是HashTable 存储Bucket数组的头指针

我们如果要删除一个元素 假如这个bucket指针存储的是‘C’ 想要去删除他，你需要将a的pNext设置为NULL

<img src="http://www.phpinternalsbook.com/_images/doubly_linked_hashtable.svg" width="360px" h alt="图片名称"/>

*****

#### HashTable 的 Bucket 

Bucket 是 HashTable 非常重要的一个结构，可以在 `zend_hash.h` 文件中找到, 我们看一下 Bucket的结构：

```
typedef struct bucket {
    ulong h;
    uint nKeyLength;
    void *pData;
    void *pDataPtr;
    struct bucket *pListNext;
    struct bucket *pListLast;
    struct bucket *pNext;
    struct bucket *pLast;
    char *arKey;
} Bucket;
```

>  - `h` 是 hash 的 key , 如果 key 是一个 `integer` 那么`nKeyLength` 将会为0，如果 key 是一个 string, `nKeyLength` 等于 `arKey` 字符串的长度
>  - pData 是存储数据的指针