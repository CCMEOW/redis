# RDB

RDB持久化可以手动执行，也可以定期执行。生成的是包括当前完整的Redis数据的文件。
RDB持久化的命令有SAVE(阻塞Redis请求直到完成)和BGSAVE(异步执行，不阻塞Redis请求)

定期执行RDB持久化的命令为 save a b，表示在a时间内发生了至少b次修改，则会执行BGSAVE命令。