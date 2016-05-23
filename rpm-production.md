# RPM包的制作

##前言

按照其软件包的格式来划分，常见的Linux发行版主要可以分为两类，**类ReadHat系列**和**类Debian系列**，这两类系统分别提供了自己的软件包管理系统和相应的工具。  

类RedHat系统中软件包的后缀是rpm，提供了同名的rpm命令来安装、卸载、升级rpm软件包；  
类Debian系统中软件包的后缀是deb，同样提供了dpkg命令来对后缀是deb

rpm的全称是Redhat Package Manager，常见的使用rpm软件包的系统主要有Fedora、CentOS、openSUSE、SUSE企业版、PCLinuxOS以及Mandriva Linux、Mageia等。  
使用deb软件包后缀的类Debian系统最常见的有Debian、Ubuntu、Finnix等。 

无论是rpm命令还是dpkg命令在安装软件包时都存在软件包依赖关系问题。   

**Fedora/CentOS 系统** 提供了 __yum__ 来自动解决软件包的安装依赖；  
**openSUSE/SUSE 系统** 提供了 __zypper__ 来自动解决软件包的安装依赖； 
**Mandriva Linux/Mageia 系统**  提供了 __urpmi__ 来自动解决软件包的安装依赖；  
**Debian/Ubuntu 系统** 提供了 __apt-*__ 命令。

这些工具本质上最终还是调用了 **rpm** (或者**dpkg**)来进行安装，但是在安装前会自动帮用户解决了软件包的安装依赖。

从软件运行的结构来说，一个软件主要可以分为三个部分：可执行程序、配置文件和动态库。  
当然还有可能会有相关文档、手册、供二次开发用的头文件以及一些示例程序等等。其他部分都是可选的，只有可执行文件是必须的。

制作rpm软件包，常用的方法是使用 **rpmbuild** 这个命令行工具。


##安装rpmbuild工具

可以执行yum命令安装工具包(能够联网的情况):

    yum install rpm* rpm-build rpmdev*

或者使用rpm包直接接安装(有rpm-build安装包的情况)：

    rpm -ivh rpm-build-4.8.0-37.el6.x86_64.rpm

如果还需要安装rpmdevtools工具，稍微麻烦一些，需要查看依赖自己来安装。


##rpmbuid打包目录

rpm打包目录有一些严格的层次上的要求。

rpm的版本<=4.4.x，rpmbuid工具其默认的工作路径是 **/usr/src/redhat**因为权限的问题，普通用户不能制作rpm包，制作rpm软件包时必须切换到 **root** 身份才可以。

rpm从4.5.x版本开始，将rpmbuid的默认工作路径移动到用户家目录下的rpmbuild目录里，即 **$HOME/rpmbuild** ，并且推荐用户在制作rpm软件包时尽量不要以root身份进行操作。

rpmbuild默认工作路径的确定，通常由在 __/usr/lib/rpm/macros__  这个文件里的一个叫做 **%\_topdir** 的宏变量来定义。如果用户想更改这个目录名，rpm官方并不推荐直接更改这个目录，而是在用户家目录下建立一个名为 **.rpmmacros** 的隐藏文件(Linux下隐藏文件,前面的点不能少)，然后在里面重新定义 **%\_topdir**，指向一个新的目录名。这样就可以满足某些用户的差异化需求了。通常情况下.rpmmacros文件里一般只有一行内容，比如：

    %_topdir    $HOME/myrpmbuildenv 


在%_topdir目录下一般需要建立6个目录  
<table>
 <tr>
  <th>目录名</th>
  <th>说明</th>
  <th>macros中的宏名</th>
 </tr>
 <tr>
  <td>BUILD</td>
  <td>编译rpm包的临时目录 </td>
  <td>%_builddir </td>
 </tr>
 <tr>
  <td>BUILDROOT </td>
  <td>编译后生成的软件临时安装目录 </td>
  <td>%_buildrootdir </td>
 </tr>
 <tr>
  <td>RPMS </td>
  <td>最终生成的可安装rpm包的所在目录</td>
  <td> %_rpmdir  </td>
 </tr>
 <tr>
  <td>SOURCES </td>
  <td>所有源代码和补丁文件的存放目录 </td>
  <td>%_sourcedir </td>
 </tr>
 <tr>
  <td>SPECS </td>
  <td>存放SPEC文件的目录</td>
  <td> %_specdir </td>
 </tr>
 <tr>
  <td>SRPMS </td>
  <td>软件最终的rpm源码格式存放路径</td>
  <td> %_srcrpmdir  </td>
 </tr>
</table>

如果有安装rpmdevtools，可以使用 rpmdev-setuptree命令在当前用户home/rpmbuild目录里自动建立上述目录。


### SPEC文件详解

####**注：** 
>liunx环境可以使用vi来编辑spec文件，windows环境可以可以使用sublime编辑spec文件。
>需要注意的是，绝对不能使用记事本来编辑，这是因为windows的默认换行符是 `\r\n`,而 liunx 的换行符是`\n`。如果spec文件中包含多余的`\r`会导致rpm包创建失败。

文件使用liunx换行符风格，也就是换行是 `\n`，所以建议使用 VI 来编辑spec文件文件，因为如果

当打包目录建立好之后，将所有用于生成rpm包的源代码、shell脚本、配置文件都拷贝到SOURCES目录里，注意通常情况下源码的压缩格式都为*.tar.gz格式。然后，将最最最重要的SPEC文件，命名格式一般是“软件名-版本.spec”的形式，将其拷贝到SPECS目录下，切换到该目录下执行：  

    rpmbuild -bb 软件名-版本.spec 

如果系统有rpmdevtools工具，可以用rpmdev-newspec -o Name-version.spec命令来生成SPEC文件的模板，然后进行修改：

    [root@localhost ~]# rpmdev-newspec -o myapp-0.1.0.spec
    Skeleton specfile (minimal) has been created to "myapp-0.1.0.spec".
    [root@localhost ~]# cat myapp-0.1.0.spec 

如果没有安装rpmdevtools，也可以自己手动创建一个spec文件

###spec文件内容：

    Name: myapp-0.1.0

    Version:
    Release: 1%{?dist}
    Summary:

    Group:
    License:
    URL:
    Source0:
    BuildRoot: %{_tmppath}/%{name}-%{version}-%{release}-root-%(%{__id_u} -n)

    BuildRequires:
    Requires:

    %description

    %prep
    %setup -q

    %build
    %configure
    make %{?_smp_mflags}

    %install
    rm -rf $RPM_BUILD_ROOT
    make install DESTDIR=$RPM_BUILD_ROOT

    %clean
    rm -rf $RPM_BUILD_ROOT

    %files
    %defattr(-,root,root,-)
    %doc

    %changelog 

SPEC文件的核心是它定义了一些“阶段”(%prep、%build、%install和%clean)，当rpmbuild执行时它首先会去解析SPEC文件，然后依次执行每个“阶段”里的指令。

###spec的头部内容说明：

		Name:                  myapp  ←←←软件包的名字
		Version:               0.1.0  ←←←软件包的版本
		Release:               1%{?dist} ←←←发布序号 
		Summary:               my first rpm ←←←软件包的摘要信息 
		Group:                 ←←←软件包的安装分类，参见/usr/share/doc/rpm-4.x.x/GROUPS这个文件 
		License:               GPL ←←←软件的授权方式 
		URL:                   ←←←源码包的下载路径或者自己的博客地址或者公司网址之类 
		Source0:               %{name}-%{version}.tar.gz ←←←源代码包的名称(默认时rpmbuid回到SOURCES目录中去找)，这里的name和version就是前两行定义的值。如果有其他配置或脚本则依次用Source1、Source2等等往后增加即可。 
		
		BuildRoot:             %{_topdir}/BUILDROOT ←←←这是make install时使用的“虚拟”根目录，最终制作rpm安装包的文件就来自这里。 
		
		BuildRequires:         ←←←在本机编译rpm包时需要的辅助工具，以逗号分隔。假如，要求编译myapp时，gcc的版本至少为4.4.2，则可以写成gcc >=4.2.2。还有其他依赖的话则以逗号分别继续写道后面。 
		Requires:              ←←←编译好的rpm软件在其他机器上安装时，需要依赖的其他软件包，也以逗号分隔，有版本需求的可以 
		
		%description           ←←←软件包的详细说明信息，但最多只能有80个英文字符。 


###制作rpm包每个阶段说明： 

制作rpm包有 **%prep**，**%build**，**%install**，**%files**，**%clean** 几个关键阶段。

#### %prep 阶段

这个阶段里通常情况，主要完成对源代码包的**解压**和**打补丁**，而解压时最常见到的就是一句指令：

	%setup -q 

将 **%_sourcedir** 目录下的源代码解压到 **%_builddir** 目录下。

#### %build 阶段

这个阶段会在 **%_builddir** 目录下执行源码包的编译。一般是执行执行常见的configure和make操作，如果有些软件需要最先执行bootstrap之类的，可以放在configure之前来做。

这个阶段我们最常见只有两条指令： 

	%configure 
	make %{?_smp_mflags} 

它就自动将软件安装时的路径自动设置成如下约定：
 
1. 可执行程序/usr/bin 
2. 依赖的动态库/usr/lib或者/usr/lib64视操作系统版本而定。 
3. 二次开发的头文件/usr/include 
4. 文档及手册/usr/share/man 

这里的 **%configure** 是个宏常量，会自动将 **prefix** 设置成 **/usr** 。另外，这个宏还可以接受额外的参数，如果某些软件有某些高级特性需要开启，可以通过给%configure宏传参数来开启。如果不用 %configure这个宏的话，就需要完全手动指定configure时的配置参数了。同样地，我们也可以给make传递额外的参数，例如：
 
	make %{?_smp_mflags} CFLAGS="" … 


#### %install 阶段

这个阶段就是执行make install操作，这个阶段会在%_buildrootdir目录里建好目录结构，然后将需要打包到rpm软件包里的文件从%_builddir里拷贝到%_buildrootdir里对应的目录里。

当用户最终用 `rpm -ivh name-version.rpm` 安装软件包时，这些文件会安装到用户系统中相应的目录里。

这个阶段最常见的两条指令是： 

	rm -rf $RPM_BUILD_ROOT 
	make install DESTDIR=$RPM_BUILD_ROOT 


其中**$RPM_BUILD_ROOT**也可以换成我们前面定义的BuildRoot变量，不过要写成**%{buildroot}**才可以，必须全部用小写，不然要报错。

如果软件有配置文件或者额外的启动脚本之类，就要手动用copy命令或者install命令你给将它也拷贝到%{buildroot}相应的目录里。用copy命令时如果目录不存在要手动建立，不然也会报错，所以推荐用install命令。  



#### %files 阶段

这个阶段主要用来说明会将%{buildroot}目录下的哪些文件和目录最终打包到rpm包里。

	%files 
	%defattr(-,root,root,-) 
	%doc 

在%files阶段的第一条命令的语法是：   

	%defattr(文件权限,用户名,组名,目录权限) 

如果不牵扯到文件、目录权限的改变则一般用%defattr(-,root,root,-)这条指令来为其设置缺省权限。所有需要打包到rpm包的文件和目录都在这个地方列出，例如：

	%files 
	%{_bindir}/* 
	%{_libdir}/* 
	%config(noreplace) %{_sysconfdir}/*.conf 
 
在安装rpm时，会将可执行的二进制文件放在/usr/bin目录下，动态库放在/usr/lib或者/usr/lib64目录下，配置文件放在/etc目录下，并且多次安装时新的配置文件不会覆盖以前已经存在的同名配置文件。  
这里在写要打包的文件列表时，既可以以宏常量开头，也可以为“/”开头，没任何本质的区别，都表示从%{buildroot}中拷贝文件到最终的rpm包里；如果是相对路径，则表示要拷贝的文件位于%{_builddir}目录，这主要适用于那些在%install阶段没有被拷贝到%{buildroot}目录里的文件，最常见的就是诸如README、LICENSE之类的文件。如果不想将%{buildroot}里的某些文件或目录打包到rpm里，则用： `%exclude dic_name` 或者 `file_name `

关于%files阶段有两个特性：
 
1. %{buildroot}里的所有文件都要明确被指定是否要被打包到rpm里。什么意思呢？假如，%{buildroot}目录下有4个目录a、b、c和d，在%files里仅指定a和b要打包到rpm里，如果不把c和d用exclude声明是要报错的； 
2. 如果声明了%{buildroot}里不存在的文件或者目录也会报错。 

 关于%doc宏，所有跟在这个宏后面的文件都来自%{_builddir}目录，当用户安装rpm时，由这个宏所指定的文件都会安装到/usr/share/doc/name-version/目录里。


#### 软件包制作阶段
这个阶段是自动完成的，所以在SPEC文件里面是看不到的，这个阶段会将 **%_buildroot** 目录的相关文件制作成rpm软件包最终放到 **%_rpmdir** 目录里。

#### %clean阶段 
编译完成后一些清理工作，主要包括对%{buildroot}目录的清空(这不是必须的)，通常执行诸如make clean之类的命令。


#### %changelog 阶段 

这是最后一个阶段，主要记录的每次打包时的修改变更日志。标准格式是： 

	* date +"%a %b %d %Y" 修改人 邮箱 本次版本x.y.z-p 
	- 本次变更修改了那些内容 




##rpm打包流程总结


1. 安装 rpmbuild 
2. 构建rpm的编译目录结构：  
rpmbuild/  
├ BUILD  
├ RPMS  
├ SOURCES  
├ SPECS  
└ SRPMS  
2. 将源码放到到 rpmbuild/SOURCES  
3. 在rpmbuild/SPECS目录创建spec文件  
4. 在rpmbuild/SPECS目录下执行打包编译：
	`rpmbuild -bb xxxxx.spec `  
5. 最终生成的rpm包


## SPEC文件模板

以下给出一个完整的，包含了打补丁、安装、卸载特性的SPEC文件模板

	Name: test 
	Version: 
	Requires: 
	%description 
	… 
	#==================================SPEC头部==================================== 
	%prep 
	%setup -q 
	%patch <==== 在这里打包 
	
	%build 
	%configure 
	make %{?_smp_mflags} 
	
	%install 
	rm -rf $RPM_BUILD_ROOT 
	make install DESTDIR=$RPM_BUILD_ROOT 
	
	%clean 
	rm -rf $RPM_BUILD_ROOT 
	
	%files 
	%defattr(-,root,root,-) 
	要打包到rpm包里的文件清单 
	%doc 
	%changelog 
	#==================================SPEC主体==================================== 
	
	%pre 
	安装或者升级软件前要做的事情，比如停止服务、备份相关文件等都在这里做。 
	%post 
	安装或者升级完成后要做的事情，比如执行ldconfig重构动态库缓存、启动服务等。 
	%preun 
	卸载软件前要做的事情，比如停止相关服务、关闭进程等。 
	%postun 
	卸载软件之后要做的事情，比如删除备份、配置文件等。




#附录：

##rpmbuild相关命令

基本格式：rpmbuild [options] [spec文档|tarball包|源码包]  

1.  从spec文档建立有以下选项：  
-bp  #只执行spec的%pre 段(解开源码包并打补丁，即只做准备)  
-bc  #执行spec的%pre和%build 段(准备并编译)  
-bi  #执行spec中%pre，%build与%install(准备，编译并安装)  
-bl  #检查spec中的%file段(查看文件是否齐全)  
-ba  #建立源码与二进制包(常用)  
-bb  #只建立二进制包(常用)  
-bs  #只建立源码包  

2.  从tarball包建立，与spec类似  
-tp #对应-bp  
-tc #对应-bc  
-ti #对应-bi  
-ta #对应-ba  
-tb #对应-bb  
-ts #对应-bs  

3.  从源码包建立  
--rebuild  #建立二进制包，通-bb  
--recompile  #同-bi  

4.  其他的一些选项  
--buildroot=DIRECTORY   #确定以root目录建立包  
--clean  #完成打包后清除BUILD下的文件目录  
--nobuild  #不进行%build的阶段  
--nodeps  #不检查建立包时的关联文件  
--nodirtokens  
--rmsource  #完成打包后清除SOURCES  
--rmspec #完成打包后清除SPEC  
--short-cricuit  
--target=CPU-VENDOR-OS #确定包的最终使用平台  



##spec文档的编写

Name: 软件包的名称，后面可使用%{name}的方式引用，具体命令需跟源包一致   
Summary: 软件包的内容概要  
Version: 软件的实际版本号，具体命令需跟源包一致  
Release: 发布序列号，具体命令需跟源包一致  

Group: 软件分组，建议使用标准分组  
软件包所属类别，具体类别有：  
Amusements/Games （娱乐/游戏）  
Amusements/Graphics（娱乐/图形）  
Applications/Archiving （应用/文档）  
Applications/Communications（应用/通讯）  
Applications/Databases （应用/数据库）  
Applications/Editors （应用/编辑器）  
Applications/Emulators （应用/仿真器）  
Applications/Engineering （应用/工程）  
Applications/File （应用/文件）  
Applications/Internet （应用/因特网）  
Applications/Multimedia（应用/多媒体）  
Applications/Productivity （应用/产品）  
Applications/Publishing（应用/印刷）  
Applications/System（应用/系统）  
Applications/Text （应用/文本）  
Development/Debuggers （开发/调试器）  
Development/Languages （开发/语言）  
Development/Libraries （开发/函数库）  
Development/System （开发/系统）  
Development/Tools （开发/工具）  
Documentation （文档）  
System Environment/Base（系统环境/基础）  
System Environment/Daemons （系统环境/守护）  
System Environment/Kernel （系统环境/内核）  
System Environment/Libraries （系统环境/函数库）  
System Environment/Shells （系统环境/接口）  
User Interface/Desktops（用户界面/桌面）  
User Interface/X （用户界面/X窗口）  
User Interface/X Hardware Support （用户界面/X硬件支持）  
* * * 
License: 软件授权方式，通常就是GPL  
Source: 源代码包，可以带多个用Source1、Source2等源，后面也可以用%{source1}、%{source2}引用    
BuildRoot: 这个是安装或编译时使用的“虚拟目录”，考虑到多用户的环境，一般定义为：`%{_tmppath}/%{name}-%{version}-%{release}-root1`
或`%{_tmppath}/%{name}-%{version}-%{release}-buildroot-%(%{__id_u} -n}`  
该参数非常重要，因为在生成rpm的过程中，执行make install时就会把软件安装到上述的路径中，在打包的时候，同样依赖“虚拟目录”为“根目录”进行操作。  
后面可使用$RPM\_BUILD\_ROOT 方式引用。  
URL: 软件的主页  
Vendor: 发行商或打包组织的信息，例如RedFlag Co,Ltd  
Disstribution: 发行版标识  
Patch: 补丁源码，可使用Patch1、Patch2等标识多个补丁，使用%patch0或%{patch0}引用  
Prefix: %{\_prefix} 这个主要是为了解决今后安装rpm包时，并不一定把软件安装到rpm中打包的目录的情况。这样，必须在这里定义该标识，并在编写%install脚本的时候引用，才能实现rpm安装时重新指定位置的功能  
Prefix: %{\_sysconfdir} 这个原因和上面的一样，但由于`%{_prefix}`指/usr，而对于其他的文件，例如/etc下的配置文件，则需要用%{\_sysconfdir}标识  
Build Arch: 指编译的目标处理器架构，noarch标识不指定，但通常都是以/usr/lib/rpm/marcros中的内容为默认值
Requires: 该rpm包所依赖的软件包名称，可以用>=或<=表示大于或小于某一特定版本，例如：  
libpng-devel >= 1.0.20 zlib  
※“>=”号两边需用空格隔开，而不同软件名称也用空格分开  
还有例如PreReq、Requires(pre)、Requires(post)、Requires(preun)、Requires(postun)、BuildRequires等都是针对不同阶段的依赖指定  
Provides: 指明本软件一些特定的功能，以便其他rpm识别  
Packager: 打包者的信息  
scription 软件的详细说明  

##spec脚本主体
spec脚本的主体中也包括了很多关键字和描述，下面会一一列举。我会把一些特别需要留意的地方标注出来。
%prep 预处理脚本  
%setup -n %{name}-%{version} 把源码包解压并放好  
注：可根据你的源码的名字格式，来确认解压后名字的格式，否则可能导致install的时候找不到对应的目录  
通常是从/usr/src/redhat/SOURCES里的包解压到/usr/src/redhat/BUILD/%{name}-%{version}中。  
一般用%setup -c就可以了，但有两种情况：一就是同时编译多个源码包，二就是源码的tar包的名称与解压出来的目录不一致，此时，就需要使用-n参数指定一下了。  
%patch 打补丁  
通常补丁都会一起在源码tar.gz包中，或放到SOURCES目录下。一般参数为：  
%patch -p1 使用前面定义的Patch补丁进行，-p1是忽略patch的第一层目录  
%Patch2 -p1 -b xxx.patch 打上指定的补丁，-b是指生成备份文件  

◎补充  
%setup 不加任何选项，仅将软件包打开。  
%setup -n newdir 将软件包解压在newdir目录。  
%setup -c 解压缩之前先产生目录。  
%setup -b num 将第num个source文件解压缩。  
%setup -T 不使用default的解压缩操作。  
%setup -T -b 0 将第0个源代码文件解压缩。  
%setup -c -n newdir 指定目录名称newdir，并在此目录产生rpm套件。  
%patch 最简单的补丁方式，自动指定patch level。  
%patch 0 使用第0个补丁文件，相当于%patch ?p 0。  
%patch -s 不显示打补丁时的信息。  
%patch -T 将所有打补丁时产生的输出文件删除。  

%build 开始构建包  
在/usr/src/redhat/BUILD/%{name}-%{version}目录中进行make的工作 ，常见写法：  
make %{?_smp_mflags} OPTIMIZE="%{optflags}"   
都是一些优化参数，定义在/usr/lib/rpm/marcros中  

%install 开始把软件安装到虚拟的根目录中  
在/usr/src/redhat/BUILD/%{name}-%{version}目录中进行make install的操作。这个很重要，因为如果这里的路径不对的话，则下面%file中寻找文件的时候就会失败。 常见内容有：  
%makeinstall 这不是关键字，而是rpm定义的标准宏命令。也可以使用非标准写法：  
make DESTDIR=$RPM_BUILD_ROOT install  
或  
make prefix=$RPM_BUILD_ROOT install  

需要说明的是，这里的%install主要就是为了后面的%file服务的。所以，还可以使用常规的系统命令：  

install -d $RPM_BUILD_ROOT/ #建立目录  
cp -a * $RPM_BUILD_ROOT/  

%clean 清理临时文件  
通常内容为：  
引用  
	[ "$RPM_BUILD_ROOT" != "/" ] && rm -rf "$RPM_BUILD_ROOT"
	rm -rf $RPM_BUILD_DIR/%{name}-%{version}

※注意区分$RPM\_BUILD\_ROOT和$RPM\_BUILD\_DIR：
$RPM\_BUILD\_ROOT是指开头定义的BuildRoot，而$RPM\_BUILD\_DIR通常就是指/usr/src/redhat/BUILD，其中，前面的才是%file需要的。

%pre rpm安装前执行的脚本  
%post rpm安装后执行的脚本  
%preun rpm卸载前执行的脚本  
%postun rpm卸载后执行的脚本  
%preun %postun 的区别是:前者在升级的时候会执行，后者在升级rpm包的时候不会执行  

%files 定义那些文件或目录会放入rpm中  
这里会在虚拟根目录下进行，千万不要写绝对路径，而应用宏或变量表示相对路径。如果描述为目录，表示目录中除%exclude外的所有文件。  
fattr (-,root,root) 指定包装文件的属性，分别是(mode,owner,group)，-表示默认值，对文本文件是0644，可执行文件是0755  

%exclude 列出不想打包到rpm中的文件  
※小心，如果%exclude指定的文件不存在，也会出错的。  
%changelog 变更日志  

##spec文档中常用的几个宏(变量)  
1. RPM\_BUILD\_DIR:    /usr/src/redhat/BUILD  
2. RPM\_BUILD\_ROOT:   /usr/src/redhat/BUILDROOT  
3. %{\_sysconfdir}:   /etc  
4. %{\_sbindir}：     /usr/sbin  
5. %{\_bindir}:       /usr/bin  
6. %{\_datadir}:      /usr/share  
7. %{\_mandir}:       /usr/share/man  
8. %{\_libdir}:       /usr/lib64  
9. %{\_prefix}:       /usr  
10. %{\_localstatedir}:   /usr/var  



##rpm常用命令

* 手动安装 rpm 包  
`rpm-ivh xxxxx.rpm`
 参数：  
 --force 即使覆盖其他包的文件也没强迫安装  
 --nodeps 即使依赖包没安装，也被强制安装  

* 查看 rpm 包信息  
`rpm-qpi xxxxx.rpm`

* 查看 rpm 包依赖  
`rpm -qpR xxxxx.rpm`

* 查看 rpm 包中包含那些文件  
`rpm -qlp xxxxx.rpm`  
 可以加grep搜索  
`rpm -qlp xxxxx.rpm|grep spec`

* 使用工具rpm2cpio提取文件： 
`rpm2cpio xxxxx.rpm |cpio -ivd xxx.jpg`

* 用rpm2cpio将rpm文件转换成cpio文件
`rpm2cpio xxxxxx.rpm >xxxxx.cpio`

* 用cpio解压cpio文件   
`cpio -i  --make-directories`

* 提取所有文件：  
`rpm2cpio xxx.rpm | cpio -vi`   
`rpm2cpio xxx.rpm | cpio -idmv`   
`rpm2cpio xxx.rpm | cpio --extract --make-directories`  

* cpio 参数说明:  
	**i** 和 **extract** 表示提取文件   
	**v** 表示指示执行进程    
	**d** 和 **make-directory** 表示根据包中文件原来的路径建立目录 
	**m** 表示保持文件的更新时间

* 查看rpm包里的pre和post install脚本：
`rpm -qp --scripts xxxxx.rpm`  

* 查看安装的过程中，代码的执行过程：  
`rpm -ih -vv xxxxx.rpm`  

* 强制卸载rpm包  
`rpm -e --nodeps xxxxx`
 没有rpm后缀

* 查询一个rpm包是否被安装
`rpm - q xxxxxx`
