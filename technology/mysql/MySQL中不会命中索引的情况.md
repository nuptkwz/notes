# 表与索引
我们简单建了一张表，如下：
![image.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy85OTA1MDg0LTc3NTFkNmU2OWQ1NDdhZmYucG5n?x-oss-process=image/format,png)
我们建立了两个索引，分别为主键索引id和普通索引product_id
![image.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy85OTA1MDg0LTJhMWU0MmZmMzNmZGUzYzEucG5n?x-oss-process=image/format,png)
# 是否命中索引
## where 条件中带 or会不会命中索引？
如果我们的where条件中存在or，及时其中带索引，也不会命中索引。这也是为什么尽量使用or的原因
我们通过主键索引查询，可以看见命中了索引：
![image.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy85OTA1MDg0LTQxMjljOTA4YzM3NDBlZmEucG5n?x-oss-process=image/format,png)
而我们在where 条件后面加上or之后，就不会命中索引了：
![image.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy85OTA1MDg0LTE3ZWUzMzg4ZjcwZmMzMWUucG5n?x-oss-process=image/format,png)
如果我们想用or又想让它命中索引，那么只能将or条件中的每一列都加上索引了：
![image.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy85OTA1MDg0LTcyMTQ1NDIyZDU2ZTM5OGEucG5n?x-oss-process=image/format,png)
所以，如果or中有一个条件没有索引的时候，还是建议使用union操作，将多个查询语句拼装在一起：
## like查询中会不会命中索引？
![image.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy85OTA1MDg0LTdjNmNlZjMxNWY1YWU3NDEucG5n?x-oss-process=image/format,png)
可以看到如果用like查询的话，%在右边会命中索引的，而%在左边则不会。当然这也不是绝对的，当我们使用索引列进行查询的时候，就都会命中索引了：
![image.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy85OTA1MDg0LTRkNTA2ZDZmMDQyNzEyZjUucG5n?x-oss-process=image/format,png)
## 负向查询条件会不会用到索引？
NOT IN,NOT LIKE,NOT EXISTS都不会命中索引





