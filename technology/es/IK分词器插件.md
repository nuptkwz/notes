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
### 测试 ik_smart（ik最简分词器）
ik_smart会将一句话拆分成几个词语
![ik_smart分词器.png](https://upload-images.jianshu.io/upload_images/9905084-aeb200df1e976dfe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 测试ik_max_word
ik_max_word最细粒度的划分词，它除了将当前词分出来，还将这个句子中可能组合的词语都列出来，穷尽词库的可能
![ik_max_word分词器.png](https://upload-images.jianshu.io/upload_images/9905084-a0119ab7e61831f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
以上是两种分词模式，但是有时候还一些我们自定义的专有名词，它就分不了了，需要我们在配置文件中进行配置，如刻威舟吃油条，刻威舟是一个人名字，但是它却分成了三个字
![自定义词语测试.png](https://upload-images.jianshu.io/upload_images/9905084-78d9e6bf6ea03f32.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### ik分词器增加自己的配置
到ik插件的config下面的IKAnalyzer.cfg.xm里面进行配置，如下：
![IKAnalyzer.cfg.xml.png](https://upload-images.jianshu.io/upload_images/9905084-211ef6cc3940ee32.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
编写自己的配置文件，加到IKAnalyzer.cfg.xml里面，如下：
![将kwz.dic加到IKAnalyzer.cfg.xml里面.png](https://upload-images.jianshu.io/upload_images/9905084-802a3de796ca9fac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

重启es，在Kibana上进行验证，如下：
![验证自定义词.png](https://upload-images.jianshu.io/upload_images/9905084-5826d4ef7064ffe3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看见，刚才“刻威舟”是分开的单词，现在“刻威舟”是一个词语，这就是自定义词的用法









