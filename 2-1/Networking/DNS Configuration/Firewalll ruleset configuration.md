sajib1616@iit-du-ASUS-EXPERTCENTER-D700ME-D500ME:~$ sudo iptables -nvxL
[sudo] password for sajib1616: 
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
    pkts      bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
    pkts      bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
    pkts      bytes target     prot opt in     out     source               destination         
sajib1616@iit-du-ASUS-EXPERTCENTER-D700ME-D500ME:~$ 




 ping 172.17.100.100
  459  ping -4 172.17.100.100
  460  ping -4 172.17.100.100/24
  461  ping -4 172.17.100.100
  462  ping -I 172.17.100.16 172.17.100.16
  463  ping -I 172.17.100.16 172.17.100.100
  464  ping -I 172.17.100.1 172.17.100.100
  465  ping -I 10.34.0.163 172.17.100.100
  466  ping -I 172.17.100.16 172.17.100.100
  467  sudo iptables -nvxL
  468  sudo iptables -nvxL INPUT
  469  sudo iptables -A INPUT
  470  sudo iptables -nvxL INPUT
  471  sudo iptables -D INPUT
  472  sudo iptables -nvxL INPUT
  473  sudo iptables -A INPUT -d 127.1.2.3 -j DROP
  474  sudo iptables -nvxL INPUT
  475  ping 127.0.0.1
  476  ping 127.1.2.3
sudo iptables -A INPUT -i  lo -j DROP