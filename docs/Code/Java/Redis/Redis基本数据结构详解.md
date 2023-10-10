# Redis基本数据结构详解

Redis 共有 5 种基本数据结构：String（字符串）、List（列表）、Set（集合）、Hash（散列）、Zset（有序集合）。

Redis 基本数据结构的底层数据结构实现如下：

| String | List                         | Hash                | Set             | Zset              |
| :----- | :--------------------------- | :------------------ | :-------------- | :---------------- |
| SDS    | LinkedList/ZipList/QuickList | Hash Table、ZipList | ZipList、Intset | ZipList、SkipList |

Redis 3.2 之前，List 底层实现是 LinkedList 或者 ZipList。 Redis 3.2 之后，引入了 LinkedList 和 ZipList 的结合 QuickList，List 的底层实现变为 QuickList。

你可以在 Redis 官网上找到 Redis 数据结构非常详细的介绍：

- [Redis Data Structures](https://redis.com/redis-enterprise/data-structures/)
- [Redis Data types tutorial](https://redis.io/docs/manual/data-types/data-types-tutorial/)