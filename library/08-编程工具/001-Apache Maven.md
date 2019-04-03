# Apache Maven

## 什么是Maven

基于项目对象模型（缩写：POM）概念，Maven利用一个中央信息片断能管理一个项目的构建、报告和文档等步骤。

Maven 是一个项目管理工具，可以对 Java 项目进行构建、依赖管理。

POM( Project Object Model，项目对象模型 ) 是 Maven 工程的基本工作单元，是一个XML文件，包含了项目的基本信息，用于描述项目如何构建，声明项目依赖，等等。

## Maven 安装

下载解压到非中文和空格的目录，配置环境变量`MAVEN_HOME`和`PATH`。

```bash
# mvn 检测是否安装成功
mvn -version
```

> 仓库配置和镜像配置

```xml
<!--路径 conf/settings.xml-->
<!--配置仓库地址-->
<localRepository>D:/.m2/repository</localRepository>
<!--配置镜像地址-->
<!--在mirrors子项目中配置阿里云镜像地址-->
<mirror>
	<id>nexus-aliyun</id>
	<mirrorOf>*</mirrorOf>
	<name>Nexus aliyun</name>
	<url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```

## Maven基本操作

### 项目JDK版本的配置

```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-compiler-plugin</artifactId>
			<version>3.6.1</version>
			<configuration>
				<source>1.8</source>
				<target>1.8</target>
				<encoding>utf-8</encoding>
			</configuration>
		</plugin>
	</plugins>
</build>
```

