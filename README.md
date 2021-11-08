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
sudo apt-get install docker-ce docker-ce-cli containerd.io
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

接著便可以用 git clone 嚐試去 clone 一個你自己的專案，它會問帳號密碼，密碼就輸入你所產生的新 token。

好了之後，可以到被 clone 下來的目錄中再執行一次

```
git pull origin
```

確認一下是不是不會再詢問密碼。

## Java openjdk 11
如果你的專案有用到 Spring Boot 或是其它 Java 的應用，用以下的指令來安裝 openjdk 11 以利之後編譯 java 時所需。

```bash
sudo apt-get install openjdk-11-jdk
```
