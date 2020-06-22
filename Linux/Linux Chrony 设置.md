# Linux chrony
## chrony是一个开源的自由软件，ep.CentOS 7 or Rehl 7的系统里，已经是默认服务了。默认配置文件在/etc/chrony.conf中。它能够保持系统时间与时间服务器NTP的同步，并且让时间始终保持同步。
***
chrony有两个核心组件，分别是：
- chronyd 守护进程，主要用于调整内核运行中运行的系统时间与时间服务器的同步。它确定计算机增减时间的比率。对此进行调整和补偿。
- chronyc 提供用户界面，用于监控性能并进行多样化配置。
***
通常系统已经默认安装。若为安装，可通过yum进行安装。
```
yum install chrony
```
加入自启动
```
systemctl enable chronyd.service
systemctl restart chronyd.service
ststus
```
防火墙firewalld设置
```
firewall-cmd —add-service=ntp —permanent
firewall-cmd —reload
#因为NTP使用123/UDP端口协议，所以运行NTP服务即可
```
***
系统默认配置文件的一些说明：
```
cat /etc/chrony.conf

# 使用pool.ntp.org项目中的公共服务器。以server开，理论上你想添加多少时间服务器都可以。
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst

# 根据实际时间计算出服务器增减时间的比率，然后记录到一个文件中，在系统重启后为系统做出最佳时间补偿调整。
driftfile /var/lib/chrony/drift

# chronyd根据需求减慢或加速时间调整，
# 在某些情况下系统时钟可能漂移过快，导致时间调整用时过长。
# 该指令强制chronyd调整时期，大于某个阀值时步进调整系统时钟。
# 只有在因chronyd启动时间超过指定的限制时（可使用负值来禁用限制）没有更多时钟更新时才生效。
makestep 1.0 3

# 将启用一个内核模式，在该模式中，系统时间每11分钟会拷贝到实时时钟（RTC）。
rtcsync

# Enable hardware timestamping on all interfaces that support it.
# 通过使用hwtimestamp指令启用硬件时间戳
#hwtimestamp eth0
#hwtimestamp eth1
#hwtimestamp *

# Increase the minimum number of selectable sources required to adjust
# the system clock.
#minsources 2

# 指定一台主机、子网，或者网络以允许或拒绝NTP连接到扮演时钟服务器的机器
#allow 192.168.0.0/16
#deny 192.168/16

# Serve time even if not synchronized to a time source.
local stratum 10

# 指定包含NTP验证密钥的文件。
#keyfile /etc/chrony.keys

# 指定日志文件的目录。
logdir /var/log/chrony

# Select which information is logged.
#log measurements statistics tracking
```
***
设置时区，查看系统当前时区
```
timedatectl
      Local time: Fri 2018-2-29 13:31:04 CST
  Universal time: Fri 2018-2-29 05:31:04 UTC
        RTC time: Fri 2018-2-29 08:17:20
       Time zone: Asia/Shanghai (CST, +0800)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: no
      DST active: n/a

如果你当前的时区不正确，请按照以下操作设置。

查看所有可用的时区：

timedatectl list-timezones

筛选式查看在亚洲S开的上海可用时区：

timedatectl list-timezones |  grep  -E “Asia/S.*”

Asia/Sakhalin
Asia/Samarkand
Asia/Seoul
Asia/Shanghai
Asia/Singapore
Asia/Srednekolymsk

设置当前系统为Asia/Shanghai上海时区：

timedatectl set-timezone Asia/Shanghai

设置完时区后，强制同步下系统时钟：

chronyc -a makestep
200 OK1

```
***
在服务集群之间的系统时间同步
因为在生产环境中，网络基本为内网结构。一般若要同步时间可到服务端同步时间即可。   
对服务端
```
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst

并添加以下内容：（表示与本机同步时间）

server 10.28.204.66 iburst

这样我们需求的一台内网时间服务器已经配置完毕。

同样在客户端注释掉其他server，并在客户端（10.28.204.65）添加以下：

server 10.28.204.66 iburst

```
***
其它给予chrony常用命令
```
$ chronyc sources -v

查看时间同步源状态：

$ chronyc sourcestats -v

设置硬件时间

硬件时间默认为UTC：

$ timedatectl set-local-rtc 1

启用NTP时间同步：

$ timedatectl set-ntp yes

校准时间服务器：

$ chronyc tracking
```
##### 需要注意的是，配置完/etc/chrony.conf后，需重启chrony服务，否则可能会不生效。
