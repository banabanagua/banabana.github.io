# vagrant 制作box

本例以制作 centos7.5 的镜像为例

## 1. 最小化安装一台centos7.5
最小化安装 centos7.5 这里就不演示了
如果用 virtualbox ，都是图形化界面的操作，按照页面的提示一步步就可以完成安装。
如果用kvm最小化安装centos7.5，创建虚拟机的指令如下：
`virt-install --connect qemu:///system --name centos7 --memory=1024 --vcpus=1 --disk path=/tmp/centos7.qcow2,device=disk,format=qcow2,bus=virtio,cache=none,size=40 --cdrom /tmp/CentOS-7-x86_64-DVD-1804.iso --os-type=linux --network bridge=br0,model=virtio,model=e1000 --hvm --virt-type=kvm --noautoconsole --graphics vnc,listen=0.0.0.0,port=5901 --machine=q35 --features kvm_hidden=on`

## 2. 虚拟机重启之后，做些必要的初始化的动作
```bash
# 创建用户并设置密码
useradd vagrant
echo "vagrant" | passwd --stdin vagrant
echo "vagrant" | passwd --stdin root

# 设置 sudoers
echo "vagrant ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# vagrant 的公钥文件下载链接 https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant.pub
# vagrant 公钥配置
su - vagrant -c "mkdir .ssh && chmod 700 .ssh"
su - vagrant -c "echo 'ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key'>.ssh/authorized_keys"
su - vagrant -c "chmod 600 .ssh/authorized_keys"

# 设置网卡开机启动
sed -i 's/ONBOOT=no/ONBOOT=yes/g' /etc/sysconfig/network-scripts/ifcfg-enp0s3

# 清除 history 记录
history -c
```

## 3. 安装增强功能（可选）
#### 3.1 点击设备-》安装增强功能
#### 3.2 安装代码如下
```bash
mount /dev/cdrom /mnt
yum install -y bzip2 gcc make perl libX11 libXt libXext libXmu
# kernel-devel 这个包找到跟内核一致的版本下载
cd /mnt &&  ./VBoxLinuxAdditions.run

# 安装完成之后，卸载之前安装包
yum history undo <ID> # 撤销之前的安装记录
```

**如果是kvm，增强功能依赖的包命令是： ` yum -y install nfs-utils nfs-utils-lib portmap`**

**如果是kvm，ubuntu2004 nfs的安装命令是:  `apt-get -yqq update && apt-get install -yqq nfs-common portmap && apt-get clean`**

## 4. 第一张网卡设置成NAT模式，并在网卡的高级选项中增加ssh应用的端口转发（经测试，该过程可不做）
主机IP：127.0.0.1 主机端口：2222 guestIP: 留空 guestPort：22

## 5. 关机虚拟机（经测试，该过程可不做。过程6会自动做）

## 6. 制作 box，命令如下
```powershell
vagrant.exe  package --base <虚拟机名称>

# kvm 制作box的命令是使用 ./create_box.sh centos7.qcow2
# 这个脚本是在 github 的vagrant-libvirt 工程中可以找到
```