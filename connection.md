# 設定一台新的機器包括改變其 IP 地址

參考資料：
[How to Enable SSH on Ubuntu 20.04](https://linuxize.com/post/how-to-enable-ssh-on-ubuntu-20-04/)

安裝好 Ubuntu 之後，需要先讓它可以 ssh 

```
sudo apt update
sudo apt install openssh-server
sudo systemctl status ssh
sudo ufw allow ssh
```

# 修改 hostname
修改 `/etc/hostname`
然後重新開機

# 修改 IP address
[How to change the IP address on Ubuntu](https://linuxhint.com/change-ip-address-ubuntu/)

一般就是去用下面 UI 的那個修改 IP，然後修改完記得把 network 關掉再重開。
