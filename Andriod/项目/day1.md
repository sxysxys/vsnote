## android
#### Application
- application在全局的manifest中定义，可以用来初始化一些全局的共享的东西，这个和在activity中的`getContext()`不一样，一般ui操作就是用`getContext`，而`getApplicationContext()`拿到的就是全局的Application。
- 不同ui的`getContext()`是不同的，全局`Application`的getContext拿到的是整个应用生命周期的上下文，但是单个ui的`getContext()`拿到的是当前的context，有些依赖于全局的组件，例如inflate、getSystemService、Intent跳转等，它们可以依赖全局的context，而不依赖于某个activity或fragment的context（当它们的父窗口(activity fragment)等如果没了，这个服务也能进行），而有些（例如dialog等）必须依赖于它们的父窗口，如果父窗口没了，它们也没有存在的必要了，这时候就不能依赖于全局的context。

#### as添加jar包
将jar包拖入lib文件夹下，然后右键jar包 -> add as library

## 百度地图测试
### 申请开发者sdk。
### 定位模块测试
1. 转向陀螺仪
`onSensorChanged()`回调，在这里处理。
### 鹰眼使用
#### 围栏报警
#### 实现轨迹纠偏(不用)
### 申请sdk
原来的测试环境下可以直接使用debug的密钥进行测试使用sdk，但是当使用发布版本apk的时候就会发现原来的密钥用不了，因为此时的应用密钥变了，我们需要拿到新密钥的文件，使用`keytool -list -v -keystore 文件名.jks`来生成相应的sha1加密，再去百度地图的控制台生成相应的访问ak。


## 整合oksocket
https://www.jianshu.com/p/8ee3ee766265
### 与服务器通信
1. 先initManager()，设置好此次连接的一些参数，包括读数据的一些格式细节setReaderProtocol
2. connect()连接；send()发送数据。将数据送入阻塞队列中，然后逐步取出按照指定的格式封装发送。具体是在write方法中。
3. 设置数据回调：registerReceiver，将相应的adapter注入。
### 心跳连接
定期发送心跳，服务端那边需要返回指定格式的数据，从而实时喂狗，一旦狗很久没喂，底层自动断开连接。
### 数据格式

## 遇到问题
- 当两个fragment的时候都有map的时候，切换map会遇到问题，使用`TextureMapView`代替`MapView`即可。




