# day11
## 服务端渲染技术NUXT
![](../截图/截屏2020-05-16%2010.14.05.png)
- 我们后台管理系统使用ajax这类的异步请求技术，不利于SEO（百度按照关键字的个数进行搜索排行）。
- 前台为了更好的SEO，使用NUXT框架（封装了node.js），客户端请求NUXT，NUXT将服务端请求的所有数据进行封装，一次性的返回给前端。

### 搭建NUXT环境
#### 安装NUXT
去github按照流程安装
#### 安装幻灯片插件
- 按照文档安装插件：[github](https://github.com/surmon-china/vue-awesome-swiper)
`npm install swiper vue-awesome-swiper --save`

- 在nuxt中按照文档进行引入：[NUXT文档](https://nuxtjs.org/guide/plugins)
#### 复制静态资源整合首页

## 后台接口编写
### 返回前几个幻灯片：创建banner模块

### 返回热点课程：去eduservice模块写
返回前几个id，或者返回观看数量前几的课程。

### 返回名师
返回前几个id

## 前端编写
### 安装axios
通过`npm install axios`安装好后，自己像vue-admin模块中那样封装一个request：
```js
import axios from 'axios'
// 创建axios实例
const service = axios.create({
  baseURL: 'http://172.16.65.10:80', // api的base_url
  timeout: 20000 // 请求超时时间
})
export default service
```
然后在各个.vue中用即可。

### 幻灯片显示
v-for

## 注意问题
- 由于之前写了linux登录执行的脚本，secureCRT由于每过一段时间就会断开连接，所以每次重新连的时候都会执行一遍那些脚本，非常费事，所以我们可以设置secureCRT每过一段时间发送一个心跳包保持连接。

## 整合redis
### 为什么使用缓存以及使用的场景
由于首页数据是经常访问的（每个用户登陆进来都要访问），并且它的数据又不常修改（名师、海报什么的基本就那几个），所以如果每次都要查数据库就太浪费了，这时候可以将这些数据添加到缓存中。
### redis的好处
通常驻留在内存中，并且可以适时候将数据进行持久化存储。
### springboot整合redis
1. 虚拟机安装redis，并启动。
2. springboot导包，并且添加配置文件（固定格式）
```java
@EnableCaching
@Configuration
public class RedisConfig extends CachingConfigurerSupport {
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        template.setConnectionFactory(factory);
        //key序列化方式
        template.setKeySerializer(redisSerializer);
        //value序列化
        template.setValueSerializer(jackson2JsonRedisSerializer);
        //value hashmap序列化
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        return template;
    }
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory) {
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        //解决查询缓存转换异常的问题
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        // 配置序列化（解决乱码的问题）,过期时间600秒
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofSeconds(600))
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer))
                .disableCachingNullValues();
        RedisCacheManager cacheManager = RedisCacheManager.builder(factory)
                .cacheDefaults(config)
                .build();
        return cacheManager;
    }
}
```
3. 在需要将返回值存入缓存的方法上添加注解`@Cacheable(key = "'selectAllBanner'",value = "banner")`并设置相应的key、value。这个注解加在查询方法上，意思是如果redis有值就拿出值返回，如果没有值就去执行你的方法，并且将这个方法返回的值在返回之前添加到redis中，添加规则是`value::key`，这个值作为redis查询中的key。
4. 访问这个方法进行测试，测试通过使用redis-cli中的`get banner::selectAllBanner`来看是否有值存在里面。

