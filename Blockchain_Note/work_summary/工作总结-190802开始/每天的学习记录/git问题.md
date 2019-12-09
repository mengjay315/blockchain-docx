## git问题

#### 查看git全局配置

git config --global  --list

#### mac 中git操作账号的保存与删除

#### git clone 报错 remote: Repository not found

https://www.jianshu.com/p/5bab36083973

## git pull和push设置密码

方法一：简单粗暴，https的方式

直接在添加远程仓库的时候在地址上写好用户名密码。例如；

https://用户名：密码@github.com/project/project.git

 

方法二：使用ssh方法，通过ssh-keygen命令生成公钥和密钥

注意几个坑；

1.提示 ssh-keygen 不是内部命令

解决：只要安装了git就有这个程序，位置在git的安装目录下，搜索一下。复制地址到电脑系统设置的path里。

2.ssh-keygen -t rsa -C "注册的邮箱"

这里存放的地址没有必要改动，否则无法使用

3.找到生成公钥，把它的内容复制到远程仓库的公钥设置里就可以使用了

最后测试一下是否设置成功 ，ssh -T [git@github.com](mailto:git@github.com)，输入yes

###  git配置用户名邮箱，全局配置/单仓库配置

https://www.cnblogs.com/xxoome/p/9183515.html

### **将本地的从其他git仓库上clone下来的代码上传push到一个新的远程git仓库中的方法**

https://blog.csdn.net/u010299133/article/details/88967476

#### 

