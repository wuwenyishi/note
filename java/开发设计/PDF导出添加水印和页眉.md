

## maven引用
```xml
<dependency>
    <groupId>com.itextpdf</groupId>
    <artifactId>itextpdf</artifactId>
    <version>5.5.13</version>
</dependency>
<dependency>
    <groupId>com.itextpdf</groupId>
    <artifactId>itext-asian</artifactId>
    <version>5.2.0</version>
</dependency>
<dependency>
    <groupId>org.apache.pdfbox</groupId>
    <artifactId>pdfbox</artifactId>
    <version>2.0.21</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>com.itextpdf.tool</groupId>
    <artifactId>xmlworker</artifactId>
    <version>5.5.13</version>
    <scope>provided</scope>
</dependency>
```
## 代码实现
```java
/**
	 * PDF预览
	 * @param response
	 * @param filename 文件名称
	 * @param rect PDF每张页面大小
	 * @param backgroundColor 背景颜色
	 * @param marginLeft 左页边距
	 * @param marginRight 右页边距
	 * @param marginTop 顶部页边距
	 * @param marginBottom 底部页面距
	 * @throws Exception
	 */
	public static void pdfExportPreview(HttpServletResponse response, String filename,Rectangle rect,BaseColor backgroundColor,
										float marginLeft, float marginRight, float marginTop, float marginBottom) throws Exception {
		response.setContentType("application/pdf");
		pdfExport(response, filename, rect, backgroundColor, marginLeft, marginRight, marginTop, marginBottom);
	}
```
```java
private static void pdfExport(HttpServletResponse response,String filename,Rectangle rect,BaseColor backgroundColor,
								  float marginLeft, float marginRight, float marginTop, float marginBottom) throws IOException {
		Document document = null;
		OutputStream os = null;
		try {
			response.addHeader("Content-Disposition", "inline;filename=" + URLEncoder.encode(filename, "UTF-8") + ".pdf");
			document =createDocument(rect,backgroundColor,marginLeft, marginRight, marginTop, marginBottom);
			os = new BufferedOutputStream(response.getOutputStream());
			// 2. 获取writer
			PdfWriter writer = PdfWriter.getInstance(document, os);
			writer.setPdfVersion(PdfWriter.PDF_VERSION_1_2);
			// 页眉/页脚
			writer.setPageEvent(new MyHeaderFooter(marginLeft));
			// 3. open()
			document.open();

			for (Element element : elementList) {
				document.add(element);
				if (newPageList.contains(element)) {
					document.newPage();
				}
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			elementList.clear();
			newPageList.clear();
			if (document != null){
				document.close();
			}
			if (os != null){
				os.close();
			}
		}
	}
```
```java
import com.itextpdf.text.*;
import com.itextpdf.text.pdf.*;
import com.itextpdf.text.pdf.draw.LineSeparator;
import lombok.SneakyThrows;

/**
 * @author chenkang
 */
public class MyHeaderFooter extends PdfPageEventHelper {
	MyHeaderFooter(){
	}
	public float marginLeft;
	MyHeaderFooter(float marginLeft){
		this.marginLeft = marginLeft;
	}
    /**
     * 模板
     */
    public PdfTemplate total;

    /**
     * 基础字体对象
     */
    public BaseFont bf = null;


    /**
     *  文档打开时创建模板
     * @see PdfPageEventHelper#onOpenDocument(PdfWriter,
     *      Document)
     */
    @Override
    public void onOpenDocument(PdfWriter writer, Document document) {
        // 共 页 的矩形的长宽高
        total = writer.getDirectContent().createTemplate(50, 50);
    }

    /**
     *
     *  关闭每页的时候，写入页眉，写入'第几页共'这几个字。*
     * @see PdfPageEventHelper#onEndPage(PdfWriter,
     *      Document)
     */
    @SneakyThrows
    @Override
    public void onEndPage(PdfWriter writer, Document document) {
        this.addPage(writer, document);
        this.addWatermark(writer);
    }

    //加分页
    public void addPage(PdfWriter writer, Document document) throws Exception {
        //设置分页页眉页脚字体
        if (bf == null) {
            bf = BaseFont.createFont("STSong-Light", "UniGB-UCS2-H", false);
        }

        Font columnTextfont = new Font(bf, Font.DEFAULTSIZE, Font.NORMAL, BaseColor.RED);
        int pageS = writer.getPageNumber();
        if (pageS != 0) {
            ClassPathResource classPathResource = new ClassPathResource("img/logo.png");
            URL url = classPathResource.getURL();
            Image image = Image.getInstance(url);
            Phrase p1 = new Phrase("", columnTextfont);
            p1.add(new Chunk(image, 0, -10));
            ColumnText.showTextAligned(writer.getDirectContent(), Element.ALIGN_LEFT, p1, document.left(), document.top() + 15, 0);

             //1.写入页眉文字
            //ColumnText.showTextAligned(writer.getDirectContent(), Element.ALIGN_LEFT, new Phrase("页眉文字内容", columnTextfont), document.left(), document.top() + 8, 0);
            //ColumnText.showTextAligned(writer.getDirectContent(), Element.ALIGN_RIGHT, new Phrase(header, columnTextfont), document.right(), document.top() + 8, 0);
            // 直线
            Paragraph p2 = new Paragraph();
            LineSeparator lineSeparator = new LineSeparator();
            lineSeparator.setLineColor(BaseColor.GRAY);
            lineSeparator.setLineWidth(1.0f);
            p2.add(new Chunk(lineSeparator));
            ColumnText.showTextAligned(writer.getDirectContent(), Element.ALIGN_LEFT, p2, document.left() - marginLeft, document.top() + 3, 0);

            // 2.写入前半部分的 第 X页/共
            String foot1 = "第 " + (pageS) + " 页 /共";
            Font fontDetail = new Font(bf, Font.STRIKETHRU, Font.NORMAL, BaseColor.BLACK);
            Phrase footer = new Phrase(foot1, fontDetail);
            // 3.计算前半部分的foot1的长度，后面好定位最后一部分的'Y页'这俩字的x轴坐标，字体长度也要计算进去 = len
            float len = bf.getWidthPoint(foot1, Font.STRIKETHRU);

            // 4.拿到当前的PdfContentByte
            PdfContentByte cb = writer.getDirectContent();

            // 5.写入页脚1，x轴就是(右margin+左margin + right() -left()- len)/2.0F
            // 再给偏移20F适合人类视觉感受，否则肉眼看上去就太偏左了
            // ,y轴就是底边界-20,否则就贴边重叠到数据体里了就不是页脚了；注意Y轴是从下往上累加的，最上方的Top值是大于Bottom好几百开外的。
            ColumnText.showTextAligned(cb, Element.ALIGN_CENTER, footer,
                    (document.rightMargin() + document.right() + document.leftMargin() - document.left() - len) / 2.0F + 20F,
                    document.bottom() - 20, 0);

            // 6.写入页脚2的模板（就是页脚的Y页这俩字）添加到文档中，计算模板的和Y轴,X=(右边界-左边界 - 前半部分的len值)/2.0F +
            // len ， y 轴和之前的保持一致，底边界-20 // 调节模版显示的位置
            cb.addTemplate(total, (document.rightMargin() + document.right() + document.leftMargin() - document.left()) / 2.0F + 20F,
                    document.bottom() - 20);
        }


    }

    //加水印
    public void addWatermark(PdfWriter writer) throws Exception {
        PdfContentByte content = writer.getDirectContent();
        content.beginText();
        BaseFont font = BaseFont.createFont("STSong-Light", "UniGB-UCS2-H", false);
        content.setFontAndSize(font, 20);
        // 设置透明度
        // 设置水印透明度
        PdfGState gs = new PdfGState();
        // 设置填充字体不透明度为0.4f
        gs.setFillOpacity(0.1f);
        content.setGState(gs);
        for (int i = 0; i < 50; i++) {
            for (int j = 0; j < 50; j++) {
                content.showTextAligned(Element.ALIGN_CENTER, "这是水印       这是水印", i * 200, j * 200, 45);
            }
        }
        // 设置水印颜色(灰色)
        content.setColorFill(BaseColor.GRAY);
        //结束
        content.endText();
    }

    /**
     *
     * 关闭文档时，替换模板，完成整个页眉页脚组件
     *
     * @see PdfPageEventHelper#onCloseDocument(PdfWriter,
     *      Document)
     */
    @Override
    public void onCloseDocument(PdfWriter writer, Document document) {
        // 7.最后一步了，就是关闭文档的时候，将模板替换成实际的 Y 值,至此，page x of y 制作完毕，完美兼容各种文档size。
        total.beginText();
        // 生成的模版的字体、颜色
        total.setFontAndSize(bf, Font.STRIKETHRU);
        //页脚内容拼接  如  第1页/共2页
        String foot2 = " " + (writer.getPageNumber()) + " 页";
        // 模版显示的内容
        total.showText(foot2);
        total.endText();
        total.closePath();
    }
}

```

**示例图：**  
![DoAJOn](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/DoAJOn.png)