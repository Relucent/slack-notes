实现yum源的创建方法
===

#1.安装createrepo工具

	yum install createrepo

 
#2.创建一个目录,用于存放rpm包

	mkdir /home/download
	cp xxx.rpm /home/download
	createrepo /home/download

执行createrepo命令后,在该目录自动搜索rpm文件,并创建repodata目录

 
3.修改yum.conf配置文件
	[main]
	cachedir=/var/cache/yum/$basearch/$releasever
	# 0删除下载文件,1不删除下载文件
	keepcache=0
	debuglevel=2
	logfile=/var/log/yum.log
	exactarch=1
	obsoletes=1
	# 0不启用验证,1启用验证
	gpgcheck=1
	plugins=1
	installonly_limit=5

 
## 4.修改下载源定义文件x.repo
vim /etc/yum.repos.d/mytest.repo

	[mytest]
	name=mytest
	#路径要设置成repodata所在目录的上级目录名
	baseurl=file:///home/download
	#1启用该配置，0不启用                                   
	enabled=1                                    
	#关闭验证      
	gpgcheck=0                                    

 
##5.安装测试

	yum install Percona-Server-server-55 Percona-Server-client-55

 
## 6.yum命令列表
1. yum clean packages 清除缓存中的软件包文件  
2. yum clean headers 清除缓存中的软件包文件头信息  
3. yum clean metadata 清除缓存中的描述信息  
4. yum clean dbcache 清除sqlite格式的描述信息
5. yum clean all 清除缓存中的所有数据信息
6. yum list all  列出所有软件包
7. yum list installed  列出所有已经安装的软件包
8. yum list available  列出可安装的软件包
9. yum list updates  列出所有可以更新的软件包
10. yum list extras 显示额外的软件包
11. yum list obsoletes  显示已经被淘汰的软件包
12. yum list recent  显示近期的软件包
13. yum makecache 把服务器的包信息下载到本地电脑缓存起来
