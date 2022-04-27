windows版本

打开cmd，进入到解压后的路径，输入redis-server redis.windows.conf，服务启动

但关闭cmd后，服务就会停止

打开cmd，进入redis文件夹下，输入redis-server.exe --service-install redis.windows.conf --loglevel verbose回车，设置服务命令（需要管理员权限）

