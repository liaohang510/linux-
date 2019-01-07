企业级调度器LVS
==    
  
数据库服务器=IO密集型+CPU密集型  

语句路由器-读写分离器  
缓存服务器：nosql服务器，对商品进行分类，排序和聚合，高度消耗资源  
heartbeat：心跳检测  

up：向上扩展，增加性能，纵向扩展   
#性价比不高，会遇到扩展瓶颈    
out：横向扩展，增加服务器数量，横向扩展  
#相对廉价，把应用服务切分/迁移到各个服务器，会降低读写IO  

● session保持   [针对有状态stateful]   
session sticky 会话粘性，记录用户访问会话  
session replication 会话复制，只适合初创阶段  
session server  独立专门的会话缓存服务器  

ipvs：内核中的协议栈上实现  
ipvsadm：用户空间的集群服务管理工具   

LB集群实现：load balance-四层调度器  
实现：  
七层调度器 基于套接字区分请求调度，一般不超过4W并发，效率低  
四层调度器 内核级调度，调度能力比七层要高得多  
分类：  
硬负载 F5-NetScaler-A10  
软负载 四层：LVS，Nginx（stream模块），HAProxy（mode tcp）..  
       七层：Nginx，HAProxy，ATS，Traefik ..  
	   
● 静态算法：仅根据算法本身和请求报文特征进行调度 ▲  
#起点公平  
#rr：round-robin，轮替，轮询算法  
#wrr：weighted rr 加权轮询算法  
（短连接，无状态，最好的算法↑）  
#sh：source ip hashing 原地址hash算法  
#dh：destination ip hashing 目标地址hash算法  

● 动态算法：额外考虑后端各real server的当前负载状态 ▲  
#结果公平    
#lc:least connections 最少连接 算法  
 overhead=activeconns*256+inactiveconns 计算当前负载  
 负载=活动连接数*256+非活动活跃数  
#wlc：weighted lc 权重最少连接 算法 ▲LVS默认算法  
 overhead=(activeconns*256+inactiveconns)/weight  
#sed 最短期望延迟 算法   
 overhead=(activeconns+1)*256/weight  
#nq 永不排队 算法  
 按权重倒序，权重大的多承重，权重小的依次降低  
#lblc  算法 ；基于本地，注重的起点公平 
#lblcr 算法 ；基于本地的最小连接，注重的起点公平   

缓存：  
CIP：客户端ip | VIP：调度器外网ip ; DIP：调度器内网IP | RIP：集群服务器ip  
LVS内核工作框架：基于ipvs，ipvs工作在INPUT链；建议不要把iptables和ipvs混用  
集群服务器内核空间：生成规则-ipvsadm  

LVS类型：工作拓扑结构和转发机制  
● NAT   
#基于多目标DNAT方式调度，通过修改请求报文的目标ip和端口，  
#为经由调度算法挑选出的某后端RS的RIP和port完成调度  
● DR  
#请求报文的iP首部，保持不变，通过在原IP报文外封装帧首部(源MAC目标MAC)完成调度  
#目标MAC是由调度算法挑选出的某后端RS的MAC地址；  
● TUN  IP隧道  
#通过在源IP报文之外再封装一个新IP首部(DIP:RIP)，完成调度  
● FULNAT  
#通过修改请求报文的源IP和目标IP，为经由调度算法挑选出的某后端RS的RIP完成调度  
#源IP（CIP->DIP) 客户请求ip转成调度器内网ip  
#目标IP（VIP->RIP） 调度器外网IP转成RS服务器RIP  

lvs-nat：   
请求包路径：CIP->路由->VIP调度器DIP->RIP  
回应包路径：RIP->DIP调度器VIP->路由->CIP  
多目标IP的DNAT，通过将请求报文中的目标地址和目标端口修改为某挑出的RS的RIP和PORT实现转发；  
1）RIP和DIP必须在同一个IP网络，且应该使用私网地址；RS的网关要指向DIP；  
2）请求报文和响应报文都必须经由Director转发；Director易于成为系统瓶颈；  
3）支持端口映射，可修改请求报文的目标PORT；  
4）vs必须是Linux系统，rs可以是任意系统；  

lvs-dr： Direct Routing，直接路由  
通过为请求报文重新封装一个MAC首部进行转发，源MAC是DIP所在的接口的MAC，目标MAC是某挑选出的RS的RIP所在接口的MAC地址；源IP/PORT，以及目标IP/PORT均保持不变；  
Director和各RS都得配置使用VIP；  
#请求包路径：CIP->路由->VIP->RIP    
#回应包路径：RIP-VIP->路由->CIP  
#路由器内网VIP解析的mac地址，必须是VIP的mac地址，而不是RS的  
#修改RS的内核参数，从而达到RS的VIP地址不响应↑mac解析请求，主推  
#避免VIP响应冲突的其它两种方式是arptables和直接路由绑定  
#配置内核默认应答方式，只回应对应接口的关联地址，且响应报文只由指定地址发出  
1) 确保前端路由器将目标IP为VIP的请求报文发往Director  ：  
	(a) 在前端网关做静态绑定；  
	(b) 在RS上使用arptables；  
	(c) 在RS上修改内核参数以限制arp通告及应答级别；  
		arp_announce  
		arp_ignore  
2) RS的RIP可以使用私网地址，也可以是公网地址；RIP与DIP在同一IP网络；   RIP的网关不能指向DIP，以确保响应报文不会经由Director；  
3) RS跟Director要在同一个物理网络；  
4) 请求报文要经由Director，但响应不能经由Director，而是由RS直接发往Client；  
5) 不支持端口映射；    
  
lvs-tun：  
请求包路径：CIP->  
#转发方式：不修改请求报文的IP首部（源IP为CIP，目标IP为VIP），而是在原IP报文之外再封装一个IP首部（源IP是DIP，目标IP是RIP）  
#将报文发往挑选出的目标RS；RS直接响应给客户端（源IP是VIP，目标IP是CIP）；  
1) DIP, VIP, RIP都应该是公网地址；  
2) RS的网关不能，也不可能指向DIP；  
3) 请求报文要经由Director，但响应不能经由Director；  
4) 不支持端口映射；  
5) RS的OS得支持隧道功能；  

lvs-fullnat：  
#拓展了内网结构，不会让RS暴露在互联网上  
#通过同时修改请求报文的源IP地址和目标IP地址进行转发；  
CIP <--> DIP | VIP <--> RIP   
1) VIP是公网地址，RIP和DIP是私网地址，且通常不在同一IP网络；因此，RIP的网关一般不会指向DIP；  
2) RS收到的请求报文源地址是DIP，因此，只能响应给DIP；但Director还要将其发往Client；  
3) 请求和响应报文都经由Director；  
4) 支持端口映射；  
注意：此类型默认不支持；  
				
ip_vs规则：ipvsadm  
  集群服务: Dispatch service   
	vip.port+tcp/vip.port+udp/FWM防火墙标记  
  集群服务的RS:  
	rip:port  
	  
--实验：通过wrr权重轮询算法 调度▲  
#通过docker配置两个容器实现  
cd /etc/yum.repo.d/  
wget   https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo  
yum install docker-ce -y  
yum install ipvsadm  -y  
#管理集群服务：-A增/-E改/-D删/-L查/-C清空/-R重载/-S保存/-t主机+tcp端口  
				-s调度算法-scheduler  
#管理RS服务器：-a增、-d删、-e改、-t主机+tcp端口、-l查、-n数字格式  
				-g:网关、-m:NAT、-i:tunneling|ipip封装、-w权重（默认1）  
				-r：RS服务器  
systemctl start dodcker  
docker image pull busybox  
容器1：  
docker run --name rs1 -it --network bridge -v   /vols/rs1:/data/web/html busybox  
cd /vols/rs1  
vim index.html  
  \<h1>RS1\</h1>  
httpd -h /data/web/html  
容器2：  
docker run --name rs2 -it --network bridge -v   /vols/rs2:/data/web/html busybox  
 vim index.html    
\<h1\>RS2</h1\>     
httpd -h /data/web/html      
vi /usr/lib/systemd/system/docker.service   
 [service]  
ExecStartPost=/usr/sbin/iptables -P FORWARD ACCEPT  
表示启动docker之后，执行该命令  
iptables -P FORWARD ACCEPT #或正常载入配置systemctl daemon-reload  
ipvsadm -Ln 查看集群信息  
定义集群服务 ↓↓   
ipvsadm -A -t 172.18.18.18:80 -s wrr 定义集群服务地址和调度算法  
定义集群RS ↓↓   
ipvsadm -a -t 172.18.18.18:80 -r 172.17.0.2:80 -m -w 1    
ipvsadm -a -t 172.18.18.18:80 -r 172.17.0.3:80 -m -w 1  
#如出现Memory allocation problem，则modprobe -r ip_vs | modprobe-r ip_vs卸载模块重装   
curl 172.18.0.66 模拟请求验证调度服务   
while true;do curl 172.18.0.66; sleep .5; done 循环请求   
ipvsadm -Z 清除计数器 ↓以下两个清空    
ipvsadm -Ln --rate 统计速率  
ipvsadm -Ln --stats 统计计数总和  
ipvsadm -e -t 172.18.0.66:80 -r 172.17.0.2 -m -w 2 修改权重   
#缺陷：调度器只复制转发，而不对RS做健康状态检查，发生故障也会照常转发
#使用7层调度nginx或者ipproxe，或者使用组件可解决该问题，或添加sorry   server   

--实验：通过dr直接路由算法 调度▲   
#请求通过调度器，回应直接发往VIP，路由器和RIP和调度器在同一网段  
#更改内核参数，阻住不必要的通告，内核添加DIP地址  
#不支持端口映射  
#同一个网段3台虚拟机实现；66,67,68在同一网段  
66:DRIP  67,68:RS01,RS02  
66:  
scp /etc/yum.repo.d/CentOS-Base.repo  67:/etc/yum.repos.d/  
scp /etc/yum.repo.d/CentOS-Base.repo  68:/etc/yum.repos.d/  
ifconfig ens33:0 172.18.0.70 netmask 255.255.255.255 broadcast 172.18.0.70 up  
0.70为VIP，与前端请求者通信,避免在局域网广播其它报文  
cd /proc/sys/net/ipv4/ens33/arp_ignore  
vim setkerner.sh  配置修改内核参数脚本  
	vip=172.18.0.70  
	mask=255.255.255.255  
	iterface="lo:0"  
	case $1 in  
	start)  
	echo 1 > /proc/sys/net/ipv4/all/arp_ignore  
	echo 2 > /proc/sys/net/ipv4/all/arp_announce  
	echo 1 > /proc/sys/net/ipv4/lo/arp_ignore  
	echo 2 > /proc/sys/net/ipv4/lo/arp_announce  
	ifconfig $interface $vip netmask $mask broadcast $vip up  
	route add -host $vip dev lo:0  
	;;  
	stop)  
	ifconfig $interface down  
	echo 0 > /proc/sys/net/ipv4/all/arp_ignore  
	echo 0 > /proc/sys/net/ipv4/all/arp_announce  
	echo 0 > /proc/sys/net/ipv4/lo/arp_ignore  
	echo 0 > /proc/sys/net/ipv4/lo/arp_announce  
	;;   
	*)  
	echo "Usage $basename $0) start|stop"  
	exit 1  
	esac  
bash setkerner.sh start  
cp setkerner.sh 67,68  
ipvsadm -A -t 172.18.0.70:80 -s wrr 指定调度地址和加权轮询算法  
ipvsadm -a -t 172.18.0.70:80 -r 172.18.0.67 -g -w 2  
ipvsadm -a -t 172.18.0.70:80 -r 172.18.0.68 -g -w 3  
ipvsadm -LN 查看是否直接路由方式  
curl 172.18.0.70 访问网页，验证调度方式  
ipvsadm -E -t 172.18.0.70:80 -s sh 修改调度算法为原地址hash算法,长连接  
culr 172.18.0.70 访问页面，将始终保持统一个服务响应  

67,68:  
yum install httpd -y  
systemctl restart chronyd  
vi /var/www/html/index.html  
  \<h1>67|68\</h1>  
systemctl start httpd   
bash setkerner.sh start  
  

ipvsadm -S 保存规则  
ipvsadm -S > /etc/sysconfig/ipvsadm 保存至配置文件，开机自动生效  
ipvsadm -C 清空规则  
ipvsadm -R < etc/sysconfig/ipvsadm 载入规则  

--实验：同时调度RS服务器的http和httpds服务 ▲  
#ipvs规则调度方式：tcp、udp、防火墙标记 [0至2的32次方任意数字]  
#把两类服务打成同一类标记，然后基于标记来处理  
cd /etc/httpd/conf.d/  
vi myvhosts.conf  
Listen 8080  
<virtualHost *:80>  
	ServerName host66.com  
	DocumentRoot "/www/vhost1"  
	<Directroy "www/vhost1">  
		Options none  
		AllowOverride none  
		Require all granted  
	</Directory>  
</VirtualHost>  
  
<virtualHost *:8080>  
	ServerName centos2.com  
	DocumentRoot "/www/vhost2"  
	<Directroy "www/vhost2">  
		Options none  
		AllowOverride none  
		Require all granted  
	</Directory>  
</VirtualHost>  

mkdir -pv /www/vhost{1,2}  
vi /www/vhost1/index.html  
\<h1>cento2.com\</h1>  
vi /www/vhost2/index.html  
\<h1>cento2.com:8080\</h1>  
systemctl restart httpd  
scp myvhosts.conf centos2:/etc/httpd/conf.d/    
scp -rp /www/ centos3:/   
  
■ RS:  
vi myvhosts.conf  
Listen 8080  
<virtualHost *:80>  
	ServerName centos1.com  
	DocumentRoot "/www/vhost1"  
	<Directroy "www/vhost1">  
		Options none  
		AllowOverride none  
		Require all granted  
	</Directory>  
</VirtualHost>  

<virtualHost *:8080>  
	ServerName centos2.com  
	DocumentRoot "/www/vhost2"  
	<Directroy "www/vhost2">  
		Options none  
		AllowOverride none  
		Require all granted  
	</Directory>  
</VirtualHost>  
vi /www/vhost1/index.html  
\<h1>cento3.com\</h1>  
vi /www/vhost2/index.html  
\<h1>cento3.com:8080\</h1>  

ipvsadm -C  
iptables -t mangle -A PREROUTING -d 172.18.0.70   
-p tcp -m multiport --dports 80,8080 -j MARK 7 --set-mark 7  
针对目标地址0.70的两个端口，同时加上iptables防火墙标记7  
iptables -vnL 查看  
ipvsadm -A -f 7 -s wrr 针对标记为7的流量做加权轮询调度   
ipvsadm -a -f 7 -r 172.18.0.67 -g -w 1   
ipvsadm -a -f 7 -r 172.18.0.68 -g -w 1   
ipvsadm -Ln  
while true; do curl 172.18.0.70;curl 172.18.0.70:8080;sleep 1;done 验证!!  


● firemark:  
	为服务提供分类器，从而实现将多个服务归类一个服务  
	Drectory会记录连接条目，然后生成计数器，只要倒计时没结束，统一客户端的请求始终发往同一服务器  
	一旦倒计时结束，默认5分钟，没命中，重新用调度算法实现  
	如果倒计时为0且命令，将自动延时2分钟  
	调度器靠连接追踪模板实现调度  
	
	
--实验 持久防火墙标记PFWM [ssh，redis] ▲  
#用任何算法，都可以持久连接，持久都几个端口上取决于机制↓  
#PCC:使用0端口[所有端口],对所有的访问，都是被持久的  
#PPC:同一客户端访问同一端口，进行持久  
#PFWM：使用防火墙标记把多端口绑定为一个服务，来之同一客户端对同一标记的所有访问，都是持久度  
67,68:  
yum install redis  
vi /etc/redis.conf  
 bind 0.0.0.0  
 systemclt restart redis  
70:  
ipvsadm -A -t 172.18.0.70:0 -s wrr -p 长连接，默认360秒  
ipvsadm -a -t 172.18.0.70:0 -r 172.18.0.67 -g -w 1  
ipvsadm -a -t 172.18.0.70:0 -r 172.18.0.68 -g -w 1  
ipvsadm -Ln  
curl 172.18.0.70:8080   
来自同一客户端，无论任何服务访问，都是绑定在同一个客户端    
其它客户端：  
yum install redis  
curl 172.18.0.70 |  curl 172.18.0.70:8080  
redis-cli -h 172.18.0.70  连接到70调度器  
>info 可现实当前登录的主机信息  
>set v1 k1  通过RS用get查看数值也能验证  


ipvsadm -C 清空规则  
ipvsadm -A -f 7 -s wrr  
ipvsadm -E -f 7 -s wrr -p 1200 防火墙标记为7的使用持久连接及加权轮询算法  
ipvsadm -a -f 7 -r 172.18.0.67 -g -w 1  
ipvsadm -a -f 7 -r 172.18.0.68 -g -w 1  
ipvsadm -A -t 172.180.70:6379 -s wrr  引用6379端口做加权轮询算法  
ipvsadm -a -t 172.18.0.70 -r 172.18.0.67 -g -w 1  
ipvsadm -a -t 172.18.0.70 -r 172.18.0.68 -g -w 1  
#如遇到内存分配问题↓↓ lsmod | grep ip_vs  
#modprobe -r ip_vs_sh ; modprobe -r ip_vs_wrr ; modprobe -r ip_vs  
#然后从新加载规则  
使用同一客户端访问web服务：  
curl 172.18.0.70:80 | curl 172.18.0.70:8080  
redis-cli -h 172.18.0.70  
>set v1 验证被调度到哪个RS  

延伸：  
加入后台服务只需要支持只读web服务，或者select语句的查询服务，一般用rr或者wrr就好  
假如web是动态网站，需要提供数据存储验证及登录等功能，因此需要提供共享储存服务；  
这样使用调度器才能保持负载均衡确保公平访问及高可用  
假如组假设myslq集群服务，也做统一存储，通过调度器均衡访问，也会有问题，多个节点操作同一数据集,inode会崩溃，且不支持该项操作，所以一般数据库都用myisam   


作业：负载均衡两个php应用（wordpress，discuzx）  
测试1）是否需要会话保持，2）是否需要共享存储  
至少两个RS，客户端登录某个wordpress注册账号，另一个客户登录看是否同步？  
负载均衡后，客户端打开网页，发表文章和图片，另一个客户端查看内容是否完整？  

&emsp;  &nbsp;