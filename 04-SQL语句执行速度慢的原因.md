# SQL语句执行速度慢的原因剖析
## 大多数情况是正常，偶尔很慢
### mysql正在刷脏块,即进行redo日志切换
- redo log日记的作用是存储更新的数据,但容量有限;再将存储数据同步到磁盘
- MySQL在改动数据时,只是暂时在内存中改变了数据形态,而磁盘中的数据要在redo log空闲时进行改变.
- 当数据库一直处于更新修改状态时,redo log日记会达到饱满状态,这时会暂停运行数据库,等redo log日记中的数据同步,所以速度会变慢
### 执行的时候遇到锁
- 在调用某一行或某张表时,其他人也在用,并且锁住了行或表,这时也需要等
- show processlist:查看当前状态
## 在数据量不变的情况下，这条SQL语句一直执行慢
####  字段没有索引
- 索引的目的是提高你的查询速度
- 在没有加索引时,需要进行全表查询,降低速度
> select * from 表 where 100 <字段 and 字段 < 100000

####  字段有索引,但写法不正确
当出现下面的写法时,仍然不会使用索引,因为系统不会帮你自动计算,只能写成:字段= 10+1
> select * from 表 where 字段 - 1 = 10

####  使用SQL里不存在的函数
当写语句时使用的函数不存在在数据库中时,则会索引失败
> select * from 表 where pow(字段,2) = 10

####  在有索引的情况下,系统也可能判断错误,走全表扫描
-  走全表扫描,即扫描整表的总行数 n
-  走索引时,如果索引中字段 c 满足的条件行数为接近或等于全表行数,找到 c 对应的主键后,还需要通过主键索引找到整行数据,这时走 c 索引不仅扫描的行数是近似n，同时还得每行数据走两次索引。
-  在系统预测到索引需要走的行数很多时,有可能不走索引,直接走全表
-  当系统预测索引基数出现差错时,在有多个索引的条件下,系统可能选错索引.
-  当索引中需要使用临时表,则可能出现上述redo log日记存储满了需要等待的情况,此时系统通过判断扫面时间进行判断                                        
-  系统还要考虑是否需要排序                                                                                                                       
 *系统如何预测判断*
-  通过索引的区分度来判断的，一个索引上不同的值越多，意味着出现相同数值的索引越少，意味着索引的区分度越高。我们也把区分度称之为基数，即区分度越高，基数越大
-  索引系统通过遍历部分数据，即采样的方式，来预测索引的基数
-  当采样出现失误,即采样基数很大,但被预测成很小时,便走全表扫描了         

*附*                                                                                                                                                                     
通过强制走索引的方式来查询，eg:
>  select * from t force index(a) where c < 100 and c < 100000;
或
> show index from t;                                                                                                                      
查询索引的基数和实际是否符合                                                                                                                                                
- 如果和实际很不符合的话，可以重新统计索引的基数:                                                            
> analyze table t;                                                                                    
最后重新统计分析。                                                                 








