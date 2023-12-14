# 在 NFS 上分享目錄

參考資料：[How to share folder with NFS in Ubuntu?](https://www.golinuxcloud.com/share-folder-with-nfs-ubuntu/#google_vignette)

以下是大概的步驟，有些可以不做。

```
sudo apt update -y
sudo apt install nfs-kernel-server -y
sudo mkdir /mnt/share
chmod 777 /mnt/share
sudo vi /etc/exports
```

`/mnt/share` 是你要分享出去的目錄

```
/mnt/share 192.168.0.*(rw,sync,subtree_check)
```

上面的 `192.168.0.*` 只是一個範例，可以定義要分享的目錄可以接受 client 的網段，避免莫名奇妙的人來存取。
