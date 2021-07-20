# elasticsearch-head-master安装
- npm install
- npm run start
# 开启跨域支持，并且让所有人可以访问
```json
http.cors.enabled: true
http.cors.allow-origin: "*"
```
# 通过ES head访问
1. 访问http://localhost:9200/
```
{
  "name" : "DESKTOP-VEJU22Q",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "-YdADrcDTWyFTiy8kDA5Hg",
  "version" : {
    "number" : "7.6.1",
    "build_flavor" : "default",
    "build_type" : "zip",
    "build_hash" : "aa751e09be0a5072e8570670309b1f12348f023b",
    "build_date" : "2020-02-29T00:15:25.529771Z",
    "build_snapshot" : false,
    "lucene_version" : "8.4.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```
2. 访问http://localhost:9100/
![image.png](https://upload-images.jianshu.io/upload_images/9905084-d0d97e32d57eed7c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


