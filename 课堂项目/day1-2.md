# guli学院项目
## day1
### mybatis-plus
- mybatis-plus它和generator不一样，它是一个mybatis的加强版，能够实现很多企业级项目的功能。
- 它封装的sql都是一些基本的操作，如果需要进行复杂sql查询，需要自己额外写xml文件。
### mp的几个基本应用
#### mp打印日志
由于mp封装了一系列操作，有时候我们想知道它实际使用的sql，使用`mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl`配置一下。
#### 自动填充时间
create_time和update_time的自动填充。
#### 乐观锁
- 乐观锁的概念：对同一个资源的事务分别提交，那么后面提交的会覆盖前面提交的事务。此时需要用到乐观锁。
- 应用：12306抢票。
#### 分页操作
- mp实现分页：本质就是自动封装上limit。
#### 逻辑删除
- 实现逻辑删除，通过flags。
#### 性能分析
##### 几种环境
dev：开发环境
test：测试环境
prod：生产环境
可以通过配置文件来修改当前环境。
##### 性能分析
性能分析的作用：可以查看sql的执行时间，如果太长就不执行。用于开发和测试环境中。如果用于生产环境中会有性能损耗。

#### mp实现条件sql查询

## day2
### 建立多module工程
项目结构：
- parent：里面放所有dependence的版本控制，不写代码，删除src
  - module1：子模块，pom里面引入parent中的包，不写代码，删除src
    - module2：子子模块，写代码用的，pom里面啥也没有

### mp代码生成器
mp代码生成器可以用来生成mvc架构代码，详见文档。
[mp文档](https://mp.baomidou.com/guide)

### 构建swagger2进行接口测试
配置swagger2
1. 先引入pom
```xml
            <dependency>
                <groupId>io.springfox</groupId>
                <artifactId>springfox-swagger2</artifactId>
                <version>${swagger.version}</version>
            </dependency>

            <!--swagger ui-->
            <dependency>
                <groupId>io.springfox</groupId>
                <artifactId>springfox-swagger-ui</artifactId>
                <version>${swagger.version}</version>
            </dependency>
```
2. 写config类。
```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket webApiConfig(){
       
        return new Docket(DocumentationType.SWAGGER_2)
                .groupName("webApi")
                .apiInfo(webApiInfo())
                .select()
                .paths(Predicates.not(PathSelectors.regex("/admin/.*")))
                .paths(Predicates.not(PathSelectors.regex("/error.*")))
                .build();
    }
    private ApiInfo webApiInfo(){
        return new ApiInfoBuilder()
                .title("网站-课程中心API文档")
                .description("本文档描述了课程中心微服务接口定义")
                .version("1.0")
                .contact(new Contact("Helen", "http://atguigu.com", "55317332@qq.com"))
                .build();
    }
}
```
3. 由于项目采用的是多模块开发，swagger的config和需要测试api的接口不在一个模块中，需要在需要测试的模块中的application加上`@ComponentScan(basePackages = "com.shen") `注解从而能扫描到别的模块。
4. 完成了swagger配置后，可以在相应的controller中加上注解。以方便测试接口的时候查看信息。
```java
@Api(description = "讲师管理")  //这些api对功能没有影响，只是方便我们测试接口时查看。
@RestController
@RequestMapping("/eduservice/teacher")
public class EduTeacherController {

    @Autowired
    IEduTeacherService iEduTeacherService;

    @GetMapping("findAll")
    @ApiOperation(value = "所有讲师列表")
    public List<EduTeacher> findAll(){
        List<EduTeacher> list = iEduTeacherService.list(null);
        return list;
    }

    @DeleteMapping("{id}")
    @ApiOperation(value = "逻辑删除讲师")
    public boolean deleteTeacher(@ApiParam(name = "id", value = "讲师ID", required = true)@PathVariable Integer id){
        return iEduTeacherService.removeById(id);
    }
}
```
5. 使用`localhost:port/swagger-ui.html`进行访问测试接口。

### 统一结果返回
与前端交互需要对数据以一定的形式进行封装，需要用同样的结构，此时可以封装出一个Result类专门用于与前端的交互。
```java
@Data
public class Result {
    @ApiModelProperty(value = "是否成功")
    private Boolean success;

    @ApiModelProperty(value = "返回码")
    private Integer code;

    @ApiModelProperty(value = "返回消息")
    private String message;

    @ApiModelProperty(value = "返回数据")
    private Map<String, Object> data = new HashMap<String, Object>();

    private Result(){};

    public static Result ok(){
        Result result = new Result();
        result.setCode(ResultCode.SUCCESS);
        result.setSuccess(true);
        result.setMessage("success");
        return result;
    }

    public static Result error(){
        Result result = new Result();
        result.setCode(ResultCode.ERROR);
        result.setSuccess(false);
        result.setMessage("error");
        return result;
    }

    //完成一种链式编程效果。
    public Result success(Boolean success){
        this.setSuccess(success);
        return this;
    }

    public Result message(String message){
        this.setMessage(message);
        return this;
    }

    public Result code(Integer code){
        this.setCode(code);
        return this;
    }

    public Result data(String key, Object value){
        this.data.put(key, value);
        return this;
    }

    public Result data(Map<String,Object> map){
        this.setData(map);
        return this;
    }
}
```
### 讲师模块开发
#### 条件分页查询实现
- 使用mybatis-plus带的分页功能，将传入的当前页和每页数进行查询。
- 将传入的对象使用mybatis的wappper进行条件过滤，实现复杂的查询。
- 对象的传入可以通过表单形式，也可以通过json形式，这两种形式都需要通过Post来提交，其实表单形式和Get后面加一堆？是一样的，但是表单封装的好一些。
```java
 @GetMapping
    @ApiOperation(value = "条件分页查询讲师")
    public Result pageTeacherCondition(@ApiParam(name = "id", value = "当前页", required = true)@PathVariable Long current,
                                       @ApiParam(name = "id", value = "一页的数量", required = true)@PathVariable Long limit,
                                       TeacherQuery teacherQuery){
        Page<EduTeacher> teacherPage = new Page<>(current,limit);
        QueryWrapper<EduTeacher> queryWrapper = new QueryWrapper<>();
        if (!StringUtils.isEmpty(teacherQuery.getName())){
            queryWrapper.like("name",teacherQuery.getName());
        }
        if (!StringUtils.isEmpty(teacherQuery.getLevel())){
                queryWrapper.eq("level",teacherQuery.getLevel());
        }
        if (!StringUtils.isEmpty(teacherQuery.getBegin())){
                queryWrapper.ge("gmt_create",teacherQuery.getBegin());
        }
        if (!StringUtils.isEmpty(teacherQuery.getEnd())){
                queryWrapper.le("gmt_create",teacherQuery.getEnd());
        }

        eduTeacherService.page(teacherPage,queryWrapper);
        long total = teacherPage.getTotal();
        List<EduTeacher> records = teacherPage.getRecords();
        return Result.ok().data("total",total).data("rows",records);
    }
//或者使用json提交，需要将提交方式改成post。
    @PostMapping("pageTeacherCondition/{current}/{limit}")
    @ApiOperation(value = "条件分页查询讲师")
    public Result pageTeacherCondition(@ApiParam(name = "current", value = "当前页", required = true)@PathVariable Long current,
                                       @ApiParam(name = "limit", value = "一页的数量", required = true)@PathVariable Long limit,
                                       @RequestBody(required = false) TeacherQuery teacherQuery){  //使用json传参。
```

#### 讲师的添加、删除、更新

### 异常处理、自定义异常
使用springmvc进行异常处理。
[文档](https://docs.spring.io/spring-boot/docs/2.0.0.RC1/reference/htmlsingle/#boot-features-sql)
可以返回json数据，也可以返回统一页面。
- 全局异常处理
- 特殊异常处理：与全局异常处理进行兼容，优先执行特殊异常。

### 全局日志
- 日志级别分为debug、info、warn、error等，级别依次提高，低的级别覆盖了高级别的信息。
- springboot通过配置文件配置日志级别：`logging.level.root=DEBUG`
- 配置输出到文件：`logging.file.path=var/log`
- 但是使用application配置毕竟程度有限，如果要更详细的定义日志的处理和输出路径，我们需要通过xml自定义日志：在resource目录下建立`logback-spring.xml`，详见博客。


