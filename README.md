# Ubuntu 20.04 設定 docker + Spring boot 網頁服務機器步驟。
這裏記錄在 Ubuntu 20.04 上設定會使用到 docker 網頁服務的新機器要做的工作。
有些設定是在 docker 與 Spring boot 下的內容便不包括在此說明中。

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
到 [https://my.zerotier.com/](https://my.zerotier.com/) 去查看自己創立好的 zerotier 網段，把機器加到 zerotier 的網域中管理。

要加入 zerotier 輸入下面的指令，後面接16碼要加入的網路代碼，然後到 zerotier 網路端去許可該節點進入。下面的指令也包括了離開以到列出所有網路的指令：

```bash
sudo zerotier-cli join ################
sudo zerotier-cli leave ################
sudo zerotier-cli listnetworks
sudo zerotier-cli status
```

[參考資料](https://zerotier.atlassian.net/wiki/spaces/SD/pages/29065282/Command+Line+Interface+zerotier-cli)

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
