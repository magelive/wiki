# iptables命令

iptables的主要功能是实现对网络数据包进出设备及转发的控制。

当数据包需要进入设备、从设备中流出或者经该设备转发、路由时，都可以使用iptables进行控制。
 
## 基础

iptables中的“四表五链”及“堵通策略”。

1. “四表”是指，iptables的功能——filter、nat、mangle、raw。
    * filter： 控制数据包是否允许进出及转发（INPUT、OUTPUT、FORWARD）,可以控制的链路有input, forward, output
    * nat：控制数据包中地址转换，可以控制的链路有prerouting, input, output, postrouting
    * mangle：修改数据包中的原数据，可以控制的链路有prerouting, input, forward, output, postrouting
    * raw：控制nat表中连接追踪机制的启用状况，可以控制的链路有prerouting, output

　　_注：在centos7中，还有security表，不过这里不作介绍_


2. “五链”是指内核中控制网络的NetFilter定义的五个规则链，分别为
    * PREROUTING：路由前
    * INPUT：数据包流入口
    * FORWARD：转发管卡
    * OUTPUT：数据包出口
    * POSTROUTING：路由后

3. 堵通策略是指对数据包所做的操作，一般有两种操作——“通（ACCEPT）”、“堵（DROP）”，还有一种操作很常见REJECT.

谈谈REJECT和DROP之间的区别，Ming写了一封信，向Rose示爱。Rose如果不愿意接受，她可以不回应Ming,这个时候Ming不确定Rose是否接到了信；Rose也可以同样写一封信，在信中明确地拒绝Ming。前一种操作就如同执行了DROP操作，而后一种操作就如同REJECT操作。

 
## 命令的语法规则

```
iptables [-t table] COMMAND [chain] CRETIRIA -j ACTION

    -t table，是指操作的表，filter、nat、mangle或raw, 默认使用filter

    COMMAND，子命令，定义对规则的管理

　　chain, 指明链路

　　CRETIRIA, 匹配的条件或标准

　　ACTION,操作动作

　　例如，不允许10.8.0.0/16网络对80/tcp端口进行访问，

    iptables -A INPUT -s 10.8.0.0/16 -d 172.16.55.7 -p tcp --dport 80 -j DROP
    　　查看iptables列表

    iptables -nL
``` 

3.链管理
　　-N, --new-chain chain：新建一个自定义的规则链；

　　-X, --delete-chain [chain]：删除用户自定义的引用计数为0的空链；

　　-F, --flush [chain]：清空指定的规则链上的规则；

　　-E, --rename-chain old-chain new-chain：重命名链；

　　-Z, --zero [chain [rulenum]]：置零计数器；　　

　　-P, --policy chain target， 设置链路的默认策略

 

4.规则管理
　　-A, --append chain rule-specification：追加新规则于指定链的尾部；

　　-I, --insert chain [rulenum] rule-specification：插入新规则于指定链的指定位置，默认为首部；

　　-R, --replace chain rulenum rule-specification：替换指定的规则为新的规则；

　　-D, --delete chain rulenum：根据规则编号删除规则；

 

5.查看规则　
　　-L, --list [chain]：列出规则；

　　-v, --verbose：详细信息；

　　　　-vv， -vvv  更加详细的信息
　　-n, --numeric：数字格式显示主机地址和端口号；

　　-x, --exact：显示计数器的精确值；

　　--line-numbers：列出规则时，显示其在链上的相应的编号；

　　-S, --list-rules [chain]：显示指定链的所有规则；

　　查看规则的一般内容：

　　

6.匹配条件
匹配条件包括通用匹配条件和扩展匹配条件。

通用匹配条件是指针对源地址、目标地址的匹配，包括单一源IP、单一源端口、单一目标IP、单一目标端口、数据包流经的网卡以及协议。

扩展匹配条件指通用匹配之外的匹配条件。

6.1通用匹配条件

[!] -s, --source address[/mask][,...]：检查报文的源IP地址是否符合此处指定的范围，或是否等于此处给定的地址；

[!] -d, --destination address[/mask][,...]：检查报文的目标IP地址是否符合此处指定的范围，或是否等于此处给定的地址；

[!] -p, --protocol protocol：匹配报文中的协议，可用值tcp, udp, udplite, icmp, icmpv6,esp, ah, sctp, mh 或者 "all", 亦可以数字格式指明协议；
[!] -i, --in-interface name：限定报文仅能够从指定的接口流入；only for packets entering the INPUT, FORWARD and PREROUTING chains.

[!] -o, --out-interface name：限定报文仅能够从指定的接口流出；for packets entering the FORWARD, OUTPUT and POSTROUTING chains.

 

6.2扩展匹配条件

隐含扩展匹配条件
-p tcp：可直接使用tcp扩展模块的专用选项；
　　[!] --source-port,--sport port[:port] 匹配报文源端口；可以给出多个端口，但只能是连续的端口范围 ；

　　[!] --destination-port,--dport port[:port] 匹配报文目标端口；可以给出多个端口，但只能是连续的端口范围 ；

　　[!] --tcp-flags mask comp 匹配报文中的tcp协议的标志位；Flags are: SYN ACK FIN RST URG PSH ALL NONE；
　　　　mask：要检查的FLAGS list，以逗号分隔；
　　　　comp：在mask给定的诸多的FLAGS中，其值必须为1的FLAGS列表，余下的其值必须为0；

　　[!] --syn： --tcp-flags SYN,ACK,FIN,RST SYN

-p udp：可直接使用udp协议扩展模块的专用选项：

　　[!] --source-port,--sport port[:port]

　　[!] --destination-port,--dport port[:port]
-p icmp
　　[!] --icmp-type {type[/code]|typename}

　　　　0/0：echo reply

　　　　8/0：echo request

显式扩展匹配条件
　　必须用-m option选项指定扩展匹配的类型，常见的有以下几种，

1、multiport

以离散或连续的 方式定义多端口匹配条件，最多15个；

　　[!] --source-ports,--sports port[,port|,port:port]...：指定多个源端口；

　　[!] --destination-ports,--dports port[,port|,port:port]...：指定多个目标端口；


iptables -I INPUT -d 172.16.0.7 -p tcp -m multiport --dports 22,80,139,445,3306 -j ACCEPT
 



2、iprange

以连续地址块的方式来指明多IP地址匹配条件；

　　[!] --src-range from[-to]

　　[!] --dst-range from[-to]


# iptables -I INPUT -d 172.16.0.7 -p tcp -m multiport --dports 22,80,139,445,3306 -m iprange --src-range 172.16.0.61-172.16.0.70 -j REJECT
 



3、time

匹配数据包到达的时间

　　--timestart hh:mm[:ss]

　　--timestop hh:mm[:ss]

　　[!] --weekdays day[,day...]

　　[!] --monthdays day[,day...]

　　--datestart YYYY[-MM[-DD[Thh[:mm[:ss]]]]]

　　--datestop YYYY[-MM[-DD[Thh[:mm[:ss]]]]]

　　--kerneltz：使用内核配置的时区而非默认的UTC；

4、string
匹配数据包中的字符

　　--algo {bm|kmp}

　　[!] --string pattern

　　[!] --hex-string pattern

　　--from offset

　　--to offset


~]# iptables -I OUTPUT -m string --algo bm --string "gay" -j REJECT

5、connlimit
用于限制同一IP可建立的连接数目

　　--connlimit-upto n

　　--connlimit-above n


~]# iptables -I INPUT -d 172.16.0.7 -p tcp --syn --dport 22 -m connlimit --connlimit-above 2 -j REJECT

6、limit
限制收发数据包的速率

　　--limit rate[/second|/minute|/hour|/day]

　　--limit-burst number


~]# iptables -I OUTPUT -s 172.16.0.7 -p icmp --icmp-type 0 -j ACCEPT

7、state
限制收发包的状态

　　[!] --state state

　　INVALID, ESTABLISHED, NEW, RELATED or UNTRACKED.

　　NEW: 新连接请求；

　　ESTABLISHED：已建立的连接；

　　INVALID：无法识别的连接；

　　RELATED：相关联的连接，当前连接是一个新请求，但附属于某个已存在的连接；

　　UNTRACKED：未追踪的连接；

state扩展：

内核模块装载：
　　nf_conntrack
　　nf_conntrack_ipv4

手动装载：
　　nf_conntrack_ftp

追踪到的连接：
　　/proc/net/nf_conntrack

调整可记录的连接数量最大值：
　　/proc/sys/net/nf_conntrack_max

超时时长：
　　/proc/sys/net/netfilter/*timeout*








## 范例

1. 禁止111.206.136.250的数据进来
```
sudo iptables -t filter -A INPUT -s 111.206.136.250 -j REJECT
```

2. 列表规则
```
sudo iptables -L -n  --line-number
```

3. 删除规则
```
sudo iptables -D INPUT 5
```


## 参考

[iptables命令使用详解](https://www.cnblogs.com/vathe/p/6973656.html)