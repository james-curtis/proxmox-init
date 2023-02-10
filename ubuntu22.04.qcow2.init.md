# ubuntu22.04.qcow2 初始化配置

# 换源

> cloud-init重生成镜像后会失效

```bash
sudo sed -i 's/archive.ubuntu.com/mirrors.nju.edu.cn/g' /etc/apt/sources.list
sudo sed -i 's/security.ubuntu.com/mirrors.nju.edu.cn/g' /etc/apt/sources.list
sudo apt update
```

# ssh登录
```bash
sudo vim /etc/ssh/sshd_config

# 放行root登录
PermitRootLogin yes

# 允许密码登录
PasswordAuthentication yes
```

# 时区
```bash
sudo timedatectl set-timezone Asia/Shanghai

# 验证
timedatectl
```

# 时间同步
```bash
sudo apt install chrony
```

修改服务器
```
sudo vim /etc/chrony/chrony.conf

# 添加一行
server time.windows.com iburst

# 重启
sudo systemctl restart chronyd

# 验证
sudo chronyc sources -v
sudo chronyc tracking
```

# qemu代理
```bash
sudo apt install qemu-guest-agent
sudo systemctl start qemu-guest-agent
sudo systemctl enable qemu-guest-agent

# 验证
sudo systemctl status qemu-guest-agent
```
