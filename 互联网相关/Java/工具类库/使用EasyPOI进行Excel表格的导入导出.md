#Java 
# 一、EasyPOI是什么

EasyPDI是对 Apche POI库的封装，可以通过一些简单的操作就可以进行Excel文件的导入导出，这这个库中还封装了一些独特的功能：
-   基于注解的导入导出,修改注解就可以修改Excel
-   支持常用的样式自定义
-   基于map可以灵活定义的表头字段
-   支持一堆多的导出,导入
-   支持模板的导出,一些常见的标签,自定义标签
-   支持HTML/Excel转换,如果模板还不能满足用户的变态需求,请用这个功能
-   支持word的导出,支持图片,Excel

一直以来，使用EasyPOI做了不少导入导出的需求，但是每次做完都是临时去看官方文档现学现用，正巧最近朋友遇到这么个需求，用到了EasyPOI来完成导入，我也正好整理整理EasyPOI的导入用法。

# 二、EasyPOI的使用
## 引入依赖
SpringBoot封装了对EasyPOI的starter，可以直接引用，也可以单独引入
```java
<!--easypoi-->  
<dependency>  
	 <groupId>cn.afterturn</groupId>  
	 <artifactId>easypoi-spring-boot-starter</artifactId>  
	 <version>4.4.0</version>  
</dependency>
```


单独引入：
```java
<dependency>
	<groupId>cn.afterturn</groupId>
	<artifactId>easypoi-base</artifactId>
	<version>4.1.0</version>
</dependency>
<dependency>
	<groupId>cn.afterturn</groupId>
	<artifactId>easypoi-web</artifactId>
	<version>4.1.0</version>
</dependency>
<dependency>
	<groupId>cn.afterturn</groupId>
	<artifactId>easypoi-annotation</artifactId>
	<version>4.1.0</version>
</dependency>
```


## EasyPOI的注解
EasyPOI是通过注解来表明需要导入导出的类、字段，一个实体类代表的是表格里的一行，实体类里面的字段代表着一行里面的一列。现在注解一共有5个，分别是：
-   @Excel 作用到类字段上面,是对Excel一列的一个描述
-   @ExcelCollection 表示一个集合,主要针对一对多的导出,比如一个老师对应多个科目,科目就可以用集合表示
-   @ExcelEntity 表示一个继续深入导出的实体,但他没有太多的实际意义,只是告诉系统这个对象里面同样有导出的字段
-   @ExcelIgnore 和名字一样表示这个字段被忽略跳过这个导导出
-   @ExcelTarget 这个是作用于最外层的对象,描述这个对象的id,以便支持一个对象可以针对不同导出做出不同处理

### 注解基本使用
下面是对EasyPOI注解的基本使用示例；
使用示例：

船艇导出实体类

```java
/**  
 * 船艇导出实体类  
 * @author jianhao  
 * @date 2021-11-10 13:58  
 */@Data  
@AllArgsConstructor  
@NoArgsConstructor  
@Accessors(chain = true)  
@ExcelTarget("exportModelBoat")  
public class ExportModelBoat {  
  
	 /**  
	 * 船名称  
	 */  
	 @NotBlank(message = "船名不能为空")  
	 @Excel(name = "船名称",orderNum = "0",needMerge = true,width = 15)  
	 String name;  

	 /**  
	 * 船只类型  
	 */  
	 @NotBlank(message = "类型不能为空")  
	 @Excel(name = "船类型",orderNum = "1",needMerge = true)  
	 String type;  

	 /**  
	 * 产品ID  
	 */ 
	 @Excel(name = "产品ID",orderNum = "2",needMerge = true)  
	 long boatid;  

	 /**  
	 * 硬件ID  
	 */ 
	 @Excel(name = "硬件ID",orderNum = "3",needMerge = true)  
	 String hardid;  

	 /**  
	 * 设备序列号  
	 */  
	 @Excel(name = "设备序列号",orderNum = "4",needMerge = true)  
	 String serial;  

	 /**  
	 * 设备序列编号  
	 */  
	 @Excel(name = "设备序列编号",orderNum = "5",needMerge = true,width = 20)  
	 String hashid;  

	 /**  
	 * 描述信息  
	 */  
	 @Length(max = 200,message = "描述信息不能超过200个字符")  
	 @Excel(name = "描述信息",orderNum = "6",needMerge = true,width = 30)  
	 String description;  

	 /**  
	 * 创建时间  
	 */  
	 @ExcelIgnore  
	 String createTime;  

	 /**  
	 * 最后修改时间  
	 */  
	 @ExcelIgnore  
	 String lastUpdateTime;  

	 /**  
	 * 该船所属设备集合  
	 */  
	 @ExcelCollection(name = "该船所属设备",orderNum = "9")  
		List<ExportModelDevice> deviceList = new ArrayList<>();  

	 /**  
	 * 该船所属信道集合  
	 */  
	 @ExcelCollection(name = "该船其他信息",orderNum = "10")  
	 List<ExportModelChannel> channelList = new ArrayList<>();  

	 /**  
	 * 该船出厂校验信息  
	 */  
	 @ExcelCollection(name = "该船出厂校验",orderNum = "11")  
	 List<ExportModelInspection> inspectionList = new ArrayList<>();

}
```

#### Excel注解

以上示例中类字段使用了Excel注解来标注，来表示在导出该对象时显示的字段名以及所属单元格的格式，该注解有许多的值可以设置，如下表：

| 属性           | 类型     | 默认值           | 功能                                                         |
| -------------- | -------- | ---------------- | ------------------------------------------------------------ |
| name           | String   | null             | 列名,支持name_id                                             |
| needMerge      | boolean  | fasle            | 是否需要纵向合并单元格(用于含有list中,单个的单元格,合并list创建的多个row) |
| orderNum       | String   | “0”              | 列的排序,支持name_id                                         |
| replace        | String[] | {}               | 值得替换 导出是{a_id,b_id} 导入反过来                        |
| savePath       | String   | “upload”         | 导入文件保存路径,如果是图片可以填写,默认是upload/className/ IconEntity这个类对应的就是upload/Icon/ |
| type           | int      | 1                | 导出类型 1 是文本 2 是图片,3 是函数,10 是数字 默认是文本     |
| exportFormat   | String   | “”               | 导出的时间格式,以这个是否为空来判断是否需要格式化日期        |
| importFormat   | String   | “”               | 导入的时间格式,以这个是否为空来判断是否需要格式化日期        |
| format         | String   | “”               | 时间格式,相当于同时设置了exportFormat 和 importFormat        |
| databaseFormat | String   | “yyyyMMddHHmmss” | 导出时间设置,如果字段是Date类型则不需要设置 数据库如果是string 类型,这个需要设置这个数据库格式,用以转换时间格式输出 |
| isColumnHidden | boolean  | false            | 导出隐藏列                                                   |
| width          | double   | 10               | 列宽                                                         |

#### ExcelCollcetions注解

该注解是表明该字段是一个集合，会让多条数据以对象的一个字段来显示（多条数据也就意味着多行数据）

| 属性     | 类型     | 默认值          | 功能                     |
| -------- | -------- | --------------- | ------------------------ |
| id       | String   | null            | 定义ID                   |
| name     | String   | null            | 定义集合列名,支持nanm_id |
| orderNum | int      | 0               | 排序,支持name_id         |
| type     | Class<?> | ArrayList.class | 导入时创建对象使用       |

#### ExcelEntity注解

标记是不是导出excel 标记为实体类,一遍是一个内部属性类,标记是否继续穿透,可以自定义内部id

| 属性 | 类型   | 默认值 | 功能   |
| ---- | ------ | ------ | ------ |
| id   | String | null   | 定义ID |





## 使用EasyPOI进行导入导出

沿用船艇实体类，在服务类代码中完成表格的导出；

在EasyPOI中分别针对导入和导出定义了工具类，分别是 `ExcelExportUtil` 和 `ExcelImportUtil` 2个类，针对这2个类可以进行导入导出操作。



### 数据导出成Excel表格

数据导出成Excel表格需要完成几个步骤：

- 准备好数据源
- 设置好导出Excel的文件格式以及sheet名
- 返回输出流



数据源从数据库中获取，将获取到的数据转化成被Excel注解标注的类，数据源就准备好了；然后通过 `ExcelExportUtils` 提供的方法来构建 `Workbook` 对象，最后将文件流写入到response中即可。

使用示例：

```java
// 从数据库获取数据
List<ModelBoat> boatList = boatService.findAll();

//从数据库获取数据并且将获取的数据进行转化
//这里使用的是Springboot Reactive 异步框架，获取数据和转化的操作自己定义即可。
List<ExportModelBoat> exportModelBoatList = new ArrayList<>();
dataStorage.getBoatRepository()
    .findAll(new Sort(Sort.Direction.ASC,"createTime"))
    .collectList()
    .flatMap(list ->{
        //根据船id去查对应的信道对象、出厂校验对象、设备对象
        for (ModelBoat boat : list) {
            ExportModelBoat exportModelBoat = new ExportModelBoat(boat);
            exportModelBoatList.add(exportModelBoat);
        }
        return Mono.empty();
    }).block();

//构建ExportParms对象，该对象可以设置该Excel文件的标题行、sheet名称以及导出的文件类型（HSSF、XSSF)
ExportParams params = new ExportParams("标题行","sheet名",ExcelType.XSSF);

//然后将该params对象传入ExcelExportUtil类的exportExcel方法中，获取到Workbook对象
//exportExcel方法需要传入3个参数，一个是params对象，一个是被注解标注的导出类，另外一个就是数据源
//需要注意的是，第二个参数和第三个参数的类型需要对应上
Workbook sheets = ExcelExportUtil.exportExcel(params, ExportModelBoat.class, exportModelBoatList);

//设置返回流的格式，避免乱码
response.setContentType("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
response.setCharacterEncoding("utf-8");
// 这里URLEncoder.encode可以防止中文乱码 当然和easyexcel没有关系
String fileName = URLEncoder.encode("ExayPOI文件导出", "UTF-8").replaceAll("\\+", "%20");
response.setHeader("Content-disposition", "attachment;filename*=utf-8''" + fileName + ".xlsx");

//最后返回写入返回流中
sheets.write(response.getOutputStream());
```

根据以上的步骤，数据导出成Excel文件也就完成了。

导出效果：

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2021-11-15_14-01.png)

> 需要注意的是，如果导出对象中存在一对多的情况，也就是有字段是集合类型的，那么该类其他字段的Excel注解需要加上一个值，`needMerge` = true，如果不设置的话，同一个对象的字段就不会纵向合并。



### Excel表格数据导入

表格的数据导入与数据导出的步骤是一样的，通过`ExcelImportUtil`工具类来实现数据导入。但是要注意的是文件表格的列名格式要和代码中被注解标注类的字段要保持一致，字段不一致的就不会解析成对象。

导入文件需要进行以下这几个步骤：

- 新建 ImportParams 对象，用于设置导入文件时的一些信息
- 获取文件，读取该文件
- 最终获得结果List

下面是示例：

```java
//新建 Importarams 对象
ImportParams importParams = new ImportParams();
//设置表头所占行数、标题行所占行数
importParams.setHeadRows(2);
importParams.setTitleRows(0);

//导入文件
//第一个参数是文件（可以是File对象，也可以是文件流），第二个对象是要被解析成的pojo类，第三个是导入参数对象
//返回值是解析成功的对象集合
List<ExportModelBoat> list = ExcelImportUtil.importExcel(file.getInputStream(), ExportModelBoat.class, importParams);

//数据存储操作
boatService.saveAll(List);
```



#### 数据导入时的数据校验

在数据导入时，会进行字段的校验，如果校验不通过则解析失败并且提示返回错误的行数以及消息。EasyPOI已经提供了这个功能选项， EasyPOI 支持 `Hibernate Validator` ，所以直接使用就可以了，因为要将错误信息以及错误行号返回，需要用到 EasyPOI 的高级用法，实现 `IExcelDataModel`与 `IExcelModel`接口，`IExcelDataModel`负责设置行号，`IExcelModel` 负责设置错误信息。

实现数据导入的数据校验要进行以下步骤：

- 新建的ImportParms对象需要开启校验功能
- 实体类加入 `Hibernate Validator` 注解，开启校验功能
- 导入的解析类对象需要实现 `IExcelDataModel` 和 `IExcelModel` 接口
- 数据校验
- 返回校验结果以及错误提示

在上面的导入数据示例中，新建了一个`ImportParams`对象，如果需要开启校验功能，需要多加一行代码：

```java
ImportParams importParams = new ImportParams();
//开启校验功能
importParams.setNeedVerify(true);
```

开启校验功能后，在实体类中需要实现`IExcelDataModel` 和 `IExcelModel` 接口，重写接口方法

> 如果使用到了 @Pattern 注解，则字段类型必须是 **String** 类型，否则会抛出异常

```java
public class ExportModelBoat implements IExcelDataModel, IExcelModel {
    
    /**
     * 船名称
     */
    @NotBlank(message = "船名不能为空")
    @Excel(name = "船名称",orderNum = "0",needMerge = true,width = 15)
    String name;
    
    /**
     * 描述信息
     */
    @Length(max = 200,message = "描述信息不能超过200个字符")
    @Excel(name = "描述信息",orderNum = "6",needMerge = true,width = 30)
    String description;
    
    /**
     * 内部ip地址
     */
    @Pattern(regexp = "^((\\d|[1-9]\\d|1\\d\\d|2[0-4]\\d|25[0-5])\\.){3}(\\d|[1-9]\\d|1\\d\\d|2[0-4]\\d|25[0-5])$",
            message = "ip地址格式错误")
    @Excel(name = "IP地址",orderNum = "1",width = 15)
    String ip;
    
    //省略其他字段
    
    /**
     * 行号
     */
    private int rowNum;

    /**
     * 错误消息
     */
    private String errorMsg;
    
    @Override
    Integer getRowNum(
        //这里设置加一是因为行数是从0开始算
    	return rowNum + 1;
    );

    @Override
    void setRowNum(Integer var1){
        this.rowNum = var1;
    };
    
    @Override
    String getErrorMsg(
    	return errorMsg;
    );
    
    @Override
    void setErrorMsg(String var1){
        this.errorMsg = var1;
    };
}
```

如果开启数据校验，那么数据导入的方式也将改变，原本是调用 `ExcelImportUtil` 类的 `importExcel` 方法实现导入，现在需要切换到另外一个方法来实现导入，通过 `importExcelMore()`方法导入，该方法所需参数与未开启校验导入是一致的，这两者的区别是返回值。

该方法的返回值是 `ExcelImportResult<T>`类型的，代码如下：

```java
public class ExcelImportResult<T> {
    //校验通过的结果集合
    private List<T> list;
    //校验失败的结果集合
    private List<T> failList;
    //是否校验失败
    private boolean verifyFail;
    private Workbook workbook;
    private Workbook failWorkbook;
    private Map<String, Object> map;
}
```

进行校验之后，会将校验成功的结果放入 `list` 字段中，校验失败的放入 `failist` 字段中，并且设置 `verifyFail` 值；

校验过程中，如果校验失败，那么就会将失败所在的行以及错误信息写入到该对象的 `rowNum` 字段和 `errorMsg`字段中，这2个字段是实现了接口而拥有的。

示例代码：

```java
//创建数据引入设置对象，设置标题行、表头位置，以及开启校验功能
ImportParams importParams = new ImportParams();
importParams.setHeadRows(2);
importParams.setTitleRows(0);
importParams.setNeedVerify(true);
ExcelImportResult<ExportModelBoat> resultList = ExcelImportUtil.importExcelMore(file.getInputStream(), ExportModelBoat.class, importParams);
//获取校验信息
System.out.println("是否校验失败: " + resultList.isVerfiyFail());
System.out.println("校验失败的集合:" + JSONObject.toJSONString(resultList.getFailList()));
System.out.println("校验通过的集合:" + JSONObject.toJSONString(resultList.getList()));
//输出
for (ExportModelBoat entity : result.getFailList()) {
    String msg = "第" + entity.getRowNum() + "行的错误是：" + entity.getErrorMsg();
    System.out.println(msg);
}
```

以上就是校验的代码情况，但是这部分只能进行基本部分的校验，但是对ExcelCollection标注集合里的对象却校验不了。

#### 对Excel集合里的对象进行校验

EasyPOI对被ExcelColleciton注解所标注的集合对象校验不了，那么我们就可以自己手动开始校验，使用`Hibernate Validator` 框架的校验功能进行手动校验。这里需要封装工具类，示例代码如下：

```java
/**
 * 校验工具类
 * @author jianhao.chen

 */
public class ValidatorUtils {

    private static Validator validatorFast = Validation.byProvider(HibernateValidator.class).configure().failFast(true).buildValidatorFactory().getValidator();
    private static Validator validatorAll = Validation.byProvider(HibernateValidator.class).configure().failFast(false).buildValidatorFactory().getValidator();

    /**
     * 校验遇到第一个不合法的字段直接返回不合法字段，后续字段不再校验
     * @Time 2021年11月11日 上午11:36:13
     * @param <T>
     * @param domain
     * @return
     * @throws Exception
     */
    public static <T> Set<ConstraintViolation<T>> validateFast(T domain) throws Exception {
        Set<ConstraintViolation<T>> validateResult = validatorFast.validate(domain);
        if(validateResult.size()>0) {
            System.out.println(validateResult.iterator().next().getPropertyPath() +"："+ validateResult.iterator().next().getMessage());
        }
        return validateResult;
    }

    /**
     * 校验所有字段并返回不合法字段
     * @Time 2021年1月11日 上午11:37:55
     * @param <T>
     * @param domain
     * @return
     * @throws Exception
     */
    public static <T> Set<ConstraintViolation<T>> validateAll(T domain) throws Exception {
        Set<ConstraintViolation<T>> validateResult = validatorAll.validate(domain);
        if(validateResult.size()>0) {
            Iterator<ConstraintViolation<T>> it = validateResult.iterator();
            while(it.hasNext()) {
                ConstraintViolation<T> cv = it.next();
                System.out.println(cv.getPropertyPath()+"："+cv.getMessage());
            }
        }
        return validateResult;
    }
}
```

以上工具类方法会返回一个Set集合，该集合存放这 `ConstraintViolation`对象，该对象可以根据方法 `getPropertyPath()`来获取校验失败的字段，根据 `getMessage()`方法获取错误信息。

示例代码：

```java
//创建数据引入设置对象，设置标题行、表头位置，以及开启校验功能
ImportParams importParams = new ImportParams();
importParams.setHeadRows(2);
importParams.setTitleRows(0);
importParams.setNeedVerify(true);
ExcelImportResult<ExportModelBoat> resultList = ExcelImportUtil.importExcelMore(file.getInputStream(), ExportModelBoat.class, importParams);
//获取校验信息
System.out.println("是否校验失败: " + resultList.isVerfiyFail());
System.out.println("校验失败的集合:" + JSONObject.toJSONString(resultList.getFailList()));
System.out.println("校验通过的集合:" + JSONObject.toJSONString(resultList.getList()));

//对所有数据进行集合校验
ArrayList<ExportModelBoat> exportModelBoats = new ArrayList<>();
exportModelBoats.addAll(resultList.getList());
exportModelBoats.addAll(resultList.getFailList());
//假设对象里面有一个集合名较deviceList，
for (ExportModelDevice exportModelDevice : boat.getDeviceList()) {
    //获取校验结果
	Set<ConstraintViolation<T>> validateResult = ValidatorUtils.validateAll(exportModelDevice);
    //如果校验失败，那么该返回值对象的size会大于0，反之则校验成功
    if(validateResult.size()>0) {
        //获取迭代器
        Iterator<ConstraintViolation<T>> it = validateResult.iterator();
        while(it.hasNext()) {
            ConstraintViolation<T> cv = it.next();
            //根据getPropertyPath()方法、getMessage()方法来获取校验失败的字段以及错误提示
            System.out.println(cv.getPropertyPath()+"："+cv.getMessage());
        }
    }
}
```

根据以上代码之后就可以校验集合对象的数据，可以满足导入导出的Excel表格需求。
