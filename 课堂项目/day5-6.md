## day5-6
### 使用阿里云oss进行存储服务
1. 注册登录阿里云。
2. 在管理控制台中创建Bucket。
3. 此时便可以通过各种api实现上传文件。

### 上传头像到阿里云
1. 调用java的api。
2. 将上传的头像文件传入接口，后端用`InputStream`进行一下转换，将文件传入阿里云oss。
3. 自己拼接访问资源的url，将url通过字符串的形式返回调用接口的前端，前端再调用其它接口存入数据库。
#### 注意事项
- 传入的头像由于名字可能相同，所以我们加一个uuid进行区分。
- 我们将资源在oss中的存储按日期作分类文件夹存储，需要对文件命名的时候加入`/aa/bb/2.jpg`这种形式，其中日期通过`LocalDateTime`进行获取和格式化操作。
- 后端使用`MutipartFile`进行接收文件，可以对`MutipartFile`获得其流对象，或者得到其`File`对象进行操作。。

### 使用nginx进行反向代理
目前我们项目中有多个模块，分别占用了两个端口：8001和8002，我们vue中接口请求的还是只写了一个端口地址，那么怎么请求多个端口呢？这时就要用到nginx了。
#### 搭建步骤
用虚拟机linux装好nginx，搭建步骤：
1. [yum安装nginx步骤和相关命令](https://www.cnblogs.com/tenghao510/p/11990353.html)
2. 装好以后，打开`/etc/nginx/conf.d/default.conf`文件，在其中添加上相应的转发路径，依据请求的url内容不同(`~/eduservice/`、`~/eduoss/`)来转发到不同的服务器端口。
3. 我们可以使用本机在虚拟机中的主机ip(vmnet8)来进行转发。  
   ```shell   
    location ~/eduservice/ {
        proxy_pass http://172.16.65.1:8001;  //本机的虚拟机ip 
    }
    location ~/eduoss/ {
        proxy_pass http://172.16.65.1:8002;
    }
    ```
4. 配置好以后按照1中的命令重启nginx，前端访问接口时即可访问nginx的`172.16.65.10:80`来起到访问多个服务器端口的效果。
#### 注意问题
- 在搭建配置完文件后，访问的时候查看日志`/var/log/nginx/error.log`一直显示`Permission not allow`，这是selinux的问题。
[解决方法1](https://www.cnblogs.com/songxingzhu/p/10063043.html),[永久关闭](https://www.iteye.com/blog/fanyc-2441673)
- 还有一个坑点：当虚拟机重启后，防火墙默认开启，这时候当nginx启动会发现根本访问不到，需要使用`systemctl disable firewalld`来关闭防火墙服务即可。
[防火墙的常见命令](https://www.cnblogs.com/chinaifae/p/10183214.html)

### 上传头像的前端部分
整合element-ui进行实现。
#### 注意问题
- 注意在请求标签action前面加上`:`，要不然无法拼接字符串。
- 当上传完头像后，页面不能自动刷新：这时候需要对页面重新进行一次请求，使用`:key="menuKey"`来实现，只要key对应的变量值一改变，页面就会自动刷新。

### 课程分类模块
#### 课程分类通过excel导入添加
使用easyexcel进行execl表格的传输
#### 前端上传excel
前后端联调。

#### 树形课程分类后端
按照前端的树形结构，后端也要返回类似的数据格式，需要创建对应的实体类，返回相应格式的数据。

#### 树形课程分类前端
整合`vue-admin`中tree下的组件，将固定格式数据变成后端返回的数据(通过aniox)。








