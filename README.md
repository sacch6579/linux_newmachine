# linux_newmachine
這裏記錄在 Ubuntu 20.04 上設定會使用到 docker 的新機器要做的工作。



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

要加入 zerotier 輸入下面的指令，後面接16碼要加入的網路代碼，然後到 zerotier 網路端去許可該節點進入。下面的指令也包括了離開以到列出所有網路的指令：

```bash
zerotier-cli join ################
zerotier-cli leave ################
zerotier-cli listnetworks
zerotier-cli status

```

[參考資料](https://zerotier.atlassian.net/wiki/spaces/SD/pages/29065282/Command+Line+Interface+zerotier-cli)

## email

```
sudo apt-get install -y ssmtp
```
[Ubuntu 使用 ssmtp 透過 Gmail 發送 email](https://noter.tw/50/ubuntu-使用-ssmtp-透過-gmail-發送-email/)
[Linux 使用 SSMTP 與 GMail 以指令或程式自動寄信教學](https://blog.gtwang.org/linux/linux-send-mail-command-using-ssmtp-and-gmail/)


## 燒機測試
[參考網址](http://takashi0922.blogspot.com/2015/07/ubuntutest-your-ubuntu-computer-ubuntu.html)

```
sudo apt-get install -y stress
```
用44線程燒cpu、三線程燒記憶體、一線程燒硬碟，時間3600秒(如果是分鐘就寫 M ，比如 1440M 就是一天)。
```
stress -c 44 -m 3 -d 1 -t 3600s
```