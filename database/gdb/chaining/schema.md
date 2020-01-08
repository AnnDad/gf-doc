
# 数据库切换

我们知道数据库的配置中有支持对默认数据库的配置，因此`DB`对象及`Model`对象在初始化的时候已经绑定到了特定的数据库上。运行时切换数据库有几种方案（假如我们的数据库有`user`用户数据库和`order`订单数据库）：
1. 通过不同的配置分组来实现。这需要在配置文件中配置不同的分组配置，随后在程序中可以通过`g.DB("分组名称")`来获取特定数据库的单例对象。
1. 通过运行时`DB.SetSchema`方法切换单例对象的数据库，需要注意的是由于修改的是单例对象的数据库配置，因此影响是全局的。
    ```go
    g.DB().SetSchema("user-schema")
    g.DB().SetSchema("order-schema")
    ```
1. 通过链式操作`Model.Schema`方法设置当前链式操作对应的数据库，没有设置的情况下使用的是其`DB`或者`TX`默认连接的数据库。
    ```go
    db.Table("user").SetSchema("user-schema").All()
    db.Table("order").SetSchema("order-schema").All()
    ```
1. 此外，假如当前数据库操作配置的用户有权限，那么可以直接通过表名中带数据库名称实现跨域操作，甚至跨域关联查询。
    ```go
    // SELECT * FROM `order`.`order` o LEFT JOIN `user`.`user` u ON (o.uid=u.id) WHERE u.id=1 LIMIT 1
    db.Table("order.order o").LeftJoin("user.user u", "o.uid=u.id").Where("u.id", 1).One()
    ```








