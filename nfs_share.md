# Ubuntu 作業系統中，在 NFS 上分享目錄

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

```
sudo systemctl enable nfs-kernel-server
sudo systemctl restart nfs-kernel-server
```

後面是還有一些防火牆的設定，可以看狀況決定要不要做，如果都是在內網操作(服務沒有任何其它對外的 public port)的話可以不用。

# MacOS 下，把分享的目錄帶進來

參考資料：[Display all exported file systems from an NFS server](https://www.cyberciti.biz/faq/howto-see-shares-on-nfs-server-exported-filesystems/)

用以下的指令可以看遠端的伺服器有分享哪些東西出來：

```
showmount -e 192.168.1.12
```

這東西也可以用來檢查自己的服務是不是有不小心分享了不該分享的東西出來。

參考資料：[macOS (OS X) Nfs mount Command](https://www.cyberciti.biz/faq/apple-mac-osx-nfs-mount-command-tutorial/)
