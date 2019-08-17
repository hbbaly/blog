# sequelize

## 安装

```sh
npm install --save sequelize
```

使用`mysql`所以还要安装

```sh
npm install --save mysql2
```

## 建立连接

### 一

```js
const Sequelize = require('sequelize');

const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  port: 3306, //端口号，默认3306
  dialect: /* 'mysql' | 'mariadb' | 'postgres' | 'mssql' 之一 */
});
```

### 二

```js
const sequelize = new Sequelize('postgres://user:pass@example.com:port/dbname');
```

### 连接池 (生产环境)

```js
const sequelize = new Sequelize(/* ... */, {
  // ...
  pool: {
    max: 5,
    min: 0,
    acquire: 30000,
    idle: 10000
  }
});
```

如果你希望最大连接池大小为 90 并且你有三个进程,则每个进程的 `Sequelize` 实例的最大连接池大小应为 30.

## 表建模

有两种方式，不过不推荐使用 `define` ,使用类方法建模.

```js
const { Model } = Sequelize;
class User extends Model {}
User.init({
  // 属性
  firstName: {
    type: Sequelize.STRING,
    allowNull: false
  },
  lastName: {
    type: Sequelize.STRING
    // allowNull 默认为 true
  }
}, {
  sequelize,
  modelName: 'user'
  // 参数
});
```

`Sequelize` 还默认为每个模型定义了字段`id`(主键),`createdAt`和`updatedAt`

你也可以添加很多参数

```js
User.init({
 // 如果未赋值,则自动设置值为 TRUE
 flag: { type: Sequelize.BOOLEAN, allowNull: false, defaultValue: true},

 // 设置默认时间为当前时间
 myDate: { type: Sequelize.DATE, defaultValue: Sequelize.NOW },

 // 将allowNull设置为false会将NOT NULL添加到列中,
 // 这意味着当列为空时执行查询时将从DB抛出错误. 
 // 如果要在查询DB之前检查值不为空,请查看下面的验证部分.
 title: { type: Sequelize.STRING, allowNull: false},

 // 创建具有相同值的两个对象将抛出一个错误. 唯一属性可以是布尔值或字符串.
 // 如果为多个列提供相同的字符串,则它们将形成复合唯一键.
 uniqueOne: { type: Sequelize.STRING,  unique: 'compositeIndex'},
 uniqueTwo: { type: Sequelize.INTEGER, unique: 'compositeIndex'},

 // unique属性用来创建一个唯一约束.
 someUnique: {type: Sequelize.STRING, unique: true},
 
 // 这与在模型选项中创建索引完全相同.
 {someUnique: {type: Sequelize.STRING}},
 {indexes: [{unique: true, fields: ['someUnique']}]},

 // primaryKey用于定义主键.
 identifier: { type: Sequelize.STRING, primaryKey: true},

 // autoIncrement可用于创建自增的整数列
 incrementMe: { type: Sequelize.INTEGER, autoIncrement: true },

 // 你可以通过'field'属性指定自定义列名称:
 fieldWithUnderscores: { type: Sequelize.STRING, field: 'field_with_underscores' },

 // 这可以创建一个外键:
 bar_id: {
   type: Sequelize.INTEGER,

   references: {
     // 这是引用另一个模型
     model: Bar,

     // 这是引用模型的列名称
     key: 'id',

     // 这声明什么时候检查外键约束. 仅限PostgreSQL.
     deferrable: Sequelize.Deferrable.INITIALLY_IMMEDIATE
   }
 },

 // 仅可以为 MySQL,PostgreSQL 和 MSSQL 的列添加注释
 commentMe: {
   type: Sequelize.INTEGER,

   comment: '这是一个包含注释的列名'
 }
}, {
  sequelize,
  modelName: 'user'
});
```
## 自定义模型参数

```js
const sequelize = new Sequelize(/* ... */, {
  // ...
define: {
    // 显示 create_time, update_time
    timestamps: true,
    // delete_time
    paranoid: true,
    createdAt: 'created_at',
    updatedAt: 'updated_at',
    deletedAt: 'deleted_at',
    underscored: true ,  // 吧所有驼峰转化为下划线
    scopes:{
      'bh':{
        attributes: {
          exclude: ['updated_at', 'deleted_at','created_at']
        },
      }
    }
  }
})
```

## 将模型与数据库同步

```js
// 创建表:
Project.sync()
Task.sync()

// 强制创建!
Project.sync({force: true}) // 这将先丢弃表,然后重新创建它

// 删除表:
Project.drop()
Task.drop()

// 事件处理:
Project.[sync|drop]().then(() => {
  // 好吧...一切都很好！
}).catch(error => {
  // oooh,你输入了错误的数据库凭据？
})
```

因为同步和删除所有的表可能要写很多行,你也可以让`Sequelize`来为做这些


```js
// 同步所有尚未在数据库中的模型
sequelize.sync()

// 强制同步所有模型
sequelize.sync({force: true})

// 删除所有表
sequelize.drop()

// 广播处理:
sequelize.[sync|drop]().then(() => {
  // woot woot
}).catch(error => {
  // whooops
})
```

因为`.sync({ force: true })`是具有破坏性的操作,可以使用`match`参数作为附加的安全检查.

`match`参数可以通知`Sequelize`,以便在同步之前匹配正则表达式与数据库名称 - 在测试中使用`force:true`但不使用实时代码的情况下的安全检查.
```js
// 只有当数据库名称以'_test'结尾时,才会运行.sync()
sequelize.sync({ force: true, match: /_test$/ });
```

## 扩展模型
`Sequelize` 模型是`ES6`类. 你可以轻松添加自定义实例或类级别的方法
```js
class User extends Model {
  // 添加一个类级别的方法
  static classLevelMethod() {
    return 'foo';
  }

  // 添加实例级别方法
  instanceLevelMethod() {
    return 'bar';
  }
}
User.init({ firstname: Sequelize.STRING }, { sequelize });
```

当然,你还可以访问实例的数据并生成虚拟的`getter`:
```js
class User extends Model {
  getFullname() {
    return [this.firstname, this.lastname].join(' ');
  }
}
User.init({ firstname: Sequelize.STRING, lastname: Sequelize.STRING }, { sequelize });

// 示例:
User.build({ firstname: 'foo', lastname: 'bar' }).getFullname() // 'foo bar'
```

## 数据类型

```js
Sequelize.STRING                      // VARCHAR(255)
Sequelize.STRING(1234)                // VARCHAR(1234)
Sequelize.STRING.BINARY               // VARCHAR BINARY
Sequelize.TEXT                        // TEXT
Sequelize.TEXT('tiny')                // TINYTEXT
Sequelize.CITEXT                      // CITEXT      仅 PostgreSQL 和 SQLite.

Sequelize.INTEGER                     // INTEGER
Sequelize.BIGINT                      // BIGINT
Sequelize.BIGINT(11)                  // BIGINT(11)

Sequelize.FLOAT                       // FLOAT
Sequelize.FLOAT(11)                   // FLOAT(11)
Sequelize.FLOAT(11, 10)               // FLOAT(11,10)

Sequelize.REAL                        // REAL        仅 PostgreSQL.
Sequelize.REAL(11)                    // REAL(11)    仅 PostgreSQL.
Sequelize.REAL(11, 12)                // REAL(11,12) 仅 PostgreSQL.

Sequelize.DOUBLE                      // DOUBLE
Sequelize.DOUBLE(11)                  // DOUBLE(11)
Sequelize.DOUBLE(11, 10)              // DOUBLE(11,10)

Sequelize.DECIMAL                     // DECIMAL
Sequelize.DECIMAL(10, 2)              // DECIMAL(10,2)

Sequelize.DATE                        // mysql / sqlite 为 DATETIME, postgres 为带时区的 TIMESTAMP
Sequelize.DATE(6)                     // DATETIME(6) 适用 mysql 5.6.4+. 小数秒支持最多6位精度
Sequelize.DATEONLY                    // DATE 不带时间.
Sequelize.BOOLEAN                     // TINYINT(1)

Sequelize.ENUM('value 1', 'value 2')  // 一个允许值为'value 1'和'value 2'的ENUM
Sequelize.ARRAY(Sequelize.TEXT)       // 定义一个数组. 仅 PostgreSQL.
Sequelize.ARRAY(Sequelize.ENUM)       // 定义一个ENUM数组. 仅 PostgreSQL.

Sequelize.JSON                        // JSON 列. 仅 PostgreSQL, SQLite 和 MySQL.
Sequelize.JSONB                       // JSONB 列. 仅 PostgreSQL.

Sequelize.BLOB                        // BLOB (PostgreSQL 为 bytea)
Sequelize.BLOB('tiny')                // TINYBLOB (PostgreSQL 为 bytea. 其余参数是 medium 和 long)

Sequelize.UUID                        // PostgreSQL 和 SQLite 的 UUID 数据类型,MySQL 的 CHAR(36) BINARY(使用defaultValue:Sequelize.UUIDV1 或 Sequelize.UUIDV4 来让 sequelize 自动生成 id).

Sequelize.CIDR                        // PostgreSQL 的 CIDR 数据类型
Sequelize.INET                        // PostgreSQL 的 INET 数据类型
Sequelize.MACADDR                     // PostgreSQL 的 MACADDR 数据类型

Sequelize.RANGE(Sequelize.INTEGER)    // 定义 int4range 范围. 仅 PostgreSQL.
Sequelize.RANGE(Sequelize.BIGINT)     // 定义 int8range 范围. 仅 PostgreSQL.
Sequelize.RANGE(Sequelize.DATE)       // 定义 tstzrange 范围. 仅 PostgreSQL.
Sequelize.RANGE(Sequelize.DATEONLY)   // 定义 daterange 范围. 仅 PostgreSQL.
Sequelize.RANGE(Sequelize.DECIMAL)    // 定义 numrange 范围. 仅 PostgreSQL.

Sequelize.ARRAY(Sequelize.RANGE(Sequelize.DATE)) // 定义 tstzrange 范围的数组. 仅 PostgreSQL.

Sequelize.GEOMETRY                    // Spatial 列. 仅 PostgreSQL (带有 PostGIS) 或 MySQL.
Sequelize.GEOMETRY('POINT')           // 带有 geometry 类型的 spatial 列. 仅 PostgreSQL (带有 PostGIS) 或 MySQL.
Sequelize.GEOMETRY('POINT', 4326)     // 具有 geometry 类型和 SRID 的 spatial 列. 仅 PostgreSQL (带有 PostGIS) 或 MySQL.
```

## Getters & setters

对象属性`getter`和`setter`函数,这些可以用于映射到数据库字段的“保护”属性,也可以用于定义“伪”属性.

```js
User.init({
  name: {
    type: Sequelize.STRING,
    allowNull: false,
    get() {
      const title = this.getDataValue('title');
      // 'this' 允许你访问实例的属性
      return this.getDataValue('name') + ' (' + title + ')';
    },
  },
  title: {
    type: Sequelize.STRING,
    allowNull: false,
    set(val) {
      this.setDataValue('title', val.toUpperCase());
    }
  }
}, { sequelize, modelName: 'employee' });

User
  .create({ name: 'John Doe', title: 'senior engineer' })
  .then(user => {
    console.log(user.get('name')); // John Doe (SENIOR ENGINEER)
    console.log(user.get('title')); // SENIOR ENGINEER
  })
```
### 用于 getter 和 setter 定义内部的 Helper 方法
检索底层属性值 - 总是使用 `this.getDataValue()`
```js
/* 一个用于 'title' 属性的 getter */
get() {
  return this.getDataValue('title')
}
```
设置基础属性值 - 总是使用 `this.setDataValue()`
```js
/* 一个用于 'title' 属性的 setter */
set(title) {
  this.setDataValue('title', title.toString().toLowerCase());
}
```

## 验证

模型验证允许你为模型的每个属性指定格式/内容/继承验证

验证会自动运行在 `create` , `update` 和 `save` 上. 你也可以调用 `validate()` 手动验证一个实例

### 属性验证器

你可以自定义验证器或使用由`validator.js`实现的几个内置验证器


```js
class ValidateMe extends Model {}
ValidateMe.init({
  bar: {
    type: Sequelize.STRING,
    validate: {
      is: ["^[a-z]+$",'i'],     // 只允许字母
      is: /^[a-z]+$/i,          // 与上一个示例相同,使用了真正的正则表达式
      not: ["[a-z]",'i'],       // 不允许字母
      isEmail: true,            // 检查邮件格式 (foo@bar.com)
      isUrl: true,              // 检查连接格式 (http://foo.com)
      isIP: true,               // 检查 IPv4 (129.89.23.1) 或 IPv6 格式
      isIPv4: true,             // 检查 IPv4 (129.89.23.1) 格式
      isIPv6: true,             // 检查 IPv6 格式
      isAlpha: true,            // 只允许字母
      isAlphanumeric: true,     // 只允许使用字母数字
      isNumeric: true,          // 只允许数字
      isInt: true,              // 检查是否为有效整数
      isFloat: true,            // 检查是否为有效浮点数
      isDecimal: true,          // 检查是否为任意数字
      isLowercase: true,        // 检查是否为小写
      isUppercase: true,        // 检查是否为大写
      notNull: true,            // 不允许为空
      isNull: true,             // 只允许为空
      notEmpty: true,           // 不允许空字符串
      equals: 'specific value', // 只允许一个特定值
      contains: 'foo',          // 检查是否包含特定的子字符串
      notIn: [['foo', 'bar']],  // 检查是否值不是其中之一
      isIn: [['foo', 'bar']],   // 检查是否值是其中之一
      notContains: 'bar',       // 不允许包含特定的子字符串
      len: [2,10],              // 只允许长度在2到10之间的值
      isUUID: 4,                // 只允许uuids
      isDate: true,             // 只允许日期字符串
      isAfter: "2011-11-05",    // 只允许在特定日期之后的日期字符串
      isBefore: "2011-11-05",   // 只允许在特定日期之前的日期字符串
      max: 23,                  // 只允许值 <= 23
      min: 23,                  // 只允许值 >= 23
      isCreditCard: true,       // 检查有效的信用卡号码

      // 自定义验证器的示例:
      isEven(value) {
        if (parseInt(value) % 2 !== 0) {
          throw new Error('Only even values are allowed!');
        }
      }
      isGreaterThanOtherField(value) {
        if (parseInt(value) <= parseInt(this.otherField)) {
          throw new Error('Bar must be greater than otherField.');
        }
      }
    }
  }
}, { sequelize });
```

## 配置

```js
class User extends Model {}
User.init({ /* bla */ }, {
  // 模型的名称. 该模型将以此名称存储在`sequelize.models`中.
  // 在这种情况下,默认为类名,即Bar. 
  // 这将控制自动生成的foreignKey和关联命名的名称
  modelName: 'bar',
  // 不添加时间戳属性 (updatedAt, createdAt)
  timestamps: false,

  // 不删除数据库条目,但将新添加的属性deletedAt设置为当前日期(删除完成时). 
  // paranoid 只有在启用时间戳时才能工作
  paranoid: true,

  // 将自动设置所有属性的字段参数为下划线命名方式.
  // 不会覆盖已经定义的字段选项
  underscored: true,

  // 禁用修改表名; 默认情况下,sequelize将自动将所有传递的模型名称(define的第一个参数)转换为复数. 如果你不想这样,请设置以下内容
  freezeTableName: true,

  // 定义表的名称
  tableName: 'my_very_custom_table_name',

  // 启用乐观锁定. 启用时,sequelize将向模型添加版本计数属性,
  // 并在保存过时的实例时引发OptimisticLockingError错误.
  // 设置为true或具有要用于启用的属性名称的字符串.
  version: true,

  // Sequelize 实例
  sequelize,
})
```

如果你希望`sequelize`处理时间戳,但只想要其中一部分,或者希望你的时间戳被称为别的东西,则可以单独覆盖每个列:

```js
class User extends Model {}
User.init({ /* bla */ }, {
  // 不要忘记启用时间戳！
  timestamps: true,

  // 我不想要 createdAt
  createdAt: false,

  // 我想 updateAt 实际上被称为 updateTimestamp
  updatedAt: 'updateTimestamp',

  // 并且希望 deletedA t被称为 destroyTime(请记住启用paranoid以使其工作)
  deletedAt: 'destroyTime',
  paranoid: true,

  sequelize,
})
```

## 数据检索

### find - 搜索数据库中的一个特定元素

```js
// 搜索已知的ids
Project.findByPk(123).then(project => {
  // project 将是 Project的一个实例,并具有在表中存为 id 123 条目的内容.
  // 如果没有定义这样的条目,你将获得null
})

// 搜索属性
Project.findOne({ where: {title: 'aProject'} }).then(project => {
  // project 将是 Projects 表中 title 为 'aProject'  的第一个条目 || null
})


Project.findOne({
  where: {title: 'aProject'},
  attributes: ['id', ['name', 'title']]
}).then(project => {
  // project 将是 Projects 表中 title 为 'aProject'  的第一个条目 || null
  // project.get('title') 将包含 project 的 name
})

```

### findOrCreate - 搜索特定元素或创建它

方法 `findOrCreate` 可用于检查数据库中是否已存在某个元素. 如果是这种情况,则该方法将生成相应的实例. 如果元素不存在,将会被创建.

如果是这种情况,则该方法将导致相应的实例. 如果元素不存在,将会被创建.

有一个空的数据库,一个 `User` 模型有一个 `username` 和 `job`

```js
User
  .findOrCreate({where: {username: 'sdepold'}, defaults: {job: 'Technical Lead JavaScript'}})
  . then(([user, created]) => {
    console.log(user.get({
      plain: true
    }))
    console.log(created)

    /*
    findOrCreate 返回一个包含已找到或创建的对象的数组,找到或创建的对象和一个布尔值,如果创建一个新对象将为true,否则为false,像这样:

    [ {
        username: 'sdepold',
        job: 'Technical Lead JavaScript',
        id: 1,
        createdAt: Fri Mar 22 2013 21: 28: 34 GMT + 0100(CET),
        updatedAt: Fri Mar 22 2013 21: 28: 34 GMT + 0100(CET)
      },
      true ]

在上面的例子中,第三行的数组将分成2部分,并将它们作为参数传递给回调函数,在这种情况下将它们视为 "user" 和 "created" .(所以“user”将是返回数组的索引0的对象,并且 "created" 将等于 "true".)
    */
  })
```

### findAndCountAll - 在数据库中搜索多个元素,返回数据和总计数

这是一个方便的方法,它结合了 `findAll` 和 `count`(见下文),当处理与分页相关的查询时,这是有用的,你想用 `limit` 和 `offset` 检索数据,但也需要知道总数与查询匹配的记录数

处理程序成功将始终接收具有两个属性的对象:

- `count` - 一个整数,总数记录匹配`where`语句和关联的其它过滤器
- `rows` - 一个数组对象,记录在`limit`和`offset`范围内匹配`where`语句和关联的其它过滤器

```js
Project
  .findAndCountAll({
     where: {
        title: {
          [Op.like]: 'foo%'
        }
     },
     offset: 10,
     limit: 2
  })
  .then(result => {
    console.log(result.count);
    console.log(result.rows);
  });
```
### findAll - 搜索数据库中的多个元素

```js
// 找到多个条目
Project.findAll().then(projects => {
  // projects 将是所有 Project 实例的数组
})

// 搜索特定属性 - 使用哈希
Project.findAll({ where: { name: 'A Project' } }).then(projects => {
  // projects将是一个具有指定 name 的 Project 实例数组
})

// 在特定范围内进行搜索
Project.findAll({ where: { id: [1,2,3] } }).then(projects => {
  // projects将是一系列具有 id 1,2 或 3 的项目
  // 这实际上是在做一个 IN 查询
})

Project.findAll({
  where: {
    id: {
      [Op.and]: {a: 5},           // 且 (a = 5)
      [Op.or]: [{a: 5}, {a: 6}],  // (a = 5 或 a = 6)
      [Op.gt]: 6,                // id > 6
      [Op.gte]: 6,               // id >= 6
      [Op.lt]: 10,               // id < 10
      [Op.lte]: 10,              // id <= 10
      [Op.ne]: 20,               // id != 20
      [Op.between]: [6, 10],     // 在 6 和 10 之间
      [Op.notBetween]: [11, 15], // 不在 11 和 15 之间
      [Op.in]: [1, 2],           // 在 [1, 2] 之中
      [Op.notIn]: [1, 2],        // 不在 [1, 2] 之中
      [Op.like]: '%hat',         // 包含 '%hat'
      [Op.notLike]: '%hat',       // 不包含 '%hat'
      [Op.iLike]: '%hat',         // 包含 '%hat' (不区分大小写)  (仅限 PG)
      [Op.notILike]: '%hat',      // 不包含 '%hat'  (仅限 PG)
      [Op.overlap]: [1, 2],       // && [1, 2] (PG数组重叠运算符)
      [Op.contains]: [1, 2],      // @> [1, 2] (PG数组包含运算符)
      [Op.contained]: [1, 2],     // <@ [1, 2] (PG数组包含于运算符)
      [Op.any]: [2,3],            // 任何数组[2, 3]::INTEGER (仅限 PG)
    },
    status: {
      [Op.not]: false,           // status 不为 FALSE
    }
  }
})
```

