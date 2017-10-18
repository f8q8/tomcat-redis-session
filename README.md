# tomcat-redis-session
tomcat和redis做session共享
【环境】

centos6.5两台A、B

apache-tomcat-7.0.78

nginx

【配置过程】

分别在A、B机器上安装tomcat，确保tomcat能够正常访问

在tomcat的webapp/ROOT中增加test.jsp文件

test.jsp

<%@ page language="java" contentType="text/html; charset=UTF-8"
        pageEncoding="UTF-8"%>
<html>
<body>
        <%
                HttpSession s = request.getSession();
                s.setAttribute("sessionId", s.getId());
        %>
        得到Session的值是
        <%=s.getAttribute("sessionId")%>
        <br />
        当前服务器名称：<%=request.getServerName()%>
        <br /> serverA <!--在A机器上写A，在B机器上写B-->
</body>
</html>


将本文附件的三个jar包拷贝到tomcat的lib目录中
编译本代码
mvn install
生成tomcat-redis-session-manager-1.2-tomcat-7-v1.2.jar
http://central.maven.org/maven2/org/apache/commons/commons-pool2/2.2/commons-pool2-2.2.jar
commons-pool2-2.2.jar

http://central.maven.org/maven2/redis/clients/jedis/2.5.2/jedis-2.5.2.jar
jedis-2.5.2.jar


在tomcat的conf目录中的content.xml中增加如下配置：

 

如果配置一台redis用以下配置：
<Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" />
<Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager"
         host="192.168.10.20"
         port="6379"
         database="3"
         password="footbar'"
         maxInactiveInterval="60" />
 
如果配置redis哨兵集群，用以下配置：
 
<Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" />
<Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager"
         port="6379"
         password="footbar"
         sentinelMaster="mymaster"
         sentinels="192.168.10.20:26379,192.168.10.233:26379"
         database="3"
         maxInactiveInterval="60" />
 

确保每个tomcat都可以正常访问

配置nginx（随便安装到A或者B都行，只安装到一台上就行了）

upstream gitlabhttp {
    server 192.168.10.20:8080;
    server 192.168.10.233:8080;
}

server {
    listen 8090;
    server_name gitlab.smallbee.io;
    location / {
        proxy_pass http://gitlabhttp;
        proxy_set_header Host $host; 
        proxy_set_header X-Real-IP $remote_addr; 
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
        proxy_set_header X-Forwarded-Proto $scheme; 
       }
    #
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root html;
    }
}

配置好后，访问nginx的地址加端口号

http://nginx:8090/test.jsp

如果sessionid保持不变，serverA和serverB是变化的，就说明配置成功了