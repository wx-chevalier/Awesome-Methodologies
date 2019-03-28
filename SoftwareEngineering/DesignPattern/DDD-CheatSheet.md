[![返回目录](https://parg.co/UCb)](https://github.com/wx-chevalier/Awesome-CheatSheets)

# Domain Driven Design CheatSheet

![](http://ata2-img.cn-hangzhou.img-pub.aliyun-inc.com/615646e6ced4c154fdb91d1ba3057a90.png)

DAO 是对于数据持久化的抽象，而 Repository 则是面向对象集合的抽象。DAO 往往是与数据库中的表强映射。

DAO is an abstraction of data persistence. Repository is an abstraction of a collection of objects.

DAO would be considered closer to the database, often table-centric. Repository would be considered closer to the Domain, dealing only in Aggregate Roots. A Repository could be implemented using DAO's, but you wouldn't do the opposite.

Also, a Repository is generally a narrower interface. It should be simply a collection of objects, with a Get(id), Find(ISpecification), Add(Entity). A method like Update is appropriate on a DAO, but not a Repository - when using a Repository, changes to entities would usually be tracked by separate UnitOfWork.

It's a repository of a specific type of objects - it allows you to search for a specific type of objects as well as store them. Usually it will ONLY handle one type of objects. E.g. AppleRepository would allow you to do AppleRepository.findAll(criteria) or AppleRepository.save(juicyApple). Note that the Repository is using Domain Model terms (not DB terms - nothing related to how data is persisted anywhere).

A repository will most likely store all data in the same table, whereas the pattern doesn't require that. The fact that it only handles one type of data though, makes it logically connected to one main table (if used for DB persistence).

# 层级划分与实体类

![image](https://user-images.githubusercontent.com/5803001/44942628-05bc1e80-ade9-11e8-9aea-25c51404638a.png)

| 对象名                     | 层名              | 描述                                                |
| -------------------------- | ----------------- | --------------------------------------------------- |
| Transfer Object/TO         | Controller        | 接入与返回层，提供视图数据聚合与统一的查询/返回格式 |
| Business Object/BO         | Service/Connector | 数据业务层，提供业务数据的聚合                      |
| Database Access Object/DAO | Model             | 元数据访问层，与数据库进行直接交互                  |

在设计数据库的时候，我们尽量避免给属性列添加额外的前缀，并且使用嵌套的结构返回多表联查的数据：

```json
{
  "user": {
    "uuid": "{uuid}",
    "name": "{name}"
  },
  "asset": {
    "uuid": "{uuid}",
    "name": "{name}"
  },
  "lessonss": []
}
```

```sh
/api/resource/get
/api/resource/getByIds

# 在交互层级上同样应该有所隐藏
/api/resource/getRelatedResourceById
/api/related-resource/getRelatedResourceByResourceId
```

```gql
query {
  Resources{
    id
  }
}

query {
  Resource($id: 1){
    id
    statisticsField(){}
    oneToOneField() {}
    oneToManyField(){}
  }
}
```

1、PO(persistant object) 持久对象

（理解为 dao 层：接收和返回的 java bean，也就是通常写在 model 包中的 model）

1. 有时也被称为 Data 对象，对应数据库中的 entity，可以简单认为一个 PO 对应数据库中的一条记录，多个记录可以用 PO 的集合。

2．在 o/r 映射的时候出现的概念,如果没有 o/r 映射,就没有这个概念存在了。
3．PO 中应该不包含任何对数据库的操作。

2、VO(value object) 值对象/ view object 表现层对象

（理解为 view 层：用于显示的 java bean）

1.主要对应页面显示（web 页面(jsp...)/swt、swing 界面）的数据对象，所以它可以和表对应，也可以不（大部分情况是表所有字段集合的子集），这根据业务的需要。

2．与 DTO 的区别是：DTO 用于无界面的 web service 传输中而 VO 用于界面的展示，可以把 DTO 转化为 VO 提供给前台。
3．例如在 struts 中，用 ActionForm 做 VO，需要做一个转换，因为 PO 是面向对象的，而 ActionForm 是和 view 对应的，要将几个 PO 要显示的属性合成一个 ActionForm，可以使用 BeanUtils 的 copy 方法。

3、BO(business object) 业务对象

（理解为 service 层中：接收和返回的 java bean）

1.从业务模型的角度看,见 UML 元件领域模型中的领域对象.封装业务逻辑的 java 对象,通过调用 DAO 方法,结合 PO,VO 进行业务操作。

2．根据业务逻辑，将封装业务逻辑为一个对象，可以包括多个 PO，通常需要将 BO 转化成 PO，才能进行数据的持久化，反之，从 DB 中得到的 PO，需要转化成 BO 才能在业务层使用。
3．关于 BO 主要有三种概念

1. 只包含业务对象的属性；
2. 只包含业务方法；
3. 两者都包含。
   在实际使用中，认为哪一种概念正确并不重要，关键是实际应用中适合自己项目的需要。

4、POJO(plain ordinary java object) 简单无规则 java 对象

（理解为各个层中：接收和返回的 java bean 统称）

1.抽象的统一概念，一个中间对象，可以转化为 PO、DTO、VO（或者说 PO、DTO 是 POJO 的不同的具体阶段的名字）。
2．POJO 持久化之后==〉PO（在运行期，由 Hibernate 中的 cglib 动态把 POJO 转换为 PO，PO 相对于 POJO 会增加一些用来管理数据库 entity 状态的属性和方法。PO 对于 programmer 来说完全透明，由于是运行期生成 PO，所以可以支持增量编译，增量调试。）
3．POJO 传输过程中==〉DTO
4．POJO 用作表示层==〉VO

5、DAO(data access object) 数据访问对象

1.是 sun 的一个标准 j2ee 设计模式,这个模式中有个接口就是 DAO，负责将 PO 持久化到数据库，也负责将数据库查询的结果集映射为 PO。

2．为业务层提供接口.此对象用于访问数据库（CRUD 操作）.通常和 PO 结合使用,DAO 中包含了各种数据库的操作方法.通过它的方法,结合 PO 对数据库进行相关的操作.夹在业务逻辑与数据库资源中间.配合 VO, 提供数据库的 CRUD 操作。

6、DTO (Data Transfer Object)数据传输对象

（理解为 controller 层中：接收和返回的 java bean）

1.用在需要跨进程或远程传输时，它不应该包含业务逻辑。

2．比如一张表有 100 个字段，那么对应的 PO 就有 100 个属性（大多数情况下，DTO 内的数据来自多个表）。但 view 层只需显示 10 个字段，没有必要把整个 PO 对象传递到 client，这时我们就可以用只有这 10 个属性的 DTO 来传输数据到 client，这样也不会暴露 server 端表结构。到达客户端以后，如果用这个对象来对应界面显示，那此时它的身份就转为 VO。
