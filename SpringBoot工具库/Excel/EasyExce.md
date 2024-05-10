# EasyExcel

EasyExcel是阿里巴巴开源的一个excel处理框架，**以使用简单、节省内存著称**。EasyExcel能大大减少占用内存的主要原因是在解析Excel时没有将文件数据一次性全部加载到内存中，而是从磁盘上一行行读取数据，逐个解析。[github地址](https://github.com/alibaba/easyexcel)

**EasyExcel特点:**

- Java领域解析、生成Excel比较有名的框架有Apache poi、jxl等。但他们都存在一个严重的问题就是非常的耗内存。如果你的系统并发量不大的话可能还行，但是一旦并发上来后一定会OOM或者JVM频繁的full gc。
- **EasyExcel采用一行一行的解析模式**，并将一行的解析结果以观察者的模式通知处理（AnalysisEventListener）
- EasyExcel是一个基于Java的简单、省内存的读写Excel的开源项目。在尽可能节约内存的情况下支持读写百M的Excel。

```XML
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>easyexcel</artifactId>
        <version>3.1.0</version>
    </dependency>
```



## 1.Excel 术语

1.整个excel称为一个 workbook

2.每个workbook中有很多sheet

3.每个sheet中有行，列，单元格



## 2.EasyExcel 简单写操作

### 2.1 创建实体类，作为Excel表头

```java
import com.alibaba.excel.annotation.ExcelProperty;
import lombok.Data;

@Data
public class User {
    //设置表头名称
    @ExcelProperty("用户编号")
    private Integer id;
    
    @ExcelProperty("用户名称")
    private String name;
}
```

### 2.2 简单写操作

```java
import com.alibaba.excel.EasyExcel;

import java.util.ArrayList;
import java.util.List;

public class ExcelTest {
    public static void main(String[] args) {
        //模拟数据
        List<User> users = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            User user = new User();
            user.setId(i);
            user.setName("xh"+i);
            users.add(user);
        }
        //设置文件的名称和路径,以.xlsx结尾
        String fileName = "D:\\excelTest.xlsx";
        //创建workbook
        EasyExcel.write(fileName,User.class)
                //创建sheet
                .sheet("写操作")
                //进行写操作
                .doWrite(users);

    }
}
```

## 3.EasyExcel 简单读操作

### 3.1 创建实体类,设置对应关系

```java
import com.alibaba.excel.annotation.ExcelProperty;
import lombok.Data;

@Data
public class User {
    //设置表头名称,设置对应的列编号
    @ExcelProperty(value = "用户编号",index = 0)
    private Integer id;

    @ExcelProperty(value = "用户名称",index = 1)
    private String name;
}
```

### 3.2 创建读取操作的监听器

实现`ReadListene`接口或者继承`AnalysisEventListener`类

```java
import com.alibaba.excel.context.AnalysisContext;
import com.alibaba.excel.metadata.data.ReadCellData;
import com.alibaba.excel.read.listener.ReadListener;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;

public class ExcelListener implements ReadListener<User> {

    public List<User> list = new ArrayList<>();

    @Override
    //一行一行去读取excel内容
    //从第二行开始读，默认第一行为表头
    public void invoke(User user, AnalysisContext analysisContext) {
        System.out.println("****"+user);
        list.add(user);
    }

    @Override
    //读取excel表头信息
    public void invokeHead(Map<Integer, ReadCellData<?>> headMap, AnalysisContext context) {
        System.out.println("表头信息："+headMap);
    }

    //读取完成后执行
    @Override
    public void doAfterAllAnalysed(AnalysisContext context) {

    }
}
```

### 3.3 简单读操作

```java
import com.alibaba.excel.EasyExcel;

public class ExcelTest {
    public static void main(String[] args) {
        //指定要读文件的名称和路径,以.xlsx结尾
        String fileName = "D:\\excelTest.xlsx";
        EasyExcel.read(fileName,User.class,new ExcelListener()).sheet().doRead();
    }
}
```



## 4. web的上传和下载

### 4.1 下载

 1.设置下载文件的类型

 2.设置响应头信息` Content-disposition `--无论什么格式都以下载的方式打开

`application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`

```java
	@GetMapping("download")
    public void download(HttpServletResponse response) throws IOException {
        response.setContentType("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
        response.setCharacterEncoding("utf-8");
       	try {
            // 这里URLEncoder.encode可以防止中文乱码 和easyexcel没有关系
            String fileName = URLEncoder.encode("课程分类","UTF-8").replaceAll("\\+", "%20");
            //设置响应头信息` Content-disposition `--无论什么格式都以下载的方式打开
            response.setHeader("Content-disposition", "attachment;filename="+ fileName + ".xlsx");
            //从数据库中查询要导出的数据
            final List<Subject> subjectList = subjectMapper.selectList(null);
            List<SubjectExcelVo> subjectExcelVoList = new ArrayList<>(subjectList.size());
            subjectList.forEach(subject -> {
                SubjectExcelVo subjectExcelVo = new SubjectExcelVo();
                BeanUtils.copyProperties(subject,subjectExcelVo);
                subjectExcelVoList.add(subjectExcelVo);
            });
            //使用输出类来进行写操作
            EasyExcel.write(response.getOutputStream(),SubjectExcelVo.class).sheet("课程分类").doWrite(subjectExcelVoList);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```



### 4.2 上传

```java
    @Override
    public void importDictData(MultipartFile file) {
        try {
            EasyExcel.read(file.getInputStream(), SubjectExcelVo.class, new ReadListener<SubjectExcelVo>() {
                @Override
                public void invoke(SubjectExcelVo subjectExcelVo, AnalysisContext context) {
                    //调用方法添加到数据库
                    Subject subject = new Subject();
                    BeanUtils.copyProperties(subject,subjectExcelVo);
                    subjectMapper.insert(subject);
                }

                @Override
                public void doAfterAllAnalysed(AnalysisContext context) {

                }
            }).sheet().doRead();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```