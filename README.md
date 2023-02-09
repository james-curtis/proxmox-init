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


# OpenStack Ceph

Glance、Cinder、Zun、Nova、Gnocchi、Manila、Swift(RadosGW)对接ceph

## keyring复制
```bash
# 在部署机操作
mkdir -p /etc/kolla/config/cinder/cinder-backup
mkdir -p /etc/kolla/config/cinder/cinder-volume
mkdir -p /etc/kolla/config/glance
mkdir -p /etc/kolla/config/gnocchi
mkdir -p /etc/kolla/config/manila
mkdir -p /etc/kolla/config/nova
mkdir -p /etc/kolla/config/zun/zun-compute

# 在Proxmox shell操作
deploy=10.10.2.81 #部署机ip

# glance
ceph auth get client.glance | ssh ${deploy} tee /etc/kolla/config/glance/ceph.client.glance.keyring

# cinder
ceph auth get client.cinder | ssh ${deploy} tee /etc/kolla/config/cinder/cinder-volume/ceph.client.cinder.keyring
ceph auth get client.cinder | ssh ${deploy} tee /etc/kolla/config/cinder/cinder-backup/ceph.client.cinder.keyring
ceph auth get client.cinder-backup | ssh ${deploy} tee /etc/kolla/config/cinder/cinder-backup/ceph.client.cinder-backup.keyring

# zun
ceph auth get client.cinder | ssh ${deploy} tee /etc/kolla/config/zun/zun-compute/ceph.client.cinder.keyring

# nova
ceph auth get client.cinder | ssh ${deploy} tee /etc/kolla/config/nova/ceph.client.cinder.keyring

# gnocchi
ceph auth get client.gnocchi | ssh ${deploy} tee /etc/kolla/config/gnocchi/ceph.client.gnocchi.keyring

# manila
ceph auth get client.manila | ssh ${deploy} tee /etc/kolla/config/manila/ceph.client.manila.keyring

```
查看结果
```bash
(venv) root@ubuntu:/etc/kolla# tree
.
├── config
│   ├── cinder
│   │   ├── cinder-backup
│   │   │   ├── ceph.client.cinder-backup.keyring
│   │   │   └── ceph.client.cinder.keyring
│   │   └── cinder-volume
│   │       └── ceph.client.cinder.keyring
│   ├── glance
│   │   └── ceph.client.glance.keyring
│   ├── gnocchi
│   │   └── ceph.client.gnocchi.keyring
│   ├── manila
│   │   └── ceph.client.manila.keyring
│   ├── nova
│   │   └── ceph.client.cinder.keyring
│   └── zun
│       └── zun-compute
│           └── ceph.client.cinder.keyring
├── globals.yml
├── globals.yml.bak
└── passwords.yml
```

## ceph.conf复制

先手动复制一份 `ceph.conf` 到 `/etc/kolla/config/ceph.conf` 

> 只需要复制 `[global]` 部分即可

然后下面命令软链接到各个组件文件夹
```
# glance
ln -s /etc/kolla/config/ceph.conf /etc/kolla/config/glance/ceph.conf

# cinder
ln -s /etc/kolla/config/ceph.conf /etc/kolla/config/cinder/ceph.conf

# zun
ln -s /etc/kolla/config/ceph.conf /etc/kolla/config/zun/zun-compute/ceph.conf

# nova
ln -s /etc/kolla/config/ceph.conf /etc/kolla/config/nova/ceph.conf

# gnocchi
ln -s /etc/kolla/config/ceph.conf /etc/kolla/config/gnocchi/ceph.conf

# manila
ln -s /etc/kolla/config/ceph.conf /etc/kolla/config/manila/ceph.conf
```
