## 路由实现

window.hashchange -> 通过前端的iframe实现

## 分页实现

> 后端

在model中放入page值，后端通过jpa的Pager帮助类实现查询

> 前端

后端往前端传一个Page封装类封装信息，thmeleaf通过调用java中写的一个pageUtil类，实现page显示和切换跳转。

## JPA

https://blog.csdn.net/qq_30054997/article/details/79420141

- 和mybatis类似，继承它的`JpaRepository<>`来实现和mbp差不多的crud，泛型前面的是bean，后面的是id类型(Long)
- 如果需要使用一些关联的sql查询：可以直接使用如下的命名方式实现：

![img](https://img-blog.csdn.net/20180422145132373)

**注意：写find后面的字段的时候应该和类中定义的一样，而不是数据库表。**

**坑点：如果一个字段是外键、那么送的时候不能送id，必须送这个外键相关联的对象。**

- 可以使用注解中写sql的形式：

  ```java
  @Modifying
  @Transactional
  @Query("update #{#entityName} set status = ?1  where id in ?2 and status <> " + StatusConst.DELETE)
  public Integer updateStatus(Byte status, List<ID> id);
  ```

- 复杂sql查询

实现`JpaSpecificationExecutor`接口，实现一个`Specification`接口送入即可。

https://www.cnblogs.com/sandea/p/7803731.html

通过`JpaSpecificationExecutor`和PageHelper可以实现复杂的分页查询。



> 几种映射关系

多对多、一对一、一对多、多对一

https://blog.csdn.net/u010588262/article/details/76667283

可以通过实体类直接将此字段关联的对象找出来，很方便。



> ExampleMatcher

主要是用来解决匹配问题，如果不需要什么其他的复杂条件（例如between等），就用这个比较方便；如果需要使用一些复杂操作，就需要复杂查询了。

```java
        Customer customer = new Customer();
        customer.setName("zhang");
        customer.setAddress("河南省");
        customer.setRemark("BB");
		//创建匹配器，即如何使用查询条件
    ExampleMatcher matcher = ExampleMatcher.matching() //构建对象
            .withStringMatcher(StringMatcher.CONTAINING) //改变默认字符串匹配方式：模糊查询
            .withIgnoreCase(true) //改变默认大小写忽略方式：忽略大小写
            .withMatcher("address", GenericPropertyMatchers.startsWith()) //地址采用“开始匹配”的方式查询
            .withIgnorePaths("focus");  //忽略属性：是否关注。因为是基本类型，需要忽略掉
    
    //创建实例
    Example<Customer> ex = Example.of(customer, matcher); 
    
    //查询
    List<Customer> ls = dao.findAll(ex);
```
- 一些dao层的元素注解

```java
@Entity //不写@Table默认为user
@Table(name="t_user") //自定义表名
```

```java
@Id //主键
@GeneratedValue(strategy = GenerationType.AUTO)//采用数据库自增方式生成主键
//JPA提供的四种标准用法为TABLE,SEQUENCE,IDENTITY,AUTO.
//TABLE：使用一个特定的数据库表格来保存主键。
//SEQUENCE：根据底层数据库的序列来生成主键，条件是数据库支持序列。
//IDENTITY：主键由数据库自动生成（主要是自动增长型）
//AUTO：主键由程序控制。
 
@Transient //此字段不与数据库关联
@Version//此字段加上乐观锁
//字段为name，不允许为空，用户名唯一
@Column(name = "name", unique = true, nullable = false)
private String name;
 
@Temporal(TemporalType.DATE)//生成yyyy-MM-dd类型的日期
//出参时间格式化
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
//入参时，请求报文只需要传入yyyymmddhhmmss字符串进来，则自动转换为Date类型数据
@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm")
private Date createTime;
 
public String getName() {
    return name;
}
public void setName(String name) {
    this.name = name;
}
```
## Layui

.use在进入页面的时候会自动执行，加载相应的组件，在use中可以使用相应的模块，以完成相应的功能。

Layui的select事件必须在form表单中才能使用

## OpenLayers

> window.location

| 属性     | 描述                              |
| -------- | --------------------------------- |
| hash     | 从井号 (#) 开始的 URL（锚）       |
| host     | 主机名和当前 URL 的端口号         |
| hostname | 当前 URL 的主机名                 |
| href     | 完整的 URL                        |
| pathname | 当前 URL 的路径部分               |
| port     | 当前 URL 的端口号                 |
| protocol | 当前 URL 的协议                   |
| search   | 从问号 (?) 开始的 URL（查询部分） |

> openlayers的图层

- view：专门用来指示的容器，包括坐标系的建立、缩放等级等。
- layer：层的概念，用来在容器中放图像的，可以包括Image、瓦片等。























































