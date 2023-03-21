# Netdata 監控

在 [Install Netdata with Docker](https://learn.netdata.cloud/docs/agent/packaging/docker) 文章中看 Docker container names resolution 這段。

```
docker run -d --name=netdata \
  --hostname=proxy \
  -p 19999:19999 \
  -v netdataconfig:/etc/netdata \
  -v netdatalib:/var/lib/netdata \
  -v netdatacache:/var/cache/netdata \
  -v /etc/passwd:/host/etc/passwd:ro \
  -v /etc/group:/host/etc/group:ro \
  -v /proc:/host/proc:ro \
  -v /sys:/host/sys:ro \
  -v /etc/os-release:/host/etc/os-release:ro \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  --restart unless-stopped \
  --cap-add SYS_PTRACE \
  --security-opt apparmor=unconfined \
  -e NETDATA_CLAIM_TOKEN=xVWSI9HIA6cW8Yi3jNaQhTf3xlfgHa3YjOEh6vt3XWfWEFhSfa9AGZKkLiC3sjvYFKO9f3qdTasK1OZL3WErZJjgs438QRx_Mpuc6_4n5VSLntVfyyjhjIARH4Wh2B_O6y3M \
  -e NETDATA_CLAIM_URL=https://app.netdata.cloud \
  -e PGID=998 \
  netdata/netdata
  ```
  
  `hostname` 要改成現在在執行的 host 的名稱，然後最後面那個 PGID 的值執行這個指令來取得 `grep docker /etc/group | cut -d ':' -f 3`。
