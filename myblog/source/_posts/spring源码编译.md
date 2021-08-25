---
title: 编译spring 5.1.X源码
categories:
  - spring 5.1.X源码
tags: spring源码
abbrlink: c9117111
date: 2021-07-10 22:52:21
---

### 环境

1. mac环境
2. JDK 1.8.0_202
3. Gradle 4.10.3

### 步骤

##### 把spring源码fork到自己的仓库

`https://github.com/spring-projects/spring-framework`

<!--more-->

##### 下载spring源码

```java
git clone git@github.com:caijianying/spring-framework.git -b 5.1.x
```

##### 确定gradle版本

进入`spring-framework`查看配置文件`cat gradle/wrapper/gradle-wrapper.properties`

```java
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-4.10.3-bin.zip
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
```

`distributionUrl`就是下载地址

##### 预编译前，修改`build.gradle`的镜像源

```java
buildscript {
	repositories {
		maven { url 'https://maven.aliyun.com/repository/spring-plugin' }
    maven { url "https://repo.spring.io/libs-spring-framewrok-build" }
	}
	dependencies {
		classpath("io.spring.gradle:propdeps-plugin:0.0.9.RELEASE")
		classpath("org.asciidoctor:asciidoctorj-pdf:1.5.0-alpha.16")
	}
}
```

##### 查看`import-into-idea.md`

1. Precompile `spring-oxm` with `./gradlew :spring-oxm:compileTestJava`
2. Import into IntelliJ (File -> New -> Project from Existing Sources -> Navigate to directory -> Select build.gradle)
3. When prompted exclude the `spring-aspects` module (or after the import via File-> Project Structure -> Modules)
4. Code away



### 问题及解决

#### 预编译过程中可能会出现`JDK tools`无法被`Kotlin`找到的问题

`Kotlin could not find the required JDK tools in the Java installation '/Library/Internet Plug-Ins/JavaAppletPlugin.plugin/Contents/Home' used by Gradle. Make sure Gradle is running on a JDK, not JRE.`

1. 检查下当前的java环境 `/usr/libexec/java_home -V`

   ![截屏2021-07-11 下午12.01.06](https://www.caijy.top//%E6%88%AA%E5%B1%8F2021-07-11%20%E4%B8%8B%E5%8D%8812.01.06.png)

   看到有两个JDK 路径，打开发现`/Library/Internet Plug-Ins/JavaAppletPlugin.plugin/Contents/Home`下的JDK确实没有jre，而通过dmg安装的`/Library/Java/JavaVirtualMachines/jdk1.8.0_202.jdk/Contents/Home`路径下有

2. 在gradle.properties中配置org.gradle.java.home，指定gradle编译使用的java环境目录

   ```java
   org.gradle.java.home=/Library/Java/JavaVirtualMachines/jdk1.8.0_202.jdk/Contents/Home

#### Kotlin版本太低

`Kotlin: API version 1.1 is no longer supported; please, use version 1.2 or greater.`

1. 修改`build.gradle` 的`apiVersion` 和`languageVersion`

   ```java
   compileKotlin {
   		kotlinOptions {
   			jvmTarget = "1.8"
   			freeCompilerArgs = ["-Xjsr305=strict"]
   			apiVersion = "1.2"
   			languageVersion = "1.2"
   		}
   	}
   ```

#### 运行单元测试，总是执行gradle打包命令

这两处选择IDEA<img src="https://www.caijy.top//1.png" alt="1" style="zoom:50%;" />

