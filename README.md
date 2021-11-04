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
sudo locale-gen en_US.UTF-8
sudo update-locale LANG=en_US.UTF-8

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

用這個指令確認是不是安裝好了

```bash
sudo docker run hello-world
```

[2]: https://docs.docker.com/engine/install/ubuntu/

## docker 使用者的權限



## git 密碼
參考資料：[How to save username and password in Git?](https://stackoverflow.com/questions/35942754/how-to-save-username-and-password-in-git)

在 Linux 上 checkout 了一個 project 之後你想要之後都不用輸入密碼直接 git pull 和 git push，你得先執行下面的指令：

```
git config --global credential.helper store
```

然後再執行一次

```
git pull origin
```
接下來就不會再問你密碼了。
