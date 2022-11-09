---
title: "Gorm使用补充点"
date: 2022-08-28T12:01:57+08:00
tags: []
categories: [MySQL,gorm,数据库]
draft: false
---

本文主要补充一些gorm的使用技巧，完整的gorm使用移步官方文档[gorm官方文档](https://gorm.io/zh_CN/docs/index.html)

## 总结

1. 配置单数表名, 再也不用写TableName

   ```go
   db, err := gorm.Open(mysql.Open(fmt.Sprintf(dsn, un, pwd, host, port, database)), &gorm.Config{
   	NamingStrategy: schema.NamingStrategy{SingularTable: true},
   })
   ```

   当然也可以传入其他参数定制命名策略

2. 创建模型迁移表时，针对`string`类型一定要给出gorm的`type`约束，否则默认是创建mysql的字符串最大数据类型`longtext`较为浪费资源

3. 使用DB前先`db.model`, 万无一失

4. 创建模型时使用基本类型的指针类型，可以使得零值保存到数据库

5. `CreatedAt`, `UpdatedAt`, mysql类型用`datetime(3)`(毫秒)

6. 支持`db.Create`创建和批量创建，创建成功时回填结构体里主键字段或结构体切片每个单元的主键字段

7. `Save`方法没有查找到，就会创建记录；查找到就会以`Save`里的参数替换整行数据，即使是零值也会替换

8. `BeforeCreate`创建前自动填写id

9. `AfterUpdate`更新后自动创建record

10. `Where`用结构体查询, 自动忽略零值, 不用手打列名

11. `ErrRecordNotFound`只会出现在`Take`, `First`,`Last`方法中，如果发生了多个错误，你可以通过 `errors.Is` 判断错误是否为 `ErrRecordNotFound`

12. `Find`结构体时: 等同于`Take`, 但没有`ErrRecordNotFound`, `RowsAffected`为`0`或`1`

13. 行锁的写法`DB.Clauses(clause.Locking{Strength: "UPDATE")`

14. `Updates`支持结构体更新, 自动忽略零值, 不用手打列名. 强行更新零值用`Select`

15. `Update`支持`gorm.Expr`SQL表达式更新

16. 事务直接用`db.Transaction`方法即可

17. `Update`更新时，如果更新失败，但是返回不一定报错。需要综合检测 `RowsAffected`

18. 零值创建、更新、作为条件等时，使用`struct`会失效，应该使用`map[string]interface{}`来指定。

19. 使用`map[string]interface{}`还可以使用SQL表达式

20. gorm外键多表联查时可以考虑使用`Preload`预加载或嵌套预加载

21. `Pluck` 用于从数据库查询单个列，并将结果扫描到切片。如果您想要查询多列，您应该使用 `Select` 和 [`Scan`](https://gorm.io/zh_CN/docs/query.html#scan)。某些业务场景下，这个`Pluck`方法很有用

22. gorm支持查询创建更新删除等前后的钩子函数

23. gorm不仅支持创建多条记录，还支持分批创建多条记录（`CreateInBatches`）或分批查询多条记录（`FindInBatches`）

24. `Scopes` 允许你指定常用的查询，可以在调用方法时引用这些查询。可以根据业务需要集成某些常用的业务查询条件。

    作用域允许你复用通用的逻辑，这种共享逻辑需要定义为类型`func(*gorm.DB) *gorm.DB`。

    ```go
    func AmountGreaterThan1000(db *gorm.DB) *gorm.DB {
      return db.Where("amount > ?", 1000)
    }
    
    func PaidWithCreditCard(db *gorm.DB) *gorm.DB {
      return db.Where("pay_mode_sign = ?", "C")
    }
    
    func PaidWithCod(db *gorm.DB) *gorm.DB {
      return db.Where("pay_mode_sign = ?", "C")
    }
    
    func OrderStatus(status []string) func (db *gorm.DB) *gorm.DB {
      return func (db *gorm.DB) *gorm.DB {
        return db.Where("status IN (?)", status)
      }
    }
    
    db.Scopes(AmountGreaterThan1000, PaidWithCreditCard).Find(&orders)
    // 查找所有金额大于 1000 的信用卡订单
    
    db.Scopes(AmountGreaterThan1000, PaidWithCod).Find(&orders)
    // 查找所有金额大于 1000 的 COD 订单
    
    db.Scopes(AmountGreaterThan1000, OrderStatus([]string{"paid", "shipped"})).Find(&orders)
    // 查找所有金额大于1000 的已付款或已发货订单
    ```

25. `Save` 用来更新，**会保存所有的字段，即使字段是零值**。单列更新推荐使用`Update`或`Updates`,多列更新但又不是所有列推荐使用`Updates`，所有列更新可以使用`Save`

26. `Select`选择字段或表、`Omit`跳过字段或表。使用 Struct 进行 `Select`（会 `select` 零值的字段）

    ```go
    db.Model(&user).Select("Name", "Age").Updates(User{Name: "new_name", Age: 0})
    // UPDATE users SET name='new_name', age=0 WHERE id=111;
    ```

27. 如果在**没有任何条件**的情况下执行批量更新，默认情况下，GORM 不会执行该操作，并返回 `ErrMissingWhereClause` 错误.对此，你必须加一些条件，或者使用原生 SQL，或者启用 `AllowGlobalUpdate` 模式，例如：

    ```go
    db.Model(&User{}).Update("name", "jinzhu").Error // gorm.ErrMissingWhereClause
    
    db.Model(&User{}).Where("1 = 1").Update("name", "jinzhu")
    // UPDATE users SET `name` = "jinzhu" WHERE 1=1
    
    db.Exec("UPDATE users SET name = ?", "jinzhu")
    // UPDATE users SET name = "jinzhu"
    
    db.Session(&gorm.Session{AllowGlobalUpdate: true}).Model(&User{}).Update("name", "jinzhu")
    // UPDATE users SET `name` = "jinzhu"
    ```

28. 如果您想在更新时跳过 `Hook` 方法且不追踪更新时间，可以使用 `UpdateColumn`、`UpdateColumns`，其用法类似于 `Update`、`Updates`

29. 删除一条记录时，删除对象需要指定主键，否则会触发 [批量 Delete](https://gorm.io/zh_CN/docs/delete.html#batch_delete)

30. 如果在没有任何条件的情况下执行批量删除，GORM 不会执行该操作，并返回 `ErrMissingWhereClause `错误.对此，你必须加一些条件，或者使用原生 SQL，或者启用 `AllowGlobalUpdate` 模式

31. 返回被删除的数据，仅适用于支持 Returning 的数据库

    ```go
    // 返回所有列
    var users []User
    DB.Clauses(clause.Returning{}).Where("role = ?", "admin").Delete(&users)
    // DELETE FROM `users` WHERE role = "admin" RETURNING *
    // users => []User{{ID: 1, Name: "jinzhu", Role: "admin", Salary: 100}, {ID: 2, Name: "jinzhu.2", Role: "admin", Salary: 1000}}
    ```

32. 如果您的模型包含了一个 `gorm.deletedat` 字段（`gorm.Model` 已经包含了该字段)，它将自动获得软删除的能力！拥有软删除能力的模型调用 `Delete` 时，记录不会从数据库中被真正删除。但 GORM 会将 `DeletedAt` 置为当前时间， 并且你不能再通过普通的查询方法找到该记录。值得注意的是：软删除不是真真的删除，如果数据表再设计的时候加入太多约束，可能会引发软删除与约束相互矛盾，例如给数据表某个字段加上`unique`的约束，显然软删除后再添加相同的数据会出问题。

    有一种折中的方案：放弃这个唯一，删除的时候还是采用软删除，但是查询的时候需要使用`First`来查询最新的一条记录，这样来看就是可以的。缺点是，无法保证数据表这个字段的唯一性。

33. 您可以使用 `Unscoped` 找到被软删除的记录，您也可以使用 `Unscoped` 永久删除匹配的记录

34. 原生DQL使用`Raw`，扫描结果使用`Scan`；原生DML使用`Exec`

35. 关联关系很好用--> 例子详解：https://juejin.cn/post/7067138794788487181

    1. `belongs to`

       `belongs to` 会与另一个模型建立了一对一的连接。这种模型的每一个实例都“属于”另一个模型的一个实例。

       例如，您的应用包含 user 和 company，并且每个 user 能且只能被分配给一个 company。下面的类型就表示这种关系。 注意，在 `User` 对象中，有一个和 `Company` 一样的 `CompanyID`。 默认情况下， `CompanyID` 被隐含地用来在 `User` 和 `Company` 之间创建一个外键关系， 因此必须包含在 `User` 结构体中才能填充 `Company` 内部结构体。

       ```go
       type User struct {
         gorm.Model
         Name      string
         CompanyID int
         Company   Company `gorm:"foreignKey:CompanyID"` 
         // 不指定引用的名字时，默认为主键。也就是引用的Company的ID字段；
         // 不指定外键的名字时，也默认为主键。也就是User的ID字段
       }
       
       type Company struct {
         ID   int
         Code string
         Name string
       }
       ```

       GORM 可以通过 `Preload`、`Joins` 预加载 belongs to 关联的记录，查看 [预加载](https://gorm.io/zh_CN/docs/preload.html) 获取详情

    2. `has one`

       `has one` 与另一个模型建立一对一的关联，但它和一对一关系有些许不同。 这种关联表明一个模型的每个实例都包含或拥有另一个模型的一个实例。

       例如，您的应用包含 user 和 credit card 模型，且每个 user 只能有一张 credit card。

       ```go
       // User 有一张 CreditCard，默认 CreditCard 的 UserID 是外键，没有USerID字段时，需要使用foreignKey来指定
       type User struct {
         gorm.Model
         Name       string     
         CreditCard CreditCard `gorm:"foreignKey:UserName;references:name"`
       }
       
       type CreditCard struct {
         gorm.Model
         Number   string
         UserName string
       }
       ```

    3. `Has Many`

       `has many` 与另一个模型建立了一对多的连接。 不同于 `has one`，拥有者可以有零或多个关联模型。

       例如，您的应用包含 user 和 credit card 模型，且每个 user 可以有多张 credit card。

       ```go
       type User struct {
         gorm.Model
         MemberNumber string
         // 重写默认外键和引用
         CreditCards  []CreditCard `gorm:"foreignKey:UserNumber;references:MemberNumber"`
       }
       
       type CreditCard struct {
         gorm.Model
         Number     string
         UserNumber string
       }
       ```

    4. `Many To Many`

       Many to Many 会在两个 model 中添加一张连接表。一般来说原生SQL建表，针对多对多关系都是需要建立中间表。

       例如，您的应用包含了 user 和 language，且一个 user 可以说多种 language，多个 user 也可以说一种 language。

       ```go
       // User 拥有并属于多种 language，`user_languages` 是连接表
       type User struct {
         gorm.Model
         Languages []Language `gorm:"many2many:user_languages;"`
       }
       
       type Language struct {
         gorm.Model
         Name string
       }
       ```

       当使用 GORM 的 `AutoMigrate` 为 `User` 创建表时，GORM 会自动创建连接表

36. 强大的实体关联

    在创建、更新记录时，GORM 会通过 [Upsert](https://gorm.io/zh_CN/docs/create.html#upsert) 自动保存关联及其引用记录。

    ```go
    user := User{
      Name:            "jinzhu",
      BillingAddress:  Address{Address1: "Billing Address - Address 1"},
      ShippingAddress: Address{Address1: "Shipping Address - Address 1"},
      Emails:          []Email{
        {Email: "jinzhu@example.com"},
        {Email: "jinzhu-2@example.com"},
      },
      Languages:       []Language{
        {Name: "ZH"},
        {Name: "EN"},
      },
    }
    // create时会自动创建所有关联的数据
    db.Create(&user)
    // BEGIN TRANSACTION;
    // INSERT INTO "addresses" (address1) VALUES ("Billing Address - Address 1"), ("Shipping Address - Address 1") ON DUPLICATE KEY DO NOTHING;
    // INSERT INTO "users" (name,billing_address_id,shipping_address_id) VALUES ("jinzhu", 1, 2);
    // INSERT INTO "emails" (user_id,email) VALUES (111, "jinzhu@example.com"), (111, "jinzhu-2@example.com") ON DUPLICATE KEY DO NOTHING;
    // INSERT INTO "languages" ("name") VALUES ('ZH'), ('EN') ON DUPLICATE KEY DO NOTHING;
    // INSERT INTO "user_languages" ("user_id","language_id") VALUES (111, 1), (111, 2) ON DUPLICATE KEY DO NOTHING;
    // COMMIT;
    // 下面的save会自动更新或创建所有关联的数据（有就替换，没有就
    db.Save(&user)
    ```

37. 若要在创建、更新时跳过自动保存，您可以使用 `Select` 或 `Omit`，例如

    ```go
    user := User{
      Name:            "jinzhu",
      BillingAddress:  Address{Address1: "Billing Address - Address 1"},
      ShippingAddress: Address{Address1: "Shipping Address - Address 1"},
      Emails:          []Email{
        {Email: "jinzhu@example.com"},
        {Email: "jinzhu-2@example.com"},
      },
      Languages:       []Language{
        {Name: "ZH"},
        {Name: "EN"},
      },
    }
    
    db.Select("Name").Create(&user)
    // INSERT INTO "users" (name) VALUES ("jinzhu", 1, 2);
    
    db.Omit("BillingAddress").Create(&user)
    // Skip create BillingAddress when creating a user
    
    db.Omit(clause.Associations).Create(&user)
    // Skip all associations when creating a user
    ```

38. Select/Omit 关联字段

    ```go
    user := User{
      Name:            "jinzhu",
      BillingAddress:  Address{Address1: "Billing Address - Address 1", Address2: "addr2"},
      ShippingAddress: Address{Address1: "Shipping Address - Address 1", Address2: "addr2"},
    }
    
    // 创建 user 及其 BillingAddress、ShippingAddress
    // 在创建 BillingAddress 时，仅使用其 address1、address2 字段，忽略其它字段
    db.Select("BillingAddress.Address1", "BillingAddress.Address2").Create(&user)
    
    db.Omit("BillingAddress.Address2", "BillingAddress.CreatedAt").Create(&user)
    ```

39. 关联模式包含一些在处理关系时有用的方法

    ```go
    // 开始关联模式
    var user User
    db.Model(&user).Association("Languages")
    // `user` 是源模型，它的主键不能为空
    // 关系的字段名是 `Languages`，关系需要是 belongs to、has one、has many、many to many
    // 如果匹配了上面两个要求，会开始关联模式，否则会返回错误
    db.Model(&user).Association("Languages").Error
    ```

40. 查找所有匹配的关联记录

    ```go
    db.Model(&user).Association("Languages").Find(&languages)
    ```

41. 查找带条件的关联

    ```go
    codes := []string{"zh-CN", "en-US", "ja-JP"}
    db.Model(&user).Where("code IN ?", codes).Association("Languages").Find(&languages)
    
    db.Model(&user).Where("code IN ?", codes).Order("code desc").Association("Languages").Find(&languages)
    ```

42. 返回当前关联的计数

    ```go
    db.Model(&user).Association("Languages").Count()
    
    // 条件计数
    codes := []string{"zh-CN", "en-US", "ja-JP"}
    db.Model(&user).Where("code IN ?", codes).Association("Languages").Count()
    ```

43. 关联模式也支持批量处理，例如：

    ```go
    // 查询所有用户的所有角色
    db.Model(&users).Association("Role").Find(&roles)
    
    // 从所有 team 中删除 user A
    db.Model(&users).Association("Team").Delete(&userA)
    
    // 获取去重的用户所属 team 数量
    db.Model(&users).Association("Team").Count()
    
    // 对于批量数据的 `Append`、`Replace`，参数的长度必须与数据的长度相同，否则会返回 error
    var users = []User{user1, user2, user3}
    // 例如：现在有三个 user，Append userA 到 user1 的 team，Append userB 到 user2 的 team，Append userA、userB 和 userC 到 user3 的 team
    db.Model(&users).Association("Team").Append(&userA, &userB, &[]User{userA, userB, userC})
    // 重置 user1 team 为 userA，重置 user2 的 team 为 userB，重置 user3 的 team 为 userA、 userB 和 userC
    db.Model(&users).Association("Team").Replace(&userA, &userB, &[]User{userA, userB, userC})
    ```

44. 你可以在删除记录时通过 `Select` 来删除具有 has one、has many、many2many 关系的记录，例如：

    ```go
    // 删除 user 时，也删除 user 的 account
    db.Select("Account").Delete(&user)
    
    // 删除 user 时，也删除 user 的 Orders、CreditCards 记录
    db.Select("Orders", "CreditCards").Delete(&user)
    
    // 删除 user 时，也删除用户所有 has one/many、many2many 记录
    db.Select(clause.Associations).Delete(&user)
    
    // 删除 users 时，也删除每一个 user 的 account
    db.Select("Account").Delete(&users)
    ```

    > **注意：** 只有当记录的主键不为空时，关联才会被删除，GORM 会使用这些主键作为条件来删除关联记录. 而且也这个`Select`关联删除只能删除当前表的关联不能删除嵌套的关联。例如user表里的Orders还可能与其他有关联，但是不能把他们也删掉

45. GORM 允许在 `Preload` 的其它 SQL 中直接加载关系（belong to\has one\has many\many to many），例如：

    ```go
    type User struct {
      gorm.Model
      Username string
      Orders   []Order `gorm:foreignKey:UserID`
    }
    
    type Order struct {
      gorm.Model
      UserID uint
      Price  float64
    }
    
    // 查找 user 时预加载相关 Order
    db.Preload("Orders").Find(&users)
    // SELECT * FROM users;
    // SELECT * FROM orders WHERE user_id IN (1,2,3,4); # 前面查找好的所有user_id
    
    db.Preload("Orders").Preload("Profile").Preload("Role").Find(&users)
    // SELECT * FROM users;
    // SELECT * FROM orders WHERE user_id IN (1,2,3,4); // has many
    // SELECT * FROM profiles WHERE user_id IN (1,2,3,4); // has one
    // SELECT * FROM roles WHERE id IN (4,5,6); // belongs to
    ```

46. 与创建、更新时使用 `Select` 类似，`clause.Associations` 也可以和 `Preload` 一起使用，它可以用来 `预加载` 全部关联，例如：

    ```go
    type User struct {
      gorm.Model
      Name       string
      CompanyID  uint
      Company    Company
      Role       Role
      Orders     []Order
    }
    
    db.Preload(clause.Associations).Find(&users)
    ```

    `clause.Associations` 不会预加载嵌套的关联，但你可以使用[嵌套预加载](https://gorm.io/zh_CN/docs/preload.html#nested_preloading) 例如：

    ```go
    db.Preload("Orders.OrderItems.Product").Preload(clause.Associations).Find(&users) 
    // 假设有A B C 三个表，这三个表依次依赖，这样就形成了嵌套的关系。可以使用Preload的嵌套预加载来实现简洁的关联查询
    // 或者使用连接查询也可以
    // 或者使用 Assosiation来关联表也可以做查询
    ```

47. GORM 允许带条件的 `Preload` 关联，类似于[内联条件](https://gorm.io/zh_CN/docs/query.html#inline_conditions)

    ```go
    // 带条件的预加载 Order
    db.Preload("Orders", "state NOT IN (?)", "cancelled").Find(&users)
    // SELECT * FROM users;
    // SELECT * FROM orders WHERE user_id IN (1,2,3,4) AND state NOT IN ('cancelled');
    
    db.Where("state = ?", "active").Preload("Orders", "state NOT IN (?)", "cancelled").Find(&users)
    // SELECT * FROM users WHERE state = 'active';
    // SELECT * FROM orders WHERE user_id IN (1,2) AND state NOT IN ('cancelled');
    ```

    `Preload`预加载时需要条件的话，只能使用内联条件或者直接改成连接查询

48. 链式方法是将 `Clauses` 修改或添加到当前 `Statement` 的方法，例如：

    `Where`, `Select`, `Omit`, `Joins`, `Scopes`, `Preload`, `Raw` (`Raw` can’t be used with other chainable methods to build SQL)…

49. 终结（方法） 是会立即执行注册回调的方法，然后生成并执行 SQL，比如这些方法：

    `Create`, `First`, `Find`, `Take`, `Save`, `Update`, `Delete`, `Scan`, `Row`, `Rows`…

50. GORM 定义了 `Session`、`WithContext`、`Debug` 方法做为 `新建会话方法`，查看[会话](https://gorm.io/zh_CN/docs/session.html) 获取详情.

    在 `链式方法`, `Finisher 方法`之后, GORM 返回一个初始化的 `*gorm.DB` 实例，不能安全地再使用。您应该使用 `新建会话方法` 来标记 `*gorm.DB` 为可共享。调用已经调用终结方法的实例会被上个实例污染，可以使用会话方式新建会话避免前面实例调用终结方法导致的条件污染

    让我们用实例来解释它：

    示例 1：

    ```go
    db, err := gorm.Open(sqlite.Open("test.db"), &gorm.Config{})
    // db is a new initialized `*gorm.DB`, which is safe to reuse
    
    db.Where("name = ?", "jinzhu").Where("age = ?", 18).Find(&users)
    // `Where("name = ?", "jinzhu")` is the first chain method call, it will create an initialized `*gorm.DB` instance, aka `*gorm.Statement`
    // `Where("age = ?", 18)` is the second chain method call, it reuses the above `*gorm.Statement`, adds new condition `age = 18` to it
    // `Find(&users)` is a finisher method, it executes registered Query Callbacks, which generates and runs the following SQL:
    // SELECT * FROM users WHERE name = 'jinzhu' AND age = 18;
    
    db.Where("name = ?", "jinzhu2").Where("age = ?", 20).Find(&users)
    // `Where("name = ?", "jinzhu2")` is also the first chain method call, it creates a new `*gorm.Statement`
    // `Where("age = ?", 20)` reuses the above `Statement`, and add conditions to it
    // `Find(&users)` is a finisher method, it executes registered Query Callbacks, generates and runs the following SQL:
    // SELECT * FROM users WHERE name = 'jinzhu2' AND age = 20;
    
    db.Find(&users)
    // `Find(&users)` is a finisher method call, it also creates a new `Statement` and executes registered Query Callbacks, generates and runs the following SQL:
    // SELECT * FROM users;
    ```

    (错误的) 示例2：

    ```go
    db, err := gorm.Open(sqlite.Open("test.db"), &gorm.Config{})
    // db is a new initialized *gorm.DB, which is safe to reuse
    
    tx := db.Where("name = ?", "jinzhu")
    // `Where("name = ?", "jinzhu")` returns an initialized `*gorm.Statement` instance after chain method `Where`, which is NOT safe to reuse
    
    // good case
    tx.Where("age = ?", 18).Find(&users)
    // `tx.Where("age = ?", 18)` use the above `*gorm.Statement`, adds new condition to it
    // `Find(&users)` is a finisher method call, it executes registered Query Callbacks, generates and runs the following SQL:
    // SELECT * FROM users WHERE name = 'jinzhu' AND age = 18
    // 会被上个实例污染，可以使用会话方式新建会话避免前面实例调用终结方法导致的条件污染
    // bad case
    tx.Where("age = ?", 28).Find(&users)
    // `tx.Where("age = ?", 18)` also use the above `*gorm.Statement`, and keep adding conditions to it
    // So the following generated SQL is polluted by the previous conditions:
    // SELECT * FROM users WHERE name = 'jinzhu' AND age = 18 AND age = 28;
    ```

    示例 3：

    ```go
    db, err := gorm.Open(sqlite.Open("test.db"), &gorm.Config{})
    // db is a new initialized *gorm.DB, which is safe to reuse
    
    tx := db.Where("name = ?", "jinzhu").Session(&gorm.Session{})
    tx := db.Where("name = ?", "jinzhu").WithContext(context.Background())
    tx := db.Where("name = ?", "jinzhu").Debug()
    // `Session`, `WithContext`, `Debug` returns `*gorm.DB` marked as safe to reuse, newly initialized `*gorm.Statement` based on it keeps current conditions
    
    // good case
    tx.Where("age = ?", 18).Find(&users)
    // SELECT * FROM users WHERE name = 'jinzhu' AND age = 18
    
    // good case
    tx.Where("age = ?", 28).Find(&users)
    // SELECT * FROM users WHERE name = 'jinzhu' AND age = 28;
    ```

51. `PreparedStmt` 在执行任何 SQL 时都会创建一个 prepared statement 并将其缓存，以提高后续的效率。这在业务里还是很有用的，一次SQL加载就可以提升后续执行效率

52. 通过 `NewDB` 选项创建一个不带之前条件的新 DB，例如：

    ```go
    tx := db.Where("name = ?", "jinzhu").Session(&gorm.Session{NewDB: true})
    
    tx.First(&user)
    // SELECT * FROM users ORDER BY id LIMIT 1
    
    tx.First(&user, "id = ?", 10)
    // SELECT * FROM users WHERE id = 10 ORDER BY id
    
    // 不带 `NewDB` 选项
    tx2 := db.Where("name = ?", "jinzhu").Session(&gorm.Session{})
    tx2.First(&user)
    // SELECT * FROM users WHERE name = "jinzhu" ORDER BY id
    ```

53. 在一个 DB 事务中使用 `Transaction` 方法，GORM 会使用 `SavePoint(savedPointName)`，`RollbackTo(savedPointName)` 为你提供嵌套事务支持。 你可以通过 `DisableNestedTransaction` 选项关闭它，例如：

    ```go
    db.Session(&gorm.Session{
      DisableNestedTransaction: true,
    }).CreateInBatches(&users, 100)
    ```

54. GORM 默认不允许进行全局 update/delete，该操作会返回 `ErrMissingWhereClause` 错误。 您可以通过将一个选项设置为 true 来启用它，例如：

    ```go
    db.Session(&gorm.Session{
      AllowGlobalUpdate: true,
    }).Model(&User{}).Update("name", "jinzhu")
    // UPDATE users SET `name` = "jinzhu"
    ```

55. 在创建、更新记录时，GORM 会通过 [Upsert](https://gorm.io/zh_CN/docs/create.html#upsert) 自动保存关联及其引用记录。 如果您想要更新关联的数据，您应该使用 `FullSaveAssociations` 模式，例如：(重要)

    ```go
    db.Session(&gorm.Session{FullSaveAssociations: true}).Updates(&user)
    // ...
    // INSERT INTO "addresses" (address1) VALUES ("Billing Address - Address 1"), ("Shipping Address - Address 1") ON DUPLICATE KEY SET address1=VALUES(address1);
    // INSERT INTO "users" (name,billing_address_id,shipping_address_id) VALUES ("jinzhu", 1, 2);
    // INSERT INTO "emails" (user_id,email) VALUES (111, "jinzhu@example.com"), (111, "jinzhu-2@example.com") ON DUPLICATE KEY SET email=VALUES(email);
    // ...
    ```

56. 声明查询字段，一种优化手段，避免`*`再次解析成表的行字段的时间消耗

    ```go
    db.Session(&gorm.Session{QueryFields: true}).Find(&user)
    // SELECT `users`.`name`, `users`.`age`, ... FROM `users` // 有该选项
    // SELECT * FROM `users` // 没有该选项
    ```

57. 事务`Begin`、`Commit`和`Rollback`不是很方便(手动事务)，推荐使用下面的方式。GORM 提供了 `SavePoint`、`Rollbackto` 方法，来提供保存点以及回滚至保存点功能

    ```go
    db.Transaction(func(tx *gorm.DB) error {
      // 在事务中执行一些 db 操作（从这里开始，您应该使用 'tx' 而不是 'db'）
      if err := tx.Create(&Animal{Name: "Giraffe"}).Error; err != nil {
        // 返回任何错误都会回滚事务
        return err
      }
    
      if err := tx.Create(&Animal{Name: "Lion"}).Error; err != nil {
        return err
      }
    
      // 返回 nil 提交事务
      return nil
    })
    ```

    GORM 支持嵌套事务，您可以回滚较大事务内执行的一部分操作，例如：

    ```go
    db.Transaction(func(tx *gorm.DB) error {
      tx.Create(&user1)
    
      tx.Transaction(func(tx2 *gorm.DB) error {
        tx2.Create(&user2)
        return errors.New("rollback user2") // Rollback user2
      })
    
      tx.Transaction(func(tx2 *gorm.DB) error {
        tx2.Create(&user3)
        return nil
      })
    
      return nil
    })
    
    // Commit user1, user3
    
    ```

58. Migrator 接口

    ```go
    // 为 `User` 创建表
    db.Migrator().CreateTable(&User{})
    
    // 将 "ENGINE=InnoDB" 添加到创建 `User` 的 SQL 里去
    db.Set("gorm:table_options", "ENGINE=InnoDB").Migrator().CreateTable(&User{})
    
    // 检查 `User` 对应的表是否存在
    db.Migrator().HasTable(&User{})
    db.Migrator().HasTable("users")
    
    // 如果存在表则删除（删除时会忽略、删除外键约束)
    db.Migrator().DropTable(&User{})
    db.Migrator().DropTable("users")
    
    // 重命名表
    db.Migrator().RenameTable(&User{}, &UserInfo{})
    db.Migrator().RenameTable("users", "user_infos")
    ```

59. 连接池

    ```go
    // 获取通用数据库对象 sql.DB ，然后使用其提供的功能
    sqlDB, err := db.DB()
    
    // SetMaxIdleConns 用于设置连接池中空闲连接的最大数量。
    sqlDB.SetMaxIdleConns(10)
    
    // SetMaxOpenConns 设置打开数据库连接的最大数量。
    sqlDB.SetMaxOpenConns(100)
    
    // SetConnMaxLifetime 设置了连接可复用的最大时间。
    sqlDB.SetConnMaxLifetime(time.Hour)
    ```

60. 执行任何 SQL 时都创建并缓存预编译语句，可以提高后续的调用速度

    ```go
    // 全局模式
    db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
      PrepareStmt: true,
    })
    
    // 会话模式
    tx := db.Session(&Session{PrepareStmt: true})
    tx.First(&user, 1)
    tx.Find(&users)
    tx.Model(&user).Update("Age", 18)
    ```

61. 您可以使用 `Table` 方法临时指定表名(和`Model()`方法类似)，例如：

    ```go
    // 根据 User 的字段创建 `deleted_users` 表
    db.Table("deleted_users").AutoMigrate(&User{})
    
    // 从另一张表查询数据
    var deletedUsers []User
    db.Table("deleted_users").Find(&deletedUsers)
    // SELECT * FROM deleted_users;
    
    db.Table("deleted_users").Where("name = ?", "jinzhu").Delete(&User{})
    // DELETE FROM deleted_users WHERE name = 'jinzhu';
    ```

62. 字段的序列化

    Serializer 是一个可扩展的接口，允许自定义如何使用数据库对数据进行序列化和反序列化

    GORM 提供了一些默认的序列化器：json、gob、unixtime，这里有一个如何使用它的快速示例

    ```go
    type User struct {
        Name        []byte                 `gorm:"serializer:json"`
        Roles       Roles                  `gorm:"serializer:json"`
        Contracts   map[string]interface{} `gorm:"serializer:json"`
        JobInfo     Job                    `gorm:"type:bytes;serializer:gob"`
        CreatedTime int64                  `gorm:"serializer:unixtime;type:time"` // 将 int 作为日期时间存储到数据库中
    }
    
    type Roles []string
    
    type Job struct {
        Title    string
        Location string
        IsIntern bool
    }
    ```

63. 复合索引列的顺序会影响其性能，因此必须仔细考虑-->后续的条件尽量符合覆盖索引原则

    您可以使用 `priority` 指定顺序，默认优先级值是 `10`，如果优先级值相同，则顺序取决于模型结构体字段的顺序()

    ```go
    type User struct {
        Name   string `gorm:"index:idx_member"`
        Number string `gorm:"index:idx_member"`
    }
    // column order: name, number
    
    type User struct {
        Name   string `gorm:"index:idx_member,priority:2"`
        Number string `gorm:"index:idx_member,priority:1"`
    }
    // column order: number, name
    
    type User struct {
        Name   string `gorm:"index:idx_member,priority:12"`
        Number string `gorm:"index:idx_member"`
    }
    // column order: number, name
    ```

64. 复合主键

    通过将多个字段设为主键，以创建复合主键，例如：

    ```go
    type Product struct {
      ID           string `gorm:"primaryKey"`
      LanguageCode string `gorm:"primaryKey"`
      Code         string
      Name         string
    }
    ```

    **注意：**默认情况下，整型 `PrioritizedPrimaryField` 启用了 `AutoIncrement`，要禁用它，您需要为整型字段关闭 `autoIncrement`：

    ```go
    type Product struct {
      CategoryID uint64 `gorm:"primaryKey;autoIncrement:false"`
      TypeID     uint64 `gorm:"primaryKey;autoIncrement:false"`
    }
    ```

65. gorm与go的类型映射

    下面是gorm结构体模型与mysql数据库类型的具体映射关系

    | gorm model field                            | mysql table field type | note                                                         |
    | ------------------------------------------- | ---------------------- | ------------------------------------------------------------ |
    | int8                                        | tinyint                |                                                              |
    | int16                                       | smallint               |                                                              |
    | int32                                       | int                    |                                                              |
    | int64                                       | bigint                 |                                                              |
    | uint8                                       | tinyint unsigned       |                                                              |
    | uint16                                      | smallint unsigned      |                                                              |
    | uint32                                      | int unsigned           |                                                              |
    | uint64                                      | bigint unsigned        |                                                              |
    | float32                                     | float                  |                                                              |
    | float64                                     | double                 |                                                              |
    | ~~complex64~~                               | 没有对应类型           | **不支持**                                                   |
    | ~~complex128~~                              | 没有对应类型           | **不支持**                                                   |
    | byte                                        | tinyint unsigned       | 等效于go的uint8                                              |
    | rune                                        | int                    | rune等效于go里的int32                                        |
    | bool                                        | tinyint(1)             | 都占一个bit位                                                |
    | time.Time                                   | datetime(3)            |                                                              |
    | string                                      | longtext               | **不添加gorm的约束，默认为longtext（mysql的字符串最大数据类型）** |
    | field string  \`gorm:"type:char(1)"\`       | char                   | **定长字符串设置为1字节不生效**                              |
    | field string  \`gorm:"type:char(5)"\`       | char(5)                | 定长                                                         |
    | field string  \`gorm:"type:varchar(1)"\`    | varchar(1)             | 可变长度，最多1个字节                                        |
    | field string  \`gorm:"type:varchar(1000)"\` | varcahr(1000)          | 可变长度，最多1000个字节                                     |
    | field string  \`gorm:"type:tinyblob"\`      | tinyblob               | 不可以添加长度，会报错                                       |
    | field string  \`gorm:"type:tinytext"\`      | tinytext               | 不可以添加长度，会报错                                       |
    | field string  \`gorm:"type:blob"\`          | blob                   | 不可以添加长度，会报错                                       |
    | field string  \`gorm:"type:text"\`          | text                   | 不可以添加长度，会报错                                       |

    **可以看出：go的数据类型通过gorm与mysql的数据类型是完全对应起来的。go的`int8`的1字节对mysql的`tinyint`的1字节，有无符号也是对应起来的，但是针对go的`string`可以添加gorm的类型约束来限制对应mysql的类型，不屑约束的话默认是`longtext`（太浪费空间了!!!）**

    **其次，非`string`类型添加长度约束会失效。而且文本或二进制类型不管大中小都无法添加长度，添加会报错**

## 从表名开始

如何定义表名? gorm的默认表名策略是模型名字的复数, 比如:

```go
type User Struct{}
```

默认表名为`users`, 但是生产习惯常用单数表名, 而且加es的复数或者sheep单复同形容易让人迷惑.

当然, gorm实现了复杂的单数变复数逻辑, 我们可以从 [http://github.com/jinzhu/inflection/inflections.go](https://link.zhihu.com/?target=http%3A//github.com/jinzhu/inflection/inflections.go) 一探究竟, 下面复习下单复同型/不可数单词, 以及特殊变复数的单词:

```go
var uncountableInflections = []string{"equipment", "information", "rice", "money", "species", "series", "fish", "sheep", "jeans", "police"}

var irregularInflections = IrregularSlice{
   {"person", "people"},
   {"man", "men"},
   {"child", "children"},
   {"sex", "sexes"},
   {"move", "moves"},
   {"mombie", "mombies"},
}
```

好吧, 写个增删改查还得过专八.

我们可以实现`gorm.schema.scheme.go.Tabler.TableName()`方法, 重写表名.

```go
func (*User) TableName() string {return "user"}
```

OK!

除了这种奇奇怪怪的写法, 在gormV2中, 我们可以使用一个配置改为单数表名, 谢天谢地.

```go
NamingStrategy: &schema.NamingStrategy{SingularTable: true}
```

`schema.NamingStrategy`实现了`gorm.schema.naming.go.Namer`接口, 我们也可以自己实现这个接口,替换gorm的命名策略, 接口简单易懂.

```go
type Namer interface {
   TableName(table string) string
   ColumnName(table, column string) string
   JoinTableName(joinTable string) string
   RelationshipFKName(Relationship) string
   CheckerName(table, column string) string
   IndexName(table, column string) string
}
```

## 列名

列名无可非议, 简简单单的下划线模式. 而且gorm会自动将`ID`转换成`id`而不是`i_d`, 那么, 就遵循golang的语言规范, 放心的在代码中使用ID吧~

让我们来看下这个特殊变小写数组.

**列名变小写?**

`gorm.schema.naming.go`

```go
// https://github.com/golang/lint/blob/master/lint.go#L770
commonInitialisms         = []string{"API", "ASCII", "CPU", "CSS", "DNS", "EOF", "GUID", "HTML", "HTTP", "HTTPS", "ID", "IP", "JSON", "LHS", "QPS", "RAM", "RHS", "RPC", "SLA", "SMTP", "SSH", "TLS", "TTL", "UID", "UI", "UUID", "URI", "URL", "UTF8", "VM", "XML", "XSRF", "XSS"}
```

`CreatedAt`, `UpdatedAt`

更加有争议和难以理解的列名,应该是这两个.

**最原始的方法 ,mysql自动添加**

如果想让mysql来做这件事, 可以这样写:

```go
create table user2
(
    ....
    create_time datetime(3) not null DEFAULT CURRENT_TIMESTAMP(3) comment '创建时间',
    update_time datetime(3) not null DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3) comment '修改时间',
   ....
);
type User struct {
   ID        int64 `gorm:"autoIncrement"`
   CreatedAt time.Time `gorm:default`
   UpdatedAt time.Time `gorm:default`
}
```

加上gorm的`default`标签, 这个标签的意思是: `gorm.Create`时, 如果该列值为零, 使用建表时的`default`值, sql语句中也就没有这一列.

`gorm.Updates`如果`Update`一个`map`,就是强制更新. 如果`Update`一个结构体, 不赋值也就是用mysql的默认值, 如果赋值就直接`Update`, 这一点很巧妙.

gorm对默认值的处理:

`gorm.callbacks.create.go:285`

```go
defaultValueFieldsHavingValue := map[*schema.Field][]interface{}{}
for _, field := range stmt.Schema.FieldsWithDefaultDBValue {
   if v, ok := selectColumns[field.DBName]; (ok && v) || (!ok && !restricted) {
      if v, isZero := field.ValueOf(rv); !isZero {
         if len(defaultValueFieldsHavingValue[field]) == 0 {
            defaultValueFieldsHavingValue[field] = make([]interface{}, stmt.ReflectValue.Len())
         }
         defaultValueFieldsHavingValue[field][i] = v
      }
   }
}
```

你要说我能看懂, 那我肯定看不懂, 但你要说看不懂, 我还知道它能干啥, 不错不错...

**更好的方法，gorm自动添加**

如果想让gorm来添加, sql就可以不用写`default`

```text
type User struct {
   ID        int64 `gorm:"autoIncrement"`
   CreatedAt time.Time
   UpdatedAt time.Time
}
```

只要列名叫`CreatedAt`和`UpdatedAt`即可.

`gorm.Create`即可自动更新`CreatedAt`和`UpdatedAt`.

但是`gorm.Updates`时可要注意了, 如果没有使用`db.Model`指定`model`结构体, 就不能更新`UpdatedAt`.

所以如果这样:

```go
db.Table(tableName).Updates(map[string]interface{})
```

是无法更新更新时间的.

正确做法是:

```go
db.Model(&User{}).Updates(map[string]interface{})
```

如果傲娇的你, 偏偏不喜欢这样的列名, 非要用`CreateTime`, 那好吧, 你可以这样: 不告诉你, 自己去查文档... [https://gorm.io/zh_CN/docs/models.html](https://link.zhihu.com/?target=https%3A//gorm.io/zh_CN/docs/models.html)

**小建议: 建议建表时使用datetime(3)秒类型**

## Create

[https://gorm.io/zh_CN/docs/create.html](https://link.zhihu.com/?target=https%3A//gorm.io/zh_CN/docs/create.html)

gormV2支持创建和批量创建, 你可以这样写:

```go
//创建
db.Create(&User{})

//批量创建
users := []*User{{},{},{}}
db.Create(&users)
// 
```

创建成功会往结构体或结构体切片回填主键的值。

## Hooks

我们可以灵活的使用hooks, 以减少重复代码在logic层对业务逻辑的侵入, 使代码更简洁. 钩子命名好像Vue啊! 哈哈哈

## Where

gorm中最常用的语句, gormv1中最常用的方法是:

```go
db.Where("uid = ?", 1)
```

gormV2支持两种新的`where`, Struct和map条件, 本文介绍struct条件:

```go
db.Where(&User{phone:"123", Age:0})

//Select * from User where phone = 123
//Age没有被查询
```

结构体查询非常好用, 你不用手动写列名, 避免了因为列名写错而导致的错误.

**注意: 当使用结构作为条件查询时，GORM 只会查询非零值字段。这意味着如果您的字段值为`0` 、`''` 、`false` 或其他[零值](https://link.zhihu.com/?target=https%3A//tour.golang.org/basics/12)，该字段不会被用于构建查询条件**

## Find, Take, First, Last

**ErrRecordNotFound**

这个东西可是实在太恶心了... 但是golang的反射又没法对ptr赋值为nil, 所以只能通过error返回. 这种处理方式也是可以理解的.

```go
err := db.Where(...).First(...).Error
// 检查 ErrRecordNotFound 错误
errors.Is(err, gorm.ErrRecordNotFound)
```

**注意: 只有`First`,`Last`,`Take`方法会产生`gorm.ErrRecordNotFound`错误哦**

**Find**

为了躲避`ErrRecordNotFound`, 我们来看下这个可爱的`Find`方法.

`Find`方法可以接受两种参数, 一种是结构体指针, 一种是数组指针.

接受数组指针很好理解, 我们可以通过数组长度来判断是否查到数据:

```go
res := []*model.User{}
ret := db.Model(&model.User{}).Where(`user_id > ?`, 0).Find(res)
```

接受结构体指针时呢? 我们只能通过ret.RowsAffected == 0来判断是否查到数据, 所以ErrRecordNotFound还有点可爱?

**注意:`db.Find(&User{}).RowsAffected`只会是`0`或`1`**

```go
res := &model.User{}
ret := db.Model(&model.User{}).Where(`user_id > ?`, 0).Find(res)
ret.RowsAffected
```

从源码来看, `Find`结构体和`Find`数组的差别如下:

`gorm.scan.go:214`

```go
switch(){
case reflect.Slice, reflect.Array:
    for initialized || rows.Next() {
    }
case reflect.Struct:
    if initialized || rows.Next() {
    }
}
```

OMG! 仅仅是for和if的差别...

那么`Find`和`Take`有什么差别呢? 你猜的没错, 只是`Take`会返回`ErrRecordNotFound`

`gorm.scan.go:239`

```go
if db.RowsAffected == 0 && db.Statement.RaiseErrorOnNotFound {
   db.AddError(ErrRecordNotFound)
}
//Find RaiseErrorOnNotFound=false
//Take RaiseErrorOnNotFound=true
```

好家伙! 我直接好家伙! 果然是大佬的代码...

**行锁**

哪里的代码有version乐观锁和share锁, 我去参观只见过`UPDATE`锁.

gormV2和V1这里确实不一样, 写法如下:

```go
DB.Clauses(clause.Locking{Strength: "UPDATE"}).Find(&users)
```

mysql行锁只在事务中有用.

## Update

gormV2的更新是我最喜欢的一部分, 非常的有趣...

**零值**

我们不再需要这样, 我讨厌的方式:

```go
var updateMap := make(map[string]interface{})
if phone != "" {
    updateMap["phone"] = phone
}
db.Updates(updateMap)
```

我们只需要这样:

```go
db.Updates(&User{Phone:phone})
```

`Updates`方法接受一个`Model`结构体, 如果更新的列为 `""`,`0`,`false`,`time.Time{}` 就会不更新该列. (如果使用gorm-curd, 连数组长度都帮你判断了,真的是太好用了!)

那你要问, 如果我非要设置这个用户的`phone`为`""`呢?

你可以这样写:

```go
db.Select("phone").Updates(&User{Phone:""})
//或者
db.Updates(map[string]interface{}{"phone":""})
```

又回去map了... 梅开二度...

**SQL 表达式**

如果需要`商品库存 - 1`, gormV2不再需要用事务取出, 再`-1` , 再存入...

可以这样:

```go
DB.Model(&product).Where("goods_id = ?", 10086).Update("quantity", gorm.Expr("quantity - ?", 1))
```

## 事务

要在事务中执行一系列操作，一般流程如下, 这个事务写法真的是太棒了!

```go
db.Transaction(func(tx *gorm.DB) error {
 // 在事务中执行一些 db 操作（从这里开始，您应该使用 'tx' 而不是 'db'）
 if err := tx.Create(&Animal{Name: "Giraffe"}).Error; err != nil {
 // 返回任何错误都会回滚事务
 return err
 }

 if err := tx.Create(&Animal{Name: "Lion"}).Error; err != nil {
 return err
 }

 // 返回 nil 提交事务
 return nil
})
```

**嵌套事务**

GORM 支持嵌套事务，可以回滚事务中的事务而不用担心外层事务回滚.

