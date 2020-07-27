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
### 


