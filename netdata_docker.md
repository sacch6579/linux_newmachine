# Netdata 監控

## 如果需要把舊的版本刪掉
[Updates for most systems](https://learn.netdata.cloud/docs/maintaining/update-netdata-agents#updates-for-most-systems)

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
  
  `hostname` 要改成現在在執行的 host 的名稱，然後最後面那個 PGID 的值執行這個指令來取得 `grep docker /etc/group | cut -d ':' -f 3`。
  
  ## GPU
參考：[Nvidia GPU collector](https://learn.netdata.cloud/docs/data-collection/monitor-anything/hardware/nvidia_smi-python.d.plugin)

