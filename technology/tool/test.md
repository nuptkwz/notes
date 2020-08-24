在eclipse中写jsp页面时，修改了xml文件，启动Tomcat7.0出现了下图情况：



并且在控制台上显示警告：SetPropertiesRule]{Server/Service/Engine/Host/Context} Setting property 'source' to 'org.eclipse.jst.jee.server:day04' did not find a matching property.

解法方法：
1.双击servers里的Tomcat

2.依次点击如下两个按钮

改为第二个位置即：Use Tomcat installation...

下面这个按钮点击：publish moduls contexts.....

按Ctr+s进行保存
最后再启动重新启动Tomcat服务器，访问8080，成功出现Tomcat页面！~~~~