> 本文主要介绍了Es的Restful Api的操作。

# 操作文档
![es的crud文档.png](https://upload-images.jianshu.io/upload_images/9905084-75286f33954c065e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 基本curd操作
## 创建索引
### 语法
```
PUT /索引名称/~类型名~/文档id
{
      请求体
}
```
### 示例
#### Kibana中新建索引index1
创建索引可以很多种方式比如命令行、浏览器、Postman、Kibana等，下面演示了通过Kibana创建索引，head进行索引查询
![新建索引index1.png](https://upload-images.jianshu.io/upload_images/9905084-8f2ca1daec14fe78.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### head中查看索引创建
![查看index1索引.png](https://upload-images.jianshu.io/upload_images/9905084-90cebec17a6eb2e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![head中的索引.png](https://upload-images.jianshu.io/upload_images/9905084-fe85a16f982a6b3f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![head中数据浏览.png](https://upload-images.jianshu.io/upload_images/9905084-778bdc85f59f2d1a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

