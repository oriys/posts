---
title: "Redis::基础::字符串"
date: 2021-08-15T06:48:14+08:00
draft: true
---

## 介绍

Redis 字符串类型是你能与 Redis key 关联的最简单的 value 类型。它是 Memcached 中唯一的数据类型，所以对于新人来说，在 Redis 中使用它也是非常自然的。

由于 Redis 的 key 是字符串，当我们把字符串类型也作为一个 value 时，我们是把一个字符串映射到另一个字符串。字符串数据类型对很多用例都很有用，比如缓存 HTML 片段或页面。

## 相关命令

`APPEND key value`

将一个 value 附加到一个 key 上

`DECR key`

将一个 key 的整数 value 递减 1

`DECRBY key decrement`

将一个 key 的整数 value 按给定的数字递减。

`GET key`

获取一个 key 的 value

`GETDEL key`

获取一个 key 的 value 并删除该 key

`GETEX key [EX seconds|PX milliseconds|EXAT timestamp|PXAT milliseconds-timestamp|PERSIST]`

获取一个 key 的 value ，并可选择设置其过期时间

- 选项
  - EX seconds -- 设置指定的过期时间，单位是秒。
  - PX milliseconds -- 设置指定的过期时间，单位是毫秒。
  - EXAT timestamp-seconds -- 设置指定的 Unix 时间，以秒为单位， key 将在该时间失效。
  - PXAT timestamp-milliseconds -- 设置指定的 Unix 时间， key 将在该时间过期，单位是毫秒。
  - PERSIST -- 删除与 key 相关的生存时间。

`GETRANGE key start end`

获取存储在一个 key 上的字符串的子串

`GETSET key value`

设置一个 key 的字符串 value 并返回其旧 value

`INCR key`

将一个 key 的整数 value 递增 1

`INCRBY key increment`

将一个 key 的整数 value 按给定的数额递增

`INCRBYFLOAT key increment`

将一个 key 的浮动 value 按给定的量增加。

`MGET key [key ...]`

获取所有给定 key 的 value

`MSET key value [key value ...]`

将多个 key 设置为多个 value

`MSETNX key value [key value ...]`

将多个 key 设置为多个 value ，只有在 key 都不存在的情况下才会这样做

`PSETEX key milliseconds value`

设置一个 key 的 value 和过期时间，单位为毫秒

`SET key value [EX seconds|PX milliseconds|EXAT timestamp|PXAT milliseconds-timestamp|KEEPTTL] [NX|XX] [GET]`

设置一个 key 的字符串 value

- 选项
  - EX seconds -- 设置指定的过期时间，单位是秒。
  - PX milliseconds -- 设置指定的过期时间，单位是毫秒。
  - EXAT timestamp-seconds -- 设置指定的 Unix 时间，以秒为单位， key 将在该时间失效。
  - PXAT timestamp-milliseconds -- 设置指定的 Unix 时间， key 将在该时间过期，单位是毫秒。
  - NX -- 只有在 key 不存在的情况下才会设置它。
  - XX -- 只设置已经存在的 key 。
  - KEEPTTL -- 保留与钥匙相关的生存时间。
  - GET -- 返回存储在 key 的旧字符串，如果 key 不存在，则返回 nil。如果存储在 key 上的 value 不是字符串，将返回一个错误，并中止 SET。

`SETEX key seconds value`

设置一个 key 的 value 和过期时间

`SETNX key value`

设置一个 key 的 value ，只在该 key 不存在的情况下使用

`SETRANGE key 的偏移 value`

从指定的偏移量开始，覆盖 key 上的部分字符串

`STRALGO LCS algo-specific-argument [algo-specific-argument ...]`

针对字符串运行算法（目前是 LCS）。

`STRLEN key`

获取存储在 key 中的 value 的长度

## 数据结构

sds 的定义在 `src/sds.h`

```c
typedef char *sds;
```

从定义上看 sds 等价于字符数组，实际上还有一部分头信息在 s[0]地址的前面。因此完整的 sds 定义由 sdshdr 头信息和一个字符数组 buf 前后两部分组成，其中 sds 的地址就是 buf 的地址。

根据字符长长度不同，Redis 提供了五种头信息，以节省内存。头信息定义如下，里面包含了数组长度，已使用长度和一个标记位，其中的 `__attribute__ ((__packed__))` 是告诉编译器取消结构在编译过程中的优化对齐，按照实际占用字节数进行对齐，是 GCC 提供的语法。

```c
/* Note: sdshdr5 is never used, we just access the flags byte directly.
 * However is here to document the layout of type 5 SDS strings. */
struct __attribute__ ((__packed__)) sdshdr5 {
    unsigned char flags; /* 3 lsb of type, and 5 msb of string length */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len; /* used */
    uint8_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len; /* used */
    uint16_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len; /* used */
    uint32_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* used */
    uint64_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};

```

## 关键函数

## 创建字符串

```C
sds _sdsnewlen(const void *init, size_t initlen, int trymalloc) {
    // 声明 sdshdr
    void *sh;
    // 声明 sds
    sds s;
    // 根据字符数组长度判断头类型
    char type = sdsReqType(initlen);
    // 空字符串通常会在创建后追加，使用 sdshdr8 是更好的选择，sdshdr5 已经弃用
    if (type == SDS_TYPE_5 && initlen == 0) type = SDS_TYPE_8;
    // 获取 sdshdr 的内存占用大小
    int hdrlen = sdsHdrSize(type);
    // 声明 flags 指针
    unsigned char *fp; /* flags pointer. */
    // 未使用量
    size_t usable;
    // 断言信息头的大小至少大于等于 0
    assert(initlen + hdrlen + 1 > initlen); /* Catch size_t overflow */
    // 根据参数尝试决定是否分配内容
    sh = trymalloc ?
         s_trymalloc_usable(hdrlen + initlen + 1, &usable) :
         s_malloc_usable(hdrlen + initlen + 1, &usable);
    // 分配失败返回 NULL
    if (sh == NULL) return NULL;
    // 如果没有分配过，整个 sds 包括头全部初始化成 0
    if (init == SDS_NOINIT)
        init = NULL;
    else if (!init)
        memset(sh, 0, hdrlen + initlen + 1);
    // flags 指针在 s 起始地址的前一位
    fp = ((unsigned char *) s) - 1;
    // 计算使用量
    usable = usable - hdrlen - 1;
    if (usable > sdsTypeMaxSize(type))
        usable = sdsTypeMaxSize(type);
    // 根据，初始化 sdshdr 结构体
    switch (type) {
        case SDS_TYPE_5: {
            *fp = type | (initlen << SDS_TYPE_BITS);
            break;
        }
        case SDS_TYPE_8: {
            SDS_HDR_VAR(8, s);
            sh->len = initlen;
            sh->alloc = usable;
            *fp = type;
            break;
        }
        case SDS_TYPE_16: {
            SDS_HDR_VAR(16, s);
            sh->len = initlen;
            sh->alloc = usable;
            *fp = type;
            break;
        }
        case SDS_TYPE_32: {
            SDS_HDR_VAR(32, s);
            sh->len = initlen;
            sh->alloc = usable;
            *fp = type;
            break;
        }
        case SDS_TYPE_64: {
            SDS_HDR_VAR(64, s);
            sh->len = initlen;
            sh->alloc = usable;
            *fp = type;
            break;
        }
    }
    // 将 init 的内容复制到 s 上
    if (initlen && init)
        memcpy(s, init, initlen);
    // 末尾置为0，表示结束
    s[initlen] = '\0';
    return s;
}
```

## 字符串拼接

```c
sds sdscatlen(sds s, const void *t, size_t len) {
    //获得当前长度
    size_t curlen = sdslen(s);
    //按照字符t的长度扩容
    s = sdsMakeRoomFor(s,len);
    if (s == NULL) return NULL;
    //拷贝t到s的后面
    memcpy(s+curlen, t, len);
    //重新设置s的长度
    sdssetlen(s, curlen+len);
    // 末尾置为0，表示结束
    s[curlen+len] = '\0';
    return s;
}
```

```c
sds sdsMakeRoomFor(sds s, size_t addlen) {
    // 声明 shshdr 和新的 sdshdr
    void *sh, *newsh;
    size_t avail = sdsavail(s);
    size_t len, newlen;
    char type, oldtype = s[-1] & SDS_TYPE_MASK;
    int hdrlen;
    size_t usable;

    /* Return ASAP if there is enough space left. */
    if (avail >= addlen) return s;

    len = sdslen(s);
    sh = (char*)s-sdsHdrSize(oldtype);
    newlen = (len+addlen);
    assert(newlen > len);   /* Catch size_t overflow */
    if (newlen < SDS_MAX_PREALLOC)
        newlen *= 2;
    else
        newlen += SDS_MAX_PREALLOC;

    type = sdsReqType(newlen);

    /* Don't use type 5: the user is appending to the string and type 5 is
     * not able to remember empty space, so sdsMakeRoomFor() must be called
     * at every appending operation. */
    if (type == SDS_TYPE_5) type = SDS_TYPE_8;

    hdrlen = sdsHdrSize(type);
    assert(hdrlen + newlen + 1 > len);  /* Catch size_t overflow */
    if (oldtype==type) {
        newsh = s_realloc_usable(sh, hdrlen+newlen+1, &usable);
        if (newsh == NULL) return NULL;
        s = (char*)newsh+hdrlen;
    } else {
        /* Since the header size changes, need to move the string forward,
         * and can't use realloc */
        newsh = s_malloc_usable(hdrlen+newlen+1, &usable);
        if (newsh == NULL) return NULL;
        memcpy((char*)newsh+hdrlen, s, len+1);
        s_free(sh);
        s = (char*)newsh+hdrlen;
        s[-1] = type;
        sdssetlen(s, len);
    }
    usable = usable-hdrlen-1;
    if (usable > sdsTypeMaxSize(type))
        usable = sdsTypeMaxSize(type);
    sdssetalloc(s, usable);
    return s;
}
```
