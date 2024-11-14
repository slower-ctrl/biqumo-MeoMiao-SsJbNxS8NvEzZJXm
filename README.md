
## 前言


**在工作我们经常会出现有多个文件，为了节省资源会将多个文件放在一起进行压缩处理；为了让大家进一步了解我先将springboot处理的方法总结如下，有不到之处敬请大家批评指正！**


### 一、文件准备：




```
https://qnsc.oss-cn-beijing.aliyuncs.com/crmebimage/public/product/2024/11/12/be353210028a3da732c8ba34073fb4ca.jpeg
https://qnsc.oss-cn-beijing.aliyuncs.com/crmebimage/public/product/2024/11/13/5bbf579109db2641249deab4be4340f6.jpeg
https://qnsc.oss-cn-beijing.aliyuncs.com/crmebimage/public/product/2024/11/13/1808773678128361474.xlsx
```


### 二、处理步骤：


#### 1\.创建一个springboot web项目 这一步在此省略.....


#### 2\.需要的方法及类的编写


##### （1）业务方法\-TestService




```
public interface TestService {
    void compressFiles(List fileUrls, HttpServletResponse response);
}
```


##### （2）业务方法实现类\-TestServiceImpl




```
@Service
@Slf4j
public class TestServiceImpl implements TestService {

    @Override
    public void compressFiles(List fileUrls, HttpServletResponse response) {
        try (ZipOutputStream zipOut = new ZipOutputStream(response.getOutputStream())) {
            for (String fileUrl : fileUrls) {
                // 1.从网络下载文件并写入 ZIP
                try {
                    URL url = new URL(fileUrl);
                    HttpURLConnection connection = (HttpURLConnection) url.openConnection();
                    connection.setRequestMethod("GET");
                    connection.connect();
                    // 2.检查响应码
                    if (connection.getResponseCode() != HttpURLConnection.HTTP_OK) {
                        throw new IOException("Failed to download file: " + fileUrl);
                    }
                    // 3.从 URL 中提取文件名
                    String pathStr = fileUrl.substring(fileUrl.lastIndexOf('/') + 1);
                    // 4.创建 ZIP 条目
                    ZipEntry zipEntry = new ZipEntry(pathStr);
                    zipOut.putNextEntry(zipEntry);
                    // 5.读取文件的输入流
                    try (InputStream inputStream = new BufferedInputStream(connection.getInputStream())) {
                        byte[] buffer = new byte[1024];
                        int length;
                        while ((length = inputStream.read(buffer)) >= 0) {
                            zipOut.write(buffer, 0, length);
                        }
                    }
                    zipOut.closeEntry();
                } catch (IOException e) {
                    log.error("Error processing file URL: " + fileUrl, e);
                    throw new RuntimeException(e);
                }
            }　　　　　　　// 6.响应信息设置处理
            response.setContentType("application/octet-stream");
            response.setHeader("Content-Disposition", "attachment;filename=test.zip");
            response.flushBuffer();
        } catch (IOException e) {
            log.error("Error compressing files", e);
            throw new RuntimeException(e);
        }
    }
}
```


##### （3）控制器类的编写\-TestController




```
/**
 * @Project:
 * @Description:
 * @author: songwp
 * @Date: 2024/11/13 14:50
 **/
@RequestMapping("test")
@RestController
@Slf4j
public class TestController {

    @Autowired
    private TestService testService;

    /**
     * 文件压缩
     *
     * @param fileUrls 要压缩的文件 URL 列表
     * @param response 响应对象
     */
    @GetMapping("/fileToZip")
    public void zip(@RequestParam("fileUrls") List fileUrls, HttpServletResponse response) {
        testService.compressFiles(fileUrls, response);
    }
}
```


### 三、方法调用展示


[![](https://img2024.cnblogs.com/blog/2156747/202411/2156747-20241113145438523-1180425776.png)](https://img2024.cnblogs.com/blog/2156747/202411/2156747-20241113145438523-1180425776.png)


#### （1）存放到桌面


[![](https://img2024.cnblogs.com/blog/2156747/202411/2156747-20241113145534929-1891329062.png)](https://img2024.cnblogs.com/blog/2156747/202411/2156747-20241113145534929-1891329062.png)


#### （2）解压response.zip文件


[![](https://img2024.cnblogs.com/blog/2156747/202411/2156747-20241113145639813-1124319126.png)](https://img2024.cnblogs.com/blog/2156747/202411/2156747-20241113145639813-1124319126.png)


[![](https://img2024.cnblogs.com/blog/2156747/202411/2156747-20241113145706931-1725910256.png)](https://img2024.cnblogs.com/blog/2156747/202411/2156747-20241113145706931-1725910256.png)


 


  * [前言](#tid-bd4Ab2)
* [一、文件准备：](#tid-Wprdis)
* [二、处理步骤：](#tid-re4TMh)
* [1\.创建一个springboot web项目 这一步在此省略.....](#tid-dfRKMY)
* [2\.需要的方法及类的编写](#tid-ein77n)
* [（1）业务方法\-TestService](#tid-yxm5F6):[slower加速器](https://jisuanqi.org)
* [（2）业务方法实现类\-TestServiceImpl](#tid-XMefQf)
* [（3）控制器类的编写\-TestController](#tid-XpcDpB)
* [三、方法调用展示](#tid-QDNt3x)
* [（1）存放到桌面](#tid-7aQ6xR)
* [（2）解压response.zip文件](#tid-JPpMrG)

   \_\_EOF\_\_

   ![](https://github.com/songweipeng)奋 斗  - **本文链接：** [https://github.com/songweipeng/p/18543941](https://github.com)
 - **关于博主：** 评论和私信会在第一时间回复。或者[直接私信](https://github.com)我。
 - **版权声明：** 本博客所有文章除特别声明外，均采用 [BY\-NC\-SA](https://github.com "BY-NC-SA") 许可协议。转载请注明出处！
 - **声援博主：** 如果您觉得文章对您有帮助，可以点击文章右下角**【[推荐](javascript:void(0);)】**一下。
     
