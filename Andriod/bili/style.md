## Style
### layout
#### LinearLayout：线性布局
- View
- orientation：指定本块下的子组件排列方式，在线性布局中必须设置。
  - horizontal：水平
  - vertical：竖直
- gravity:将子组件放在本块中的位置
  - center：中心
- weight：此块相对于父块此时剩余部分的权重，如果有两个字块都为1，那么就是对半分；如果是子块是3和1，那么就是将此时父块剩余部分3：1分。

#### RelativeLayout：相对布局

### Button
- 绑定事件：几乎所有控件都能绑定事件，绑定事件一般有两种方法：一种是通过在activity中添加方法；一种是直接在onCreate中取得元素绑定：
```java
        t1 = findViewById(R.id.tv_1);

        t1.setOnClickListener(v -> {
            Toast.makeText(TextButtonActivity.this,"文本框1被点击了",Toast.LENGTH_SHORT).show();
        });
```
- 自定义背景：可以在drawable中画自己想要的背景，一般selector和shape进行配合使用。
  - selector：用来定义事件触发时候的shape
  - shape：自定义形状
    - solid：实心的样式
    - stroke：空心的样式
    - corners：圆角的样式
- 图片：在drawable中可以存放png图片用于text中：
```xml
        android:drawableLeft="@drawable/usericon"
        android:drawablePadding="5dp"
```
#### EditText
作用：是一种编辑文本的控件，每次输入值都可以触发事件。
```xml
    <EditText
        android:id="@+id/et_1"
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:textSize="16sp"
        android:textColor="#FFCC00"
        android:hint="手机号"
        android:inputType="number"
        />

    <EditText
        android:id="@+id/et_2"
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:textSize="16sp"
        android:textColor="#FFCC00"
        android:hint="密码"
        android:inputType="textPassword"
        android:layout_below="@id/et_1"
        android:layout_marginTop="15dp"
        />
```
#### RadioButton
- RadioButton：点选框
- 只能选一个的radio组：RadioGroup
  - `checked：true`：默认选中这个
#### CheckBox
- ` cb5.setOnCheckedChangeListener`：当改变状态的时候回触发这个函数
- 如果不使用原生的控件样子，可以修改`button`属性，将`button`属性连接上一个selector。
  - button属性定义的是前面的那个点，background定义的是整个背景，他们都能使用事件来触发修改。
  - 使用`<item android:state_checked="true" android:drawable="@drawable/success"/>`配置相应点击的icon。

#### Button衍生控件
- ToggleButton
- Swith

### View
#### TextView
- text：文字
- ellipsize：如果为end，那么如果放不下文字就会变成点
- drawableRight：文字+图片（右边）
- drawablePadding：设置此图与文字之间的距离
- textsize：设置文字的大小，一般使用sp作为单位
- 跑马灯
```xml
    <TextView
        android:id="@+id/tv_5"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textColor="#000000"
        android:text="flyflyflyflyflyflyflyflyflyflyflyflyflyflyflyflyflyflyflyflyflyflyflyf
        lyflyflyflyflyflyflyflyflyflyflyflyflyflyfly"
        android:textSize="24sp"
        android:singleLine="true"
        android:ellipsize="marquee"
        android:marqueeRepeatLimit="marquee_forever"
        android:focusable="true"
        android:focusableInTouchMode="true"
        android:clickable="true"/>
```
#### ImageView：使用第三方库加载网络图片
- 常用属性
  - `scaleType`
    - `fitXY`：撑满控件，宽高比可能发生改变
    - `centerCrop`：宽高比不变，让照片成比例放大
  - `src`：图片源加载的路径
- 加载网络图片：引用glide库
  1. 在`Manifest.xml`中配置：`<uses-permission android:name="android.permission.INTERNET"/>`
  2. 在activity中取到ImageView控件，通过以下代码获取图片：
   ```java
   iv2 = findViewById(R.id.iv_2);
        Glide.with(this).load("https://dasi-edu.oss-cn-beijing.aliyuncs.com/2020/05/14/c974cdbb82c244419c605f1dc95c3817123.png").into(iv2);
   ```
#### ListView、GridView
都是list控件，整体使用流程如下：
1. 创建相应的Activity和layout，并在layout中声明元素、在activity中取得元素。
2. 写一个adapter，重写方法：
   - getCount：需要定义几个元素
   - getView：每个元素上应该怎么显示，第一次送入的view为空，第二次的view是上一次返回的view。
3. 再写一个layout作为元素的显示view。
- geiview中如何取到指定的布局：通过`view = layoutInflater.inflate(R.layout.activity_grid_item, null);`，其中layoutInflater是`Gridactivity`对应的layout布局，在构造方法中通过
```java
    public MyGridAdapter(Context context) {
        this.context = context;
        layoutInflater = LayoutInflater.from(context);
    }
```
送入`Gridactivity.this`取得，然后需要通过一个layoutInflater取得另一个。

#### ScrollView
- 垂直滚动的布局方式，直接在xml中声明`<ScrollView>`
- 注意：这个里面只能有一个元素，需要将需要滚动的元素放到一个layout中作为子元素。

#### HorizontalScrollView
- 水平滚动的布局方式，直接在xml中声明。
- 注意：里面只能有一个元素，同上。

#### RecyclerView(important)
- 使用方式见代码，重要概念：viewholder、adapter等
- 分割线
- grid
- linear
- 瀑布流
- XRecyclerView：github开源库

#### WebView
- 作用：为了访问一些本地或者网络的url，来在app内实现上网。
- 几个重要的部分：
```java
        webView.getSettings().setJavaScriptEnabled(true);
        webView.setWebViewClient(new MyWebViewClient());
        webView.setWebChromeClient(new MyWebChromeClient());
        webView.loadUrl("https://m.baidu.com");
```

### 弹出组件
#### Toast
弹出的提示框，可以设置显示位置，显示的东西等。
#### dialog
一个类似表单的弹出框。
#### ProgressBar
- 进度条
  ```xml
      <ProgressBar
        android:id="@+id/pb_3"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        style="@style/Widget.AppCompat.ProgressBar.Horizontal"
        android:layout_marginTop="10dp"
        android:max="100"
        android:progress="10"
        android:secondaryProgress="30"/>
  ```
- 旋转框
  ```xml
      <ProgressBar
        android:id="@+id/pb_2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        style="@android:style/Widget.ProgressBar"
        android:layout_marginTop="10dp"/>
  ```
  其中style属性指明了此时到底是使用什么样式的进度，可以自己选择图片自定义style。

#### ProgressDialog
弹出一个带有进度条的dialog，可以自己设置弹出的样式，添加按钮等操作。

#### 自定义dialog
1. 先画出一个layout框。
2. 再创建继承Dialog类，实现oncreate，并设置各种事件等。
3. 在需要弹框的时候new这个Dialog对象，并show()调用oncreate()。
4. 也可以使用style样式，其实style就是我们在xml中设置的各种样式，给控件设置一个style就可以让控件的xml少写点东西，看起来简洁一些。

#### PopupWindow
给控件加入一个下拉框，步骤：
1. 先画出一个下拉框。
2. 添加按钮点击事件，点击按钮创建一个PopupWindow对象：
   ```java
   popupWindow = new PopupWindow(v,dia7.getWidth(), ViewGroup.LayoutParams.WRAP_CONTENT);
            popupWindow.setOutsideTouchable(true); // 点击外面的区域也可以关闭
            popupWindow.setFocusable(true);  // 获得焦点，使得可以通过点击按钮来打开关闭。
            popupWindow.showAsDropDown(dia7); // 设置在这个控件下面展开。
   ```
















