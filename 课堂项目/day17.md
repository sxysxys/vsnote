# day17：啥也没干的一天
## 问题
由于今天早上在解压canal的时候，它的压缩包中的文件夹和系统文件夹一样，但是我在解压时候给它安排目录，所以把系统文件夹替换了，系统就崩了。
## 解决
### 导出数据问题的解决
- 没有办法恢复，只能打算重装系统了，一开始还是像之前那样，把docker下挂载出的`mysql：data文件夹`拉出来想在本地mac的mysql上无缝接入，一点反应都没有，不管怎么按照网上的方法改文件都没用，所以就放弃了这种方法。暂且把它当做不同系统的不兼容。
- 之后查到网上的方法，使用`mysqldump`来进行sql脚本的导出。使用`mysqldump -uroot -p --all-databases > /all.sql`来将mysql中所有的数据库一并导出到根目录下的`all.sql`中。但是要注意，如果是linux中的mysql，那么此sql文件在linux的根目录下；但是如果是使用docker的话，那么它导出到本容器的根目录下。需要使用`docker copy`命令将容器中的文件导出到宿主机中。
- 由于之前折腾了半天本地mac数据库，基本被折腾坏了，索性将mysql直接卸载，参考[干净卸载mysql](https://www.jianshu.com/p/276c1271ae14)
- 接着在新安装的mysql中执行之前导出的sql文件，将数据库完全恢复到本地。

### 重新安装linux
参考鸟哥的linux。
1. 配置好本地的虚拟机硬件，uefi启动（这样默认加载启动就是gpt分区）、2g内存、40g硬盘、nat模式、选好iso镜像（进去再选也行）。
2. 进入引导界面，重点在配置分区那边、记得一定要对`/boot/efi`进行分区（50m）即可。对`/home 、 /  、  swap`这几个采用`lvm`来分区。
3. 配置一下网络ip地址，启动。
4. 启动完后，这时候应该会连不上网，可以进入`/etc/sysconfig/network-scripts/ifcfg-ens33`进行相应的配置。[参考博客](https://blog.csdn.net/BokeyGeGe/article/details/78631328)
5. 装相应的环境。由于linux安装自带了java8环境，所以无需下载。[参考博客](https://www.cnblogs.com/guipeng/p/11146679.html)

### 坑点
- 依旧是selinux和防火墙这些，之前都遇到过了。
- 启动nacos的时候还是说需要java路径，直接这是因为java环境变量配到了用户登陆的时候，而这些`systemctl`都是系统启动就要加载的，这时候还没走到环境变量那一步，所以我们直接在`startup.sh`最前面加上java的环境变量路径。（重复设置无所谓）

