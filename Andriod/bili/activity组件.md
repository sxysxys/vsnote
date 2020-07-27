## activity
### Manifest属性
- manifest用来声明activity，只有在manifest中声明的activity才能被使用。
- label：设置此页面的标题部分
- theme：设置主体属性，`android:theme="@style/Theme.AppCompat.DayNight.NoActionBar"/>`代表上面没有框，在application上设置代表所有的activity上面都没有框
- 设置横竖屏：`android:screenOrientation="landscape"`
- 设置启动activity：`<intent-filter>`

### 生命周期
1. `onCreate()`：每次进入activity执行。
2. `onStart()`：执行完created执行。
3. `onResume()`：每次重新进入activity（可能之前返回home了或者接了个电话回来）执行。
4. `onPause() onStop()`：暂停，一般切出去就会执行。
5. `ondestory()`：退出activity执行。

### 跳转和数据传递
- 显式跳转
  ```java
            Intent intent = new Intent(MainActivity.this, ToastActivity.class);
            startActivity(intent);
  ```
- 隐式跳转
  ```java
    Intent intent = new Intent();
    intent.setAction("android.shen.test");  //这个是在manifest中设置的属性
    startActivity(intent);
  ```
  ```xml
        <activity android:name=".TestActivity" >
            <intent-filter>
                <action android:name="android.shen.test" />

                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </activity>
  ```
- 数据传递
   - 使用Bundle放入Intent中进行数据传递。
- 数据交互：如果需要将下一个页面关闭的数据传到上一个页面，下面这几个缺一不可。
  - `startActivityForResult`：启动activity，并且带过一个id过去，当跳转回来的时候会带回来。
  - `setResult(Activity.RESULT_OK,intent);`：代表将一个intent存入result，如果关闭此activity，会将这个intent带到返回的那个activity。
  - `onActivityResult`：当从别的activity返回的时候并且携带了一个result(intent)的时候会执行这个方法。

### 启动模式
#### 概念
每个activity调用都在一个任务栈里面
- standard：跳转的时候，表示直接new一个activity在栈顶。
- singleTop：跳转的时候，表示入如果栈顶元素是本activity的实例，就不new这个activity了（不执行oncreate），直接执行onNewintent()方法。
- singleTask：只要这个activity在任务栈中存在，就不new了，直接复用。并且在任务栈中，该activity上面的activity会被清除。
- singleInstance（一般不会使用）：不管是哪个任务栈，只要在任何一个任务栈中存在这个activity，就复用。这种模式下每个activity占有一个新的任务栈。
#### 设置
- 设置启动模式：在manifest中设置每个activity的启动模式`launchMode`的时候进行设置。
- 设置任务栈`taskAffinity`：也可以对每个activity设置任务栈名称，那么这些activity的生命周期就是相对于这个任务栈的。