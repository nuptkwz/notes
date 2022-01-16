> 本文主要介绍了Es的Restful Api的操作。

# 操作文档
![es的crud文档.png](https://upload-images.jianshu.io/upload_images/9905084-75286f33954c065e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 基本curd操作
## 创建
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

#### 指定类型创建索引
es基本数据类型
参考：https://blog.csdn.net/kris1025/article/details/81279339

除了基本类型外，可以自定义类型，例如：
![创建index2索引的数据结构.png](https://upload-images.jianshu.io/upload_images/9905084-6c65b8d8bab35119.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 默认类型
创建相关的字段和值，es会根据字段值自动加上类型，例如：
![创建加入数据.png](https://upload-images.jianshu.io/upload_images/9905084-567c14e995783d82.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![查看索引字段类型.png](https://upload-images.jianshu.io/upload_images/9905084-b8adb0441f61a896.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 查询
### 语法
```
GET xxx
```
### 示例
![image.png](https://upload-images.jianshu.io/upload_images/9905084-462055b8c7a95460.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 获取_cat命令获取es更多当前的信息
#### 示例
![查询es健康状态.png](https://upload-images.jianshu.io/upload_images/9905084-6f6ad5645bc19dc3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9905084-0b8261ddcaf43643.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 修改
修改命令分为两种，一种是PUT更新，另一种是POST更新
### PUT更新
PUT更新会用当前对象全局覆盖，如：
![修改语法.png](https://upload-images.jianshu.io/upload_images/9905084-8ec21ed755419743.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9905084-bd39d4505b638690.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### POST更新
![image.png](https://upload-images.jianshu.io/upload_images/9905084-9f78b556bb0aba7e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/9905084-f757d85fda044f6d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 删除
### 语法
```
DELETE xxx
```
![image.png](https://upload-images.jianshu.io/upload_images/9905084-2da48e05f205f783.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
