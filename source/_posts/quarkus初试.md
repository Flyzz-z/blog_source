---
title: Quarkus初试
tags: [java,云原生]
index_img: https://blog-img-1304606000.file.myqcloud.com/img/20220131000458.png
---



# Quarkus初试

## Quarkus是什么

 引用Red Hat官网的话

> Quarkus 是一个为 Java 虚拟机（JVM）和原生编译而设计的全堆栈 Kubernetes 原生 [Java 框架](https://www.redhat.com/zh/topics/cloud-native-apps/what-is-a-Java-framework)，用于专门针对容器优化 Java，并使其成为[无服务器](https://www.redhat.com/zh/topics/cloud-native-apps/what-is-serverless)、[云](https://www.redhat.com/zh/topics/cloud)和 [Kubernetes](https://www.redhat.com/zh/topics/containers/what-is-kubernetes) 环境的高效平台。

  看它的简介，它支持现有常用的java标准，库，框架，另外还支持GraalVM用于进行原生编译。接下来就记录跟着官网的get started在windows操作系统初次尝试Quarkus的过程。

## 第一个Quarkus程序

[官网教程](https://quarkus.io/guides/getting-started)

  这个跟着官网做就行，过程中也没有出现什么问题。

## 原生可运行Java

### 生成windos可执行文件

1. 首先要安装GrallVM,我安装了Oracle GrallVM CE.之后配置环境变量`GRAALVM_HOME`

2. 使用gu安装native-image. `${GRAALVM_HOME}/bin/gu install native-image`

3. 生成可执行文件（windows提前安装VS Build tools）。

   ```bash
   cmd /c 'call "C:\Program Files (x86)\Microsoft Visual Studio\2017\BuildTools\VC\Auxiliary\Build\vcvars64.bat" && mvn package -Pnative'
   ```

4. 在target文件夹中就可以看见生成的xx-runner.exe了，`./xxx.exe`即可运行。

### 生成Linux可执行文件

  如果本地没有安装GrallVM,或者不是Linux平台，则可以选择借助容器来进行原生编译生成Linux可执行文件。

  借助docker，同样windos下需要vs build环境:

```
./mvnw package -Pnative -Dquarkus.native.container-build=true -Dquarkus.native.container-runtime=docker
```

  这里我报了一个错，根据官网用`-Dquarkus.native.remote-container-build=true`替换`-Dquarkus.native.container-build=true`后成功。感觉电脑没买好，8g内存直接跑满。完成后target文件夹中可以看见生成的`xxx-runner`。

### 生成镜像

   生成容器镜像，要保证容器应用在运行，这里的话即docker运行中。

```
 ./mvnw package -Pnative -Dquarkus.native.container-build=true -Dquarkus.container-image.build=true
```

  这样会先构建Linux可执行文件，之后为它构建镜像。如果已经提前生成了可执行文件，则可以：

```bash
docker build -f src/main/docker/Dockerfile.native -t quarkus-quickstart/getting-started .
```

#### Dockerfile.native简要分析

```dockerfile
1. FROM registry.access.redhat.com/ubi8/ubi-minimal:8.4
2. WORKDIR /work/
3. RUN chown 1001 /work \
    && chmod "g+rwX" /work \
    && chown 1001:root /work
4. COPY --chown=1001:root target/*-runner /work/application

5. EXPOSE 8080
6. USER 1001

7. CMD ["./application", "-Dquarkus.http.host=0.0.0.0"]
```

1. 以`ubi-minimal:8.4`为基础镜像构建，Red Hat提供的一个可用于生产的镜像。

2. 指定work为工作目录。

3. 为目录指定用户和权限。“g+rwX"表示组权限为读写执行。

4. 拷贝可执行文件到工作目录。

5. 暴露8080端口。

6. 切换用户1001。

7. 运行applicatin并暴露8080端口，启动容器。

   

完成后就可以在容器中看见生成的镜像了，运行容器：

```bash
docker run -i --rm -p 8080:8080 quarkus-quickstart/getting-started
```



## 杂项

### powershell和cmd中的环境变量

CMD:

```
set  :查看所有环境变量
set xxx :查看特定环境变量
临时修改设置：
set xxx=xxx
SET XX=%XX%yy  在XX变量后追加yy
SET xx= 设置空值即删除
永久设置：
setx /m
```

PowerShell:

```
ls env:   查看所有环境变量
$env:xxx 查看特定环境变量
ls env:xx  可使用*通配符
临时设置修改：
$env:xxx=xxx
$env:xxx+=";c:\temp"  追加
删除
del env:xxx
```

### docker更新后再次犯病

命令行管理员身份执行：`netsh winsock reset`

### 编译可执行文件时报错

**Native-image building on Windows currently only supports target architecture: AMD64 (?? unsupported)**

  开始菜单 => Visual Studio  Installer => 修改生成工具 => 语言包勾选英文，去掉中文。

成功解决。



