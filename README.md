## 什么是sonarqube？

​	随着项目团队规模日益壮大，项目代码量也越来越多。且不说团队成员编码水平层次不齐，即便是大佬，也难免因为代码量的增加和任务的繁重而忽略代码的质量，最终的问题便是bug的增多和代码债务的堆积。因此，采用了自动化代码review的工具，**SonarQube**。

## 版本选择

因为组内开发都是用 Java8的问题，sonar支持最高版本是7.9 综合部署和软件的综合考虑选择了 7.6

![sonar7.6](https://tva1.sinaimg.cn/large/008vxvgGly1h73gr73t89j31gm05kab2.jpg)

![如何找到sonar 历史版本的下载url](https://tva1.sinaimg.cn/large/008vxvgGly1h73grjtxubj31dg0u0gq6.jpg)

## 我遇到的问题

### sonar 启动异常

#### fail to start soarQube

```sh
luojiajun@qa-test-webserver-12:/srv/sonarqube-7.6/bin/linux-x86-64$ ./sonar.sh start
Starting SonarQube...
Failed to start SonarQube.
```

**解决方案**:

**一定请到 /sonarqube-7.6/logs 到这个路径下面看日志报错**，以下是我接入过程中发现的问题以及如何解决的。

1. JVM没有找到如果是的话 -> 修改 sonarqube-7.6/conf/wrapper.conf 文件 指定JVM路径。

   ```sh
   # Path to JVM executable. By default it must be available in PATH.
   # Can be an absolute path, for example:
   #wrapper.java.command=/path/to/my/jdk/bin/java
   #下方是 替换 后的java地址
   wrapper.java.command=/srv/jdk/jdk1.8.0_311/bin/java 
   ```

2. Process [es] is stopped

```
Running SonarQube...
wrapper  | --> Wrapper Started as Console wrapper  | Launching a JVM...
jvm 1    | Wrapper (Version 3.2.3) http://wrapper.tanukisoftware.org
jvm 1    |   Copyright 1999-2006 Tanuki Software, Inc.  All Rights Reserved.
jvm 1    |
jvm 1    | 2017.12.14 18:45:28 INFO  app[][o.s.a.AppFileSystem] Cleaning or creating temp directory /etc/sonarqube/temp
jvm 1    | 2017.12.14 18:45:28 INFO  app[][o.s.a.es.EsSettings] Elasticsearch listening on /127.0.0.1:9001
jvm 1    | 2017.12.14 18:45:29 INFO  app[][o.s.a.p.ProcessLauncherImpl] Launch process[[key='es', ipcIndex=1, logFilenamePrefix=es]] from     [/etc/sonarqube/elasticsearch]: /etc/sonarqube/elasticsearch/bin/elasticsearch -Epath.conf=/etc/sonarqube/temp/conf/es
jvm 1    | 2017.12.14 18:45:29 INFO  app[][o.s.a.SchedulerImpl] Waiting for Elasticsearch to be up and running
jvm 1    | 2017.12.14 18:45:29 WARN  app[][o.s.a.p.AbstractProcessMonitor] Process exited with exit value [es]: 137
jvm 1    | 2017.12.14 18:45:29 INFO  app[][o.s.a.SchedulerImpl] Process [es] is stopped
jvm 1    | 2017.12.14 18:45:29 INFO  app[][o.s.a.SchedulerImpl] SonarQube is stopped
jvm 1    | 2017.12.14 18:45:29 INFO  app[][o.e.p.PluginsService] no modules loaded
jvm 1    | 2017.12.14 18:45:29 INFO  app[][o.e.p.PluginsService] loaded plugin [org.elasticsearch.transport.Netty4Plugin]
jvm 1    | 2017.12.14 18:45:30 WARN  app[][i.n.u.i.MacAddressUtil] Failed to find a usable hardware address from the network interfaces; using random bytes: 05:2b:7f:2f:de:90:ca:4a
wrapper  | <-- Wrapper Stopped
```

**非常关键！ 非常关键！ 非常关键！**
**启动之前先使用 chown 命令将sonarqube-7.6及其子目录授权给一个非root的用户，sonarqube及其es等软件禁止 root账户启动，因此需要切换一个非root账户，授权的用户需要有bin目录及其子目录的读取和可执行的权限。**

### 插件相关问题

#### Error while downloading plugin 'l10nzh' with version '9.6'. No compatible plugin found.

因为低于7.9的版本不支持 直接在sonarqube 中直接下载安装插件，所以我们所有的第三方插件需要手动导入到 /opt/sonarqube/extensions/plugins/目录下

我当前用的是7.6在 以 导入汉化包为例子：

jar包下载地址是 https://github.com/xuhuisheng/sonar-l10n-zh/releases/tag/sonar-l10n-zh-plugin-1.26.

重启一下即可

```sh
luojiajun@qa-test-webserver-12:/srv/sonarqube-7.6/bin/linux-x86-64$ sh sonar.sh restat
Stopping SonarQube...
Waiting for SonarQube to exit...
Stopped SonarQube.
```

![汉化后的效果](https://tva1.sinaimg.cn/large/008vxvgGly1h73sehxjvpj316q0u077p.jpg)

#### 如何安装配置 Pmd?

1. https://github.com/JacksonZhangHuaQuan/sonar-pmd-plugin mvn clean install 将jar文件复制到extensions/plugins目录下即可。

2. 【推荐这个！不需要配置，直接导入到 extensions/plugins 目录下即可直接使用！】 https://github.com/ZuccRoger/sonar-pmd-plugin/blob/master/sonar-pmd-plugin-3.2.1.jar

#### 配置 sonar-gitlab-plugin

GitLab 用户令牌生成
GitLab 安装完成后，我们需要需要根据 3.1.3 中的要求，生成用户令牌。

具体步骤如下：

登录具有管理员权限的账号（一定要是管理员身份）

访问地址 http://gitlab.example.com/profile/personal_access_tokens 进入令牌生成页面

输入令牌名称，勾选 api 、read_user、sudo 权限，点击【创建】按钮，即可生成用户令牌


![gitlab 创建api token](https://tva1.sinaimg.cn/large/008vxvgGly1h74sguo0ffj31p20u00wd.jpg)

若 sonar-gitlab-plugin 成功安装，则在 SonarQube 的配置页面可以见到如下页面：

具体路径为：登录 admin 账号 -> 点击顶部导航栏【配置】按钮 -> 点击【通用设置】中的【GitLab】选项卡

在该页面中，需要修改以下参数的值：

| 参数名称           | 参数标识                 | 目标值                            |
| ------------------ | ------------------------ | --------------------------------- |
| GitLab url         | sonar.gitlab.url         | http://gitlab.example.com         |
| GitLab User Token  | sonar.gitlab.user_token  | GitLab 的用户令牌，获取方式如上图 |
| GitLab API version | sonar.gitlab.api_version | v4                                |

注：上述参数中，GitLab User Token 是核心，设置错误的 token，将直接影响到 SonarQube 与 GitLab 的协作。特别要注意的是，生成该 token 的用户一定要具备管理员权限，否则 SonarQube 无法对 GitLab 上的所有项目进行评论。

## 推荐安装的插件

1. chinese（是个汉化包帮助小白更快上手sonarqube）
2. Pmd https://github.com/jborgers/sonar-pmd/releases
3. sonar-gitlab-plugin

## 使用效果截图

![所有的项目列表](https://tva1.sinaimg.cn/large/008vxvgGly1h74tk3nqwpj319j0u0dib.jpg)

![单个项目总览](https://tva1.sinaimg.cn/large/008vxvgGly1h74tkijjhnj31be0u0jup.jpg)

![现存的代码坏味道截图](https://tva1.sinaimg.cn/large/008vxvgGly1h74tkx0em6j31ch0u0wi9.jpg)

## 参考资料

喝水不忘挖井人，感谢一下以下各位up主的无私奉献，帮助我更快地能够将sonar部署完成！

1. [SonarQube系列一、Linux安装与部署](https://www.cnblogs.com/7tiny/p/11269774.html)
2. [SonarQube - 中文插件安装](https://blog.csdn.net/gw5205566/article/details/103387117)
3. [pmd3.0.0集成p3c](https://blog.csdn.net/Zhang_Jackson/article/details/87969174)
4. [基于 GitLab+SonarQube 搭建自动化代码检测平台](https://blog.csdn.net/magicpenta/article/details/106880267)