# 利用 LINE notify 監控登入狀態

利用 crontab 來定期執行 script ，該 script 檢查過去24小時的登入狀態，並總結報告到 LINE Notify。

## 取得 LINE Notify TOKEN
參考資料：[LINE Notify API Document](https://notify-bot.line.me/doc/en/)
這部份就不再贅述。

## crontab 設定
參考資料：[Linux 系統上使用 crontab 工作排程](https://blog.gtwang.org/linux/linux-crontab-cron-job-tutorial-and-examples/)

## 安裝 aureport

參考資料：[How to track successful and failed login attempts in Linux](https://www.2daygeek.com/check-track-successful-failed-login-attempts-linux/)

在 Ubuntu 20.04 以下面的指令來安裝

```
sudo apt-get install auditd
```

安裝完之後用這樣的指令來啟動(安裝完之後不啟動好像它也會啟動)

```
sudo service auditd start
```

然後我們就可以用 `aureport -au -i` 這樣的指令來看從安裝至今的登入記錄了，包括登入失敗和登入成功的所有記錄。然而我們需要的是過去24小時的登入記錄，那麼我們需要執行類似像 `sudo aureport -au -i -ts 03/07/2023 00:00:00 -te 03/08/2023 00:00:00` 這樣有時間範圍的指令。在 crontab 裏如果用 root 身份執行不需要使用 sudo 所以到時候要把 sudo 拿掉。

那麼我們來寫個 script。下面是 `/usr/local/sbin/line_notify_login.sh` 接著把檔案改成可執行：

```bash
#!/bin/bash
SERVER="server name"
# 下面兩行先把時間給定出來
START="$(date --date="yesterday" +"%m/%d/%Y") 00:00:00"
END="$(date --date="today" +"%m/%d/%Y") 00:00:00"
# 取得 aureport 並存入 RESULT
RESULT=$(aureport -au -i -ts ${START} -te ${END})
# 處理一些統計
YES=$(echo "$RESULT" | grep " yes " | wc -l)
YES_AUTHS=$(echo "$RESULT" | grep " yes " | awk '{ print $4 }' | sort | uniq | sed ':a; N; $!ba; s/\n/,/g')
NO=$(echo "$RESULT" | grep " no " | wc -l)
NO_AUTHS=$(echo "$RESULT" | grep " no " | awk '{ print $4 }' | sort | uniq | sed ':a; N; $!ba; s/\n/,/g')
REPORT="${SERVER}: 登入成功 ${YES} 次，由帳號 ${YES_AUTHS}，登入失敗 ${NO} 次，由帳號 ${NO_AUTHS}"
#下面的 token要換成你自己的 line notify token
TOKEN="xxx"
SCRIPT="curl -X POST -H 'Authorization: Bearer ${TOKEN}' -F 'message=${REPORT}' https://notify-api.line.me/api/notify"
/bin/bash -c "$SCRIPT"
```

然後 crontab 那邊就設定成
```
30 08 * * *     /usr/local/sbin/line_notify_login.sh
```
可以讓它每天早上八點半的時候執行一次。
