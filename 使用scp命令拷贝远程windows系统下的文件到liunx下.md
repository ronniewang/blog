## 使用scp命令拷贝远程windows系统下的文件到liunx下

我的系统是Win7

由于Win7不支持ssh，需要先安装ssh服务，可以从[这里](https://www.bitvise.com/ssh-server-download)下载安装

安装之后即可执行命令了

例如输入如下命令

`scp wsy@192.168.1.128:/e:/idea-workspace/robot/target/spider-robot.jar /opt/spider/spider-robot.jar`

输入密码

ok，`192.168.1.128`机器上的`/e:/idea-workspace/robot/target/spider-robot.jar`就到`/opt/spider/spider-robot.jar`了

> 注意windows路径的输入
