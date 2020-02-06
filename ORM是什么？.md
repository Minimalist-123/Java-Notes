它是什么
字面意思
实际上就是个缩写，表示对象-关系映射的缩写

O->Object
RM->Relational (关系) Mapping(映射)
代表什么思想
实际上就是一种把数据库映射成对象的想法

数据库的表（table） --> 类（class）
记录（record，行数据）--> 对象（object）
字段（field）--> 对象的属性（attribute）
比如说查询语句 SELECT id, first_name, last_name, phone, birth_date, sex FROM persons WHERE id = 10

对应到代码就是

res = db.执行数据库(sql);
name = res[0]["FIRST_NAME"];
那么ORM的写法就是

p = Person.get(10);
name = p.first_name;
这样的好处就是可以不需要了解数据库底层，因为它不需要接触SQL语句

所以ORM有这样一些优点

ORM生态已经比较完备，有很多的工具支持
天生的MVC，ORM就是天生的Model
可以不写SQL了。
它的缺点来说：

复杂查询很难做到，做到了性能也很差
学习成本比较高
由于不需要接触SQL所以无法定制一些特殊的SQL
命名规范
目前公认最规范的是Ruby 语言的 Active Record。Active Record 对于对象和数据库表的映射，有一些命名限制。

（1）一个类对应一张表。类名是单数，且首字母大写；表名是复数，且全部是小写。比如，表 books 对应类 Book。

（2）如果名字是不规则复数，则类名依照英语习惯命名，比如，表 mice 对应类 Mouse，表 people 对应类 Person。

（3）如果名字包含多个单词，那么类名使用首字母全部大写的骆驼拼写法，而表名使用下划线分隔的小写单词。比如，表 book_clubs 对应类 BookClub，表 line_items 对应类 LineItem。

（4）每个表都必须有一个主键字段，通常是叫做 id 的整数字段。外键字段名约定为单数的表名 + 下划线 + id，比如 item_id 表示该字段对应 items 表的 id 字段。

示例库 OpenRecord
OpenRecord 是仿 Active Record 的，将其移植到了 JavaScript，而且实现得很轻量级，学习成本较低。我写了一个示例库，请将它克隆到本地。

连接数据库
使用 ORM 的第一步，就是你必须告诉它，怎么连接数据库（完整代码看这里）。

// demo01.js
const Store = require('openrecord/store/sqlite3');

const store = new Store({
  type: 'sqlite3',
  file: './db/sample.db',
  autoLoad: true,
});

await store.connect();
连接成功以后，就可以操作数据库了。

Model
没啥好说的，就是ORM的框架会把表转成类对象

CRUD 操作
也没什么好说的，增删改查都从查询语句变成了调用方法

关系
表与表之间的关系（relation），分成三种。

一对一（one-to-one）：一种对象与另一种对象是一一对应关系，比如一个学生只能在一个班级。
一对多（one-to-many）： 一种对象可以属于另一种对象的多个实例，比如一张唱片包含多首歌。
多对多（many-to-many）：两种对象彼此都是 "一对多" 关系，比如一张唱片包含多首歌，同时一首歌可以属于多张唱片。
了解到这就足够用了

作者：本大少_
链接：https://www.jianshu.com/p/73c9c79392f2
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
