[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheets)

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
