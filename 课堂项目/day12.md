# day12
## 单点登录sso
### 分布式中常见的几种方式
1. session复制：将一个模块中的session复制到其他模块中。
2. token：登录到一个模块中以后，按照用户输入的信息生成一个密钥返回当做用户的cookie，当用户访问不同的模块的时候，都会携带这个密钥，服务端进行解密从而验证用户的登陆。
3. cookie+redis：将用户登录信息传入redis，在redis中生成key，并将用户信息作为value进行存储，返回key给用户cookie，每次访问不同的模块都用cookie去redis中进行判断。

### 整合jwt用作token
写jwt工具类：
```java
package com.shen.commonutils;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jws;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.util.StringUtils;

import javax.servlet.http.HttpServletRequest;
import java.util.Date;

/**
 * @author helen
 * @since 2019/10/16
 */
public class JwtUtils {

    public static final long EXPIRE = 1000 * 60 * 60 * 24;
    public static final String APP_SECRET = "ukc8BDbRigUDaY6pZFfWus2jZWLPHO";

    /**
     * 创建token
     * @param id
     * @param nickname
     * @return
     */
    public static String getJwtToken(String id, String nickname){

        String JwtToken = Jwts.builder()
                .setHeaderParam("typ", "JWT")
                .setHeaderParam("alg", "HS256")
                .setSubject("guli-user")
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + EXPIRE))
                .claim("id", id)
                .claim("nickname", nickname)
                .signWith(SignatureAlgorithm.HS256, APP_SECRET)
                .compact();

        return JwtToken;
    }

    /**
     * 判断token是否存在与有效
     * @param jwtToken
     * @return
     */
    public static boolean checkToken(String jwtToken) {
        if(StringUtils.isEmpty(jwtToken)) return false;
        try {
            Jwts.parser().setSigningKey(APP_SECRET).parseClaimsJws(jwtToken);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
        return true;
    }

    /**
     * 判断token是否存在与有效
     * @param request
     * @return
     */
    public static boolean checkToken(HttpServletRequest request) {
        try {
            String jwtToken = request.getHeader("token");
            if(StringUtils.isEmpty(jwtToken)) return false;
            Jwts.parser().setSigningKey(APP_SECRET).parseClaimsJws(jwtToken);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
        return true;
    }

    /**
     * 根据token获取会员id
     * @param request
     * @return
     */
    public static String getMemberIdByJwtToken(HttpServletRequest request) {
        String jwtToken = request.getHeader("token");
        if(StringUtils.isEmpty(jwtToken)) return "";
        Jws<Claims> claimsJws = Jwts.parser().setSigningKey(APP_SECRET).parseClaimsJws(jwtToken);
        Claims claims = claimsJws.getBody();
        return (String)claims.get("id");
    }
}
```

### 整合阿里云短信服务
1. 开通阿里云短信服务：开通签名管理和模板管理。签名管理是开通短信服务；模板管理是发送短信的格式。
2. 通过整合文档进行短信的发送：
```java
    @Test
    public void sendMessage(){
        Boolean isSuccess = send(PhoneConstant.KEY_ID, PhoneConstant.KEY_SECRET);
        System.out.println(isSuccess);
    }

    public static Boolean send(String accessId,String accessSecret){
        DefaultProfile profile = DefaultProfile.getProfile("cn-hangzhou", accessId, accessSecret);
        IAcsClient client = new DefaultAcsClient(profile);

        //固定的部分
        CommonRequest request = new CommonRequest();
        request.setSysMethod(MethodType.POST);
        request.setSysDomain("dysmsapi.aliyuncs.com");
        request.setSysVersion("2017-05-25");
        request.setSysAction("SendSms");
        //自己设置的部分
        request.putQueryParameter("RegionId", "cn-hangzhou");  //固定
        request.putQueryParameter("PhoneNumbers", "13075595172");  //电话
        request.putQueryParameter("SignName", "大司在线教育网站");   //注册后的签名
        request.putQueryParameter("TemplateCode", "SMS_190720225");  //模板名称
        String code = RandomUtil.getFourBitRandom();
        Map<String,String> map = new HashMap<>();
        map.put("code",code);
        request.putQueryParameter("TemplateParam", JSONObject.toJSONString(map));  //传入验证码：转json发送
        try {
            CommonResponse response = client.getCommonResponse(request);
            System.out.println(response.getData());
            return response.getHttpResponse().isSuccess();
        } catch (ServerException e) {
            e.printStackTrace();
        } catch (ClientException e) {
            e.printStackTrace();
        }
        return false;
    }
```
#### 坑点
- 进行springboottest测试的时候，一直报错找不到test类，原来是需要将test的路径和源springboot的路径保持一致，要不然就会找不到。[解决方法](https://my.oschina.net/u/1017791/blog/3024441)
- 在@Test执行的方法优先会去取test目录中`application.properties`的value。

## 注册 登陆的实现
### 注册接口实现
#### 验证码接口
注册时将手机号传入，通过阿里云短信服务将验证码发到手机，并且将手机号和验证码存入redis中（5分钟失效）。
#### 注册接口
将用户的信息封装一个vo传入，这个vo中有手机号和刚刚手机收到的验证码，传入进行判断，实现注册。

### 注册前端实现
整合前端页面，用到js的定时器来使能发送验证码倒计时的功能。
```js
setInterval(() => {
          --this.second;
          this.codeTest = this.second
          if (this.second < 1) {
            clearInterval(result);
            this.sending = true;
            //this.disabled = false;
            this.second = 60;
            this.codeTest = "获取验证码"
          }
        }, 1000);
      }
每1s中执行一下这个定时器。
```
#### 坑点
- http相应状态码的问题，由于我们的后端异常都是通过json返回，这时候状态码依旧是200，也就是成功，但是我们需要进入axios异常处理阶段（根据状态码判断），所以我们需要在then中自行取出状态码进行判断。

### 登录接口整体实现
1. 将用户的手机号和密码传入，判断正确后根据用户id和名称返回一个token。
2. 前端拿到token放到cookie中，配置一个过滤器，<font color=red>将cookie中的东西放入header中传入后端。</font>
3. 后端拿到header中的token，通过token拿出id从而取出用户所有信息。
4. 将用户信息返回到cookie中，跳转到首页。
5. 首页中拿出数据进行判断显示。

### 登录接口前端
使用`js-cookie`技术操作cookie。

#### 坑点
- 后端返回的json对象在封装到cookie后再取出来会变成json字符串，需要使用`JSON.parse()`进行转换才能绑定到data。
- <font color=red>为什么有时候导入插件的时候需要进行`Vue.create()`，这是因为这些插件封装了一些真正的标签模板对象，而还有些插件例如`axios`、`js-cookie`这些里面都是一些js方法，所以只需要import包即可。</font>
- 对于token的使用：第一是为了减少查询数据库的频率，第二是为了分布式使用；所以不用session。
- 为什么当传递token的时候需要配置过滤器将cookie放入header中？<font color=red>那是因为cookie永远是伴随着服务端的，每个cookie对应了一个域名，它只会在请求特定的服务端域名的时候自己送过去；而我们项目部署、nginx反向代理都会导致域名变化，所以cookie根本就不会传过去；这时候需要乘着在请求之前，使用js获取cookie技术还能获取到的时候，赶紧把它加到这一次的请求头中，以便把token送入后端。</font>

