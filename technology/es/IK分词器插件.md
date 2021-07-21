# 前言
>分词是把一段中文或者别的划分成一个个的关键字,我们在搜索时候会把自己的信息进行分词,会把数据库中或者索引库中的数据进行分词,然后进行一个匹配操作,默认的中文分词器是将每个字看成一个词,比如"中国人很喜欢吃油条"，会被分为“中 国 人 很 喜 欢 吃 油 条”这几个字，而采用IK分词器可以分为“中国人 很 喜欢 吃 油条”，看下上面分词后的结果，肯定是 ik 的分词结果比较符合中文。IK提供了两个分词算法:ik_smart和ik_max_word，其中ik_smart为最少切分,ik_max_word为最细粒度划分。
# 安装
## 下载
https://github.com/medcl/elasticsearch-analysis-ik

![elasticsearch-analysis-ik.png](https://upload-images.jianshu.io/upload_images/9905084-69bbcd0c0efe6726.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 解压到ES plugins文件夹下
![image.png](https://upload-images.jianshu.io/upload_images/9905084-996dbf1dc379e7d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 重启ES
![加载ik插件.png](https://upload-images.jianshu.io/upload_images/9905084-e2e175ea81a97ffa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 检查插件
也可以通过ES自带的工具查看ik插件，命令行执行 ./elasticsearch-plugin list
![elasticsearch-plugin进行插件查询.png](https://upload-images.jianshu.io/upload_images/9905084-55f88cc9c5aa29d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 使用Kibana进行测试





