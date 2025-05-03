# 项目初始部署maven依赖报错问题解决

**一、出现的报错问题如下：**

`Could not find artifact org.activiti:activiti-spring-boot-starter:pom:7.10.0 in aliyunmaven (https://maven.aliyun.com/repository/public)`

**二、解决问题的过程**

- 首先，我使用的仓库和该项目该有的依赖都已经从up提供的资料里拷贝到自己的本地了。
- 根据报错信息的解释就是：这个依赖无法在阿里云的maven公共仓库中找到。
- 我去[Maven Repository](https://mvnrepository.com/)这个公共库里查询`org.activiti:activiti-spring-boot-starter:pom:7.10.0`这个依赖。
- 在搜索到的依赖文档里发现下方有提示信息：**Note**: this artifact is located at **[Alfresco](https://mvnrepository.com/repos/alfresco-public)** repository (https://artifacts.alfresco.com/nexus/content/repositories/public/)

- 起初我认为只是需要将这个仓库的URL加到maven的settings.xml文件中就可以了。后来发现依旧报错。
- 接着去检查pom.xml文件，发现了以下提示（up写好的提示）

```xml
 <!--如果activiti依赖下载不了，可以配置如下地址进行下载-->
<repositories>
    <repository>
        <id>activiti-releases</id>
     <url>https://artifacts.alfresco.com/nexus/content/repositories/activiti-releases</url>
     </repository>
</repositories>
```

- 我把这个加上去后重新刷新maven，发现依旧报同样的错误。那就继续查找解决方案，排错。
- 最后发现在我的maven的settings.xml文件里配置的阿里云镜像源的写法是：

![image-20250503212049014](images\image-20250503212049014.png)

- 在`<mirrorOf>`标签里如果写的是`*`，表示**所有仓库请求**（包括你在 `pom.xml` 里显式写的 `<repositories>`）都要被这个镜像拦截，然后改为走这个 `<url>`。这样我的maven就无法扫描到本地配置好的仓库，需要将`<mirrorOf>`标签里的内容更改为`central`，意思是只代理Maven的中央仓库。然后如果在项目的pom.xml文件里加上`<repositories>`标签来引入其它的仓库进行依赖的下载。

- 所以最后把maven配置文件settings.xml里的镜像源的`<mirrorOf>`改为`<mirrorOf>central</mirrorOf>`后，重新刷新maven即可。

![image-20250503212218961](images\image-20250503212218961.png)

**细节补充**

- 在联网下载依赖产生错误，无法正确下载依赖后，往往会在配置好的仓库里，如本地仓库的相应的依赖的包中产生以`.lastUpdated`结尾的文件，我们需要将这些文件删除，否则会阻止Maven使用该目录，导致后续无法正确下载依赖。