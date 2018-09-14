### mongodb 性能优化

##### 常用的分析工具

###### mongotop
`mongotop --host host:port -u user -p pswd --authenticationDatabase=admin`
> 可以查看mongo执行操作数，连接数，使用内存情况
###### mongostat
`mongostat --host host:port -u user -p pswd --authenticationDatabase=admin`
> 可以查看mongo每个collection占用cpu的时间

> `db.serverStatus() -- 查询当前db状态信息`
> `db.collection.stats() -- 某个collection的统计信息，比如平均document大小`

###### system.profile 排查慢查询语句
> `db.getProfilingLevel() -- 查询慢查询跟踪级别 {0-关闭， 1-定义慢查询操作，2-全部跟踪}`
> `db.setProfilingLevel(1,1000)  -- 设备慢查询跟踪级别为1,1000ms阈值`
> `db.getProfilingStatus()  -- 查询当前设置`
> `db.system.profile.find().sort({$natrual: -1}).limit(3) -- 查询最佳的慢查询语句`

###### explain 查询计划分析
`db.collection.find({...}).explain(true)`
> 分析查询计划结果，看看是否有**COLLSCAN**，还有就是查询扫描的Document数totalDocsExamined，正常情况下，totalDocsExamined应该很接近于nReturned，否则就是索引建立不当



### mongodb 一些注意点
###### mongodb索引 和 mysql索引
> mongodb和mysql查询引擎在使用索引时有很大的差异

mysql中，我们可以建立单独字段的索引，很多时候不需要创建联合索引，mysql查询引擎，在执行查询的时候，会尽可能多的使用到我们创建的索引，提高查询效率

mongodb查询引擎，在执行查询的时候，通常只会命中一个最优的索引，不会用到其他的索引，
```
例如我们要查询 name=a,age=b 的数据：
我们分别对name,age建立索引，查询是，MongoDB会先通过name索引查询出所有的document，然后使用age=b进行filter，若name=a的数量比较大，索引的作用就不明显了

但如果我们建立了name和age的联合索引，MongoDB就会通过name和age的联合索引查询到数据

```





###### mongodb kill ops
有个很耗时的操作，我们可以通过如下方法，结束这个耗时操作
```
1. db.currentOp() 找到opId
2. db.killOp(opId) 
```

###### mongodb 创建索引，删除索引
mongodb创建索引，最好在collection创建的时候，就创建好；
在已存在大量数据的collection上面创建索引,删除索引，需要很谨慎

1. 创建索引默认会锁库，期间所有的操作都会失败
2. 就算使用后台方法创建索引，即使不锁库，也很占用cpu
3. 删除索引时，一定要确认没有查询会用到它，如果删除了一个查询会用到的索引，那就悲剧了，大量的慢查询，导致cpu利用率很高，最好mongodb挂掉
4. 如果要优化索引，那就先创建索引，然后在删除之前的那个索引


###### mongodb中skip,limit
skip,limit可以用于分页查询。
skip并不能提高查询效率，并且skip越大，查询效率越低。






### Link
- [mongo常用运维.pdf](https://github.com/meacial/mongo_note/blob/master/mongo%E5%B8%B8%E7%94%A8%E8%BF%90%E7%BB%B4.pdf)
