## 1.判断ppp是否可用
首先，检查服务器是否支持搭建pptp，在shell中输入：
```
cat /dev/ppp
```
如果提示“没有那个设备或地址”则可用，但如果出现的是“Permission denied”就换个服务器吧
## 2.安装所需的软件包
- PPP：点对点协议
- PPTP:点对点通道协议
- IPtables:设定IP包转发规则

#### **（1)安装PPP**
```
yum install ppp
```
#### **(2)安装IPtables**
通常Linux系统中都自带安装好了iptables，如果没有可以使用yum进行安装

```
yum install iptables
```
#### ***tips:***
在CentOS 7或RHEL 7或Fedora中防火墙由firewalld来管理,不能直接使用service命令，而接下来的操作使用的是Service命令，假如采用传统命令请执行以下命令
```
systemctl stop firewalld
systemctl mask firewalld
yum install iptables-services
systemctl enable iptables
```
#### **(3)安装PPTP套件**
在centos中shell执行
```
wget http://poptop.sourceforge.net/yum/stable/rhel6/x86_64/pptpd-1.4.0-1.el6.x86_64.rpm
rpm -ivh pptpd-1.4.0-1.el6.x86_64.rpm
```
#### **(4)进行配置**
使用vi/vim编辑/etc/sysctl.conf文件,添加下面一行
```
net.ipv4.ip_forward = 1
```
复制并执行以下命令

```
sysctl -p
echo "localip 192.168.240.1" >> /etc/pptpd.conf
echo "remoteip 192.168.240.101-200" >> /etc/pptpd.conf
echo "ms-dns 8.8.8.8" >> /etc/ppp/options.pptpd
echo "ms-dns 8.8.4.4" >> /etc/ppp/options.pptpd
iptables --flush POSTROUTING --table nat
iptables --flush FORWARD
iptables -A INPUT -p tcp -m tcp --dport 1723 -j ACCEPT
iptables -A INPUT -p gre -j ACCEPT
iptables -t nat -A POSTROUTING -s 192.168.240.0/24 -o eth0 -j MASQUERADE
service iptables save
service pptpd restart
service iptables restart
chkconfig pptpd on
chkconfig iptables on

```
以上主要设置了VPN所使用的IP地址段、包转发规则等。VPN客户端将分配到的地址为：192.168.240.* 地址段，通过VPN服务器的eth0以太网接口转发对外的数据包（可以使用ifconfig查看本机对外通信的接口是否为eth0，若不是可自行更改）。

#### **（5）设置VPN账号和密码**
你任意添加VPN账户，使用Vim编辑文件 /etc/ppp/chap-secrets 一行一个账户，如：
```
username pptd password *
```
分别对应账户名、vpn服务、密码、ip(\*表示允许来自任意ip的连接)
现在可以开始在电脑或手机上连接vpn上网了

#### **(6)客户端连接**
具体的去百度，协议选pptp
