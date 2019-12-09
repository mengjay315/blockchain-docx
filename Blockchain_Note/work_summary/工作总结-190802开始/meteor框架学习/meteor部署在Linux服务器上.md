## Meteor部署在服务器上（生产环境部署）

### Meteor部署 (Nginx+pm2+mongodb)

https://www.jianshu.com/p/6db84b83ac39

## pm2学习使用

Node.js Production Process Manager with a built-in Load Balancer. [http://pm2.keymetrics.io](http://pm2.keymetrics.io/)

http://pm2.keymetrics.io/docs/usage/application-declaration/#generate-configuration

### pm2.json在实际场景中的应用

https://www.jianshu.com/p/a1c2963c00cc

### pm2运行json文件启动node项目及pm2-web的安装

https://blog.csdn.net/weixin_34315665/article/details/92719785

### [pm2进阶使用]

(https://www.cnblogs.com/coolslider/p/8510993.html)

### Meteor部署在CentOS环境下

https://www.wandouip.com/t5i14266/



------------



### meteor 连接外置MongoDB

https://cloud.tencent.com/developer/ask/99322

### Windows开发环境 Meteor + MongoDB 独立环境部署团队多端协作开发配置从入门到把坑填平

https://www.jianshu.com/p/a7bab365defa

### 创建一个meteor项目

https://blog.csdn.net/Long861774/article/details/82588625

### Meteor 开发环境 mongodb 的连接

https://www.mycode.net.cn/database/1362.html

### 使用 meteor shell 进行管理

通过 meteor shell 管理 mongodb 不需要指定端口，你只要在 meteor 项目启动后的项目目录下执行 `meteor mongo` 就可以连接到数据库了。如下：

```
myCode:~/Project/microduino$ meteor mongo
MongoDB shell version: 2.6.7
connecting to: 127.0.0.1:3001/meteor
meteor:PRIMARY>
```

这样就连接到了当前项目的 mongodb 数据库，通过 `help` 命令可以看到帮助。

![image-20191021092837847](../../工作总结-190802开始/meteor框架学习/assets/image-20191021092837847.png)

----

### 