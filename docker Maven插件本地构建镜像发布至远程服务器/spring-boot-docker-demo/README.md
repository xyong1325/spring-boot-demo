# spring-boot-docker 


**A创建spring boot 项目**

1、_在 pom.xml 中 ，使用 Spring Boot 2.0 相关依赖、添加 web 和测试依赖_ 
````xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.0.RELEASE</version>
</parent>

 <dependencies>
     <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
 </dependencies>
````

2、创建一个controller

```java
@RestController
 public class DockerController {
     @RequestMapping("/")
     public String index() {
         return "hello docker  !";
     }
 }
```
3、_在src-> main 文件夹下面 新建  docker 文件夹  以及 Dockerfile 文件_
```DockerFile
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ADD spring-boot-docker-1.0.jar app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```
4、_在 pom.xml 中添加  docker 插件_
```xml
<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
			<plugin>
				<groupId>com.spotify</groupId>
				<artifactId>docker-maven-plugin</artifactId>
				<version>1.0.0</version>
				<configuration>
					<forceTags>true</forceTags>
					<imageName>${docker.image.prefix}/${project.artifactId}</imageName>
					<dockerHost>http://xx.xx.xx.xx:2375<!--服务器地址--></dockerHost>
					<dockerDirectory>src/main/docker</dockerDirectory>
					<resources>
						<resource>
							<targetPath>/</targetPath>
							<directory>${project.build.directory}</directory>
							<include>${project.build.finalName}.jar</include>
						</resource>
					</resources>
				</configuration>
			</plugin>
		</plugins>
	</build>
```

**B服务器配置**

1、_服务器上面 安装docker环境_
  * 安装
  ```
   yum install docker
 ```
  * 配置镜像 vim /etc/docker/daemon.json
  ``` 
      {
        "registry-mirrors": ["https://ayeqnkc6.mirror.aliyuncs.com"]
       }
   ```
   * 启动 docker
     ```
          systemctl restart docker
     ```
   * 输入 docker -v 返回 版本号就代表正常
2、开启docker远程api(远程服务器必须开启)
  * 在/etc/systemd/system目录下新建目录docker.service.d，并新建文件http-proxy.conf，添加以下内容保存
  ```
   [Service]
   ExecStart=
   ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
  ```
  * 重新载入系统配置、重启 docker 服务
  ```
   systemctl daemon-reload;
   service docker restart;
```
  * 添加安全组开放端口 2375
