---
title: jenkins Rest Api试用
date: 2019-05-29 10:29:26
tags: jenkins
---

# 安装

使用Jenkins Rest Api需要安装Jenkins服务，windows版本的可以直接安装为本地service。Jenkins的默认端口为8080，如果需要，可以在安装路径下找到jenkins.xml文件，修改`--httpPort`参数为指定端口。

安装完成后，jenkins服务自动启动，访问`localhost:[port]`后按照提示完成初始配置，包括安装推荐插件。如果遇到插件安装失败的情况，大多是因为插件由于网络原因不可达，需要升级插件的站点，一般都可以解决。升级站点的可选地址包括但不局限于：`http://ftp.tsukuba.wide.ad.jp/software/jenkins/updates/current/update-center.json`，可以根据需要替换。

# 配置

本次案例主要实现通过Rest Api实现项目远程部署等操作，所以需要如下组件：

- 用户凭证

- 全局工具配置
  - Maven
  - JDK
  - Git
- 环境配置
  - 邮件通知
- 项目配置
  - 一个demo项目，托管到github
  - git
  - build
  - 构建后操作

## 用户凭证

访问`[Jenkins_url]/credentials`，e.g.，`http://localhost:11111/credentials`。

配置github和tomcat凭证，后序pull代码和发布到tomcat需要使用到。

## 全局工具配置

访问`[Jenkins_url]/configureTools`，e.g.，`http://localhost:11111/configureTools`，根据页面提示配置Maven，Git，JDK。

## 环境配置

访问`[Jenkins_url]/configure`，e.g.，`http://localhost:11111/configure`进入系统配置界面，填写邮件通知的SMTP服务器。Jenkins提供了测试按钮来测试邮件配置是否成功。

## 项目配置

首先需要新建一个构建maven项目的job，接着进行相应的环境配置，访问`[Jenkins_url]/job/[jobname]/configure`，e.g.，`http://localhost:11111/job/maven-hello-world/configure`进入目标页面。

### git

在General签页源码管理项目中选择github仓库和凭证信息，指定需要关联的分支。

### build

在Build签页选择构建时使用的pom文件，键入构建的命令和选项，e.g.，`clean package -Dmaven.test.skip=true`。

### 构建后操作

构建完成之后，可以将代码打包发布到指定的tomcat容器（需要安装deploy to container插件），填写war包路径，这里指的是相对于Jenkins工作空间根路径的相对路径、访问路径、指定配置好的tomcat及其凭证。

# 测试

Jenkins的官方文档中有这样一句话：

> Many objects of Jenkins provide the remote access API. They are available at `/.../api/` where "..." portion is the object for which you'd like to access.

很多jenkins对象都提供了rest api，可以通过在url后追加`/api/`的方式来查看提供的api，更直观地操作是在jenkins界面的右下角处，如果有[Rest Api]()字样，直接点击跳转即可。

本次案例中使用的是`http://localhost:11111/job/maven-hello-world/`。

## Build

### case1

```html
Method: POST
Url：http://localhost:11111/job/maven-hello-world/build
Response：
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html;charset=utf-8"/>
        <title>Error 403 No valid crumb was included in the request</title>
    </head>
    <body>
        <h2>HTTP ERROR 403</h2>
        <p>Problem accessing /job/maven-hello-world/build. Reason:
			<pre>    No valid crumb was included in the request</pre>
        </p>
        <hr>
        <a href="http://eclipse.org/jetty">Powered by Jetty:// 9.4.z-SNAPSHOT</a>
        <hr/>
    </body>
</html>
```

提示需要`crumb`信息，这个信息是用来防止跨站点请求伪造的，解决此问题有两种方案：

1. 关闭CSRF安全策略，不可取
2. 请求一个crumb，<font color="red">需要加上jenkins用户名密码，其他请求也是</font>

#### subcase，get crumb

```json
// GET请求crumb
GET http://localhost:11111/crumbIssuer/api/json
// 返回json
{
    "_class": "hudson.security.csrf.DefaultCrumbIssuer",
    "crumb": "7325742b8283ee86bd78d1cb82765a53",
    "crumbRequestField": "Jenkins-Crumb"
}
```

### 带crumb的case1

在case1的基础上，在请求头加上`"Jenkins-Crumb":"7325742b8283ee86bd78d1cb82765a53"`，再次发送请求，返回`Status : 201 Created`，表示构建进入调度队列，jenkins会调度执行。

值得注意的是，build任务的响应头中包括了一个位置信息：

```
Date →Wed, 29 May 2019 03:31:47 GMT
X-Content-Type-Options →nosniff
Location →http://localhost:11111/queue/item/16/
Content-Length →0
Server →Jetty(9.4.z-SNAPSHOT)
```

Location指明了这次build触发的queueId：16

## 获取构建详情

```html
Method: GET/POST
Url: http://localhost:11111/job/maven-hello-world/16/api/xml
Response:
<mavenModuleSetBuild _class='hudson.maven.MavenModuleSetBuild'>
    <action _class='hudson.model.CauseAction'>
        <cause _class='hudson.model.Cause$UserIdCause'>
            <shortDescription>Started by user admin</shortDescription>
            <userId>admin</userId>
            <userName>admin</userName>
        </cause>
    </action>
    <action _class='hudson.plugins.git.util.BuildData'>
        <buildsByBranchName>
            <refsremotesoriginmaster _class='hudson.plugins.git.util.Build'>
                <buildNumber>16</buildNumber>
                <marked>
                    <SHA1>b5e89b3eba13533ab141e3e2a71f768613f8ec63</SHA1>
                    <branch>
                        <SHA1>b5e89b3eba13533ab141e3e2a71f768613f8ec63</SHA1>
                        <name>refs/remotes/origin/master</name>
                    </branch>
                </marked>
                <revision>
                    <SHA1>b5e89b3eba13533ab141e3e2a71f768613f8ec63</SHA1>
                    <branch>
                        <SHA1>b5e89b3eba13533ab141e3e2a71f768613f8ec63</SHA1>
                        <name>refs/remotes/origin/master</name>
                    </branch>
                </revision>
            </refsremotesoriginmaster>
        </buildsByBranchName>
        <lastBuiltRevision>
            <SHA1>b5e89b3eba13533ab141e3e2a71f768613f8ec63</SHA1>
            <branch>
                <SHA1>b5e89b3eba13533ab141e3e2a71f768613f8ec63</SHA1>
                <name>refs/remotes/origin/master</name>
            </branch>
        </lastBuiltRevision>
        <remoteUrl>https://github.com/nywitness/springmvc-demo.git</remoteUrl>
        <scmName></scmName>
    </action>
    <action _class='hudson.plugins.git.GitTagAction'></action>
    <action></action>
    <action _class='hudson.maven.reporters.MavenAggregatedArtifactRecord'></action>
    <action></action>
    <action></action>
    <action></action>
    <building>false</building>
    <displayName>#16</displayName>
    <duration>26345</duration>
    <estimatedDuration>29204</estimatedDuration>
    <fullDisplayName>maven-hello-world #16</fullDisplayName>
    <id>16</id>
    <keepLog>false</keepLog>
    <number>16</number>
    <queueId>16</queueId>
    <result>SUCCESS</result>
    <timestamp>1559100714301</timestamp>
    <url>http://localhost:11111/job/maven-hello-world/16/</url>
    <builtOn></builtOn>
    <changeSet _class='hudson.plugins.git.GitChangeSetList'>
        <kind>git</kind>
    </changeSet>
    <mavenArtifacts></mavenArtifacts>
    <mavenVersionUsed>3.6.1</mavenVersionUsed>
</mavenModuleSetBuild>
```

可以看到构建结果`<result>SUCCESS</result>`，表示构建成功，再通过tomcat访问，看看是否发布成功即可。如果没有成功，需要查看构建后操作中配置的tomcat信息是否正确，并且需要在Build之前就已经启动tomcat，否则会报错。

