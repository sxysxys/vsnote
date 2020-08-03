## Fragment
### 概念
- 依赖于activity，一个activity中可以有多个fragment
- 可以和activity进行相互获取
- 有自己独立的生命周期
### 常见使用方法
1. 创建fragment和activity，对Fragment重写`onCreateView`和`onViewCreated`两个方法，第一个是返回一个fragment view，第二个是对view中的元素进行设置值。
2. 在activity的layout中创建一个framelayout，用来作为fragment的容器
3. 在activity中可以对容器中进行添加
   ```java
   // 拿到afragment
        aFragment = new AFragment();
        // 将fragment添加到容器
        getSupportFragmentManager().beginTransaction().add(R.id.fra_container,aFragment).commit();
   ```
### fragment和activity的数据传递
1. 通过newInstance创建fragment，并且对fragment添加上一个bundle。
2. 在fragment的`onViewCreated()`方法中拿到bundle，传递到view中的组件中。
### fragment和activity的绑定关系
- 某些时候，fragment和activity可能会进行脱绑，这时候fragment可能没有activity的引用，可以通过`getActivity()`进行判断是否能取到值。
- 可以在onAttach()方法中取到activity的引用，这样一直绑定上activity。
### fragment的调用栈
1. 在调用fragment的时候进行绑定上一个tag，`getSupportFragmentManager().beginTransaction().add(R.id.fra_container, this.aFragment,"a").commit();`
2. 取到tag对应的fragment，将此时的fragment进行隐藏，并且送入新的fragment：
   ```java
     Fragment a = getFragmentManager().findFragmentByTag("a");
     f (a != null) {
     // 这样设置，当返回到上一个fragment的时候不会重新加载，而是会直接拿到afragment
     getFragmentManager().beginTransaction()
     .hide(a).add(R.id.fra_container,bfragment)
     .addToBackStack(null).commit();
   ```
3. 如果使用replace会直接将fragment在调用栈中移除，虽然不会重新加载activity，但是会重新加载fragment的`onViewCreated`，重新设置view的值。
   ```java
   getFragmentManager().beginTransaction()
                        .replace(R.id.fra_container,bfragment)
                        .addToBackStack(null).commit();
   ```
### fragment和activity进行通信
1. 可以通过回调接口实现，具体看代码。
2. 可以通过`getActivity()`实现。





