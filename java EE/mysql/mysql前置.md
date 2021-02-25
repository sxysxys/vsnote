mysql安装

![截屏2021-02-24 11.37.27](mysql前置.assets/截屏2021-02-24 11.37.27.png)

服务器上的mysql是安装在docker中的。

## mysql配置文件

找到相应的my.cnf文件，进行相应的配置。

> 配置文件的组成

- `log-bin`：用于主从复制
- `log-error`：默认是关闭，记录启动和关闭的详细信息
- 慢查询日志
- frm文件：存放表结构
- myd：存放表数据
- myi：存放表索引

##  mysql的架构

![image-20210224133215228](mysql前置.assets/image-20210224133215228.png)

**插件式的存储引擎将查询和其他系统任务以及数据的存储提取相分离。**

1. 连接层
2. 服务层
3. 引擎层
4. 存储层（文件系统）

## mysql存储引擎

![截屏2021-02-24 13.40.49](mysql前置.assets/截屏2021-02-24 13.40.49.png)

## sql执行慢的原因

1. 查询语句写的烂
2. 索引失效
3. join
4. 服务器调优设置

> 手写sql顺序

![截屏2021-02-24 15.12.16](mysql前置.assets/截屏2021-02-24 15.12.16.png)

> mysql机读顺序

![截屏2021-02-24 14.52.43](mysql前置.assets/截屏2021-02-24 14.52.43.png)

![截屏2021-02-24 14.55.03](mysql前置.assets/截屏2021-02-24 14.55.03.png)

> join查询关联

七种情况

![截屏2021-02-24 15.15.35](mysql前置.assets/截屏2021-02-24 15.15.35.png)

![截屏2021-02-24 15.18.38](mysql前置.assets/截屏2021-02-24 15.18.38.png)



## Explain语句性能解析

> mysql常见瓶颈

1. cpu饱和
2. io瓶颈
3. 硬件性能

> explain语句作用

用来解析分析mysql是如何优化运行你的sql语句的。

![截屏2021-02-24 15.42.53](mysql前置.assets/截屏2021-02-24 15.42.53.png)

> 使用

explain+sql语句

> 表头格式

![截屏2021-02-24 15.48.02](mysql前置.assets/截屏2021-02-24 15.48.02.png)

- id：select语句操作的顺序（表的读取顺序）

  - 如果是子查询，id大的优先级高，越先执行
  - id相同的从上到下执行

-  select_type：执行的操作类型

  ![截屏2021-02-24 16.27.05](mysql前置.assets/截屏2021-02-24 16.27.05.png)

- type：

  https://www.cnblogs.com/benbenhan/articles/13212861.html

  ![截屏2021-02-24 16.43.05](mysql前置.assets/截屏2021-02-24 16.43.05.png)

  - ALL：此时字段上没有建立索引，并且全表扫描主键，性能最差。
  - system：表中只有一条记录（等于系统表）
  - const：将常量送到where后面，通过索引一次就找到了，例如主键索引和唯一索引
  - eq_ref：和const不同在于const指定检索常量，而eq_ref一般用于联表中，后者扫描每一行，而前者只需要针对每一行的值进行唯一性检索
  - ref：非唯一性的索引检索，此时如果使用非唯一索引进行扫描，常量也会变成ref
  - range：使用索引检索的时候使用了between这种语句，这样需要通过索引定位到初始值，然后一个一个开始扫描。
  - index：在索引上进行全表扫描，例如扫描全表主键id。

- possible_keys和key
  - `possible_keys`显示可能应用在这张表中的索引，但是不一定实际查询的时候真实使用了。
  - key：实际最终用到的索引

- key_len：索引的长度

- ref： 此时用索引的值对应的到底是什么，也就是索引匹配的值是什么

- rows：估算需要查找多少行

  ![截屏2021-02-24 20.52.49](mysql前置.assets/截屏2021-02-24 20.52.49.png)

- extra：

  - `Using fileSort`: 排序时没有使用到索引，性能差![截屏2021-02-24 21.25.43](mysql前置.assets/截屏2021-02-24 21.25.43.png)
  - `Using temporary`：使用临时表，性能差![截屏2021-02-24 21.33.26](mysql前置.assets/截屏2021-02-24 21.33.26.png)
  - `Using index`：![截屏2021-02-24 21.38.20](mysql前置.assets/截屏2021-02-24 21.38.20.png)![截屏2021-02-24 21.40.13](mysql前置.assets/截屏2021-02-24 21.40.13.png)
  - `using join buffer`：使用了连接缓存（为什么少用join）
  - `impossible where`：where总是false

![截屏2021-02-24 21.45.07](mysql前置.assets/截屏2021-02-24 21.45.07.png)

## 索引

### 1. 原理

> 数据结构

使用b+树，由于其数据块与磁盘匹配，极大减少了io次数。

https://time.geekbang.org/column/article/69236

> 几个索引概念

1. 单值索引：一个索引只包含单列
2. 唯一索引：索引列的值必须唯一
3. 复合索引：索引列包含多列

> 语法

![截屏2021-02-25 10.38.25](mysql前置.assets/截屏2021-02-25 10.38.25.png)

> **覆盖索引**

​	如果执行的语句是 select ID from T where k between 3 and 5，这时只需要查 ID 的值，而 ID 的值已经在 k 索引树上了，因此可以直接提供查询结果，不需要回表。也就是说，在这个查询里面，索引 k 已经“覆盖了”我们的查询需求，我们称为覆盖索引。

作用：减少树的搜索次数，提高性能

![截屏2021-02-24 20.15.00](mysql前置.assets/截屏2021-02-24 20.15.00.png)

> 前缀索引

最左索引原则

> 索引下推

mysql5.6后，可以判断索引后再回表

https://time.geekbang.org/column/article/69636

### 2. 索引优缺点

- 优点：排序查找，查找的时候增加了速度。
- 缺点：占用空间、降低更新插入速度

> 建立索引规则

**1. 适合建立索引：**

- 主键
- 频繁查询
- 外键
- 按顺序order by查询
- 按顺序group by

**2. 不适合建立索引：**

- 频繁更新
- where用不到
- 表记录太少
- 某字段如果大量重复（例如：国籍中国）

> 注意事项

1. 尽量选择组合索引（注意最左前缀原则）
2. 查询中排序`order by`字段，如果按照索引顺序去访问将大大提高排序速度![截屏2021-02-25 10.51.11](mysql前置.assets/截屏2021-02-25 10.51.11.png)

### 3. 索引优化案例

> 单表优化

- 建表

```sql
CREATE TABLE IF NOT EXISTS `article` (
`id` INT(10) UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT, `author_id` INT(10) UNSIGNED NOT NULL, `category_id` INT(10) UNSIGNED NOT NULL, `views` INT(10) UNSIGNED NOT NULL, `comments` INT(10) UNSIGNED NOT NULL, `title`VARBINARY(255) NOT NULL, `content` TEXT NOT NULL);
INSERT INTO `article`(`author_id`,`category_id`, `views`, comments, title, `content`) VALUES
(1, 1, 1, 1,'1', '1'),
(2, 2, 2, 2, '2', '2'),
(1, 1, 3, 3, '3','3');
```

- 需要优化的sql：

  ```sql
  explain select id, author_id from article where category_id=1 and comments > 1 order by views limit 1;
  ```

- 建立索引：

  ```sql
  create index idx_article_ccv on article(comments, category_id, views);
  
  mysql> explain select id, author_id from article where comments > 1 and category_id = 1 order by views limit 1;
  +----+-------------+---------+------------+-------+-----------------+-----------------+---------+------+------+----------+---------------------------------------+
  | id | select_type | table   | partitions | type  | possible_keys   | key             | key_len | ref  | rows | filtered | Extra                                 |
  +----+-------------+---------+------------+-------+-----------------+-----------------+---------+------+------+----------+---------------------------------------+
  |  1 | SIMPLE      | article | NULL       | range | idx_article_ccv | idx_article_ccv | 4       | NULL |    2 |    33.33 | Using index condition; Using filesort |
  +----+-------------+---------+------------+-------+-----------------+-----------------+---------+------+------+----------+---------------------------------------+
  1 row in set, 1 warning (0.00 sec)
  // 此时用到了文件排序，需要进行优化
  ```

- 优化索引

  ```sql
  create index idx_article_cv on article(category_id, views);
  
  +----+-------------+---------+------------+------+--------------------------------+----------------+---------+-------+------+----------+-------------+
  | id | select_type | table   | partitions | type | possible_keys                  | key            | key_len | ref   | rows | filtered | Extra       |
  +----+-------------+---------+------------+------+--------------------------------+----------------+---------+-------+------+----------+-------------+
  |  1 | SIMPLE      | article | NULL       | ref  | idx_article_ccv,idx_article_cv | idx_article_cv | 4       | const |    2 |    66.67 | Using where |
  +----+-------------+---------+------------+------+--------------------------------+----------------+---------+-------+------+----------+-------------+
  1 row in set, 1 warning (0.00 sec)
  // 此时文件不再排序
  ```

> 两表优化

结论：左连接要在右表建立索引，右连接在左表建立索引。

因为对于左连接来说，左边必须要全部遍历，而在左边加索引其实也就是将all变成了index，而对于右边而言，是ref和all的区别。

> 三表优化

和两表连接类似。

> 结论

1. 使用小结果驱动大结果集，也就是在联表的时候，需要将数量小的表放在连接的驱动一侧，进行全表扫描。
2. 保证被join表用上索引。
3. 优先优化内层循环。

### 4. 索引失效

![截屏2021-02-25 15.04.38](mysql前置.assets/截屏2021-02-25 15.04.38.png)

1. 如果不使用最佳左前缀法则，也就是没有最左字段的搜索，那么会变成全表扫描all；并且尽量不跳过索引中的列，如果跳过，那么索引不完全。（带头大哥不能死，中间兄弟不能断）![截屏2021-02-25 15.14.43](mysql前置.assets/截屏2021-02-25 15.14.43.png)

2. 索引上不能使用函数。![截屏2021-02-25 15.19.54](mysql前置.assets/截屏2021-02-25 15.19.54.png)

3. 范围之后全失效。![截屏2021-02-25 15.23.44](mysql前置.assets/截屏2021-02-25 15.23.44.png)

4. 尽量使用覆盖索引。

5. 索引中不能使用符号（例如>  !  <>  is null等）

   `explain select * from staffs where name != 'July'`

   `explain select * from staffs where name is null`

6. 在使用like语句的时候，需要把%放在右边才能使用到索引。

   > 如何解决`%xx%`无法使用索引的问题？

   **使用覆盖索引即可，如果查询结果在索引里，那么此时mysql会优化成使用使用索引遍历index；而如果结果不在覆盖索引中，mysql认为如果使用索引还要进行回表，那么此时会选择主键遍历all。**

7. varchar类型不能失去单引号，失去单引号会导致索引失效，因为mysql进行了整数到字符串的转换，也就是进行了第2条的操作（使用了函数）。

8. 使用or也有可能导致索引失效：

   `explain select * from staffs where name='July' or name='z3'`，在or的时候，mysql也要全表遍历，如果优化成index的话，由于此时返回的*，那么需要回表，此时mysql可能会选择all。



### 5. 面试题



































