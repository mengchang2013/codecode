# Redis

## 数据类型
String(字符串)、hash(哈希)、list(列表)、set(集合)、zset(sorted set:有序集合)
## 命令
```shell
keys # 打印所有的key(切记不能用在生产环境)
dbsize #显示key总数
exists key #判断key是否存在（1-是，0-否）
del key [key ...] #删除key,允许指定多个key
expire key seconds #设置key过期时间
ttl key #显示key剩余的过期时间
persist key #去除key过期时间
type key #返回key的类型
```
