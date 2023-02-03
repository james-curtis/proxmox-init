# proxmox-init

换源 & 安装基础软件
```bash
sed -i 's|^deb http://ftp.debian.org|deb https://mirrors.ustc.edu.cn|g' /etc/apt/sources.list
sed -i 's|^deb http://security.debian.org|deb https://mirrors.ustc.edu.cn/debian-security|g' /etc/apt/sources.list
source /etc/os-release
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/pve $VERSION_CODENAME pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list
rm -f /etc/apt/sources.list.d/pve-enterprise.list
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/ceph-quincy bullseye main" > /etc/apt/sources.list.d/ceph_mirror.list

apt update -y && apt upgrade -y && apt install fio git openvswitch-switch bcache-tools supervisor -y
```

安装ceph之前执行
```bash
while true; do rm -f /etc/apt/sources.list.d/ceph.list; sleep 0.2; done
```

开机自动恢复调优参数，文件位置 `/etc/supervisor/conf.d/bcache-tuning.conf`
```
[program:bcache-tuning]
command=/usr/bin/bcache-tuning
startsecs=0
startretries=0
user=root
```

#### 工具推荐

- bcache-status：https://gist.github.com/adamryczkowski/8b9a1e55ac85a2ee83e2
- bcache-tuning：https://github.com/james-curtis/bcache-tuning
