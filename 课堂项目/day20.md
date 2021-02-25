# day20
## nacos配置中心
我们在项目中所有的properties都放到了各个项目中，非常难以管理，如果修改了什么那么我们一个一个到服务器文件中修改就会变的非常麻烦，所以我们可以将所有的properties都放到nacos配置中心中。

### 配置步骤
1. 开启nacos服务。
2. 按照指定的规则创建相应的配置文件：例如`servicexx-dev.properties`，将本地的配置文件复制过去
![](../截图/截屏2020-05-29%2010.41.43.png)
3. 在代码创建`bootstrap.properties`，里面指定nacos的配置中心地址。
```properties
#配置中心地址
spring.cloud.nacos.config.server-addr=172.16.65.10:8848

spring.profiles.active=dev   //和配置文件第二部分一样

# 该配置影响统一配置中心中的dataId
spring.application.name=service-statistics
```
4. 导入相应的springcloud和nacos整合包。
```pom
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```
5. 启动服务，就可以读取了。

### 其他操作
#### 命名空间
由于我们的项目一般都有几个环境：
- 生产环境dev
- 上线部署prod
- 测试环境test

我们可以对这几个环境分别创建properties，当然可以使用后缀方式，但是这样多了就变得不好管理。所以我们可以使用创建不同的命名空间，然后再`bootstrap.properties`中加上一个配置，指定相应的命名空间：
```properties
spring.cloud.nacos.config.namespace=ca27f023-68b8-48d4-9942-cab2c341973b   //命名空间号
```

#### 一个项目多个properties
可以创建多个properties，需要在`bootstrap.properties`中加上配置：
```properties
spring.cloud.nacos.config.ext-config[0].data-id=redis.properties
# 开启动态刷新配置，否则配置文件修改，工程无法感知
spring.cloud.nacos.config.ext-config[0].refresh=true

spring.cloud.nacos.config.ext-config[1].data-id=jdbc.properties
spring.cloud.nacos.config.ext-config[1].refresh=true
```

## git管理

## 项目部署
### 传统方法
可以直接进入项目父目录，使用`mvn clean package`进行打包，打包出来一个jar文件，然后可以直接使用`java -jar xxx.jar`运行。
### 使用jekins进行自动化部署到docker中
#### 步骤
1. 在linux中安装好java、maven、git、docker，并配置好相应的环境变量。
2. 将项目提交到git远程仓库中，并且在项目中写好`dockerfile`
3. 执行jekins的脚本，启动jekins，访问8080端口进行配置。
4. 在下载插件之前切换源，把源切换成国内的。
5. 下载插件。
6. 配置好mvn、java、git本地路径。
7. 填写git远程仓库的地址，docker执行脚本等信息，点击开始部署。

docker.sh脚本如下：
```shell
#!/bin/bash
#maven打包
mvn clean package
echo 'package ok!'
echo 'build start!'
cd ./
service_name="eureka-server"
service_prot=8761
#查看镜像id
IID=$(docker images | grep "$service_name" | awk '{print $3}')
echo "IID $IID"
if [ -n "$IID" ]
then
    echo "exist $SERVER_NAME image,IID=$IID"
    #删除镜像
    docker rmi -f $service_name
    echo "delete $SERVER_NAME image"
    #构建
    docker build -t $service_name .
    echo "build $SERVER_NAME image"
else
    echo "no exist $SERVER_NAME image,build docker"
    #构建
    docker build -t $service_name .
    echo "build $SERVER_NAME image"
fi
#查看容器id
CID=$(docker ps | grep "$SERVER_NAME" | awk '{print $1}')
echo "CID $CID"
if [ -n "$CID" ]
then
    echo "exist $SERVER_NAME container,CID=$CID"
    #停止
    docker stop $service_name
    #删除容器
    docker rm $service_name
else
    echo "no exist $SERVER_NAME container"
fi
#启动
docker run -d --name $service_name --net=host -p $service_prot:$service_prot $service_name
#查看启动日志
#docker logs -f  $service_name
```
#### 整体开发部署流程
我们写完代码之后，将项目提交到远程仓库，自动化工具可以远程把我们的项目拉过来，使用mvn自动下载依赖，然后执行docker脚本(创建项目jar包，并且创建相应的镜像和容器并启动），实现在docker容器中自动化部署。