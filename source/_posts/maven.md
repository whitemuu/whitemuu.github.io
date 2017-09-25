---
title: maven
date: 2017-02-22 19:02:56
desc:
tags:
  - maven
  - intellij
---

<!-- more -->
# 私服nexus

### 阿里镜像

```xml
    <mirror>
      <id>alimaven</id>
      <mirrorOf>central</mirrorOf>        
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    </mirror>

```
### 从nexus下载依赖 ###

~/.m2/setting.xml
```xml
<mirrors>
	 <mirror>
		 <!--此处配置所有的构建均从私有仓库中下载 *代表所有，也可以写central -->
		 <id>nexus</id>
		 <mirrorOf>*</mirrorOf> <!--或者是central-->
		 <url>http://localhost:8080/nexus/content/groups/public/</url>
	 </mirror>
 </mirrors>
```

### 布署到nexus `mvn deploy` ###

setting.xml
```xml
<server>
  <id>releases</id>
  <username>admin</username>
  <password>admin123</password>
</server>
<server>
  <id>snapshots</id>
  <username>admin</username>
  <password>admin123</password>
</server>
```
pom.xml
```xml
<distributionManagement>
		<repository>
			<id>releases</id>
			<name>Internal Releases</name>
			<url>http://localhost:8080/nexus-2.7.0-06/content/repositories/releases/</url>
		</repository>
		<snapshotRepository>
			<id>snapshots</id>
			<name>Internal Snapshots</name>
			<url>http://localhost:8080/nexus-2.7.0-06/content/repositories/snapshots/</url>
		</snapshotRepository>
</distributionManagement>
```

# 纯maven操作笔记heima

打jar、war在pom一个标签里，tag.todo GAV标签里有个<packaging>标签

Maven命令
```sh
Mvn clean
清除上一次构建时的文件

Mvn compile
编译命令，会将main目录下的源代码进行编译

Mvn test
编译测试代码的命令，会将test目录下的源代码进行编译

Mvn package
打包命令，可以将工程打成jar包或者war包

Mvn install
安装命令，会将本地打好的包，安装到本地仓库中。

Mvn clean test
先执行清除，在执行测试命令
Clean 可以和test、compile、install、package等命令进行组合，即先清除再执行后续命令。
```

---

## maven parent 和 dependency的区别

parent <dependencyManagement> <pluginManagemaent> 标签管理依赖

<dependency>标签引入的其实全是pom，所以才能够引入被引入依赖的依赖，因为被引入依赖的pom里有其自己<dependency> 标签



pom指定jdk版本：配置编译plugin

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-compiler-plugin</artifactId>
	<configuration>
		<source>1.7</source>
		<target>1.7</target>
		<encoding>UTF-8</encoding>
	</configuration>
</plugin>
```

## eclipse创建maven聚合工程

在父工程上面新建maven module，新工程目录就会在父工程下面（真实文件夹层面）——其实就是intellij的module概念，故intellij天生和maven结构一致。

service如何依赖dao层？在pom文件添加dao依赖

```xml
<dependency>
    <groupId>${project.groupId}</groupId>
    <artifactId>dao</artifactId>
    <version>${project.version}</version>
    <scope>test</scope>
</dependency>
```

不清楚的概念：父工程的module标签，---这是嵌套module即manage工程pom里才有的。

这里他只给controller层选择的war，工程发布结构是怎样的呢？



# 依赖范围

| scope    | 主代码classpth | 测试代码classpath | 打包后运行classpath | 例子                         |
| -------- | ----------- | ------------- | -------------- | -------------------------- |
| compile  | Y           | Y             | Y              | log4j                      |
| test     |             | Y             |                | junit                      |
| provided | Y           | Y             |                | servlet-api                |
| runtime  |             |               | Y              | JDBC driver implementation |

![](http://of01cnpwt.bkt.clouddn.com/md20161020215729.png)

依赖传递

| 第一\第二直截依赖 | compile  | test | provided | runtime  |
| --------- | -------- | ---- | -------- | -------- |
| compile   | compile  |      |          | runtime  |
| test      | test     |      |          | test     |
| provided  | provided |      | provided | provided |
| runtime   | runtime  |      |          | runtime  |

## Maven生命周期,生命周期分成phrase（阶段）

> 在一个生命周期中，运行某个阶段的时候，它之前的所有阶段都会被运行

* clean Lifecycle
  ```
  pre-clean
  clean
  post-clean
  ```

* Default Lifecycle

  ```sh
  validate 
  generate-sources 
  process-sources 
  generate-resources 
  process-resources 复制并处理资源文件，至目标目录，准备打包。 
  *compile 编译项目的源代码。 
  process-classes 
  generate-test-sources 
  process-test-sources 
  generate-test-resources 
  process-test-resources 复制并处理资源文件，至目标测试目录。 
  test-compile 编译测试源代码。 
  process-test-classes 
  *test 使用合适的单元测试框架运行测试。这些测试代码不会被打包或部署。 
  prepare-package 
  *package 接受编译好的代码，打包成可发布的格式，如 JAR 。 
  pre-integration-test 
  integration-test 
  post-integration-test 
  verify 
  *install 将包安装至本地仓库，以让其它项目依赖。 
  deploy 将最终的包复制到远程的仓库，以让其它开发人员与项目共享。 
  ```

* Site Lifecycle

  ```
  pre-site 执行一些需要在生成站点文档之前完成的工作 
  site 生成项目的站点文档 
  post-site 执行一些需要在生成站点文档之后完成的工作，并且为部署做准备 
  site-deploy 将生成的站点文档部署到特定的服务器上 
  ```




# pom inheritance

`lib/maven-model-builder-3.0.3.jar:org/apache/maven/model/pom-4.0.0.xml`
​          ⊢`conf/setting.xml`
​                      ⊢`~/.m2/setting.xml`

## pom-4.0.0.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
Licensed to the Apache Software Foundation (ASF) under one or more contributor license agreements.  See the NOTICE file distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file to you under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License.  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the specific language governing permissions and limitations under the License. -->

<!-- START SNIPPET: superpom -->
<project>
	<modelVersion>4.0.0</modelVersion>

	<repositories>
		<repository>
			<id>central</id>
			<name>Central Repository</name>
			<url>https://repo.maven.apache.org/maven2</url>
			<layout>default</layout>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
	</repositories>

	<pluginRepositories>
		<pluginRepository>
			<id>central</id>
			<name>Central Repository</name>
			<url>https://repo.maven.apache.org/maven2</url>
			<layout>default</layout>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
			<releases>
				<updatePolicy>never</updatePolicy>
			</releases>
		</pluginRepository>
	</pluginRepositories>

	<build>
		<directory>${project.basedir}/target</directory>
		<outputDirectory>${project.build.directory}/classes</outputDirectory>
		<finalName>${project.artifactId}-${project.version}</finalName>
		<testOutputDirectory>${project.build.directory}/test-classes</testOutputDirectory>
		<sourceDirectory>${project.basedir}/src/main/java</sourceDirectory>
		<scriptSourceDirectory>${project.basedir}/src/main/scripts</scriptSourceDirectory>
		<testSourceDirectory>${project.basedir}/src/test/java</testSourceDirectory>
		<resources>
			<resource>
				<directory>${project.basedir}/src/main/resources</directory>
			</resource>
		</resources>
		<testResources>
			<testResource>
				<directory>${project.basedir}/src/test/resources</directory>
			</testResource>
		</testResources>
		<pluginManagement>
			<!-- NOTE: These plugins will be removed from future versions of the super POM -->
			<!-- They are kept for the moment as they are very unlikely to conflict with lifecycle mappings (MNG-4453) -->
			<plugins>
				<plugin>
					<artifactId>maven-antrun-plugin</artifactId>
					<version>1.3</version>
				</plugin>
				<plugin>
					<artifactId>maven-assembly-plugin</artifactId>
					<version>2.2-beta-5</version>
				</plugin>
				<plugin>
					<artifactId>maven-dependency-plugin</artifactId>
					<version>2.8</version>
				</plugin>
				<plugin>
					<artifactId>maven-release-plugin</artifactId>
					<version>2.3.2</version>
				</plugin>
			</plugins>
		</pluginManagement>
	</build>

	<reporting>
		<outputDirectory>${project.build.directory}/site</outputDirectory>
	</reporting>

	<profiles>
		<!-- NOTE: The release profile will be removed from future versions of the super POM -->
		<profile>
			<id>release-profile</id>

			<activation>
				<property>
					<name>performRelease</name>
					<value>true</value>
				</property>
			</activation>

			<build>
				<plugins>
					<plugin>
						<inherited>true</inherited>
						<artifactId>maven-source-plugin</artifactId>
						<executions>
							<execution>
								<id>attach-sources</id>
								<goals>
									<goal>jar</goal>
								</goals>
							</execution>
						</executions>
					</plugin>
					<plugin>
						<inherited>true</inherited>
						<artifactId>maven-javadoc-plugin</artifactId>
						<executions>
							<execution>
								<id>attach-javadocs</id>
								<goals>
									<goal>jar</goal>
								</goals>
							</execution>
						</executions>
					</plugin>
					<plugin>
						<inherited>true</inherited>
						<artifactId>maven-deploy-plugin</artifactId>
						<configuration>
							<updateReleaseInfo>true</updateReleaseInfo>
						</configuration>
					</plugin>
				</plugins>
			</build>
		</profile>
	</profiles>

</project>
		<!-- END SNIPPET: superpom -->
```

## pom.xml template

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.itheima.parent</groupId>
	<artifactId>itheima-parent</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>pom</packaging>

	<!-- 集中定义依赖版本号 -->
	<properties>
		<junit.version>4.10</junit.version>
		<spring.version>4.1.3.RELEASE</spring.version>
	</properties>

	<dependencyManagement>
		<dependencies>
			<!-- 单元测试 -->
			<dependency>
				<groupId>junit</groupId>
				<artifactId>junit</artifactId>
				<version>${junit.version}</version>
				<scope>test</scope>
			</dependency>

			<!-- Spring -->
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-context</artifactId>
				<version>${spring.version}</version>
			</dependency>
           ...
	</dependencyManagement>

	<build>
		<finalName>${project.artifactId}</finalName>
		<plugins>
			<!-- 资源文件拷贝插件 -->
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-resources-plugin</artifactId>
				<version>2.7</version>
				<configuration>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
			<!-- java编译插件 -->
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.2</version>
				<configuration>
					<source>1.7</source>
					<target>1.7</target>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
		</plugins>
		<pluginManagement>
			<plugins>
				<!-- 配置Tomcat插件 -->
				<plugin>
					<groupId>org.apache.tomcat.maven</groupId>
					<artifactId>tomcat7-maven-plugin</artifactId>
					<version>2.2</version>
				</plugin>
			</plugins>
		</pluginManagement>
	</build>
```



[1]: 	&quot;黑马maven教案&quot;
[2]: 	http://juvenshun.iteye.com/blog/359256	"重要"
