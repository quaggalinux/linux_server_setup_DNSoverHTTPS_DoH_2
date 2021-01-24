# linux_server_setup_DNSoverHTTPS_DoH_2
linux服务器另外一种设置DNS查询为DNS over HTTPS(DoH)方式   
   
之前的一个repo已经介绍了利用cloudflare程序设置linux服务器DNS查询为DNS over HTTPS(DoH)方式,下面介绍另外一种方式,利用了DNSproxy这个程序,DNSproxy是AdguardTeam在github上的一个项目,原理与cloudflare程序差不多   
   
首先下载dnsproxy的linux运行版   
   
$su   
   
#cd /   
   
具体机器的系统架构及操作系统版本可查询https://github.com/AdguardTeam/dnsproxy/releases 连接   
   
#wget https://github.com/AdguardTeam/dnsproxy/releases/download/v0.33.8/dnsproxy-linux-amd64-v0.33.8.tar.gz   
   
解压并拷贝到运行目录  
  
#tar xvf dnsproxy-linux-amd64-v0.33.8.tar.gz  
  
#mv -f linux-amd64/dnsproxy /usr/bin/  
  
检查运行权限,如果没有就置运行权限     
   
#ls -l /usr/bin/dnsproxy   
   
运行程序验证可用   
   
#dnsproxy --version   


------配置Systemd服务启动方式------

确保是root用户并在根目录,检查是否已经有system目录   
#ls -l /usr/lib/systemd   
   
没有的的话创建system目录   
#mkdir /usr/lib/systemd/system   
   
编辑dnsproxy服务文件   
   
#nano /usr/lib/systemd/system/dnsproxy.service   
   
写入以下内容,dnsproxy的各种参数可以参考https://github.com/AdguardTeam/dnsproxy 连接的介绍,关键是 -u 参数紧跟的DoH服务器可以换成其他的,甚至是像IPv6的DoH服务器地址https://[2606:4700:4700::1001]/dns-query , 各个DoH服务商DoH的IP地址大家可以自己查,但是务必是查找服务商的IP已经直接签发IP证书的,就是可以直接https://ip 的这种,验证很简单,就是把这个地址按照https://ip/dns-query 格式放到浏览器浏览,如果正常出现小锁头而没有安全告警就是这个IP地址直接签发了证书,如果还用域名连接DoH查DNS的话,底层没有办法脱离原来DNS的限制,等于整台linux服务器还是受制于原来的DNS查询方式   
   
[Unit]   
Description=dnsproxy   
After=network.target   
   
[Service]   
TimeoutStartSec=30   
ExecStart=/usr/bin/dnsproxy -l 127.0.0.1 -p 53 -u https://9.9.9.10/dns-query -u https://1.0.0.1/dns-query  
ExecStop=/bin/kill $MAINPID   
   
[Install]   
WantedBy=multi-user.target   
   
保存并退出
   
设置开机启动dnsproxy    
#systemctl enable dnsproxy    
   
启动dnsproxy服务  
#systemctl start dnsproxy
   
检查服务状态  
#systemctl status dnsproxy  
  
检查网络连接状态及dnsproxy是否有在监听端口53  
  
#netstat -tunap  
   
检查DNS监听服务器地址  
   
#nslookup www.163.com   
应该输出下面的DNS服务器地址信息,但这时系统里面的DNS服务器设置仍然是原来的   
   
Server:		127.0.0.53  
Address:	127.0.0.53#53   
   
然后修改netplan或network-manager的DNS服务器为127.0.0.1及"::1"如果有IPv6的DoH服务器设置的话,如果修改有疑问可以参考之前我另外一个repo的介绍  
   
修改完成后重启机器  
  
然后检查系统DNS服务器是否显示为127.0.0.1  
   
#systemd-resolve --status  
  
这样配置完成后,系统任何地方再也没有任何传统DNS服务器IP地址的出现了,各种系统服务及程序就可以透明地使用纯DoH的查询服务了  
   



