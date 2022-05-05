windows版本

打开cmd，进入到解压后的路径，输入redis-server redis.windows.conf，服务启动

但关闭cmd后，服务就会停止

打开cmd，进入redis文件夹下，输入redis-server.exe --service-install redis.windows.conf --loglevel verbose回车，设置服务命令（需要管理员权限）

服务启动后可以运行redis客户端，redis-cli.exe

客户端内的命令有：set, get......

事务标记开始：MULTI 

命令入队：INCR

执行：EXEC  注意当EXEC前出现命令(语法？)错误，redis会终止事务进行报错，

   而出现命令(语法没错，数据key，value？)错误，redis会忽略这条指令继续执行事务。

监视key是否被改动过：watch 在发现key被更改后，在执行EXEC是就会返回nil，表示事务无法被触发。
