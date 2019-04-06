### 数据库规范


**创建**

写 SQL 文件，创建应用的所有数据表。比如叫 `create.sql`。

比如其中一个表：

```
CREATE TABLE `tb_total_assets` (
  `id` int unsigned NOT NULL AUTO_INCREMENT,
  `uid` int unsigned NOT NULL COMMENT '用户id',
  `projectId` int unsigned NOT NULL COMMENT '项目id',
  `assets` double NOT NULL COMMENT '资产数额',
  `createdAt` timestamp NULL DEFAULT NULL,
  `updatedAt` timestamp NULL DEFAULT NULL,
  `deletedAt` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_tb_total_assets_uid` (`uid`)
) ENGINE=MyISAM AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
```

里面涉及到的几个注意点：

+ 搞清楚 MySQL（或者使用的其他数据库）有哪些数据类型，具体的使用场景是什么（比如 tinyint，decimal 等等类型）
+ 使用 COMMENT 进行注释
+ 对于在应用中会被作为 Where 条件查询的列，可以加索引
+ 设置字符集：CHARSET=utf8;
+ 认真思考字段是否允许 NULL 值，有没有默认值

在应用数据库初始化时直接运行 `create.sql` 就可以。

**外键**

不使用外键。具体讨论可以看：[知乎](https://www.zhihu.com/search?type=content&q=%E4%B8%8D%E7%94%A8%E5%A4%96%E9%94%AE)

需要注意的是，**在应用中我们需要自行维护出现在多个表中的实体的更新和删除**。比如你有一个商品名更新了，不仅要更新商品表，还要更新有商品名字段的其他表。

**表结构变更**

写一个 SQL，比如：

```
ALTER TABLE table_name
ADD column_name datatype
```

然后交给值班 DBA 同学，在应用上线之前停机更新时进行变更处理（涉及数据库变更必须停机上线）。

这个过程必须有文档记录，包括变更的 SQL，变更的原因等等。等于是给 DBA 提交一个工单。DBA 同学如果发现 SQL 有问题，也需要进行把关。

**事务**

如果涉及一个关键变量的并发读写（比如商品库存），需要用事务对更新操作进行隔离。

**ORM 使用**

我们用的 ORM 是 [gorm]()。ORM 在我们的使用场景中，主要起的作用有两个：

+ 一、数据库数据到 Go struct 的双向映射。简单的说就是把查出来的数据自动绑定到一个结构体上。插入数据的时候同理。
+ 二、通过一些函数调用生成 SQL 来查询数据库。简单的说就是通过一些 Find，Select 之类的 API 来进行查询，让我们不用写裸 SQL。

此外的 ORM 功能，基本就很少用了。特别是 ORM 自动创建数据库和数据迁移，这部分我们会手动完成，来对数据库形成更好的控制。

> 题外话：此前我们用 Flask 的时候对 ORM 依赖非常严重。相比之下 gorm 基本和裸写 SQL 区别不大，我们在做一些复杂查询的时候基本也是首先直接写 SQL 来看看查询的正确性。熟悉 SQL 是非常重要的，后端主流还是不会对 SQL 那边做太多的封装。毕竟你的查询进入了数据库，还要经过查询引擎的各种优化。所以如果你的查询本身都是生成出来的，这个就太黑盒了。直接写表结构创建的 SQL 也是一个道理，我们需要对表结构有直接的控制。多写写 SQL 也会让我们对数据库更加了解。



