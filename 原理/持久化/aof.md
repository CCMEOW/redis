# AOF

和RDB每次生成一个新的完整数据的文件不同，AOF每次修改同一个文件。并且RDB写入文件的是key和value的值，而AOF写入的则是Redis命令本身。每当Redis有新的修改命令时，都会追加到现有的AOF文件后。
但要注意的是，由于操作系统的限制，追加到文件后并不一定实时写到磁盘中，而是写到缓冲区中，同时操作系统提供了相关命令(fsync和fdatasync)，将缓冲区的内容刷到磁盘中。

AOF文件有上限，但超过上限后，则使用AOF重写。

#### AOF重写
AOF重写，会创建一个新的文件，根据当前Redis服务器中的数据，生成一份新的命令，大大减少了原有AOF文件的大小。

由于Redis是单线程服务器，为了在重写时仍然能够接受新的请求，Redis会开启一个新的进程去做重写。同时在重写时，服务器不仅接受新的命令，还会将其写到两个
区域: AOF缓冲区和AOF重写缓冲区。当将现有数据重写到新的AOF文件的工作完成后，Redis会将AOF重写缓冲区的内容写到重写的文件中。这样新的AOF文件与当前Redis数据库数据一致，就可以用新的AOF文件覆盖旧的文件了。

例如，假如执行以下命令:   
```text
redis> RPUSH list "A" "B"
(integer) 2

redis> RPUSH list "C"
(integer) 3

redis> LPOP list
"A"
```
AOF文件需要记录3条命令: 
```text
*2$6SELECT
$10*4$5RPUSH$4list$1A$1B
*3$5RPUSH$4list$1C
*2$4LPOP$4list
```
而AOF重写文件只需要记录一条命令: 
```text
*2$6SELECT
$10*4$5RPUSH$4list$1B$1C
```