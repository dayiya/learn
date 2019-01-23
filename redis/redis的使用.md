# 1、redis的简介
Redis 是一个开源的使用 ANSI C 语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value 数据库

# 2、String类型
## 2.1 Redis 的 Key
Redis 的 key 是字符串类型，但是 key 中不能包括边界字符，由于 key 不是 binary safe的字符串，所以像"my key"和"mykey\n"这样包含空格和换行的 key 是不允许的。

### 2.1.1 key 相关指令介绍

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
11. **move key db-index** 将 key 从当前数据库移动到指定数据库。返回 1 表示成功。0 表示 key不存在或者已经在指定数据库中。

## 2.2 Redis 的 vaule

redis 提供五种数据类型：string,hash,list,set 及 sorted set。

### 2.2.1 string 类型

string 是最基本的类型，而且 string 类型是二进制安全的。意思是 redis 的 string 可以包含任何数据。比如 jpg 图片或者序列化的对象。从内部实现来看其实 string 可以看作 byte数组，最大上限是 1G 字节。

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

### 2.2.2 Hash类型
hash是一个string类型的field和value的映射表。添加，删除操作都是0(1)(平均)。hash特别适合用于存储对象。相对于将对象的每个字段存成单个String类型。将一个对象储存在hash类型中会占用更少的内存，并且可以更方便的存取整个对象。省内存的原因是新建一个hash对象时开始是用zipmap（又称small hash）来存储的。这个zipmap其实并不是hash table，但是zipmap相比正常的hash实现可以节省不少hash本身需要的一些元数据存储开销。尽管zipmap的添加，删除，查找都是0(n)，但是由于一般对象的field数量都不太多。所以用zipmap也是很快的，也就是说添加删除平均还是0(1)。如果field或者value的大小超出一定限制后，redis会在内部自动将zipmap替换成正常的hash实现。这个实现可以在配置文件中指定。
* hash-max-zipmap-entries 64 #配置字段最多 64 个
* hash-max-zipmap-value 512 #配置 value 最大为 512 字节

**hash类型数据操作指令简介**
1. **hset key field value**：设置hash field为指定值，如果key不出在，则创建。
2. **hget key field**：获取指定的hash field。
3. **hmget key filed1 ... fieldN**：获取全部指定的hash filed。
4. **hmset key filed1 value1 ... fileN valueN**：同时设置hash的多个field。
5. **hincrby key field integer**：将指定的hash filed加上指定的值。成功返回hash filed变更后的值。
6. **hexists key field**：检测指定field是否存在。
7. **hdel key field**：删除指定的hash field。
8. **hlen key**：返回指定的hash的field的数量。
9. **hkeys key**：返回hash的所有field。
10. **hvals key**：返回hash的所有value。
11. **hgetall**：返回hash的所有filed和value。

<img alt="redis的使用-20151da2.png" src="assets/redis的使用-20151da2.png" width="" height="" >

### 2.2.3 List类型
list是一个链表结构，可以理解为一个每个子元素都是string类型的双向链表。主要功能是push、pop、获取一个范围的所有值等。操作中key理解为链表的名字。

**hash类型数据操作指令简介**
1. **lpush key string**：在key对应的list的头部添加字符串元素，返回1表示成功，0表示key存在且不是list类型。
2. **rpush key string**：在key对应list的尾部添加字符串元素。
3. **llen key**：返回key对应的list的长度，如果key不存在返回0，如果key对应类型不是list返回错误。
4. **lrang key start end**：返回指定区间的元素，下表从0开始，负值表示从后面计算，-1表示倒数第一个元素，key不存在返回空列表。
5. **ltrim key index value**：截取list中指定区间的元素，成功返回1，key不存在返回错误。
6. **lset key index value**：设置list中指定下表的元素值，成功返回1，key或者下标不存在返回错误。
7. **lrem key count value**：从list的头部（count正数）或尾部（count负数）删除一定数量（count）匹配value的元素，返回删除的元素数量。count为0时候删除全部。
8. **lpop key**：从list的头部删除并返回删除元素。如果key对应的list不存在或者是空返回nil，如果key对应值不是list返回错误。
9. **rpop key**：从list尾部删除并返回删除元素。
10. **blpop key1 ... keyN timeout**：从左到右扫描，返回对第一个非空list进行lpop操作并返回。比如 blpop list1 list2 list3 0 ,如果list1不存在，list2，list3 都是非空则对 list2 做 lpop 并返回从 list2 中删除的元素。如果所有的 list 都是空或不存在，则会阻塞 timeout秒，timeout 为 0 表示一直阻塞。当阻塞时，如果有 client 对 key1...keyN 中的任意 key进行 push 操作，则第一在这个 key 上被阻塞的 client 会立即返回。如果超时发生，则返回nil。有点像 unix 的 select 或者 poll。
11. **brpop**：同blpop，一个是从头部删除，一个是从尾部删除。

<img alt="redis的使用-b3e45e7b.png" src="assets/redis的使用-b3e45e7b.png" width="" height="" >

### 2.2.4 set类型
set 是无序集合，最大可以包含(2 的 32 次方-1)个元素。set 的是通过 hash table 实现的，所以添加，删除，查找的复杂度都是 O(1)。hash table 会随着添加或者删除自动的调整大小。需要注意的是调整 hash table 大小时候需要同步（获取写锁）会阻塞其他读写操作。可能不久后就会改用跳表（skip list）来实现。跳表已经在 sorted sets 中使用了。关于 set 集合类型除了基本的添加删除操作，其它有用的操作还包含集合的取并集(union)，交集(intersection)，差集(difference)。通过这些操作可以很容易的实现 SNS 中的好友推荐和 blog 的 tag 功能。

**set类型数据操作指令简介**
1. **sadd key member**：添加一个string元素到key对应set集合中，成功返回1，如果元素已经在集合中则返回0，key对应的set不存在返回错误。
2. **srem key member**：从key对应的set中移除指定元素，成功返回1，如果member在集合中不存在或者key不存在返回0，如果key对应的不是set类型的值返回错误。
3. **spop key**：删除并返回key对应set中随机的一个元素，如果set是空或者key不存在返回nil。
4. **srandmember key**：同spop，随机去set中的一个元素，但是不删除元素。
5. **smove srckey dstkey member**：从srckey对应set中移除member并添加到dstkey对应的set中。整个操作是原子的，成功返回1，如果member在srckey中不存在返回0，如果key不是set类型返回错误。
6. **scard key**：返回set的元素个数，如果set是空或者key不存在返回0.
7. **siismember key member**：判断member是否在set中，存在返回1,0表示不存在或者key不存在。
8. **sinter key1 key2 ... keyN**：返回所有给定key的交集。
9. **sinterstore dstkey key1 key2 ... keyN**：返回所有给定key的交集，并保存交集到dstkey下。
10. **sunion key1 key2 ... keyN**：返回所有给定key的并集。
11. **sunionstore dstkey key1 key2 ... keyN**：返回所有给定key的并集，并保存并集到dstkey下。
12. **sdff key1 key2 ... keyN**：返回所有给定key的差集。
13. **sdffstore dstkey key1 key2 ... keyN**：返回所有给定的key的差集，并保存差集到dstkey下。
14. **smembers key**：返回key对应set的所有元素，结果是无序的。

<img alt="redis的使用-f8d6910c.png" src="assets/redis的使用-f8d6910c.png" width="" height="" >

### 2.2.5 sorted set类型
sorted set 是有序集合，它在 set 的基础上增加了一个顺序属性，这一属性在添加修改元素的时候可以指定，每次指定后，会自动重新按新的值调整顺序。可以理解了有两列的mysql 表，一列存 value，一列存顺序。操作中 key 理解为 sorted set 的名字。

**sorted set类型数据操作指令简介**
1. **add key score member**：添加元素到集合，元素在集合中存在则更新对应的score。
2. **zrem key member**：删除指定元素，1表示成功，如果元素不存在返回0.
3. **zincrby key incr member**：增加对应 member 的 score 值，然后移动元素并保持 skip list 保持有序。返回更新后的 score 值。
4. **zrank key member**：返回指定元素在集合中的排名（下标），集合中元素是按 score 从小到大排序的。
5. **zrevrank key member**：同上,但是集合中元素是按 score 从大到小排序。
6. **zrange key start end**：类似 lrange 操作从集合中去指定区间的元素。返回的是有序结果。
7. **zrevrange key start end**：同上，返回结果是按 score 逆序的。
8. **zrangebyscore key min max**:返回集合中 score 在给定区间的元素。
9. **zcount key min max**: 返回集合中 score 在给定区间的数量。
10. **zcard key**: 返回集合中元素个数。
11. **zscore key element**: 返回给定元素对应的 score。
12. **zremrangebyrank key min max**: 删除集合中排名在给定区间的元素。
13. **zremrangebyscore key min max**: 删除集合中 score 在给定区间的元素.
