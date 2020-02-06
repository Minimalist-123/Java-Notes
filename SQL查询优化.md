数据库服务器优化步骤
整个流程划分成了观察（Show status）和行动（Action）两个部分。字母 S 的部分代表观察（会使用相应的分析工具），字母 A 代表的部分是行动（对应分析可以采取的行动）。


image.png
在S1部分，需要观察服务器的状态是否存在周期性的波动。如果存在周期性的波动，有可能是周期性节点的原因，比如双十一，促销活动等。这样的话，可以通过A1 加缓存或者更改缓存失效策略；
如果缓存策略没有解决，或者不是周期性波动的原因，需要进一步分析查询延迟和卡顿的原因。进行S2这一步，此时需要打开慢查询。慢查询可以帮我们定位执行慢的SQL语句。通过设置long_query_time参数定义“慢”的阈值，如果SQL执行时间超过了long_query_time，则会认为是慢查询。收集到慢查询后，通过分析工具对慢查询日志进行分析；
知道了执行慢的SQL语句之后，可以针对性地用EXPLAIN查看对应SQL语句的执行计划，或者使用SHOW PROFILE查看SQL中每一个步骤的时间成本。这样就可以了解SQL查询慢是因为执行时间长，还是等待时间长；
如果是SQL等待时间长，则进入A2步骤中。可以调优服务器的参数，比如适当增加数据库缓冲池等。如果是SQL执行时间长，就可以进入A3。此时需要考虑是索引涉及的问题？还是查询关联的数据表过多？还是因为数据表的字段设计问题。
如果A2和A3都不能解决问题，需要思考数据库自身的SQL查询性能是否已经达到了瓶颈，如果没有达到性能瓶颈，需要重新检查。若已经到达了性能瓶颈，进入A4阶段，需要考虑增加服务器，采用读写分离的架构，或者考虑对数据库分库分表等。
SQL调优的三个步骤
获取慢查询SQL

// 查看慢查询是否开启，以及慢查询日志文件的位置
mysql > show variables like '%slow_query_log';
// 开启慢查询
mysql > set global show_query_log='ON'；
// 查看慢查询的时间阈值
mysql > show variables like '%long_query_time%';
// 设置慢查询的时间阈值
mysql > set global long_query_time = 3;
通过mysqldumpslow工具（这个工具是个 Perl 脚本，你需要先安装好 Perl）提取想要查询的SQL语句，mysqldumpslow命令的具体参数如下：

-s：采用 order 排序的方式，排序方式可以有以下几种。分别是 c（访问次数）、t（查询时间）、l（锁定时间）、r（返回记录）、ac（平均查询次数）、al（平均锁定时间）、ar（平均返回记录数）和 at（平均查询时间）。其中 at 为默认排序方式；
-t：返回前 N 条数据 ；
-g：后面可以是正则表达式，对大小写不敏感；
使用EXPLAIN查看执行计划
EXPLAIN 可以帮助我们了解数据表的读取顺序、SELECT 子句的类型、数据表的访问类型、可使用的索引、实际使用的索引、使用的索引长度、上一个表的连接匹配条件、被优化器查询的行的数量以及额外的信息（比如是否使用了外部排序，是否使用了临时表等）等

mysql >  EXPLAIN SELECT comment_id, product_id, comment_text, product_comment.user_id, user_name FROM product_comment JOIN user on product_comment.user_id = user.user_id ;
image.png

数据表的访问类型所对应的 type 列是我们比较关注的信息。type 可能有以下几种情况：
image.png
all 是最坏的情况，因为采用了全表扫描的方式。index 和 all 差不多，只不过 index 对索引表进行全扫描，这样做的好处是不再需要对数据进行排序，但是开销依然很大；
range 表示采用了索引范围扫描，这里不进行举例，从这一级别开始，索引的作用会越来越明显，因此我们需要尽量让 SQL 查询可以使用到 range 这一级别及以上的 type 访问方式；
使用SHOW PROFILE查看SQL的具体执行成本
SHOW PROFILE 相比 EXPLAIN 能看到更进一步的执行解析，包括 SQL 都做了什么、所花费的时间等。

// 查看profiling是否开启
mysql > show variables like 'profiling';
// 开启profiling
mysql > set profiling = 'ON'；
// 查看当前会话都有哪些profies
mysql > show profiles;
// 查看上一个查询的开销
mysql > show profile;
// 查看指定Query ID的开销
mysql  > show profile for query 2;
image.png



作者：LittleJessy
链接：https://www.jianshu.com/p/28738a2eaa03
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
