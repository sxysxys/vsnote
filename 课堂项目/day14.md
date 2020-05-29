# day14
## 整合前台课程列表和讲师列表
### 讲师列表
- 由于是前台，所以我们不能使用传统的vue加载方式。详见博客。
- 这里使用一个`nuxt.js`自带的异步加载数据的方式，也就是还没有将组建和数据的请求同步操作。[文档](https://nuxtjs.org/guide/async-data)
```js
    asyncData({params,error}){
    return course.getCourseInfo(params.id)   //这样就能获取到路由的param
          .then(response => {
            return {
               courseInfo: response.data.data.courseInfo,
               chapterAndVideo: response.data.data.chapterAndVideo
            }
          })
  }
```
- 不使用`element-ui`自带的分页插件，我们需要在后台通过mybatis-plus的分页功能多获取一些信息传回前台。
```java
        List<EduCourse> records = eduCoursePage.getRecords();
        long current = eduCoursePage.getCurrent();
        long total = eduCoursePage.getTotal();
        long size = eduCoursePage.getSize();
        long pages = eduCoursePage.getPages();
        boolean hasNext = eduCoursePage.hasNext();
        boolean hasPrevious = eduCoursePage.hasPrevious();
        map.put("items",records);
        map.put("current",current);
        map.put("total",total);
        map.put("pages",pages);
        map.put("size",size);
        map.put("hasNext",hasNext);
        map.put("hasPrevious",hasPrevious);
```
### 课程列表
和讲师列表类似。
### 注意问题
- aysnc如果需要同时发出多个异步请求操作，这时候的语法:
```js
    async asyncData({params,error}){
      let [response1, response2] = await Promise.all([
        course.getCourseList(1,8,{}),   //直接传个空值到后端。
        course.getAllSubject()
      ])
      return {
        data: response1.data.data,
        subjectNestedList: response2.data.data.list,
      }
    },
```
- 如上面`course.getCourseList(1,8,{}),`，如果在axios中写明了一个参数data，但是却不给值的话，会默认传null，后端就会发生空指针异常。所以我们就算没有数据，也可以使用`{}`的方式传一个对象，只不过对象中啥都是空值。

## 整合阿里云视频播放
### 参考文档
技术官网的阿里云文件夹中的文档。
### 思路
- 我们可以直接使用地址去请求视频，但是这样无法播放加密的视频。
- 所以我们使用一种添加视频id的方法，思路就是每次点击播放的时候首先根据视频id路由到新页面，在新页面中将视频id传到后台获取凭证，再使用`mounted()`：在数据渲染完成后再执行。

```html
<template>
  <div>
    <!-- 阿里云视频播放器样式 -->
    <link rel="stylesheet" href="https://g.alicdn.com/de/prismplayer/2.8.1/skins/default/aliplayer-min.css" >
    <!-- 阿里云视频播放器脚本 -->
    <script charset="utf-8" type="text/javascript" src="https://g.alicdn.com/de/prismplayer/2.8.1/aliplayer-min.js" />

    <!-- 定义播放器dom -->
    <div id="J_prismPlayer" class="prism-player" />
  </div>
</template>
<script>
import vod from '@/api/vod'
export default {
    layout:'video',
    asyncData({params,error}){
    return vod.getAuth(params.id)   //这样就能获取到路由的param
          .then(response => {
            return {
               auth: response.data.data.videoAuth,
               vid:params.id
            }
          })
    },
    mounted(){ //页面渲染之后
        var player = new Aliplayer({
            id: 'J_prismPlayer',
            width: '100%', 
            //使用凭证播放
            // source:'https://outin-ba334611953111ea97e500163e1c9256.oss-cn-shanghai.aliyuncs.com/sv/243c7171-1722c499517/243c7171-1722c499517.mp4?Expires=1590144442&OSSAccessKeyId=LTAIwkKSLcUfI2u4&Signature=ZB0UX2rop2LJshXdKiLRxncRR%2BQ%3D',
            encryptType:'1',//如果播放加密视频，则需设置encryptType=1，非加密视频无需设置此项
            vid : this.vid,
            playauth : this.auth,
            height: '500px',
            cover: 'http://guli.shop/photo/banner/1525939573202.jpg', // 封面
            qualitySort: 'asc', // 清晰度排序
            mediaType: 'video', // 返回音频还是视频
            autoplay: false, // 自动播放
            isLive: false, // 直播
            rePlay: false, // 循环播放
            preload: true,
            controlBarVisibility: 'hover', // 控制条的显示方式：鼠标悬停
            useH5Prism: true, // 播放器类型：html5
        },
        function(player){
            console.log('播放器创建好了。')
        });
    }
}
</script>
```

## 课程评论功能的实现
### 评论接口实现
先建表。
#### 查询评论接口
将课程id传入，将所有对应的评论进行返回。
#### 添加评论接口
- 添加评论接口，将评论的的课程id和老师id信息传入，并通过jwttoken判断`dasi_token`是否过期，如果过期返回异常，前端通过拦截器拦截异常，并且同时清空用户登陆信息，跳转到login界面。
```js
// response的拦截器，判断code来进行相应的操作。
service.interceptors.response.use(
    response => {
      //debugger
      if (response.data.code == 28004) {   //通过jwt判断是否登录。
          console.log("response.data.resultCode是28004")
          // 返回 错误代码-1 清除ticket信息并跳转到登录页面
          cookie.set('ucenter','',{domain: 'localhost'})
          cookie.set('dasi_token','',{domain: 'localhost'})
          //debugger
          window.location.href="/login"
          return
      }else {
          return response;
        }
      }
    },
    error => {
      return Promise.reject(error.response)   // 返回接口返回的错误信息
  });
```
- 如果jwttoken判断成功，就通过从jwttoken中取出用户id，通过用户id去调用ucenter模块（使用feign和nacos）获取返回的用户信息。
- 发送方：
```java
@Component
@FeignClient(name="service-ucenter",fallback = UcenterFeignClient.class)
public interface UcenterClient {
    @GetMapping("/educenter/member/getInfoById/{userId}")
    public UcenterMember getInfoById(@PathVariable("userId") String userId);
}
```
- 接收方：
```java
    @GetMapping("getInfoById/{userId}")
    public com.shen.commonutils.UcenterMember getInfoById(@PathVariable("userId") String userId){
        UcenterMember member = ucenterMemberService.getById(userId);
        com.shen.commonutils.UcenterMember ucenterMember = new com.shen.commonutils.UcenterMember();
        BeanUtils.copyProperties(member,ucenterMember);
        return ucenterMember;
    }
```
- 调用获取用户信息，加入传入的comment对象中，并存入数据库。
#### 坑点
- 模块调用之间，如果需要返回对象的时候，需要建立一个公共的对象，因为接收的服务方返回的对象发送方可能没有，所以需要在共同依赖的模块中建立一个共同的对象，以便数据的封装。

### 评论前端实现
常规操作。
#### 坑点
- a标签的`undisable`属性只能使得其不能被选中，但是不能使得其不触发点击事件，所以我们可以在点击事件中进行一下判断，以防请求出错。
```js
      gotoPage(page){
        if(page>this.data.pages){
          return false
        }
        return comment.getComments(page,8,this.courseId)
        .then(response => {
          this.data=response.data.data
      })
      },
```

