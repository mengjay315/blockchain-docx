## postgresql数据库使用

### 同时返回查询总数及分页数据

#### SQL性能优化第一篇之分页数据与Count数据一次性获取

https://blog.csdn.net/qq_31122833/article/details/83894509

### 怎样处理从后台传来大量数据的分页操作

### https://www.cnblogs.com/zttDream/p/6688541.html

### 在分页查询的情况下，前端或者移动如何处理后端数据的实时更新

### https://segmentfault.com/q/1010000007334996

### Mysql分页查询获取totalCount大幅提升性能的办法总结

https://blog.csdn.net/qq_34325438/article/details/83095495

### 大数据量实时统计排序分页查询(并发数较小时)的几点建议

https://blog.csdn.net/anmuxf7123/article/details/101460987

### 【MySQL】大数据统计 - 优化GROUP BY的使用

https://blog.csdn.net/weixin_38762114/article/details/79005583

### Mysql查询优化——中间表方法优化count()统计大数据量总数问题

https://blog.csdn.net/zhangdaiscott/article/details/60584036

### PostgreSQL分页count超级无敌巨慢

https://www.jiweichengzhu.com/article/64a2b98c1b284923bc25f513f1e9d92a

### **关于select Count()的使用和性能问题**

https://blog.csdn.net/qq_15037231/article/details/79351268

### **高性能MySQL之Count统计查询**

https://blog.csdn.net/qq_15037231/article/details/81179383

### **select count(*)底层究竟做了什么？**

https://blog.csdn.net/Dome_/article/details/88929215

## 临时解决方案：

1、利用pg_class查询，但是却无法带上where条件，全表扫描时可以考虑；

2、建立计数表，insert和delete之后，实时更新数量；

3、**定时查询符合条件的数量，存入缓存；**

此问题，可以说是依旧没有解决，我选择的第三种方案，只不过是避开了实时查询慢的问题，在定时器中查询的时候，依旧很慢，不过这个慢只有代码知道，对于用户是无感的！

-----

每页显示多少条数据，pageNumber:页数， pageSize：每页显示多少条数据

这个pageSize应该是前端传给我的吗？



