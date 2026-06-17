---
layout: post
title: "EasyExcel 实现导出 Excel 文件不可编辑"
date: 2024-03-15 00:00:00
author: ingerchao
toc: true
category: blog
tag:
- Java
- EasyExcel


---

## 背景

在实际业务场景中，有时需要导出 Excel 文件，但希望用户只能查看而不能编辑文件内容。例如：

1. **财务报表** - 确保数据的完整性和准确性
2. **审计报告** - 防止数据被篡改
3. **合同文件** - 保证文件的法律效力
4. **数据快照** - 记录某个时间点的数据状态

EasyExcel 是阿里巴巴开源的 Excel 处理工具，性能优秀且使用简单。本文将介绍如何通过 EasyExcel 实现导出的 Excel 文件不可编辑。

## 实现方案

### 核心思路

通过 Excel 的工作表保护功能，设置工作表为只读状态，用户无法编辑单元格内容。

### 实现步骤

#### 步骤一：创建自定义拦截器

EasyExcel 提供了丰富的拦截器机制，可以通过自定义拦截器在特定事件发生时执行自定义逻辑。

```java
import com.alibaba.excel.write.handler.AbstractSheetWriteHandler;
import com.alibaba.excel.write.metadata.holder.WriteSheetHolder;
import com.alibaba.excel.write.metadata.holder.WriteWorkbookHolder;
import lombok.extern.slf4j.Slf4j;
import org.apache.poi.xssf.streaming.SXSSFSheet;

@Slf4j
public class UneditableSheetWriteHandler extends AbstractSheetWriteHandler {

    @Override
    public void afterSheetCreate(WriteWorkbookHolder writeWorkbookHolder,
                                 WriteSheetHolder writeSheetHolder) {
        // 获取当前工作表
        SXSSFSheet sheet = (SXSSFSheet) writeSheetHolder.getSheet();

        // 启用工作表锁定
        sheet.enableLocking();

        log.info("Sheet locking enabled for sheet: {}", sheet.getSheetName());
    }
}
```

**关键点说明：**

1. 继承 `AbstractSheetWriteHandler` 抽象类
2. 重写 `afterSheetCreate` 方法，在工作表创建后执行
3. 调用 `enableLocking()` 方法启用工作表锁定

#### 步骤二：注册拦截器

在创建 ExcelWriter 时，注册自定义的拦截器：

```java
import com.alibaba.excel.EasyExcel;
import com.alibaba.excel.ExcelWriter;
import java.io.OutputStream;

// 创建 ExcelWriter 并注册拦截器
ExcelWriter excelWriter = EasyExcel.write(out)
    .withTemplate(template)
    .registerWriteHandler(new UneditableSheetWriteHandler())
    .build();

// 写入数据
// ...

// 关闭 writer
excelWriter.close();
```

## 完整示例

### 示例一：基础导出

```java
import com.alibaba.excel.EasyExcel;
import com.alibaba.excel.ExcelWriter;
import com.alibaba.excel.write.metadata.WriteSheet;
import lombok.extern.slf4j.Slf4j;
import java.io.FileOutputStream;
import java.io.OutputStream;
import java.util.List;

@Slf4j
public class ExcelExportService {

    public void exportData(List<User> users, String filePath) {
        try (OutputStream out = new FileOutputStream(filePath)) {
            ExcelWriter excelWriter = EasyExcel.write(out)
                .head(User.class)
                .registerWriteHandler(new UneditableSheetWriteHandler())
                .build();

            WriteSheet writeSheet = EasyExcel.writerSheet("用户数据")
                .build();

            excelWriter.write(users, writeSheet);
            excelWriter.finish();

            log.info("Excel exported successfully to: {}", filePath);
        } catch (Exception e) {
            log.error("Failed to export Excel", e);
            throw new RuntimeException("Export failed", e);
        }
    }
}
```

### 示例二：基于模板导出

```java
import com.alibaba.excel.EasyExcel;
import com.alibaba.excel.ExcelWriter;
import com.alibaba.excel.write.metadata.WriteSheet;
import com.alibaba.excel.write.metadata.fill.FillConfig;
import java.io.InputStream;
import java.io.OutputStream;
import java.util.List;

public void exportWithTemplate(InputStream templateStream,
                               OutputStream out,
                               List<Order> orders) {
    ExcelWriter excelWriter = EasyExcel.write(out)
        .withTemplate(templateStream)
        .registerWriteHandler(new UneditableSheetWriteHandler())
        .build();

    WriteSheet writeSheet = EasyExcel.writerSheet()
        .build();

    FillConfig fillConfig = FillConfig.builder()
        .forceNewRow(Boolean.TRUE)
        .build();

    excelWriter.fill(orders, fillConfig, writeSheet);
    excelWriter.finish();
}
```

### 示例三：多 Sheet 导出

```java
public void exportMultipleSheets(List<User> users,
                                 List<Order> orders,
                                 OutputStream out) {
    ExcelWriter excelWriter = EasyExcel.write(out)
        .registerWriteHandler(new UneditableSheetWriteHandler())
        .build();

    // 第一个 Sheet
    WriteSheet userSheet = EasyExcel.writerSheet(0, "用户数据")
        .head(User.class)
        .build();
    excelWriter.write(users, userSheet);

    // 第二个 Sheet
    WriteSheet orderSheet = EasyExcel.writerSheet(1, "订单数据")
        .head(Order.class)
        .build();
    excelWriter.write(orders, orderSheet);

    excelWriter.finish();
}
```

## 高级配置

### 设置密码保护

如果需要更严格的保护，可以设置密码：

```java
@Slf4j
public class PasswordProtectedSheetWriteHandler extends AbstractSheetWriteHandler {

    private String password;

    public PasswordProtectedSheetWriteHandler(String password) {
        this.password = password;
    }

    @Override
    public void afterSheetCreate(WriteWorkbookHolder writeWorkbookHolder,
                                 WriteSheetHolder writeSheetHolder) {
        SXSSFSheet sheet = (SXSSFSheet) writeSheetHolder.getSheet();

        // 启用工作表锁定并设置密码
        sheet.enableLocking();
        sheet.protectSheet(password);

        log.info("Sheet protected with password for sheet: {}", sheet.getSheetName());
    }
}
```

使用方式：

```java
ExcelWriter excelWriter = EasyExcel.write(out)
    .head(User.class)
    .registerWriteHandler(new PasswordProtectedSheetWriteHandler("your_password"))
    .build();
```

### 选择性锁定

如果只希望锁定某些列或行，可以在拦截器中进行更精细的控制：

```java
@Slf4j
public class SelectiveLockSheetWriteHandler extends AbstractSheetWriteHandler {

    @Override
    public void afterSheetCreate(WriteWorkbookHolder writeWorkbookHolder,
                                 WriteSheetHolder writeSheetHolder) {
        SXSSFSheet sheet = (SXSSFSheet) writeSheetHolder.getSheet();

        // 启用工作表锁定
        sheet.enableLocking();

        // 设置列锁定（示例：锁定 A 列到 D 列）
        for (int i = 0; i <= 3; i++) {
            sheet.lockColumn(i);
        }

        // 设置行锁定（示例：锁定第 1 行）
        sheet.lockRow(0);

        log.info("Selective locking enabled for sheet: {}", sheet.getSheetName());
    }
}
```

## 常见问题

### 问题一：锁定后无法复制数据

**现象：** 用户无法从 Excel 中复制数据。

**解决方案：**
```java
@Override
public void afterSheetCreate(WriteWorkbookHolder writeWorkbookHolder,
                             WriteSheetHolder writeSheetHolder) {
    SXSSFSheet sheet = (SXSSFSheet) writeSheetHolder.getSheet();
    sheet.enableLocking();

    // 允许选择单元格
    sheet.setLockedAttribute(Sheet.LOCKED_ATTRIBUTE_SELECT_LOCKED_CELLS, false);

    // 允许选择未锁定的单元格
    sheet.setLockedAttribute(Sheet.LOCKED_ATTRIBUTE_SELECT_UNLOCKED_CELLS, false);
}
```

### 问题二：锁定后无法排序和筛选

**现象：** 用户无法对数据进行排序和筛选。

**解决方案：**
```java
@Override
public void afterSheetCreate(WriteWorkbookHolder writeWorkbookHolder,
                             WriteSheetHolder writeSheetHolder) {
    SXSSFSheet sheet = (SXSSFSheet) writeSheetHolder.getSheet();
    sheet.enableLocking();

    // 允许排序
    sheet.setLockedAttribute(Sheet.LOCKED_ATTRIBUTE_SORT, false);

    // 允许自动筛选
    sheet.setLockedAttribute(Sheet.LOCKED_ATTRIBUTE_AUTO_FILTER, false);
}
```

### 问题三：如何解除锁定

**方案一：** 不注册锁定拦截器

```java
// 正常导出，不锁定
ExcelWriter excelWriter = EasyExcel.write(out)
    .head(User.class)
    .build();
```

**方案二：** 动态控制是否锁定

```java
public class ConditionalLockSheetWriteHandler extends AbstractSheetWriteHandler {

    private boolean shouldLock;

    public ConditionalLockSheetWriteHandler(boolean shouldLock) {
        this.shouldLock = shouldLock;
    }

    @Override
    public void afterSheetCreate(WriteWorkbookHolder writeWorkbookHolder,
                                 WriteSheetHolder writeSheetHolder) {
        if (shouldLock) {
            SXSSFSheet sheet = (SXSSFSheet) writeSheetHolder.getSheet();
            sheet.enableLocking();
        }
    }
}

// 使用
ExcelWriter excelWriter = EasyExcel.write(out)
    .head(User.class)
    .registerWriteHandler(new ConditionalLockSheetWriteHandler(false)) // 不锁定
    .build();
```

## 最佳实践

### 1. 根据业务场景选择锁定策略

**严格锁定：** 财务报表、审计报告等
- 启用工作表锁定
- 设置密码保护
- 禁止所有编辑操作

**宽松锁定：** 数据快照、参考数据等
- 启用工作表锁定
- 允许选择单元格
- 允许复制数据

### 2. 提供用户提示

在导出的 Excel 中添加说明：

```java
// 在拦截器中添加注释
@Override
public void afterSheetCreate(WriteWorkbookHolder writeWorkbookHolder,
                             WriteSheetHolder writeSheetHolder) {
    SXSSFSheet sheet = (SXSSFSheet) writeSheetHolder.getSheet();
    sheet.enableLocking();

    // 在 A1 单元格添加说明
    Row row = sheet.createRow(0);
    Cell cell = row.createCell(0);
    cell.setCellValue("此文件为只读，请勿编辑");
}
```

### 3. 记录导出日志

```java
@Override
public void afterSheetCreate(WriteWorkbookHolder writeWorkbookHolder,
                             WriteSheetHolder writeSheetHolder) {
    SXSSFSheet sheet = (SXSSFSheet) writeSheetHolder.getSheet();
    sheet.enableLocking();

    // 记录导出信息
    log.info("Exported locked sheet: {}, workbook: {}",
        sheet.getSheetName(),
        writeWorkbookHolder.getWorkbook().getSheetName(0));
}
```

## 总结

通过 EasyExcel 的拦截器机制，可以轻松实现 Excel 文件的锁定功能。

**核心要点：**

1. **自定义拦截器** - 继承 `AbstractSheetWriteHandler`
2. **启用锁定** - 调用 `enableLocking()` 方法
3. **注册拦截器** - 在创建 ExcelWriter 时注册
4. **灵活配置** - 可以设置密码、选择性锁定等

**应用场景：**

- 财务报表导出
- 审计报告生成
- 数据快照保存
- 合同文件导出

掌握这个技巧，可以在需要保护数据完整性的场景中发挥重要作用。

---

## 参考资料

- [EasyExcel GitHub 仓库](https://github.com/alibaba/easyexcel)
- [EasyExcel 官方文档](https://easyexcel.opensource.alibaba.com/)
- [ISSUE: 如何设置单元格为只读属性](https://github.com/alibaba/easyexcel/issues/1859)
