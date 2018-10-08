# linux-__NO.3__  
文件管理  
==
#文件系统  
--
文件和目录被组织成一个单根倒置树结构 
文件系统从根目录下开始，用“/”表示    
文件名称区分大小写  
以.开头的文件为隐藏文件  
路径分隔的/  
文件有两类数据：元数据：metadata  数据：data 
  
![image](http://i2.51cto.com/images/blog/201801/19/2e756907ab2ac4a81919d472460b6683.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)  
#文件名规则  
--
文件名最长255个字节  
包括路径在内文件名称最长4095个字节  
<font color="Aqua">蓝色</font>-->目录<font color="LawnGreen">&emsp;绿色</font>-->可执行文件<font color="Red">&emsp;红色</font>-->  
&emsp;&emsp;压缩文件<font color="LightSkyBlue">&emsp;浅蓝色</font>-->链接文件<font color="Gray">&emsp;灰色</font>-->其他文件  
文件名称大小写敏感  

#文件系统结构
--
/boot：引导文件存放目录，内核文件(vmlinuz)、引导加载器    
/bin：所有用户使用的基本命令  
/sbin：管理类的基本命令；  
/lib：启动时程序依赖的基本共享库文件以及内核模块文件(/lib/modules)  
/lib64：x86_64系统上的辅助共享库文件存放位置  
/etc：配置文件目录  
/home/：普通用户家目录  
/root：管理员的家目录  
/media：便携式移动设备挂载点  
/mnt：临时文件系统挂载点  
/dev：设备文件及特殊文件存储位置  
&emsp;b: block device，随机访问  
&emsp;c: character device，线性访问  
/opt：第三方应用程序的安装位置  
/srv：系统上运行的服务用到的数据  
/tmp：临时文件存储位置  
/user:通用共享，只读数据  
/var:可变数据文件 variable data files  
/proc:输出内核与进程信息相关的虚拟文件系统  
/sys：输出当前系统上硬件设备相关信息虚拟文件系统  
/selinux:安全策略信息相关存储位置  

#Linux下的文件类型  
--  

\-&emsp;普通文件   
d&emsp;目录文件    
b&emsp;块设备    
c&emsp;字符设备    
l&emsp;符号链接文件    
p&emsp;管道文件pipe  
s&emsp;套接字文件socket

#显示当前工作目录
pwd  
  -p 显示物理路径
  -L 联系链接路径（默认）  

#绝对和相对路径  
--  
**绝对路径**  
以正斜杠开始  
/目录开头，完整的文件的位置路径  
**相对路径名**  
 不以斜线开始
当前路径开头，指定一个文件名  
基名：basename  
目录名：dirname  

#更改目录
--  
cd 改变目录  
切换至上级目录：cd..  
切换至当前用户主目录：cd  
切换至之前目录：cd-  

#列出目录内容  
--
\--ls [options] [files_or_dirs]  
ls -a&emsp;包含隐藏文件  
ls -l&emsp;显示额外的信息  
ls -R&emsp;目录递归通过  
ls -ld&emsp;目录和符号链接信息  
ls -1&emsp;文件分行显示  
ls –S&emsp;按从大到小排序  
ls –t&emsp;按mtime排序  
ls –u&emsp;配合-t选项，显示并按atime从新到旧排序  
ls –U&emsp;按目录存放顺序显示  
ls –X&emsp;按文件后缀排序  

#查看文件状态
stat
文件：metadata，data  
atime，读取文件内容  
mtime，改变文件内容（数据）  
ctime，元数据发生改变  

#文件通配符  
--
匹配零个或多个字符  
匹配任何单个字符  
~ 当前用户家目录  
~mage 用户mage家目录    
[0-9]匹配数字范围  
[a-z]：字母  
[A-Z]：字母  
[wang]匹配列表中的任何的一个字符  
[^wang]匹配列表中的所有字符以外的字符  

#文件通配符  
预定义的字符类：man 7 glob  
[:digit:]：任意数字，相当于0-9  
[:lower:]：任意小写字母  
[:upper:]: 任意大写字母  
[:alpha:]: 任意大小写字母  
[:alnum:]：任意数字或字母  
[:blank:]：水平空白字符  
[:space:]：水平或垂直空白字符  

#练习  
--
1、显示/var目录下所有以l开头，以一个小写字母结尾，且中间出现至少一位数字的文件或目录     
ls  /var/l*[0-9]*[[:lower:]]    

2/显示/etc目录下以任意一位数字开头，且以非数字结尾的文件或目录   
ls  /etc/[[:gidit:]]*[^[:gidit:]]  

3、显示/etc/目录下以非字母开头，后面跟了一个字母及其它任意长度任意字符的文件或目录  
ls  /etc/[^[:alpha:]][[:alpha:]]*

4、显示/etc/目录下所有以rc开头，并后面是0-6之间的数字，其它为任意字符的文件或目录  
ls /etc/rc[0-6]* 

5、显示/etc目录下，所有以.d结尾的文件或目录  
ls /etc/*.d 

6、显示/etc目录下，所有.conf结尾，且以m,n,r,p开头的文件或目录  
ls  /etc/[mnrp]*.conf  

7、只显示/root下的隐藏文件和目录  
ls -ad /root/.*   

8、只显示/etc下的非隐藏目录  
ls -d /etc/[^.]*     

#创建空文件和刷新时间
--
touch   [OPTION]... FILE...  
-a&emsp;仅改变atime和ctime  
-m&emsp;仅改变mtime和ctime  
-t&emsp;[[CC]YY]MMDDhhmm[.ss]  
&emsp;指定atime和mtime的时间戳  
-c&emsp;如果文件不存在，则不予创建  

#复制文件和目录cp  
--  

cp[OPTION]... [-T] SOURCE DEST (默认不用加-T)  
cp[OPTION]... SOURCE... DIRECTORY   
cp[OPTION]... -t DIRECTORY SOURCE...  
-i：覆盖前提示（root默认定义cp -i别名） 
–n:不覆盖，注意两者顺序  
-r, -R: 递归复制目录及内部的所有内容  
-d：不复制源文件，只复制链接名  
-p：保留文件属性→所有者权限，属主属组，时间戳
-f：强制  
-u:只复制源比目标 更新的or不存在 的文件
-b：若目标存在，覆盖前先备份DEST
-v:显示复制过程

#练习
--
1、定义别名命令baketc，每天将/etc/目录下所有文件，备份到/app独立的子目录下，并要求子目录格式为backupYYYY-mm-dd，备份过程可见  
alias baketc='cp -av /etc/   /app/backup\`date +%F`'  
2、创建/app/rootdir目录，并复制/root下所有文件  到该目录内，要求保留原有权限  
cp -rp /root/ /app/rootdir

#移动和重命名文件  
-- 

mv [OPTION]... [-T] SOURCE DEST  
mv [OPTION]... SOURCE... DIRECTORY  
mv [OPTION]... -t DIRECTORY SOURCE...   
常用选项：  
-i 交互式(提示)  
-f 强制  
-b 目标存在，覆盖前先备份  

#删除
--
常用选项：  
-i 交互式  
-f 强制删除  
-r 递归  

#目录操作  
--
**tree 显示目录树**  
-d: 只显示目录  
-L level：指定显示的层级数目  
-P pattern: 只显示由指定pattern匹配到的路径  
**mkdir创建目录**  
-p: 存在于不报错，且可自动创建所需的各目录  
-v: 显示详细信息  
-m MODE: 创建目录时直接指定权限  
**rmdir删除空目录** (不常用)  
-p: 递归删除父空目录  
-v: 显示详细信息  
rm-r递归删除

#练习  
--
(1) 如何创建/testdir/dir1/x, /testdir/dir1/y, /testdir/dir1/x/a, /testdir/dir1/x/b, /testdir/dir1/y/a, /testdir/dir1/y/b  
mkdir -p /testdir/dir1/{x,y}/{a,b}  

(2) 如何创建  /testdir/dir2/x,/testdir/dir2/y,/testdir/dir2/x/a,/testdir/dir2/x/b  
mkdir -p /data/testdir/dir2/{x/{a,b},y}

(3) 如何创建/testdir/dir3, /testdir/dir4, /testdir/dir5, /testdir/dir5/dir6,   /testdir/dir5/dir7  
mkdir -p testdir/dir{3,4,5/dir{6,7}}  

#索引节点  
--
--inode（index node）表中包含文件系统所有文件列表  
包括：文件类型，权限，UID，GID  
&emsp;&emsp;&emsp;链接数,文件大小，时间戳，数据块指针，其它数据  
**目录**    
文件引用一个是inode号  
人是通过文件名来引用一个文件  
一个目录是目录下的文件名和文件inode号之间的映射       

#硬链接  
--
ln  filename [linkname]  
（不能跨越驱动器或分区）

#软链接  
--  
ln -s filename [linkname]  
（不能跨越驱动器或分区）  

**硬链接和软链接区别：**  
硬&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;软  
1）同一个文件&emsp;&emsp;不同文件  
2）不能跨分区&emsp;&emsp;能跨分区  
3）连接数不增长&emsp;&emsp;链接数增加  
4）节点编号一样&emsp;&emsp;节点编号不一样  
5）原始文件删除，硬链接>=1正常访问  
&emsp;&emsp;原始文件删除，软链接失效  
6）相对路径写法不一样 &emsp;&emsp;大小不一样  
7）不支持目录创建&emsp;&emsp;支持目录创建  
8）相对路径节点一样&emsp;&emsp;相对路径节点不一样   

#\--file确定文件内容  
--   
file [options] \<filename>...  
-b&emsp;列出文件辨识结果时，不显示文件名称    
-f&emsp;filelist列出文件filelist中文件名的文件类型  
-F&emsp;使用指定分隔符号替换输出文件名后默认的”:”分隔符  
-L&emsp;查看对应软链接对应文件的文件类型  
--help&emsp;显示命令在线帮助  

