## day1问题汇总

> layui动态添加option

一开始使用固定的格式`$("#robot1").append(new Option(item.robotName,item.robotCode));`添加，不知道为什么添加不上

后来网上查看了一下layui的动态添加option方法，需要引入form组件：

```javascript
var options = '';
var selectCode = $('#hid_robot').val();
$.each(res.data,function(index,item){
    if (item.robotCode == selectCode) {
        options += "<option selected value='"+item.robotCode+"'>"+item.robotCode+"</option>";
    } else {
        options += "<option value='"+item.robotCode+"'>"+item.robotCode+"</option>";
    }
});
$('#robotInfo').html(options);//往下拉菜单里添加元素
form.render();
```

> 前端的疯狂报错

不知道为啥，前端thmeleaf渲染一直报明明在页面中一个不存在的字段的错，搞了一个下午，发现是注释里有这个字段，这估计是idea中前端的一个bug。

> thmeleaf判断传入的对象如果为空就不渲染

使用`${missionInfo?.remark}`方式判断。

> 后端的jpa复杂查询

需要实现一个功能，就是前端提交一个表单，但是同时提交了两个时间，需要一个between操作。

如果单纯使用ExampleMatcher，发现并不能实现between操作，但是如果直接使用复杂查询，那么对每个字段都要进行判断，非常的麻烦。

只有使用了一个比较low的方法，判断前端有没有传入时间，使用if else来进行区分判断。

> textArea的thmeleaf

```javascript
<textarea placeholder="请输入内容" class="layui-textarea" name="remark">[[${missionInfo?.remark}]]</textarea>
```

## day2问题汇总

> jpa循环依赖问题

待解决。

> 表格的内容过长省略

需要在table上加上`table-layout:fixed`，保证table中的列可调节，从而可以在th属性上设置一下width等属性，`WORD-BREAK:break-all`表示如果超出就打省略号。

```html
<table class="layui-table timo-table" style="table-layout:fixed;WORD-BREAK:break-all;" lay-skin="nob">
    <thead>
    <tr>
        <th width="5%" class="timo-table-checkbox">
            <label class="timo-checkbox"><input type="checkbox">
                <i class="layui-icon layui-icon-ok"></i></label>
        </th>
        <th width="35%">任务编号/名称/类型</th>
        <th width="30%">任务内容</th>
        <th width="10%">任务状态</th>
        <th width="10%">AGV小车编号</th>
        <th width="10%">备注</th>
        <th width="15%">时间轴</th>
        <th width="15%">操作</th>
    </tr>
    </thead>
```

