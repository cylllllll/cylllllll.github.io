---
title: easyexcel 
categories: Java
date: 2019-04-02 23:00:00

---



最近项目要实现大数据量的导入导出，试用了一下easyexcel感觉还挺好用的。

*easyexcel是 Alibaba 的开源项目，项目地址 ：https://github.com/alibaba/easyexcel*

# 1. 依赖

首先需要添加依赖，推荐使用最新的，可以在Maven仓库中查询；

```
<!-- https://mvnrepository.com/artifact/com.alibaba/easyexcel -->
<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>easyexcel</artifactId>
	<version>1.1.1</version>
</dependency>
```

# 2. 优势

> Java解析、生成Excel比较有名的框架有Apache poi、jxl。但他们都存在一个严重的问题就是非常的耗内存，poi有一套SAX模式的API可以一定程度的解决一些内存溢出的问题，但POI还是有一些缺陷，比如07版Excel解压缩以及解压后存储都是在内存中完成的，内存消耗依然很大。easyexcel重写了poi对07版Excel的解析，能够原本一个3M的excel用POI sax依然需要100M左右内存降低到KB级别，并且再大的excel不会出现内存溢出，03版依赖POI的sax模式。在上层做了模型转换的封装，让使用者更加简单方便。

* 读任意大小的03、07版Excel不会OO
* 读Excel自动通过注解，把结果映射为java模型
* 写Excel时候自定义是否需要写表头
* 写Excel通过注解将表头自动写入Excel
* 写任意大07版Excel不会OOM
* 写小量数据的03版Excel（不要超过2000行）

#3. 自定义类

 * Temple

   作为模版可以将字段含义写入，在导入导出时可以讲导入导出内容与模版中实体类映射，也可作为模版导出；

   ```java
   public class Temple extends BaseRowModel {
   
       @ExcelProperty(value = "姓名" ,index = 0)
       private String name;
   
       @ExcelProperty(value = "年龄",index = 1)
       private String age;
   
       @ExcelProperty(value = "邮箱",index = 2)
       private String email;
   
       @ExcelProperty(value = "地址",index = 3)
       private String address;
   
       @ExcelProperty(value = "性别",index = 4)
       private String sax;
   
       @ExcelProperty(value = "高度",index = 5)
       private String heigh;
   
       @ExcelProperty(value = "备注",index = 6)
       private String last;
   }
   ```

 * ExcelListener

   监听类，可以根据需要，自定义处理获取到的数据；

   ```java
   public class ExcelListener extends AnalysisEventListener {
       //自定义用于暂时存储data。
       //可以通过实例获取该值
       private List<Object> datas = new ArrayList<>();
       /**
        * 通过 AnalysisContext 对象还可以获取当前 sheet，当前行等数据
        */
       @Override
       public void invoke(Object object, AnalysisContext context) {
           //数据存储到list，供批量处理，或后续自己业务逻辑处理。
           datas.add(object);
           //根据自己业务做处理
           doSomething(object);
       }
       private void doSomething(Object object) {
       }
       @Override
       public void doAfterAllAnalysed(AnalysisContext context) {
           //  解析结束销毁不用的资源
               datas.clear();
       }
       public List<Object> getDatas() {
           return datas;
       }
       public void setDatas(List<Object> datas) {
           this.datas = datas;
       }
   }
   ```

* ExcelUtil
   * 导出方法，将需要的导出的内容以及其表头，导出的一个Sheet页上。其中fileName，sheetName 分别是导出文件的文件名和 Sheet 名，list 为需要导出的数据；

     ```java
     public static void writeExcel(HttpServletResponse response, List<? extends BaseRowModel> list,String fileName, String sheetName, BaseRowModel object) {
             ExcelWriter writer = new ExcelWriter(getOutputStream(fileName, response), ExcelTypeEnum.XLSX);
             Sheet sheet = new Sheet(1, 0, object.getClass());
             sheet.setSheetName(sheetName);
             writer.write(list, sheet);
             writer.finish();
         }
     ```

     

   * 导入方法，读取导入的Excel文件的内容。其中 sheet 的序号从1开始，用于导入指定一Sheet页的内容；headLineNum 为表头行数，一般表头为一行；excel 是 MultipartFile 类型的文件对象；

     ```java
      public static List<Object> readExcel(MultipartFile excel, BaseRowModel rowModel, int sheetNo,int headLineNum) {
             ExcelListener excelListener = new ExcelListener();
             ExcelReader reader = getReader(excel, excelListener);
             if (reader == null) {
                 return null;
             }
             reader.read(new Sheet(sheetNo, headLineNum, rowModel.getClass()));
             return excelListener.getDatas();
         }
     ```


# 4. 遇到的问题

easyexcel 1.0.0 版本存在一些bug，建议使用1.1.X版本；1.1.X版本使用的poi和poi:ooxml版本为3.17；

在 3.17 版本之后，居中等方法已过时：

原来为 `cellStyle.setAlignment(XSSFCellStyle.ALIGN_CENTER)` 更换为`cellStylestyle.setAlignment(HorizontalAlignment.CENTER)`



