## go-redis做缓存

### Golang Gin实践-优化你的应用结构和实现Redis缓存

https://segmentfault.com/a/1190000015140508)

https://book.eddycjy.com/golang/gin/application-redis.html

-----

### mac安装redis

brew install redis

启动redis

这个命令会在后台启动redis服务，并且每一次登录系统，都会自动重启

brew services start redis

假如你不需要后台启动服务，你可以使用配置文件启动：

```
redis``-``server ``/``usr``/``local``/``etc``/``redis.conf
```

这个命令会读取redis的配置文件，并且在redis运行的过程中也会看到实时的日志打印。启动成功，

### 连接redis

```
# 不需要身份认证时
redis-cli -p 6379 -h 127.0.0.1
```

 

```
# 需要身份认证时，输入如下命令
redis-cli -p 6379 -h 127.0.0.1 -a yourpassword
# or
redis-cli -p 6379 -h 127.0.0.1
# 登录进去之后再进行身份认证
127.0.0.1:6379> auth 0903
```

-------

### mySql与Redis做二级缓存

https://www.jianshu.com/p/d8dad949057b

https://www.jb51.net/article/155802.htm

### go操作redis

https://www.cnblogs.com/zhaohaiyu/p/11626969.html)

### Go 分布式缓存方案

https://www.v2ex.com/t/595606

https://github.com/seaguest/cache

主要是内存+redis 二级缓存，支持多机同步

内存部分是采用的是 sync.Map, 读取缓存的时候先从内存读，如果未读到则去读 redis，如果 redis 未读到，则根据定义的加载函数加载到 redis 和内存。

缓存有 lazy 模式，为了避免缓存被击穿，可以设置 lazy 模式，缓存数据存活的时间更久，但是每次读取的时候依然会判断数据是否是最新的，不是最新的话会异步加载更新。

通过 redis 的 Publish/Subscribe 功能，实现缓存的分布式更新，目前仅实现删除同步。

--------

### Golang之redis

https://www.cnblogs.com/pyyu/p/8341946.html

### 基于Go语言与MySQL和Redis的交互实现二级缓存的小案例

https://blog.csdn.net/lili9415/article/details/100132065

