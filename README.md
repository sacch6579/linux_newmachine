# Ubuntu 20.04 設定 docker + Spring boot 網頁服務機器步驟。
這裏記錄在 Ubuntu 20.04 上設定會使用到 docker 網頁服務的新機器要做的工作。
有些設定是在 docker 與 Spring boot 下的內容便不包括在此說明中。

## 安裝 ssh 
如果 Ubuntu 裏並沒有安裝 ssh 就得先安裝才有辦法利用 ssh 遠端登入，參考
1. [Ubuntu 作業系統SSH安裝及設定教學](https://hackmd.io/@W855Yo-6R22n28iWVYZdVw/HJwUo8yzT)

```
apt update
apt upgrade
apt install openssh-server
```
如果要修改 ssh 的設定就修改 `/etc/ssh/sshd_config`
然後如果修改完要重啟 ssh 服務就執行 `/etc/init.d/ssh restart`
然後最後要確認 openSSH 的服務是否有被預設啟動 `systemctl is-enabled ssh`
要預設啟動服務就下 `sudo systemctl enable ssh` 指令

## 新增使用者

```bash
sudo adduser <使用者名稱>
```

接著就用 passwd 設定使用者密碼，去 /etc/sudoers 設定權限如果你的使用者需要較高權限。

## 改變時區

參考[修正Ubuntu的時區設定][1]的方法：

```bash
sudo dpkg-reconfigure tzdata
```

[1]: https://help.ubuntu.com/community/UbuntuTime	"修正Ubuntu的時區設定"


## 修正 locale 的警告
```
sudo locale-gen en_US.UTF-8
sudo update-locale LANG=en_US.UTF-8
```

## 安裝 docker

參照[在 Ubuntu 安裝 docker][2]的方法：

先設定好 repository：

```bash
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

新增 docker 的官方 GPG key：

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

設定穩定版的 repository：

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

安裝 docker engine

```
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

用這個指令確認是不是安裝好了

```bash
sudo docker run hello-world
```

[2]: https://docs.docker.com/engine/install/ubuntu/

## docker 使用者的權限
用下面這個指令來加現在的使用者到 docker 的使用者列表中
```bash
sudo usermod -aG docker ${USER}
```

## docker-compose

[How To Install and Use Docker Compose on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04)

## git 密碼
參考資料：[How to save username and password in Git?](https://stackoverflow.com/questions/35942754/how-to-save-username-and-password-in-git)

現在 git 需要 Personal access tokens [Generate new token](https://github.com/settings/tokens/new)，當你有一台新機器需要登入的密碼時，便到這裏去申請自己的新 token，給它一個名字和有效期限，然後到你的機器上執行以下的指令：


```bash
git config --global credential.helper store
```

接著便可以用 git clone 嚐試去 clone 一個你自己的專案，它會問帳號密碼，密碼就輸入你所產生的新 token。Token 的產生可以到下面這個[網頁](https://github.com/settings/tokens)：


好了之後，可以到被 clone 下來的目錄中再執行一次

```
git pull origin
```

確認一下是不是不會再詢問密碼。

## Java openjdk 11
如果你的專案有用到 Spring Boot 或是其它 Java 的應用，用以下的指令來安裝 openjdk 11 以利之後編譯 java 時所需。

```bash
sudo apt-get install -y openjdk-11-jdk
```

## zerotier

### 安裝 zerotier

```
curl -s https://install.zerotier.com | sudo bash
```

或是

```
curl -s 'https://raw.githubusercontent.com/zerotier/ZeroTierOne/master/doc/contact%40zerotier.com.gpg' | gpg --import && \
if z=$(curl -s 'https://install.zerotier.com/' | gpg); then echo "$z" | sudo bash; fi
```
做好之後機器就會得到一個10碼獨一無二的代碼。

### 加入 zerotier
到 [https://my.zerotier.com/](https://my.zerotier.com/) 去查看自己創立好的 zerotier 網段，把機器加到 zerotier 的網域中管理。

要加入 zerotier 輸入下面的指令，後面接16碼要加入的網路代碼，然後到 zerotier 網路端去許可該節點進入。下面的指令也包括了離開以到列出所有網路的指令：

```bash
sudo zerotier-cli join ################
sudo zerotier-cli leave ################
sudo zerotier-cli listnetworks
sudo zerotier-cli status
```

[參考資料](https://zerotier.atlassian.net/wiki/spaces/SD/pages/29065282/Command+Line+Interface+zerotier-cli)


## 更改 hostname
`sudo hostnamectl set-hostname <想要改的名稱>`

## Netdata

管理 netdata 儲存資料的時間：[Change how long Netdata stores metrics](https://learn.netdata.cloud/guides/longer-metrics-storage)

到下面這個檔案：

```
/etc/netdata/netdata.conf
```

去修改 history 那一行，我習慣弄到五天的時間長度，估計大概是 86400 x 5 = 432000 秒。

```
history = 432000
page cache size = 320
dbengine disk space = 2560
```

現在暫時我的設定是這樣。之後再討論是不是ok... 

### 在有 nVidia GPU 的機器上啟動 GPU 監控的功能

下面這一頁有提到需要修改 `go.d/nvidia_smi.conf` 不過我看了一下好像沒有什麼需要改。

[啟用 nVidia GPU 監控](https://app.netdata.cloud/spaces/1000lpr/rooms/all-nodes/nodes/49a82e14-1ee5-448d-ad13-870ff76914ab#metrics_correlation=false&after=-900&before=0&utc=Asia%2FTaipei&offset=%2B8&timezoneName=Taipei&modal=&modalTab=&modalParams=&selectedIntegrationCategory=data-collection.hardware-devices-and-sensors&integrationsModalOpen=true&selectedIntegration=go.d.plugin-nvidia_smi-Nvidia_GPU&selectedIntegrationTab=&96ee29b6-e81f-4cae-abcb-41e0519623fb-49a82e14-1ee5-448d-ad13-870ff76914ab-chartName=menu_docker_submenu_containers)

安裝 `apt install mlocate` 使用 locate 找到 `go.d.conf` 的位置，系統很小的話應該一下子就找到了。
找到有 nvidia 的那行改掉，讓它啟動。

[重啟netdata服務](https://learn.netdata.cloud/docs/maintaining/starting-and-stopping-netdata-agents)

## email

如果你需要發送 email 給自己或是其它同伴：

```
sudo apt-get install -y ssmtp
```

然後用 `` sudo vi /etc/ssmtp/ssmtp.conf`` 編輯郵件的設定檔。

```config
# 接收系統郵件的 Email (應該也是收信的 email?)
root=<your email>@gmail.com

# 使用 GMail 的 MTA 送信 
mailhub=smtp.gmail.com:587

# 設定 hostname
hostname=<your hostname>

# 允許使用者設定 Email 的 From 欄位
FromLineOverride=YES

# Google 帳號與密碼
AuthUser=<your email>@gmail.com
AuthPass=<your password>

# 啟用安全加密連線
UseSTARTTLS=YES
UseTLS=YES

# 輸出除錯資訊
Debug=YES
```

這上面的 password 要到 Gmail security 來設定，到 [Gmail Security](https://myaccount.google.com/u/3/security) 選取 App passwords 來新增應用程式密碼。

[Ubuntu 使用 ssmtp 透過 Gmail 發送 email](https://noter.tw/50/ubuntu-使用-ssmtp-透過-gmail-發送-email/)

[Linux 使用 SSMTP 與 GMail 以指令或程式自動寄信教學](https://blog.gtwang.org/linux/linux-send-mail-command-using-ssmtp-and-gmail/)

[以 Java 執行 SMTP 來傳送電子郵件](https://docs.aws.amazon.com/zh_tw/ses/latest/DeveloperGuide/send-using-smtp-java.html)


## 燒機測試
[參考網址](http://takashi0922.blogspot.com/2015/07/ubuntutest-your-ubuntu-computer-ubuntu.html)

```
sudo apt-get install -y stress
```
用44線程燒cpu、三線程燒記憶體、一線程燒硬碟，時間3600秒(如果是分鐘就寫 M ，比如 1440M 就是一天)。
```
stress -c 44 -m 3 -d 1 -t 3600s
```

## 檢查系統問題

[How to View System Log Files on Ubuntu 20.04 LTS](https://vitux.com/view-system-log-files-ubuntu/)

下面這個指令可以看現在的系統 log，

```bash
sudo cat /var/log/syslog
```

[Debugging System Crash](https://help.ubuntu.com/community/DebuggingSystemCrash)

下面這個則是看 crash log，可以看到很多天前的資料。
```bash
sudo vi /var/log/kern.log
```
## NFS 設定
參考資料：
1. [Network File System (NFS)](https://ubuntu.com/server/docs/network-file-system-nfs)

## 掛載磁碟陣列

用軟體掛載磁碟陣列的參考文件 [Ubuntu 20.04 Root on ZFS](https://openzfs.github.io/openzfs-docs/Getting%20Started/Ubuntu/Ubuntu%2020.04%20Root%20on%20ZFS.html) ，第一碟用的是 SSD 比較不怕掛掉，剩下的資料就習慣掛到 `/data` 去，用軟體的方式去掛載多顆本機的硬碟。

另外兩個參考文件：

[Z 檔案系統 (ZFS)](https://docs.freebsd.org/zh-tw/books/handbook/zfs)

[Troubleshooting ZFS](https://openzfs.github.io/openzfs-docs/Basic%20Concepts/Troubleshooting.html)

有時候沒有掛載要重新掛載的話要用這個

```
sudo zpool import data
```

## 安全性

### 限制登入網段
[限制 Linux 的 SSH 連線設定](https://cynthiachuang.github.io/Linux_SSH_Access_Control/)

[Restrict SSH login to a specific IP or host](https://docs.rackspace.com/support/how-to/restrict-ssh-login-to-a-specific-ip-or-host/)
修改
```
/etc/hosts.allow
/etc/hosts.deny
```

這兩個檔案來限制與允許由外部進來的連線。比方說在 `/etc/hosts.allow` 中加入
```
sshd:192.168.31.0/24,192.168.99.10/16
```
就是允許 `192.168.31.0/24` 還有 `192.168.99.10/16` 這兩個網段登入的動作。既然有允許就有拒絕，那麼在 `/etc/hosts.deny` 中加入
```
sshd:ALL
```
便可以拒絕除了 `/etc/hosts.allow` 定義的網段以外的登入動作。要設定這些之前要非常小心，因為不小心就有可能讓自己登不進去。

### DDoS 攻擊
[How to check for and stop DDoS attacks on Linux](https://www.techrepublic.com/article/how-to-check-for-and-stop-ddos-attacks-on-linux/)

[How to stop/prevent SSH bruteforce](https://serverfault.com/questions/594746/how-to-stop-prevent-ssh-bruteforce)



## 資料庫備份

備份可能會用到的資源：

[How to Automate MySQL Database Backups in Linux](https://sqlbak.com/blog/how-to-automate-mysql-database-backups-in-linux)

[Linux Time Machine](https://github.com/cytopia/linux-timemachine)



```bash
sudo apt-get update 
sudo apt-get install -y postfix mailutils
```



```bash
# Backup storage directory 
backupfolder=/var/backups
# Notification email address 
recipient_email=<username@mail.com>
# MySQL user
user=<user_name>
# MySQL password
password=<password>
# Number of days to store the backup 
keep_day=30 
sqlfile=$backupfolder/all-database-$(date +%d-%m-%Y_%H-%M-%S).sql
zipfile=$backupfolder/all-database-$(date +%d-%m-%Y_%H-%M-%S).zip 
# Create a backup 
sudo mysqldump -u $user -p$password --all-databases > $sqlfile 
if [ $? == 0 ]; then
  echo 'Sql dump created' 
else
  echo 'mysqldump return non-zero code' | mailx -s 'No backup was created!' $recipient_email  
  exit 
fi 
# Compress backup 
zip $zipfile $sqlfile 
if [ $? == 0 ]; then
  echo 'The backup was successfully compressed' 
else
  echo 'Error compressing backup' | mailx -s 'Backup was not created!' $recipient_email 
  exit 
fi 
rm $sqlfile 
echo $zipfile | mailx -s 'Backup was successfully created' $recipient_email 
# Delete old backups 
find $backupfolder -mtime +$keep_day -delete
```

## 遠端登入免密碼
其實很簡單，如果還沒產生 key，就先執行

```java
ssh-keygen
```

有了 key 之後，就可以：

```java
ssh-copy-id <帳號>@<遠端的 hostname/IP>
```
