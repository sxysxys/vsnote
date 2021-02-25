## service服务
### 基本使用
1. 重写一个服务类。
2. 在activity中调用服务类，对服务进行开启关闭：
   ```java
       /**
     * 开启服务
     * @param view
     */
    public void startServiceClick(View view) {
        Intent intent = new Intent();
        intent.setClass(this,TestService.class);
        startService(intent);
    }
    /**
     * 关闭服务
     * @param view
     */
    public void stopServiceClick(View view) {
        Intent intent = new Intent();
        intent.setClass(this,TestService.class);
        stopService(intent);
    }
   ```
### 生命周期
- 创建的时候执行`onCreate()`，然后在activity中再开启服务`onStartCommand()`，如果之前create过就不会再create了。
- 关闭的service的时候执行`onDestory()`
### 性质
长期在后台运行，就算退出了activity，它依旧在后台运行。
## 绑定服务
```java
    /**
     * 绑定服务
     * @param view
     */
    public void bindServiceClick(View view) {
        Intent intent = new Intent();
        intent.setClass(this,TestService.class);

        // 绑定服务，如果没有服务就创建服务。
        bindService(intent,mServiceConnection,BIND_AUTO_CREATE);
    }
        /**
     * 解除绑定
     * @param view
     */
    public void unbindServiceClick(View view) {
        if (mServiceConnection != null) {
            unbindService(mServiceConnection);
        }
    }

    // 回调接口，抽象成了服务和activity的连接对象。
    private ServiceConnection mServiceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
            Log.d(TAG,"bind success...");
            TestService.InnerBind ib = (TestService.InnerBind) iBinder;
            ib.sayHallo();
        }

        @Override
        public void onServiceDisconnected(ComponentName componentName) {

        }
    };
```
### 与开启服务区别
- 开启服务长期在后台运行，不能通过绑定成功后的`IBinder`进行通信。并且关闭activity还存在，除非关闭服务。
- 绑定服务可能不能长期在后台运行，因为关闭activity的时候也会停止服务；但是能通过`IBinder`进行通信。

### 总结
![](../../截图/截屏2020-07-29%2014.28.30.png)

## 项目实战：模拟调用第三方服务
见代码
### 问题
不知道为啥绑定不上远程的服务。

## 项目实战：做一个音乐播放器
详见源码。
### 开发流程
1. 设计接口。
2. 设计ui界面。
3. 服务实现接口。
### 服务的设计
按照上面的图片，先创建服务，再绑定；每次退出activity服务依然在跑，重新回界面的时候再绑定上即可。

