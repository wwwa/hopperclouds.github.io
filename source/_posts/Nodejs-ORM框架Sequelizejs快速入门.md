title: Nodejs ORM框架Sequelizejs快速入门
date: 2016-09-12 23:10:10
# 类别
categories:
  - 技术
# 标签
tags:
  - ORM
  - Nodejs
---
作者: Adam

## 什么是`ORM`？
简单的讲就是对`SQL`查询语句的封装，让我们可以用`OOP`的方式操作数据库，优雅的生成安全、可维护的`SQL`代码。直观上，是一种`Model`和`SQL`的映射关系。

```Javascript
const User = sequelize.define('user', {
    id: {
        type: Sequelize.INTEGER,
        allowNull: false,
        autoIncrement: true,
        primaryKey: true
    },
    email: {
        type: Sequelize.STRING,
        allowNull: false,
        validate: {
            isEmail: true
        },
        unique: true
    }
})
```
```SQL
CREATE TABLE IF NOT EXISTS `users` (
  `id` INTEGER PRIMARY KEY AUTOINCREMENT,
  `email` VARCHAR(255) NOT NULL UNIQUE
  `createdAt` DATETIME NOT NULL,
  `updatedAt` DATETIME NOT NULL,
  UNIQUE (email)
);
```

## 那么什么是Sequelize？

Sequelize是一款基于Nodejs功能强大的异步ORM框架。
同时支持PostgreSQL, MySQL, SQLite and MSSQL多种数据库，很适合作为Nodejs后端数据库的存储接口，为快速开发Nodejs应用奠定扎实、安全的基础。

既然Nodejs的强项在于异步，没有理由不找一个强大的支持异步的数据库框架，与之配合，双剑合并～

```javascript
//引入框架
var Sequelize = require('sequelize');
//初始化链接（支持连接池）
var sequelize = new Sequelize('database', 'username', 'password',  {
  host: 'localhost',
  dialect: 'mysql'|'sqlite'|'postgres'|'mssql',

  pool: {
    max: 5,
    min: 0,
    idle: 10000
  },

  // SQLite only
  storage: 'path/to/database.sqlite'
});
//定义数据模型
var User = sequelize.define('user', {
  username: Sequelize.STRING,
  birthday: Sequelize.DATE
});
//初始化数据
sequelize.sync().then(function() {
  return User.create({
    username: 'janedoe',
    birthday: new Date(1980, 6, 20)
  });
}).then(function(jane) {
  //获取数据
  console.log(jane.get({
    plain: true
  }));
}).catch(function (err) {
  //异常捕获
  console.log('Unable to connect to the database:', err);
});
```

## Sequelize有哪些特色？

> 强大的模型定义，支持虚拟类型。Javascript虽然被很多人诟病杂乱无章法，但是函数即对象这个特色，可以说是我的最爱，非常灵活强大。

```javascript
var Foo = sequelize.define('foo', {
  firstname: Sequelize.STRING,
  lastname: Sequelize.STRING
}, {
  getterMethods   : {
    fullName       : function()  { return this.firstname + ' ' + this.lastname }
  },
  setterMethods   : {
    fullName       : function(value) {
        var names = value.split(' ');
        this.setDataValue('firstname', names.slice(0, -1).join(' '));
        this.setDataValue('lastname', names.slice(-1).join(' '));
    },
  }
});
```

> 支持完善的数据验证，减轻前后端的验证压力。

```javascript
var ValidateMe = sequelize.define('foo', {
  foo: {
    type: Sequelize.STRING,
    validate: {
      is: ["^[a-z]+$",'i'],     // 全匹配字母
      is: /^[a-z]+$/i,          // 全匹配字母，用规则表达式写法
      not: ["[a-z]",'i'],       // 不能包含字母
      isEmail: true,            // 检查邮件格式
      isUrl: true,              // 是否是合法网址
      isIP: true,               // 是否是合法IP地址
      isIPv4: true,             // 是否是合法IPv4地址
      isIPv6: true,             // 是否是合法IPv6地址
      isAlpha: true,            // 是否是字母
      isAlphanumeric: true,     // 是否是数字和字母
      isNumeric: true,          // 只允许数字
      isInt: true,              // 只允许整数
      isFloat: true,            // 是否是浮点数
      isDecimal: true,          // 是否是十进制书
      isLowercase: true,        // 是否是小写
      isUppercase: true,        // 是否大写
      notNull: true,            // 不允许为null
      isNull: true,             // 是否是null
      notEmpty: true,           // 不允许为空
      equals: 'specific value', // 等于某些值
      contains: 'foo',          // 包含某些字符
      notIn: [['foo', 'bar']],  // 不在列表中
      isIn: [['foo', 'bar']],   // 在列表中
      notContains: 'bar',       // 不包含
      len: [2,10],              // 长度范围
      isUUID: 4,                // 是否是合法 uuids
      isDate: true,             // 是否是有效日期
      isAfter: "2011-11-05",    // 是否晚于某个日期
      isBefore: "2011-11-05",   // 是否早于某个日期
      max: 23,                  // 最大值
      min: 23,                  // 最小值
      isArray: true,            // 是否是数组
      isCreditCard: true,       // 是否是有效信用卡号

      // 自定义规则
      isEven: function(value) {
        if(parseInt(value) % 2 != 0) {
          throw new Error('请输入偶数!')
        }
      }
    }
  }
});
```

> Sequelize的查询非常全面和灵活

```javascript
Project.findAll({
  //复杂过滤，可嵌套
  where: {
    id: {
      $and: {a: 5}           // AND (a = 5)
      $or: [{a: 5}, {a: 6}]  // (a = 5 OR a = 6)
      $gt: 6,                // id > 6
      $gte: 6,               // id >= 6
      $lt: 10,               // id < 10
      $lte: 10,              // id <= 10
      $ne: 20,               // id != 20
      $between: [6, 10],     // BETWEEN 6 AND 10
      $notBetween: [11, 15], // NOT BETWEEN 11 AND 15
      $in: [1, 2],           // IN [1, 2]
      $notIn: [1, 2],        // NOT IN [1, 2]
      $like: '%hat',         // LIKE '%hat'
      $notLike: '%hat'       // NOT LIKE '%hat'
      $iLike: '%hat'         // ILIKE '%hat' (case insensitive)  (PG only)
      $notILike: '%hat'      // NOT ILIKE '%hat'  (PG only)
      $overlap: [1, 2]       // && [1, 2] (PG array overlap operator)
      $contains: [1, 2]      // @> [1, 2] (PG array contains operator)
      $contained: [1, 2]     // <@ [1, 2] (PG array contained by operator)
      $any: [2,3]            // ANY ARRAY[2, 3]::INTEGER (PG only)
    },
    status: {
      $not: false,           // status NOT FALSE
    }
  }
})
Project.all()
Project.findById
Project.findByOne
Project.findOrCreate
Project.findAndCountAll

Project.count()
Project.max()

//CRUD
Project.create()
Project.save()
Project.update()
Project.destroy()

//批量
User.bulkCreate([])

//排序
something.findOne({
  order: [
    'name',
    // 返回 `name`
    'username DESC',
    // 返回 `username DESC`
    ['username', 'DESC'],
    // 返回 `username` DESC
    sequelize.fn('max', sequelize.col('age')),
    // 返回 max(`age`)
    [sequelize.fn('max', sequelize.col('age')), 'DESC'],
    // 返回 max(`age`) DESC
    [sequelize.fn('otherfunction', sequelize.col('col1'), 12, 'lalala'), 'DESC'],
    // 返回 otherfunction(`col1`, 12, 'lalala') DESC
    [sequelize.fn('otherfunction', sequelize.fn('awesomefunction', sequelize.col('col'))), 'DESC']
    // 返回 otherfunction(awesomefunction(`col`)) DESC, 有可能是无限循环
    [{ raw: 'otherfunction(awesomefunction(`col`))' }, 'DESC']
    // 也可以这样写
  ]
})

// 分页查询
Project.findAll({ limit: 10 })
Project.findAll({ offset: 8 })
Project.findAll({ offset: 5, limit: 5 })
```

> 关联查询 `include` 支持嵌套，这可能是`ORM`里面最难的部分。

```javascript
var User = sequelize.define('user', { name: Sequelize.STRING })
  , Task = sequelize.define('task', { name: Sequelize.STRING })
  , Tool = sequelize.define('tool', { name: Sequelize.STRING })

Task.belongsTo(User) // 增加外键属性 UserId 到 Task
User.hasMany(Task) // 给 Task 增加外键属性 userId
User.hasMany(Tool, { as: 'Instruments' }) // 给 Task 增加自定义外键属性 InstrumentsId

Task.findAll({ include: [ User ] })
User.findAll({ include: [{
    model: Tool,
    as: 'Instruments',
    where: { name: { $like: '%ooth%' } }
}] })
User.findAll({ include: ['Instruments'] })

var User = this.sequelize.define('user', {/* attributes */}, {underscored: true})
  , Company  = this.sequelize.define('company', {
    uuid: {
      type: Sequelize.UUID,
      primaryKey: true
    }
  });

User.belongsTo(Company); // 增加 company_uuid 外键属性到 user
User.belongsTo(UserRole, {as: 'role'});
// 自定义外键属性 roleId 到 user 而不是 userRoleId

User.belongsTo(Company, {foreignKey: 'fk_companyname', targetKey: 'name'}); // 增加自定义外键属性 fk_companyname 到 User
Person.hasOne(Person, {as: 'Father', foreignKey: 'DadId'})
// Person 增加外键属性 DadId

Coach.hasOne(Team)  // `coachId` 作为 Team 的外键属性

Project.hasMany(User, {as: 'Workers'})
// 给 User 增加外键属性 projectId ／ project_id

Project.belongsToMany(User, {through: 'UserProject'});
User.belongsToMany(Project, {through: 'UserProject'});
// 创建新的模型: UserProject 包含外键属性：projectId 和 userId
```

> `Sequelize`还有完善的迁移同步数据方案,`migrate so easy`。

```
//$ sequelize db:migrate //用命令直接生成模版脚本，接下来的还是写js
module.exports = {
  up: function(queryInterface, Sequelize) {
    // 需要修改数据库的操作
  },

  down: function(queryInterface, Sequelize) {
    // 取消修改的操作
  }
}
```

好的，快餐吃到这里，希望大家喜欢`nodejs`，能够快速开发`Node App`～

