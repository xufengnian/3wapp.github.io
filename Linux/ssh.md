## SSH

### 参数

option | note
 --- | -----
 -D | SSH 建立一个 socket 监听 ,把数据转发到目标机器上
-p | 指定登录端口
-N | 只连接远程主机，不打开远程shell
-T | 不为这个连接分配TTY, 和 -N 参数一起使用，代表这个SSH连接只用来传数据，不执行远程操作
-f | SSH连接成功后，转入后台运行,如果想关闭,只能通过 kill 命令去杀掉进程
-q | 静默模式

## ssh 隧道

ssh隧道，所有的网络通讯都是加密的。又被称作端口转发，因为ssh隧道通常会绑定一个本地端口，所有发向这个端口的数据包，都会被加密并透明地传输到远端系统。

ssh隧道有3种类型：

* 动态端口转发（Socks 代理）
* 本地端口转发
* 远端端口转发

### 动态端口转发

动态端口允许通过配置一个本地端口，把通过隧道到数据转发到远端的所有地址。本地的应用程序需要使用Socks协议与本地端口通讯。此时SSH充当Socks代理服务器的角色。

* 命令格式

  `ssh -D [bind_address:]port`

* 参数说明

  `bind_address`: 指定绑定的IP地址，默认情况会绑定在本地的回环地址（即127.0.0.1），如果空值或者为*会绑定本地所有的IP地址，如果希望绑定的端口仅供本机使用，可以指定为localhost。

  `port`: 指定本地绑定的端口

* 使用场景

  假设X网络（192.168.18.0/24）有主机A（192.168.18.100）,Y网络（192.168.2.0/24）有主机B（192.168.2.100）和主机C（192.168.2.101），已知主机A可以连接主机B，但无法连接主机C。

  在主机A执行 `$ ssh -D localhost:8080 root@192.168.2.100` , 然后主机A上的应用程序就可以通过 `SOCKS5 localhost:8080` 访问主机C上的服务

* 常用命令

  ```
  # socket代理:
  ssh -qTfnN -D port remotehost
  ```

* 优点

  配置一个代理服务就可以访问远端机器和与其所在子网络的所有服务

* 缺点

  应用程序需要额外配置SOCKS代理，若应用程序不支持代理配置则无法使用

### 本地端口转发

通过SSH隧道，将一个远端机器能够访问到的地址和端口，映射为一个本地的端口

* 命令格式

  `ssh -L [bind_address:]port:host:hostport`

* 参数说明

  `bind_address` 指定绑定的IP地址，默认情况会绑定在本地的回环地址（即127.0.0.1），如果空值或者为*会绑定本地所有的IP地址，如果希望绑定的端口仅供本机使用，可以指定为localhost。

  `port` 指定本地绑定的端口

  `host` 指定数据包转发目标地址的IP，如果目标主机和ssh server是同一台主机时该参数指定为localhost

  `host_port` 指定数据包转发目标端口

* 使用场景

  假设X网络（192.168.18.0/24）有主机A（192.168.18.100）,Y网络（192.168.2.0/24）有主机B（192.168.2.100）和主机C（192.168.2.101），已知主机A可以连接主机B，但无法连接主机C。A主机需要访问C主机的VNC服务（5900端口）

  在A主机上建立本地转发端口5901, `$ ssh -L 5901:192.168.2.101:5900 root@192.168.2.100`, 然后本地vnc客户端通过5901端口打开c主机的vnc服务 , `$ open vnc://localhost:5901`

* 优点

  无需设置代理

* 缺点

  每个服务都需要配置不同的端口转发

### 远端端口转发

远程端口转发用于某些单向阻隔的内网环境，比如说NAT，网络防火墙。在NAT设备之后的内网主机可以直接访问公网主机，但外网主机却无法访问内网主机的服务。如果内网主机向外网主机建立一个远程转发端口，就可以让外网主机通过该端口访问该内网主机的服务。可以把这个内网主机理解为“内应”和“开门者”

* 命令格式

  `ssh -R [bind_address:]port:host:hostport`

* 参数说明

  `bind_address` 指定绑定的IP地址，默认情况会绑定在本地的回环地址（即127.0.0.1），如果空值或者为*会绑定本地所有的IP地址，如果希望绑定的端口仅供本机使用，可以指定为localhost。

  `port` 指定本地绑定的端口

  `host` 指定数据包转发源地址的IP，如果源主机和ssh server是同一台主机时该参数指定为localhost

  `host_port` 指定数据包转发源端口

* 使用场景

  假设X网络（192.168.18.0/24）有主机A（192.168.18.100）,Y网络（192.168.2.0/24）有主机B（192.168.2.100）和主机C（192.168.2.101），已知主机A可以通过SSH访问登录B主机，但反向直接连接被禁止，主机B和主机C可以相互访问。若主机C想访问主机A的VNC服务（5900端口）。

  在主机A执行如下命令，开放B主机远端端口转发。`$ ssh -R 5900:192.168.2.100:5901 root@192.168.2.100
  `, 然后主机C连接主机B的5901端口, `$ open vnc://192.168.2.100:5901`

* 优点

  可以穿越防火墙和NAT设备

* 缺点

  每个服务都需要配置不同的端口转发

### 如何禁止端口转发

设置ssh服务配置文件`/etc/ssh/sshd_config`: `AllowTcpForwarding no`