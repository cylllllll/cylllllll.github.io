---
title: KendoUI遇到的问题总结 
categories: Java
date: 2019-06-30 23:23:23


---

# KendoUI遇到的问题总结

1. #### kendoLov带出的值出现 null和undefined

如果数据库中数据为空，查询出来，界面上会显示null，点击新建按钮可能会出现undefined，这是因为template的问题，这样写会出错：

```javascript
template: function (dataItem) {
  if(dataItem.meaning!=null || dataItem.meaning=""){
    return dataItem.meaning;
  }
     return ""
},
```

应将“ !=null ”的时候 将“ || ” 换成“ && ”

2. #### 修改某个字段之后 点击保存按钮没有任何效果

原因： __status = ''

HAP框架判断界面是新增还是修改或者是删除，都是通过"__status "。如果没有值，controller是不会操作数据的。

新建： __status =  "add"

更新： __status = ‘"update"

删除： __status = "delete"

如果  "___status"实在是没有值，可以手动设置值。 viewModel.model.set("_status",  "add");

3. #### 检查Grid行中是否还有数据没有保存

使用dataSource.hasChanges() 方法。

```javascript
if(!dataSource1.hasChanges()){
    kendo.ui.showInfoDialog({
        message: '还有未保存的数据，请先保存!'
    });
}
```

4. #### 检查Grid选中的数据是否有脏数据

删除数据时，需要判断选中的数据是否是脏数据（即未保存的数据）

`$(“#Grid”).data(“kendoGrid”).selectedDataItems()[i].dirty`     

5. #### .Grid保存之后自动刷新数据，加载数据源

   `$(‘#grid1’).data(‘kendoGrid’).dataSource.page(1);`

6. #### Input框 回车之后可以查询

```javascript
$('#query-form input').keydown(function (e) {
     if (e.keyCode == 13) {
          e.target.blur();
          viewModel.queryResource(e);
      }
});      
```

7. #### 锁住grid的列

`locked:true`

```javascript
{
field:'test',
title: '<@spring.message "test.test"/>',
width:'100px',
locked:true 
},
```

8. #### Grid列开始日期、结束日期限制大小

```javascript
{
    field: "startDate",
    title: '<@spring.message "test.startDate"/>',
    width: 120,
    format: "{0:yyyy-MM-dd}",
    editor: function(container, options){
        //获得到期时间
        var end = options.model.endDate;
        var opts={
            format:"yyyy-MM-dd"
        }
        if(end){
            opts.max=end;
        }
        $('<input name="' + options.field + '"/>')
            .appendTo(container)
            .kendoDatePicker(opts);
    }
},
{
    field: "endDate",
    title: '<@spring.message "test.endDate"/>',
    width: 120,
    format: "{0:yyyy-MM-dd}",
    editor: function(container, options){
        //获得开始时间
        var start = options.model.startDate;
        var opts={
            format:"yyyy-MM-dd"
        }
        //设置min属性 限制最小的可选日期
        if(start){
            opts.min=start;
        }
        $('<input name="' + options.field + '"/>')
            .appendTo(container)
            .kendoDatePicker(opts);
    }
},
```

9. #### Lov带出的字段不可编辑

`container.removeClass(‘k-edit-cell’); `

10. #### Grid保存之后设置某列不可编辑

```javascript
editable:function(col,item){
    if(col=="test"){
        if(this.id != "" && this.id != null){ 
            return false;
        }
    }
    return true;
}
```

11. #### Tooltip 移动到列上会显示所有数据内容

* 首先设置样式

```css
<!--设置tooltip的样式-->
<style>
    div[role=tooltip].k-tooltip{
        padding: 2px;
        background: #5c9acf;
    }
    .k-tooltip-content{
        padding: 4px;
        text-align: left;
        background: #fff;
        color: #666;
    }
    .k-callout {
        border-bottom-color: #5c9acf;
    }
</style>
```

* 设置数据显示的样式

```javascript
attributes:{style: "white-space:nowrap;text-overflow:ellipsis;"},
```

* 添加Tooltip

```javascript
$("#Grid").kendoTooltip({
    show: function(e){
        if($.trim(this.content.text()) !=""){
            $('[role="tooltip"]').css("visibility", "visible");
        }
    },
    hide: function(){
        $('[role="tooltip"]').css("visibility", "hidden");
    },
    filter: "td:nth-child(n+3)",
    content: function(e){
        var element = e.target[0];
        if(element.offsetWidth < element.scrollWidth){
            var text = $(e.target).text();
            return '<div style="min-width:100px;max-width: 1000px;">' + text + '</div>';
        }else{
     $('[role="tooltip"]').css("visibility", "hidden");//解决鼠标一开始放在上面出现空模块
            return "";
        }
    }
}).data("kendoTooltip");

```

* 使用Tooltip会出现一个问题，如果行上出现复选框，复选框很难选中。

  解决方法：把复选框放到最左或者最右

  最左，修改n+3中的“ 3 ” ,以达到想要的效果

  最右，过滤掉最后一列：filter: “td:nth-child(n+3) :not-last-child”,