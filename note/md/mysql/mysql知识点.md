1. 根据user表新建表user1，并复制其数据

    > create table user1 as select * from user; # user部分克隆user1表结构

2. 根据user表新建表user2，不复制其数据

    * > create table user2 as select * from user where 1=2; # user2部分克隆user表结构
    * > create table user2 like user; # user2完整克隆user表结构

3. 在表结构相同时将user1的数据复制到user2表中

    > insert into user2 select * from user1

4. 将user1表的指定字段的值复制到user2的指定字段中

    > insert into user2(name) select name from user1

5. 删除表中数据

    * > truncate table user3; # 对于有外键约束的数据直接报错、不能加where、属于DML语句、操作后直接生效不会放到事物中、不能回滚也不能触发触发器

6. 删除整个表
    * > drop table user3; # 对于有外键约束的数据直接报错、不能加where、属于DML语句、操作后直接生效不会放到事物中、不能回滚也不能触发触发器

7. 区别

    * 删除内容

        > drop操作会删除表结构、依赖的约束、索引以及触发器，并且会将依赖该表的所有存储过程和视图设置为invalid。truncate和delete则是只会删除表中的数据，并不会删除表结构。因此如果该表以后不再需要的话可以使用drop，而如果后续还需要的话可以通过truncate或delete，因为这样可以不需要再重新建立表。

    * 删除空间

        > delete操作并不会更改所占用的区的空间，高水位线不会发生改变。drop操作就会直接删除整个表空间。truncate则是相对于先执行drop操作，然后再执行create操作，执行完成后会恢复初始的表空间。

    * 语句类型

        > delete 语句是DML语句，这个操作会放到 rollback segement 中，事务提交之后才生效，并且可以执行对应的触发器。truncate、drop是DDL操作，会包含implicit commit，因此不能回滚，也不能不触发触发器。

    * 效率

        > 从第二点的描述中可以推断truncate的执行效率要低于drop操作。delete操作则是按照行记录一行一行的进行删除，因此其效率更低。

    * 安全性

        > 使用 drop 和 truncate会导致整个表中的数据都被删除，需格外注意。如果仅想删除部分数据可用 delete，但是需要注意where子句得到的范围，会占用rollback segement。
