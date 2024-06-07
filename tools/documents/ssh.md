## VSCode 远程连接 CentOS8

### CentOS 端

```bash
sudo yum install openssh-server	# 安装 openssh

systemctl start sshd.service	# 启动ssh服务
		
service network restart			# 重启网络

systemctl enable sshd.service	# 设置开启启动ssh服务

systemctl status sshd.service	# 查看ssh服务状态
```

```bash
ip addr # 查看 ip
```

