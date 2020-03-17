# 基于 POI 封装 ExcelUtil 精简的 Excel 导入导出1111



![poi](https://user-gold-cdn.xitu.io/2019/7/25/16c27fd32f261a74?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)poi



# 注

本文是使用 org.apache.poi 进行一次简单的封装，适用于大部分 excel 导入导出功能。过程中可能会用到反射，如若有对于性能有极致强迫症的同学，看看就好。

# 序

由于 poi 本身只是针对于 excel 等office软件的一个工具包，在一些常规的 excel 导入导出时，还需要再做一次精简的封装，简化代码耦合。

## 一、现状

本人经历过几家公司的代码封装，导入导出一般存在下面的情况。

### 1.1 导入

1. 传入文件地址，返回 Sheet 对象，在业务代码中进行循环遍历，做相对应的类型转换，业务处理（二零零几年的代码框架）
2. 传入文件地址，返回 List<String, Object> 的对象，外部直接做强转
3. 传入文件地址，返回 List<String, String> 的对象，外部将字符串对象转换为对应的类型

总结：如果只有上述的选择，本人是比较倾向于第二种，毕竟对外层是非常友好的

### 1.2 导出

1. 直接在逻辑代码中进行遍历封装sheet，传入到生成file的方法中（二零零几年的代码框架）
2. 先循环遍历 List 对象，转换为 List<Map<String, String>> 对象，带上 fieldName 传入到封装好excel生成的方法中，内部则使用 map.get() 方法操作
3. 直接将 List 对象带上 fieldName 传入到封装好excel生成的方法中，内部将 Model 对象转换为 JSONObject，然后使用 jsonObj.get() 方法操作
4. 先将 List 转换为 JSONArray ，带上 fieldName 传入到封装好excel生成的方法中，内部将 Model 对象转换为 JSONObject，然后使用 jsonObj.get() 方法操作。（使用这种做法，据分析应该是为了执行 jsonConfig.registerJsonValueProcessor(Date.class, new JsonDateValueProcessor("yyyy-MM-dd HH:mm:ss")); 这行代码，可能是为了解决日期类型格式问题）

总结：如果只有上述的选择，本人是比较倾向于第三种，第三种只遍历一次，并且外部未做处理。但是按第四种模式来看，那么第三种模式还是会存在日期格式问题，这个我们后续再分析如何处理。

## 二、导入

### 2.1 方法定义

```
/**
* excel导入
* @param keys		字段名称数组，如  ["id", "name", ... ]
* @param filePath	文件物理地址
* @return 
* @author yzChen
* @date 2016年12月18日 下午2:46:51
*/
public static List<Map<String, Object>> imp(String filePath, String[] keys)
    throws Exception {}
复制代码
```

### 2.2 循环处理模块

```
// 遍历该行所有列
for (short j = 0; j < cols; j++) {
    cell = row.getCell(j);
    if(null == cell) continue;	// 为空时，下一列
    
    // 根据poi返回的类型，做相应的get处理
    if(Cell.CELL_TYPE_STRING == cell.getCellType()) {
        value = cell.getStringCellValue();
    } else if(Cell.CELL_TYPE_NUMERIC == cell.getCellType()) {
        value = cell.getNumericCellValue();
        
        // 由于日期类型格式也被认为是数值型，此处判断是否是日期的格式，若时，则读取为日期类型
        if(cell.getCellStyle().getDataFormat() > 0)  {
            value = cell.getDateCellValue();
        }
    } else if(Cell.CELL_TYPE_BOOLEAN == cell.getCellType()) {
        value = cell.getBooleanCellValue();
    } else if(Cell.CELL_TYPE_BLANK == cell.getCellType()) {
        value = cell.getDateCellValue();
    } else {
        throw new Exception("At row: %s, col: %s, can not discriminate type!");
    }
    
    map.put(keys[j], value);
}
复制代码
```

### 2.3 使用

```
String filePath = "E:/order.xls";
String[] keys = new String[]{"id","brand"};

List<Map<String, Object>> impList;
try {
    impList = ExcelUtil.imp(filePath, keys);
    
    for (Map<String, Object> map : impList) {
        System.out.println(map.get("brand"));
    }
} catch (Exception e) {
    e.printStackTrace();
}
复制代码
```

### 2.4 分析

1. 入口只需要传入文件名称，以及外部需要读取的key即可
2. 内部处理，则针对 数值型、日期类型、字符串 类型已经做了对应处理，外部则直接进行强转对应的类型即可

## 三、导出

### 3.1 方法定义

```
/**
* excel导出
* @param fileNamePath	导出的文件名称
* @param sheetName	导出的sheet名称
* @param list		数据集合
* @param titles		第一行表头
* @param fieldNames	字段名称数组
* @return
* @throws Exception    
* @author yzChen
* @date 2017年5月6日 下午3:53:47
*/
public static <T> File export(String fileNamePath, String sheetName, 
    List<T> list, String[] titles, String[] fieldNames) throws Exception {}
复制代码
```

### 3.2 循环处理模块

```
// 遍历生成数据行，通过反射获取字段的get方法
for (int i = 0; i < list.size(); i++) {
    t = list.get(i);
    HSSFRow row = sheet.createRow(i+1);
    Class<? extends Object> clazz = t.getClass();
    for(int j = 0; j < fieldNames.length; j++){
        methodName = "get" + capitalize(fieldNames[j]);
        try {
            method = clazz.getDeclaredMethod(methodName);
        } catch (java.lang.NoSuchMethodException e) {	//	不存在该方法，查看父类是否存在。此处只支持一级父类，若想支持更多，建议使用while循环
            if(null != clazz.getSuperclass()) {
                method = clazz.getSuperclass().getDeclaredMethod(methodName);
            }
        }
        if(null == method) {
            throw new Exception(clazz.getName() + " don't have menthod --> " + methodName);
        }
        ret = null == method.invoke(t) ? null : method.invoke(t) + "";
        setCellGBKValue(row.createCell(j), ret + "");
    }
}
复制代码
```

### 3.3 使用

```
String[] titles = new String[]{"Id", "Brand"};
String[] fieldNames = new String[]{"id", "brand"};
List<Order> expList = new ArrayList<Order>();
Order order = new Order();
order.setId(1L);
order.setBrand("第三方手动阀");
expList.add(order);
order = new Order();
order.setId(2L);
order.setBrand("scsdsad");
expList.add(order);

String fileNamePath = "E:/order.xls";
try {
    ExcelUtil.export(fileNamePath, "订单", expList, titles, fieldNames);
} catch (Exception e) {
    e.printStackTrace();
}
复制代码
```

### 3.4 总结

1. 入口主要是需要传入 List 数据集合，以及 fieldNames 字段名称
2. 内部处理，是直接通过反射获得 get 方法的返回值，进行强转为字符串进行导出
3. 为了兼容继承父类的一些共有字段的设计，则加上了一层父类的方法读取

## 四、关于日期类型导出处理

### 1.1 日期字段导出指定格式内容

1. 建议在 Model 类中，新增一个扩展字段，并封装一个 get 方法，内容则只是对原字段进行转换，导出时，fieldName 则传递扩展字段即可。如 createTime，示例如下：

```
private Date createTime;
private String createTimeStr;	// 扩展字段

public Date getCreateTime() {
    return createTime;
}

public void setCreateTime(Date createTime) {
    this.createTime = createTime;
}

public String getCreateTimeStr() {
    createTimeStr = DateUtil.formatDatetime(this.createTime);
    return createTimeStr;
}
```