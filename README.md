# proxmox-init

换源 & 安装基础软件
```bash
sed -i 's|^deb http://ftp.debian.org|deb https://mirrors.ustc.edu.cn|g' /etc/apt/sources.list
sed -i 's|^deb http://security.debian.org|deb https://mirrors.ustc.edu.cn/debian-security|g' /etc/apt/sources.list
source /etc/os-release
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/pve $VERSION_CODENAME pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list
rm -f /etc/apt/sources.list.d/pve-enterprise.list
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/ceph-quincy bullseye main" > /etc/apt/sources.list.d/ceph_mirror.list

apt install fio git openvswitch-switch bcache-tools -y
apt update -y && apt upgrade -y
```

安装ceph之前执行
```bash
while true; do rm -f /etc/apt/sources.list.d/ceph.list; sleep 0.2; done
```
