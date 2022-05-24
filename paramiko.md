# PYTHON之SSH远程登录

一、SSH简介

　　SSH（Secure Shell）属于在传输层上运行的用户层协议，相对于Telnet来说具有更高的安全性。

二、SSH远程连接

　　SSH远程连接有两种方式，一种是通过用户名和密码直接登录，另一种则是用过密钥登录。（目前只用用户名和密码登录）

​		用户名和密码登录

```python
import paramiko

ssh = paramiko.SSHClient()
#跳过了远程连接中选择‘是’的环节
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
ssh.connect('IP',22,'用户名','密码')
stdin, stdout, stderr = ssh.exec_command('df')
```

>>>>>>> 
