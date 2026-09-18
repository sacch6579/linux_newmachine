下面是一版新的，可以直接存成例如 `README_ubuntu_26_04.md`。

```markdown
# Ubuntu 26.04 設定 Docker + Spring Boot 網頁服務機器步驟

本文記錄 Ubuntu 26.04 LTS 新機器常用初始化流程，適用於 Docker、Spring Boot、HOC/CMS、資料庫維運等服務主機。

## 1. 基本更新

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y curl ca-certificates gnupg git vim tmux htop unzip
```

## 2. SSH

若尚未安裝 SSH server：

```bash
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
sudo systemctl status ssh
```

修改設定：

```bash
sudo vim /etc/ssh/sshd_config
sudo systemctl restart ssh
```

## 3. 新增使用者

```bash
sudo adduser <username>
sudo usermod -aG sudo <username>
```

## 4. Hostname

```bash
sudo hostnamectl set-hostname <hostname>
```

## 5. 時區

```bash
sudo timedatectl set-timezone Asia/Taipei
timedatectl
```

## 6. Locale

若 SSH 登入出現 `LC_CTYPE: cannot change locale (UTF-8)`，建議使用 Ubuntu 內建的 `C.UTF-8`：

```bash
sudo update-locale LANG=C.UTF-8 LC_CTYPE=C.UTF-8
```

或使用英文 UTF-8：

```bash
sudo locale-gen en_US.UTF-8
sudo update-locale LANG=en_US.UTF-8 LC_CTYPE=en_US.UTF-8
```

登出後重新登入確認：

```bash
locale
locale -a
```

## 7. 安裝 Docker Engine 與 Docker Compose Plugin

移除可能衝突的舊套件：

```bash
sudo apt remove -y docker.io docker-compose docker-compose-v2 docker-doc docker-buildx podman-docker containerd runc || true
```

設定 Docker 官方 repository：

```bash
sudo apt update
sudo apt install -y ca-certificates curl

sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

sudo tee /etc/apt/sources.list.d/docker.sources > /dev/null <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

安裝 Docker：

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

確認：

```bash
sudo docker run hello-world
docker compose version
```

讓目前使用者可執行 Docker：

```bash
sudo usermod -aG docker "$USER"
```

登出再登入後確認：

```bash
docker ps
```

## 8. Java

新專案建議使用 Java 21：

```bash
sudo apt install -y openjdk-21-jdk
java -version
javac -version
```

若舊專案需要 Java 11，再另外安裝：

```bash
sudo apt install -y openjdk-11-jdk
```

## 9. Git Token

建議使用 Personal Access Token。

```bash
git config --global credential.helper store
git config --global user.name "<your name>"
git config --global user.email "<your email>"
```

第一次 `git clone` 或 `git pull` 時，密碼輸入 GitHub token。

## 10. ZeroTier

```bash
curl -s https://install.zerotier.com | sudo bash
sudo zerotier-cli join <network_id>
sudo zerotier-cli status
sudo zerotier-cli listnetworks
```

離開網路：

```bash
sudo zerotier-cli leave <network_id>
```

## 11. Netdata

若需要主機監控，可安裝 Netdata。安裝後可調整保留時間：

```bash
sudo vim /etc/netdata/netdata.conf
```

常用設定範例：

```ini
history = 432000
page cache size = 320
dbengine disk space = 2560
```

重啟：

```bash
sudo systemctl restart netdata
```

## 12. Email 發送

新機器建議優先使用 `msmtp`：

```bash
sudo apt install -y msmtp msmtp-mta mailutils
```

若只是簡單舊式設定，也可使用 `ssmtp`：

```bash
sudo apt install -y ssmtp
```

Gmail 請使用 App Password，不要使用主要登入密碼。

## 13. 壓力測試

```bash
sudo apt install -y stress
```

例如燒 CPU、記憶體、磁碟 1 小時：

```bash
stress -c 44 -m 3 -d 1 -t 3600s
```

## 14. 系統 log

Ubuntu 26.04 建議優先使用 journalctl：

```bash
journalctl -xe
journalctl -u ssh
journalctl -u docker
```

傳統 log 仍可看：

```bash
sudo less /var/log/syslog
sudo less /var/log/kern.log
```

## 15. NFS / ZFS

NFS：

```bash
sudo apt install -y nfs-common
```

ZFS：

```bash
sudo apt install -y zfsutils-linux
```

重新匯入 zpool：

```bash
sudo zpool import
sudo zpool import <pool_name>
```

## 16. SSH 安全限制

可在 `/etc/ssh/sshd_config` 搭配 `AllowUsers`、`Match Address` 或防火牆規則限制登入來源。

修改後：

```bash
sudo systemctl restart ssh
```

## 17. 常用檢查

```bash
ip addr
ip route
hostname -I
df -h
free -h
lsblk
systemctl status docker
docker ps
```
```
