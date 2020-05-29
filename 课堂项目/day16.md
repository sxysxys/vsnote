# day16
## 修复观看和购买bug
当用户支付后，或者免费视频，就显示立即观看，否则显示立即购买。
### 坑点
- 由于我们需要往后端送入课程id和会员id两个值，去订单表中查询，所以我们选择送token，然后对jwttoken解密进行判断。但是我们发现后端根本取不到jwttoken。找了很久原因，发现是拦截器没有起作用，后来发现是异步请求`async`只在页面刷新的第一次会经过拦截器，后面对页面进行手动刷新都不会走拦截器了。没有办法，只能将`async`变成了普通的data()形式，在`created()`中提交。
- <font color=red>由于后端送入会员id和课程id，需要调用订单模块去查询状态，从而判断是不是已经购买，我在熔断器的`fallback`方法中抛出了异常，然后前台后台就一直报错，熔断器这块到时候一定要好好看一下。</font>

## 后台统计模块
### 生成数据模块
#### 更新当日统计表接口
将日期传入，使用模块调用`ucenter`，查询当天注册人数、登录人数等信息返回，将这些统计信息插入数据库中。
#### 坑点
- <font color=red>nginx一直报错：`1 upstream prematurely closed connection while reading response header from upstream, client: 10.0.2.77, server: gis.oneconcern.com, request:`，请求不到接口，使用swagger测试以后发现接口能用。前端这边也没啥问题，就一直以为是nginx的配置问题，但是其他的也是那样配置的，就这个接口访问不了。后来排查发现，原来是这个服务模块的端口居然启动了别的服务(`airserver`)，所以产生了这种情况，但是又搞不懂，既然他启用了别的服务，那么idea怎么又能使用这个端口启动这个模块的服务呢？</font>
#### 生成数据前端实现
使用一个element的日期组件，添加按钮将日期请求接口。

#### 生成数据系统定时任务
如果一直自己手动选取日期更新数据库当然可以，但是这样非常不方便，我们使用系统的定时任务实现。
##### springboot整合定时任务
1. 在启动类上开启定时任务`@EnableScheduling`
2. 写一个`@Component`，里面对使用的方法使用`@Scheduled`配置定时任务：
```java
    /**
     * 每天凌晨一点将前一天的数据送入，进行数据的更新操作。
     */
    @Scheduled(cron = "0 0 1 * * ? ")
    public void updateSta(){
        String beforeDay = LocalDateTime.now().withDayOfMonth(LocalDateTime.now().getDayOfMonth()-1).format(ISO_LOCAL_DATE);
        statisticsDailyService.insertCount(beforeDay);
    }
```
上面的cron代表cron表达式，它告诉系统什么时候，隔多久进行定时任务。
cron表达式生成地址：`https://cron.qqe2.com/`

### 显示数据模块
#### 前端整合echarts
- 看echarts官方文档例子整合。
- 要注意，如果当我们查询出来的数据横坐标很多显示不下的时候，echarts会自动的帮我们将横坐标数据进行一个合理的展示（空几个值显示一次）。

#### 后端接口实现
送入需要筛选条件的值，后端返回一个map，map中put横坐标和纵坐标，并且对应的value为List，这样前端的data才能转换为数组的格式。


