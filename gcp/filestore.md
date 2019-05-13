## How to mount filestore:

Install NFS package:
```
sudo apt-get -y install nfs-common
```

Get shell command and execute it on the server:
```
eval $(gcloud filestore instances list --format=json | jq -r '.[] | @sh "fs_ip=\(.networks[0].ipAddresses[0]) fs_name=\(.fileShares[0].name)"')
echo mount $fs_ip:/$fs_name /mnt
```
