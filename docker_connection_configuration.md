# 完整的docker服务器配置过程

1. docker run -p 10022:22 --runtime=nvidia --ipc=host --shm-size=32G --gpus all --name your_name -v /:/mnt --privileged=true -it your_docker_image /bin/bash
   
   - 这个使用的是所有gpu，建议直接指定gpu，不要用all，用all容易有0号显卡爆显存的问题
   
   - 映射这一步中的 -v /:/mnt --privileged=true 非常重要，把宿主机/mnt里的所有目录映射到容器里，打开宿主机上的文件时只要在前面加上/mnt即可，eg. cd /mnt/mnt/users/...
   
   - 再次进行创建，只使用gpu 6：docker run -p 10023:22 --runtime=nvidia --ipc=host --shm-size=32G --gpus='"device=6"' --name your_name -v /:/mnt -it your_docker_image /bin/bash（如果加上--privileged=true就还是使用所有的gpu，要使用多gpu可以--gpus='"device=5,6"'）

显示root@（容器ID比如是12345）

接着进入已经创建好的容器：docker exec -it 12345 /bin/bash，如果没有启动就运行：docker start 12345

2. apt update  
   apt install -y openssh-server

3. mkdir /var/run/sshd

4. 修改密码和用户名（用户名默认是root，此处不改），输入passwd进行密码的修改，记住密码 

5. sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config  
   sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd  
   echo "export VISIBLE=now" >> /etc/profile

6. 进入config修改设置：vim /etc/ssh/sshd_config

发现这个容器里没有安装vim，使用yum安装也发现没有yum，直接用`apt-get install vim*`即可。

vim的命令和使用可参考：https://blog.csdn.net/ycsdn10/article/details/121933411

7. 进入config文件后：将 `PermitRootLogin` 参数修改为 `yes`，将 `PasswordAuthentication` 参数修改为 `yes`，将`PubkeyAuthentication` 改为 `yes`

8. 重启ssh服务：`service ssh restart`

9. 进行验证：

```python
docker port your_name 22
# 如果前面的配置生效了，你会看到如下输出
# 0.0.0.0:10022
ssh root@(连接的服务器的IP地址) -p 10022
# 密码是你前面自己设置的
# 如果能进入docker的命令行，则表示成功
```

10. 验证完进行pycharm里的配置：
    - 接下来配置PyCharmTools > Deployment > Configuration
    - 再配置PyCharm的File > Setting > Project > Python Interpreter设置远程解释器
    - 注意端口的对应关系，其他的都是跟之前配置过的一致

在选择解释器的时候，Interpreter路径查看方法（在docker中输入）：which python，得到路径：/opt/conda/bin/python

如果terminal中输入python报错-bash: python: command not found，则解决方案是建立软链接： ln -sf /opt/conda/bin/python3 /usr/bin/python。需要注意的是前面的路径就是真实docker中的python的路径（which python显示的路径），后面的写法是统一的。

一些命令的补充：

- docker删除镜像：docker rmi （image名称）

- docker删除容器：docker rm -f 镜像的ID
