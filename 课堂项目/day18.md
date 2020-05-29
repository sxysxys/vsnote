# day18
## 安装mysql-linux
因为使用canal需要mysql进行同步数据，所以在linux中安装mysql用于练习。
1. 安装mysql：使用yum安装[安装流程](https://www.cnblogs.com/z0909y/p/10772854.html)
2. 安装的时候发现安装的很慢，选择切换成阿里的镜像源进行安装。

#### 坑点
安装好mysql后，按照上面的博客进行设置，但是出现了一系列问题，详见我写的博客。

## 使用canal进行数据同步
### 为什么使用canal
之前都是使用feign模块调用来实现获取其他数据库信息的，但是这样效率很低，当需要数据的时候反而会等很久，所以我们使用canal随时同步数据库，这样就可以在本地数据库检索数据了。
### 操作步骤
1. 首先在linux中安装好mysql。
2. 解压canal，修改其中的配置文件，例如数据库地址、同步表信息。
```shell
#需要改成自己的数据库信息
canal.instance.master.address=192.168.44.132:3306
#需要改成自己的数据库用户名与密码
canal.instance.dbUsername=canal
canal.instance.dbPassword=canal
#需要改成同步的数据库表规则，例如只是同步一下表
#canal.instance.filter.regex=.*\\..*
canal.instance.filter.regex=guli_ucenter.ucenter_member
```
3. 编写启动脚本，启动canal。
4. 在代码中创建相应的模块，连接上虚拟机中的canal，这时候linux中数据库中的表（配置文件中配置过的）发生变化，数据库的同名表就也会发生数据的改变（增删改），要注意数据库和表的名字得一样。


