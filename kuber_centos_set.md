## CentOS Setting
1. selinux 비활성화
```bash
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config
```

2. swap 비활성화
```
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

3. 방화벽 비활성화
```
systemctl disable firewalld
systemctl stop firewalld
```

4. iptables 커널 설정
```
cat <<EOF>> /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF>> /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system
```

5. hosts 설정
```
# ip 192.168.56.0으로 가상 NAT 네트워크 구성
cat <<EOF >> /etc/hosts
192.168.56.xxx k8s-master
192.168.56.xxx k8s-worker1
192.168.56.xxx k8s-worker2
EOF
```

6. cent os package update
```
yum update -y
```
## Kubernetes 설치
1. kubernetes yum repository 설정
```
cat <<EOF>> /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

(cent os libcgroup 설치필요할지도)
```
curl https://download.docker.com/linux/centos/docker-ce.repo -o /etc/yum.repos.d/docker-ce.repo
sed -i 's/$releasever/8/g' /etc/yum.repos.d/docker-ce.repo
dnf install docker-ce -y # error, reporting ' nothing provides libcgroup needed by docker-ce-3:20.10.12-3.el8.x86_64'

systemctl enable --now docker
```

2. Docker 설치 (centos 8과는 잘 안맞음, 수동 설치 필요) (podman 삭제가 필요할 수도 있음)
yum install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm -y
yum install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-19.03.6-3.el7.x86_64.rpm -y
yum install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-cli-19.03.6-3.el7.x86_64.rpm -y
yum install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.3.ce-1.el7.noarch.rpm -y