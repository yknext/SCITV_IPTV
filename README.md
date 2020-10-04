```
# SCITV_IPTV
chengdu telecom iptv with udpxy and pon stick
双网卡下IPTV配置，默认网关，静态路由，dns解析
目标机器enp1s0 为内网接口，连接路由器，设置默认网关用于互联网连接
目标机器enp2s0 为IPTV接口，连接光猫IPTV接口，配置静态路由，非默认网关

1./etc/sysconfig/network-scripts/ifcfg-enp2s0
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
#不设置默认网关，双网卡设置会冲突导致无法连接enp1s0互联网
DEFROUTE=no
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=no
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp2s0
UUID=5a39d007-478a-428f-af16-0f11371274c9
DEVICE=enp2s0
ONBOOT=yes
#配置光猫Mac地址
MACADDR=54:93:59:*******

2./etc/dhcp/dhclient.conf
#防止IPTV dns更小刷掉/etc/resolv.conf中正常网络dns导致无法上网
prepend domain-name-servers 223.6.6.6,223.5.5.5;
interface "enp2s0" {
  # 发送终端名，这个抓盒子发的包直接送出去就好了，是个32字节的字符串
  send host-name "00109**********************";
  # 发送机顶盒的MAC地址，我的华为的盒子是54:93:59开头的
  send dhcp-client-identifier "54:93:59:*****";
  # 电信用了Option60验证终端是否为盒子，按照抓包出来的字符串原样发送
  send vendor-class-identifier "SCITV";
  request subnet-mask, classless-static-routes, static-routes,
          routers, domain-name-servers, host-name, domain-name,
          interface-mtu, broadcast-address, ntp-servers,
          dhcp-lease-time, dhcp-server-identifier,
          dhcp-renewal-time, dhcp-rebinding-time, domain-search;
  require subnet-mask, domain-name-servers, routers;
}

3./etc/NetworkManager/NetworkManager.conf
#设置dhcp使用dhclient，不设置dhclient默认不会启动(centos8下测试)
dhcp=dhclient
#设置后执行ps -ef 查看存在以下进程
#/sbin/dhclient -d -q -sf /usr/libexec/nm-dhcp-helper -pf /run/NetworkManager/dhclient-enp1s0.pid -lf /var/lib/NetworkManager/dhclient-b862fde9-d76d-4436-ba6e-2f53e365d85d-enp1s0.lease -cf /var/lib/NetworkManager/dhclient-enp1s0.conf enp1s0
#/sbin/dhclient -d -q -sf /usr/libexec/nm-dhcp-helper -pf /run/NetworkManager/dhclient-enp2s0.pid -lf /var/lib/NetworkManager/dhclient-5a39d007-478a-428f-af16-0f1c371274c9-enp2s0.lease -cf /var/lib/NetworkManager/dhclient-enp2s0.conf enp2s0

4./etc/NetworkManager/dispatcher.d/30-iptv-route.sh
#IPTV网口启动后,自动设置静态路由配置,组播段和源服务器IP段(成都电信)
iptv_interface=enp2s0
action=up
#set static route
if [[ ( $1 == $iptv_interface ) && ( $2 == $action ) ]]
then
  ip route add 182.138.0.0/15 via $DHCP4_ROUTERS dev enp2s0
  ip route add 224.0.0.0/4    via $DHCP4_ROUTERS dev enp2s0
fi
pon stick
成都电信，IPTV 单播vlan 43 组播3990，语音电话 vlan45
```
