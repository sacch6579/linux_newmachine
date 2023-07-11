# Netdata 監控

之前會使用用 docker 版來安裝 netdata 不過後來發現 docker 版沒有辦法自己更新，現在覺得還是本機安裝比較方便。

安裝 netdata 很簡單，只要在想要新增的 War Room 右上角按 Add Nodes 就會有安裝的指示。
安裝完之後可能 netdata 不會自動啟動，要用下面網頁面的令來設定讓它啟動。

[Start, stop, or restart the Netdata Agent](https://learn.netdata.cloud/docs/maintaining/starting-and-stopping-netdata-agents#:~:text=Using%20systemctl%20%2C%20service%20%2C%20or%20init.&text=This%20is%20the%20recommended%20way,run%20sudo%20systemctl%20restart%20netdata%20.)

基本上就是 
```
# 開機啟用 netdata
sudo systemctl enable netdata
# 開始 netdata 服務
sudo systemctl start netdata
```

## GPU
參考：[Nvidia GPU collector](https://learn.netdata.cloud/docs/data-collection/monitor-anything/hardware/nvidia_smi-python.d.plugin)


## Docker install

## 如果需要把舊的版本刪掉
[Updates for most systems](https://learn.netdata.cloud/docs/maintaining/update-netdata-agents#updates-for-most-systems)

首先把新的 image 拉下來
```
docker pull netdata/netdata:latest
```

然後停止並刪除舊的 container

```
docker stop netdata
docker rm netdata
```

## 安裝

在 [Install Netdata with Docker](https://learn.netdata.cloud/docs/agent/packaging/docker) 文章中看 Docker container names resolution 這段。

在 Netdata cloud 的 Manage War Room / Add Nodes 選單中，找到 docker 安裝片段：

```
docker run -d --name=netdata \
  <... 其它省略 ... > 
  --hostname=<你要加的host名稱> \
  -e PGID=998 \
  netdata/netdata
  ```
  
`hostname` 要改成現在在執行的 host 的名稱，然後最後面那個 PGID 的值執行這個指令來取得 

```
grep docker /etc/group | cut -d ':' -f 3
```
  

