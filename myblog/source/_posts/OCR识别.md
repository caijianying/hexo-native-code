---
title: PDF发票OCR识别
categories:
  - OCR识别
tags: OCR识别
abbrlink: 4a4871b6
date: 2021-03-23 22:52:21
---
#### PDF发票OCR识别

>基本解析过程是解析发票中的二维码，解析出发票的5要素：发票号码、发票代码、发票金额、开票日期、校验码

##### 引入maven

```xml
		<!--pdf发票解析-->
        <dependency>
            <groupId>org.apache.pdfbox</groupId>
            <artifactId>pdfbox</artifactId>
            <version>2.0.20</version>
        </dependency>
        <dependency>
            <groupId>com.google.zxing</groupId>
            <artifactId>core</artifactId>
            <version>3.1.0</version>
        </dependency>
        <dependency>
            <groupId>com.google.zxing</groupId>
            <artifactId>javase</artifactId>
            <version>3.1.0</version>
        </dependency>

		<!--现在项目一般都放这个包 视情况而定 -->
        <!-- https://mvnrepository.com/artifact/org.apache.commons/commons-lang3 -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.4</version>
        </dependency>

```

##### 解析pdf的完整代码

```java

    /**
     * 获取电子发票pdf文件中的发票信息
     *
     * @param pdfFile 电子发票
     * @return 发票信息
     */
    public InvoiceInfo getInvoiceInfo(File pdfFile) {
        try {
            List<BufferedImage> imageList = extractImage(pdfFile);
            if (imageList.isEmpty()) {
                log.info("pdf中未解析出图片，返回空");
                return null;
            }

            MultiFormatReader formatReader = new MultiFormatReader();
            Result result = null;
            //正常解析出来有3张图片，第一张是二维码，其他两张图片是发票上盖的章
            BinaryBitmap binaryBitmap = new BinaryBitmap(new HybridBinarizer(new BufferedImageLuminanceSource(imageList.get(0))));
            Map hints = new HashMap();
            hints.put(EncodeHintType.CHARACTER_SET, "utf-8");
            result = formatReader.decode(binaryBitmap);
            if (result == null || StringUtils.isEmpty(result.getText())) {
                log.info("pdf中的第一张图片没有解析了字符串信息，直接返回空");
                return null;
            }
            log.info("从电子发票中识别出的信息为：{}", result == null ? "" : result.getText());

            // 读取到的信息为 ： 01，发票类型，发票代码，发票号码，发票金额，开票日期，校验码，随机产生的摘要信息
            String[] infos = result.getText().split(",");
            if (infos.length != 8) {
                log.info("pdf中的第一张图片解析出的字符串数组长度不为8，返回空。");
                return null;
            }

            InvoiceInfo invoice = new InvoiceInfo();
            invoice.setInvoiceType(Integer.parseInt(infos[1])); //发票类型
            invoice.setInvoiceCode(infos[2]); //发票代码
            invoice.setInvoiceNo(infos[3]); // 发票号码
            invoice.setTotalAmount(new BigDecimal(infos[4])); // 发票金额
            invoice.setInvoiceDate(infos[5]); //开票日期
            invoice.setInvoiceCheckcode(infos[6]); // 校验码

            return invoice;
        } catch (Exception e) {
            log.info("解析pdf中的二维码出现异常", e);
            return null;
        }
    }


    /**
     * 提取电子发票里面的图片
     *
     * @param pdfFile 电子发票文件对象
     * @return pdf中解析出的图片列表
     * @throws Exception
     */
    private List<BufferedImage> extractImage(File pdfFile) throws Exception {
        List<BufferedImage> imageList = new ArrayList<>();

        PDDocument document = PDDocument.load(pdfFile);
        try {
            PDPage page = document.getPage(0); //电子发票只有一页
            PDResources resources = page.getResources();

            for (COSName name : resources.getXObjectNames()) {
                if (resources.isImageXObject(name)) {
                    PDImageXObject obj = (PDImageXObject) resources.getXObject(name);
                    imageList.add(obj.getImage());
                }
            }
        } finally {
            document.close();//流未关闭会导致文件删除不掉
        }
        return imageList;
    }
```

##### File对象

> 考虑到本地文件、文件链接和字节流的情况。生成file对象的方法由3种。

###### 本地文件

> 这种情况是最简单的。`File file = new File(filePath);`

###### http连接

> 需要把链接转化成inputStream
>
> ```java
> String remotePdf = "http://XXXXX/rB4r9WBZqmaAeuXVAADICoWEwVQ508.PDF";
> InputStream is = null;
> FileOutputStream fos = null;
> File file = null;
> try {
>     file = TempFile.createTempFile("tmp", ".pdf");
>     fos = new FileOutputStream(file);
>     URL url = new URL(remotePdf);
>     is = url.openStream();
>     int len = 0;
>     byte[] buffer = new byte[1024];
>     while ((len = is.read(buffer)) != -1) {
>         fos.write(buffer, 0, len);
>     }
>     InvoiceInfo invoiceInfo = getInvoiceInfo(file);
> } catch (Exception e) {
> 
> } finally {
>     try {
>         if (fos != null) {
>             fos.close();
>         }
>         if (is != null) {
>             is.close();
>         }
>         file.delete();
>     } catch (Exception e) {
>         e.printStackTrace();
>     }
> }
> ```

###### 字节流

> 需要把链接转化成inputStream，其他部分跟http连接的处理方式一样，创建临时文件和关闭流。
>
> `InputStream is = new ByteArrayInputStream(download)`