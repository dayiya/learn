# learn
Self-compiled technical documents related to java learning，This is for your reference only.
与JAVA学习相关的自编技术文档，仅供参考。
##redis的使用
###String类型
    获取所有的value：keys *
    设置值：set key value ，设置key多次会覆盖
    取值：get key
    删除值：del key；setnx（not exist）：setnx age 19如果不存在进行设置，存在就不需要设置，返回0
    设置过期时间：setnx key time value；setex color 10 red 设置color的有效期为10秒，10秒后返回nil（在redis里面表示为空）
    替换字符串:setrange；set email 123456789@qq.com；setrange email 10 163.com 10表示从第几位开始替换后面的字符串
    mset:设置多个值。mget获取多个值 mset k1 v1 k2 v2 k3 v3  mget k1 k2 k3
    返回旧值并设置新值的方法：getset key newValue
    对某一个值进行递增和递减：incr key； decr key；其中value须为Integer类型
    对某个值进行指定长度的递增和递减：incrby key [步长]； decrby key [步长]
    字符串追加方法：append[key]
    获取字符串长度：strlen[key]
