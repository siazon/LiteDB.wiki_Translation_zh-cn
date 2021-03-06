LiteDB是一个ACID（原子性、一致性、隔离性、持久性）数据库，因此在并发访问时你的数据事务总是一致的。

### 事务日志

要保证数据完整性和容错性，在写入数据文件之前LiteDB使用一个临时文件来写入所有变化。在直接写到磁盘前，LiteDB创建一个临时文件(称为提交日志文件)来存储所有脏页面。日志总是与数据库文件位于同一个文件夹下，除了添加了`-journal`外，它与数据库文件的名称相同。

- 如果在写日志文件期间有任何错误，事务都会被终止 - 并且原数据文件不会有任何变化。
- 在写入所有提交页面到日志文件后，LiteDB开始将这些页面写入到你的数据文件。如果这时候出现一个错误，你的数据文件会损坏，因为不是所有页面都被写入了。LiteDB需要使用日志文件来再次重新写入所有页面。这个恢复操作会出现在你下次尝试打开数据库时。
- 当所有操作结束时，日志文件会被删除。

日志默认是启用的。你可以在连接字符串中禁用它来获得对某些磁盘的快速写操作！

```C#
var db = new LiteDatabase("filename=mydata.db; journal=false");
``` 

在v2版本的长事务期间，LiteDB使用日志文件作为变化页面的缓存。在内存中有5000个脏页面后，LiteDB在当前的日志文件中创建一个检查点并保存所有的脏页面。这可以防止在长事务期间消耗太多内存。
