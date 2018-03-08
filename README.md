一、背景
在平时的后台开发中，直接访问数据库，导致数据库资源被占用的原因大致有三种：1.使用复杂SQL语句，如join,leftjoin。 2.对无索引字段筛选结果，包括对索引字段使用函数。3. 查询返回大量数据结果。这些操作都将导致数据访问速度慢或者网络传输慢，影响与数据库交互性能。
二、解决的问题
针对2情况，可以在数据库里对相应字段加索引，唯一索引和普通索引使用的结构都是B-tree,执行时间复杂度都是O(logn)。针对3情况，一般避免返回大量结果。针对1情况，常用的方法是使用中间层或者将查询放到主机上运算，如使用redis做缓存等方法。本组件参考SQL语法设计出一套表格数据结构和相应SQL操作库，主要针对1情况将结果缓存到本地主机，使用相应成员方法，完成类似SQL语法操作，同时大大降低跨表查询数据的组合难度。
三、举例
例如对数据库表查询的筛选条件，要跨表A,B,C,D，常用的方法是根据业务逻辑，由小到大筛选出表格的子集，然后用in键值顺序得出最终子集，例如Qa<Qb<Qc<Qd，根据两表之间的关联键值如iUserID，从小到大筛选出最终的结果，可能是Qa & Qb & Qc & Qd(集合的与操作)，最终拼接这4个子集；
如果遇到复杂的业务逻辑，各个子集不能区分出大小，导致在合并各个子集数据时还要再去掉的部分冗余数据；
另一个常用的优点在于，如果要带出除了主集合A字段信息外，还要提取出关联集合B的其他字段，只需要将两张表join或者leftjoin即可，这种表格操作方式十分方便提取，大大减轻开发难度。
使用该组件，针对上述4个集合，只需要使用pT_All = pTA.join(pTB,keyJoin).join(pTC,keyJoin).join(pTAD,keyJoin)即可，业务逻辑实现简单易懂。
同理，和普通SQL查询要求一样，每个集合Qa,Qb,Qc,Qd必须经过严格筛选，切忌筛选出大量数据，例如经纪人助理权限时，将筛选出所有经纪人对应的所有会员，这将严重拖慢数据库返回速度，得不偿失。
四、优缺点
优点：
1、将原始数据采集到本地微服务中运算，降低数据库计算负担，尤其是类似如join、leftjoin、group by等SQL函数
2、降低程序针对业务逻辑的设计复杂度，降低程序开发困难，降低bug出现概率
3、针对查询和统计模块，只需要将大量原始数据放在后台进行连表，集合，统计处理，从而避免在统计维度众多时需要多次数据库查询操作，在跨表搜索时，信息检索、信息拼接等繁杂问题
缺点：
1、针对优点1，可能发生采集到大量数据到本地主机情况，反而导致数据库因返回大量数据而降低性能，需要人工优化避免
2、当存在跨表筛选条件时，如果不区分集合大小，要想返回固定条数的最终结果可能会少；因为join后的最终结果有可能是各个集合的真子集，即结果少于参与各个join的集合；对于这个问题，还是需要对主表最后进行筛选，其他表可以预先进行join。

