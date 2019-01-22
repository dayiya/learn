##1、redis的简介
* Redis 是一个开源的使用 ANSI C 语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value 数据库
##2、String类型
###2.1. Redis 的 Key
* Redis 的 key 是字符串类型，但是 key 中不能包括边界字符，由于 key 不是 binary safe的字符串，所以像"my key"和"mykey\n"这样包含空格和换行的 key 是不允许的。
####2.1.1. key 相关指令介绍
1. **exits key** 检测指定 key 是否存在，返回 1 表示存在，0 不存在
2. **del key1 key2 ...... keyN** 删除给定 key,返回删除 key 的数目，0 表示给定 key 都不存在
3. **type key** 返回给定 key 值的类型。返回 none 表示 key 不存在,string 字符类型，list 链表类型 set 无序集合类型......
4. **keys pattern** 返回匹配指定模式的所有 key
5. **randomkey** 返回从当前数据库中随机选择的一个 key,如果当前数据库是空的，返回空串
6. **rename oldkey newkey** 重命名一个 key,如果 newkey 存在，将会被覆盖，返回 1 表示成功，0 失败。可能是 oldkey 不存在或者和 newkey 相同。
7. **renamenx oldkey newkey** 同上，但是如果 newkey 存在返回失败。
8. **expire key seconds** 为 key 指定过期时间，单位是秒。返回 1 成功，0 表示 key 已经设置过过期时间或者不存在。
9. **ttl key** 返回设置过过期时间 key 的剩余过期秒数。-1 表示 key 不存在或者未设置过期时间。
10. **select db-index** 通过索引选择数据库，默认连接的数据库是 0,默认数据库数是 16 个。返回 1
表示成功，0 失败。
11. **move key db-index** 将 key 从当前数据库移动到指定数据库。返回 1 表示成功。0 表示 key
不存在或者已经在指定数据库中。
###2.2. Redis 的 vaule
redis 提供五种数据类型：string,hash,list,set 及 sorted set。
####2.2.1. string 类型
string 是最基本的类型，而且 string 类型是二进制安全的。意思是 redis 的 string 可以
包含任何数据。比如 jpg 图片或者序列化的对象。从内部实现来看其实 string 可以看作 byte数组，最大上限是 1G 字节。

**string 类型数据操作指令简介**
1. **set key value**：设置 key 对应 string 类型的值，返回 1 表示成功，0 失败。
2. **setnx key value**：如果 key 不存在，设置 key 对应 string 类型的值。如果 key 已经存在，返回 0。
3. **get key**：获取 key 对应的 string 值,如果 key 不存在返回 nil
4. **getset key value**：先获取 key 的值，再设置 key 的值。如果 key 不存在返回 nil。
5. **mget key1 key2 ...... keyN**：一次获取多个 key 的值，如果对应 key 不存在，则对应返回 nil。
6. **mset key1 value1 ...... keyN valueN**：一次设置多个 key 的值，成功返回 1 表示所有的值都设置了，失败返回 0 表示没有任何值被设置。
7. **msetnx key1 value1 ...... keyN valueN**：一次设置多个 key 的值，但是不会覆盖已经存在的 key
8. **incr key**：对 key 的值做++操作，并返回新的值。注意 incr 一个不是 int 的 value 会返回错误，incr 一个不存在的 key，则设置 key 值为 1。
9. **decr key**：对 key 的值做--操作，decr 一个不存在 key，则设置 key 值为-1。
10. **incrby key integer**：对 key 加上指定值 ，key 不存在时候会设置 key，并认为原来的 value
是 0。
11. **decrby key integer**：对 key 减去指定值。decrby 完全是为了可读性，我们完全可以通过 incrby一个负值来实现同样效果，反之一样。
12. **strlen key**：获取字符串长度

<img alt="redis的使用-e2ffab53.png" src="assets/redis的使用-e2ffab53.png" width="" height="" >

####2.2.2 Hash类型
TODO
