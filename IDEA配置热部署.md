# IDEA配置热部署

## 一、没有布置热部署时的痛点

在使用IDEA进行Java开发（尤其是Spring boot项目）时，最常见的问题：

- **每次修改代码后，都需要重新启动项目**
  - 改了一行Controller逻辑，就要重新运行，得等十几秒甚至更久才能重新访问，测试功能。
  - 启动项目又要跑数据库连接、加载Bean、初始化缓存......

- **开发节奏被严重打断**
  - 改完代码还要等待启动，思路容易被打断
  - 多次启动还会消耗内存，风扇狂转



## 二、为什么要使用热部署

**热部署能够很好的解决上述的痛点**

> 热部署 = 不重启服务器也能让代码变更生效。

**热部署的好处**

- 大幅提升开发效率

> 修改代码 -> 保存 -> 自动更新 -> 浏览器刷新立即生效!

- 支持快速迭代和调试

> 尤其是前后端联调、逻辑修复，不用每次都Rerun

- 更流畅的开发体验

> 不需要频繁重启Spring boot项目，对电脑内存和CPU的负担也更小



## 三、实现热部署的常见方式



| 热部署方式                | 说明                                | 适用情况       |
| ------------------------- | ----------------------------------- | -------------- |
| spring-boot-devtools      | 官方推荐方式，自动重启但更快        | 通用           |
| JRebel                    | 商业工具，真正的热加载class，不重启 | 企业、专业开发 |
| DCEVM + Hotswap Agent     | 高级JVM插件                         | 高级用户       |
| IDEA的自动编译 + 自动构建 | 开发时轻量修改                      | 小修改         |



## 四、如何用`spring-boot-devtools`实现热部署？

`spring-boot-devtools`是Spring boot 官方提供的热部署工具，支持在开发时自动重启应用（比手动重启方便），实现**保存即生效**的效果。

**在pom.xml中的配置**

添加`spring-boot-devtools`依赖

```xml
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-devtools</artifactId>
                <scope>runtime</scope> <!--加上<scope>runtime</scope>，可以避免在项目打包发布时被引入-->
                <optional>true</optional> <!--将依赖标记为可选，避免传递给其他模块-->
            </dependency>
```

> 因为`spring-boot-devtools`是由spring boot官方提供的，所以不需要指定版本号。
>
> 版本号由Spring boot的父依赖`spring-boot-dependencies`自动统一管理（即“依赖版本管理BOM”机制）

**idea的配置**

- 设置一：

File->Settings->Build,Execution,Deployment->Compiler

勾选：`Build project automatically`

<img src="images\image-20250511215950342.png" alt="image-20250511215950342" style="zoom: 50%;" />



- 设置二：

在旧版本的IDEA中（2021.2及以下）：

按快捷键：ctrl + alt + shift + /

在打开的弹窗中选择Registry

勾选：compiler.automake.allow.when.app.running



在新版本的IDEA中（2021.2及以上）

File->Settings->Advanced Settings

勾选：`Allow auto-make to start even if developed application is currently running`

<img src="images\image-20250511220315517.png" alt="image-20250511220315517" style="zoom:50%;" />



**application.yml文件配置**

```yaml
 spring: 
      devtools:
        restart:
          enabled: true  # 设置在该启动类开启热部署
          additional-paths: src/main/java  # 需要重启的
```















