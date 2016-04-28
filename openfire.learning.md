#Openfire服务端源代码开发配置指南

##目标
1.下载源码
2.配置Eclipse项目
3.ANT编译项目
4.运行Openfire项目


#[一]、下载源码

打开网址：http://www.igniterealtime.org/downloads/source.jsp 选择目前最新版本 openfire_src 下载。


#[二]、配置Eclipse项目


1、把下载好的 openfire_src.zip 压缩包直接解压到Eclipse的工作目录，结构如下：

	openfire_src\
		build\
		documentation\
		resources\
		src\
		work\
		changelog.html
		LICENSE.html
		README.html


2、把 openfire_src\build\eclipse 目录下的文件夹setting、文件classpth、文件project全部copy到 openfire_src\ 目录下，修改成Eclipse工程配置文件格式：.classpath .project .settings
（Window下可以用命令提示行 rem classpath .classpath 来改名，Liunx下可以用mv命令重命名）

3、然后打开Eclipse，选择 File –> Import… –> Existing Projects into Workspace  选择 openfire_src 导入即可。


4、项目导入后编译错误的解决

目录：/openfire_src/src/plugins/clustering/src/java 报错是因为缺少coherence相应的包：coherence.jar、coherence-work.jar。
具体信息可以参考：openfire_src/src/plugins/clustering/lib/README.TXT 中的说明。
可以从其官网下载：http://www.oracle.com/technetwork/middleware/coherence/downloads/index.html。
下载jar包后copy到目录：/openfire_src/src/plugins/clustering/lib 下，把这两个jar 添加到classpath中，直接在/openfire_src/.classpath 文件中添加如下内容即可：
 
	<classpathentry kind="lib" path="src/plugins/clustering/lib/coherence.jar"/>
	<classpathentry kind="lib" path="src/plugins/clustering/lib/coherence-work.jar"/>
 
目录：/openfire_src/src/plugins/sip/src/java 报错是因为 SipCommRouter.java和SipManager.java 这两个类没有实现抽象方法和完成异常处理，最简单的解决办法是利用Eclipse自动修复功能进行修复即可。



#[三]、ANT编译项目

Eclipse已经集成了Ant，所以我们只需要在 /openfire_src/build/build.xml 文件右击，选择Run As –> Ant Build 即可完成编译，编程成功后，会在/openfire_src/的跟目录下生成两个新的文件夹：target 和 work 。


#[四]、运行Openfire项目

1、配置资源文件

在Build Path配置中把  /openfire_src/src/i18n 、/openfire_src/src/resources/jar 、/openfire_src/build/lib/dist 文件夹添加到 Source 中

2、配置启动参数

选择Run –> Run Configurations… 左边的Java Application，单击右键，选择 New:
把默认name：New_configuration 修改成：ServerStarter
选中Main选项卡，点击Browse按钮选择 openfire_src 项目;单击Search 按钮输入：ServerStarter 自动过滤后选择：ServerStarter – org.jivesoftware.openfire.starter：

选中Arguments选项卡，在VM arguments中填入：
-DopenfireHome=“${workspace_loc:openfire_src}/target/openfire”
注意：项目路径 ${workspace_loc:openfire_src}

选中Common选项卡，将Debug和Run打钩（方便之后快速启动），然后点击apply，再点击run：


3、运行后控制台日志如下：

	Openfire 4.0.0 [Mar 12, 2015 22:33:33 PM]
	Admin console listening at http://127.0.0.1:9090

 4、浏览器中输入地址： http://127.0.0.1:9090 回车：

	看到界面表示Openfire的源码配置、导入、编译、启动已经圆满成功了。


[五]、参考
	https://community.igniterealtime.org/docs/DOC-1020
