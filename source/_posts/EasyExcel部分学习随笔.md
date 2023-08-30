---
title: EasyExcel部分学习随笔
date: 2023-08-29 11:20:37
tags: [Java,EasyExcel]
categories: [EasyExcel]
---
### 自己写了一个工具类
```java

import cn.hutool.core.util.StrUtil;
import com.alibaba.excel.EasyExcel;
import com.suntang.dcm.common.dto.ExportExcelDTO;
import com.suntang.dcm.common.response.Result;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;
import org.springframework.web.multipart.MultipartFile;

import javax.servlet.http.HttpServletResponse;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.OutputStream;
import java.net.URLEncoder;
import java.util.ArrayList;
import java.util.List;

@Slf4j
@Component
public class ExcelUtil {

    /**
     * 校验文件格式是否是Excel
     *
     * @return
     */
    public static boolean verifyFileFormat(MultipartFile file) {
        String originalFilename = file.getOriginalFilename();
        log.info("originalFilename:{}", originalFilename);
        if (StrUtil.isNotBlank(originalFilename)) {
            assert originalFilename != null;
            originalFilename = originalFilename.substring(originalFilename.lastIndexOf(".") + 1);
            return "xls".equals(originalFilename) || "xlsx".equals(originalFilename);
        }
        return true;
    }

    /**
     * 封装一下基于EasyExcel的导出方法
     *
     * @param response
     * @param excelDTO
     * @throws IOException
     */
    public static void export(HttpServletResponse response, ExportExcelDTO excelDTO) throws IOException {
        String fileName = URLEncoder.encode(excelDTO.getFileName(), "UTF-8");
        response.setContentType("application/vnd.ms-excel");
        response.setCharacterEncoding("utf-8");
        response.setHeader("Content-disposition", String.join("", "attachment;filename=", fileName, ".xlsx"));
        EasyExcel.write(response.getOutputStream(), excelDTO.getBaseClass())
                .sheet(excelDTO.getSheetName())
                .doWrite(excelDTO.getDataList());
    }

    /**
     * 导出路径上的Excel模板
     *
     * @param response
     * @param filePath
     * @param exportFileName
     * @throws IOException
     */
    public static void exportExcelTemplate(HttpServletResponse response, String filePath, String exportFileName) throws IOException {
        log.info("ExcelUtil.exportExcelTemplate.filePath：{}", filePath);
        // 构建文件路径
        File file = new File(filePath);
        if (!file.exists()) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            response.setContentType("text/plain;charset=UTF-8");
            response.getWriter().write("文件不存在。");
            return;
        }
        String fileName = URLEncoder.encode(exportFileName, "UTF-8");
        response.setContentType("application/vnd.ms-excel");
        response.setCharacterEncoding("utf-8");
        response.setHeader("Content-disposition", String.join("", "attachment;filename=", fileName, ".xlsx"));
        // 将文件写入响应输出流
        try (FileInputStream fis = new FileInputStream(file);
             OutputStream os = response.getOutputStream()) {
            byte[] buffer = new byte[4096];
            int bytesRead;
            while ((bytesRead = fis.read(buffer)) != -1) {
                os.write(buffer, 0, bytesRead);
            }
            os.flush();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }


    /**
     * 有表头没数据的模板
     *
     * @param response
     * @param headerList
     * @param exportFileName
     * @throws IOException
     */
    public static void exportExcelByHeaderList(HttpServletResponse response, List<String> headerList, String exportFileName) throws IOException {
        List<List<String>> headers = new ArrayList<>();
        headerList.forEach(res -> {
            List<String> column = new ArrayList<>();
            column.add(res);
            headers.add(column);
        });
        String fileName = URLEncoder.encode(exportFileName, "UTF-8");
        response.setContentType("application/vnd.ms-excel");
        response.setCharacterEncoding("utf-8");
        response.setHeader("Content-disposition", String.join("", "attachment;filename=", fileName, ".xlsx"));
        EasyExcel.write(response.getOutputStream())
                .head(headers)
                .sheet("sheet1")
                .doWrite(new ArrayList<>());
    }


}

```
ExportExcelDTO

```java

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.List;

/**
 * @author YoonaDa
 * @email lintiaoda@suntang.com
 * @description:
 * @date 2022-03-09 18:07
 */
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ExportExcelDTO {

    /**
     * 导出时显示的文件名
     */
    private String fileName;

    /**
     * 默认第一个sheet的名字
     */
    private String sheetName;

    /**
     * 数据转换映射的类
     */
    private Class<?> baseClass;

    /**
     * 列表数据
     */
    private List<?> dataList;
    
}

```
DynamicEasyExcelImportUtil （动态表头导入）

```java
import com.alibaba.excel.EasyExcelFactory;
import com.alibaba.excel.util.CollectionUtils;
import com.alibaba.nacos.shaded.com.google.common.collect.Lists;
import com.suntang.dcm.dg.listener.DynamicEasyExcelListener;

import java.io.ByteArrayInputStream;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;
import java.util.Objects;

/**
 * @author YoonaDa
 * @email lintiaoda@suntang.com
 * @description:
 * @date 2023-05-10 16:54
 */
public class DynamicEasyExcelImportUtil {

    /**
     * 动态获取全部列和数据体
     *
     * @param stream         excel文件流
     * @param parseRowNumber 指定读取行
     * @return
     */
    public static List<Map<String, String>> parseExcelToView(byte[] stream, Integer parseRowNumber) throws Exception {
        if (Objects.isNull(parseRowNumber)) {
            // 默认从第一行开始解析数据
            parseRowNumber = 1;
        }
        DynamicEasyExcelListener readListener = new DynamicEasyExcelListener();
        EasyExcelFactory.read(new ByteArrayInputStream(stream))
                .registerReadListener(readListener)
                .headRowNumber(parseRowNumber)
                .sheet(0)
                .doRead();
        List<Map<Integer, String>> headList = readListener.getHeadList();
        if (CollectionUtils.isEmpty(headList)) {
            throw new Exception("Excel未包含表头");
        }
        List<Map<Integer, String>> dataList = readListener.getDataList();
        if (CollectionUtils.isEmpty(dataList)) {
            throw new Exception("Excel未包含数据");
        }
        //获取头部,取最后一次解析的列头数据
        Map<Integer, String> excelHeadIdxNameMap = headList.get(headList.size() - 1);
        //封装数据体
        List<Map<String, String>> excelDataList = Lists.newArrayList();
        for (Map<Integer, String> dataRow : dataList) {
            Map<String, String> rowData = new LinkedHashMap<>();
            excelHeadIdxNameMap.forEach((key, value) -> rowData.put(value, dataRow.get(key)));
            excelDataList.add(rowData);
        }
        return excelDataList;
    }

}
```

```java 
List<Map<String, String>> dataList = DynamicEasyExcelImportUtil.parseExcelToView(file.getBytes(), 1);
```