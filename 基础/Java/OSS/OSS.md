# 回顾文件上传

> 前端页面

```html
<form action="/upload" enctype="multipart/form-data" method="post">
    <input type="hidden" name="dir" value="image">
    <input type="file" name="file">
    <input type="submit" value="提交">
</form>
```

> 图片路径处理

```java
/**
 * @author ：zsy
 * @date ：Created 2022/1/13 21:11
 * @description：
 */
public class ImageUtil {
    private static String projectImgPath = System.getProperty("user.dir") + File.separator;

    public static String getImgStorePath(String imageName, String dir) {
        File curFile = new File(projectImgPath + dir);
        if (!curFile.exists()) {
            curFile.mkdirs();
        }
        String imgName = imageName.substring(imageName.indexOf("."));
        return curFile.getPath()
                + File.separator
                + UUID.randomUUID().toString().substring(0, 4)
                + imgName;
    }
}
```

> 上传图片

```java
@PostMapping("/upload")
public String upload(@RequestParam("file") MultipartFile multipartFile,
                     HttpServletRequest request,
                     HttpServletResponse response) throws IOException {
    if (multipartFile == null) {
        return "上传失败";
    }
    String dir = request.getParameter("dir");
    FileInputStream fis = null;
    try {
        String path = uploadService.upload(multipartFile, dir);
        fis = new FileInputStream(new File(path));
        response.setContentType("image/png");
        ServletOutputStream out = response.getOutputStream();
        byte[] bytes = new byte[1024];
        int rc = 0;
        while ((rc = fis.read(bytes)) != 0) {
            out.write(bytes, 0, rc);
        }
        out.flush();
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (fis != null) {
            fis.close();
        }
    }
    return "123";
}
```

> service

```java
/**
 * @author ：zsy
 * @date ：Created 2022/1/13 20:18
 * @description：
 */
@Service
public class UploadServiceImpl implements UploadService {
    @Override
    public String upload(MultipartFile file, String dir) {
        String imgStorePath = ImageUtil.getImgStorePath(file.getOriginalFilename(), dir);
        try {
            file.transferTo(new File(imgStorePath));
        } catch (IOException e) {
            e.printStackTrace();
            return "上传失败";
        }
        return imgStorePath;
    }
}
```

# 阿里云oss文件存储对象

[文档](https://help.aliyun.com/document_detail/32008.html?spm=5176.87240.400427.45.24f84614CspZUw)

导入oss的SDK依赖

```xml
<dependency>
    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-sdk-oss</artifactId>
    <version>3.10.2</version>
</dependency>
```

> 代码

```java
/**
 * @author ：zsy
 * @date ：Created 2022/1/13 23:35
 * @description：
 */
@Service
public class OssUploadServiceImpl implements OssUploadService {
    // yourEndpoint填写Bucket所在地域对应的Endpoint。以华东1（杭州）为例，Endpoint填写为https://oss-cn-hangzhou.aliyuncs.com。
    String endpoint = "oss-cn-beijing.aliyuncs.com";
    // 阿里云账号AccessKey拥有所有API的访问权限，风险很高。强烈建议您创建并使用RAM用户进行API访问或日常运维，请登录RAM控制台创建RAM用户。
    String accessKeyId = "";
    String accessKeySecret = "";
    // 填写Bucket名称，例如examplebucket。
    String bucketName = "dunkcode";
    OSS ossClient = null;
    {
        // 创建OSSClient实例。
        ossClient = new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);
    }

    public String uploadFile(MultipartFile file) {
        try {
            String originalFilename = file.getOriginalFilename();
            InputStream inputStream = file.getInputStream();
            String newFilename = "images/" + System.currentTimeMillis()
                    + UUID.randomUUID().toString().substring(0, 4)
                    + originalFilename.substring(originalFilename.indexOf("."));
            ObjectMetadata objectMetadata = new ObjectMetadata();
            objectMetadata.setContentType("image/jpg");
            ossClient.putObject(bucketName, newFilename, inputStream, objectMetadata);

            return "https://" + bucketName + "." + endpoint + "/" + newFilename;
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 关闭OSSClient。
            ossClient.shutdown();
        }
        return "";
    }
}
```

