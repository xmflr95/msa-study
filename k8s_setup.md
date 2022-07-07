# K8S(Kubernetes) 설치과정
VM 구성과 AWS, Azure를 이용한 구성 설치과정 정리(순서 뒤죽박죽인 상태)

## 1. 설정
### 1.1. VM(Virtualbox)
|HOSTNAME|IP|ROLE|RESOURCE|비고|
|-------|-----|----------|------------|---|
|k8s-master|192.168.56.10|control-plane, master node|disk(30G),cpu(2),mem(2G)||
|k8s-node1|192.168.56.11|worker node|disk(30G),cpu(2),mem(2G)||
|k8s-node2|192.168.56.12|worker node|disk(30G),cpu(2),mem(2G)||

### 1.2. AWS(Master, Worker1), AZURE(Worker2)
|HOSTNAME|IP|ROLE|RESOURCE|비고|
|-------|-----|----------|------------|---|
|k8s-master|172.31.35.102|control-plane|disk(30G),cpu(2),mem(2G)||
|k8s-worker1|172.31.13.164|\<none\>|disk(30G),cpu(1),mem(1G)||
|k8s-worker2|10.0.0.4|\<none\>|disk(30G),cpu(1),mem(1G)||

## 2. Centos Settings
```sh
# 0. Docker 설치 및 활성화
# Docker 설치
# https://docs.docker.com/engine/install/centos/ 사이트 자료를 참고하여 설치한다.
yum -y update
yum install -y yum-utils
 
# Docker repository 시스템에 추가
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum-config-manager --enable docker-ce-nightly
 
# 최신버전의 Docker 설치(Install Docker Engine)
yum -y install docker-ce docker-ce-cli containerd.io
 
# Docker 데몬 시작 및 부팅 시 Docker 데몬 자동 시작
systemctl start docker
# 재부팅시 docker daemon 자동 실행
systemctl enable docker
 
# Docker 실행중인지 확인
systemctl status docker

# 1. selinux 비활성화
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config

# 2. SWAP 비활성화
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# 3. 방화벽 끄기
systemctl disable firewalld
systemctl stop firewalld

# 4. iptables 설정
# 4-1.
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
# 4-2.
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
# 4-3.
sudo sysctl --system

# 호스트(/etc/hosts) 파일 편집
cat <<EOF >> /etc/hosts
${master-node-ip} k8s-master
${worker-node-1-ip} k8s-worker1
${worker-node-2-ip} k8s-worker2
EOF

# 내경우
cat <<EOF >> /etc/hosts
192.168.56.10 k8s-master
192.168.56.11 k8s-node1
192.168.56.12 k8s-node2
EOF

yum update -y
```

## 3. K8S(Kubernetes) 설치
```sh
# 1. kubernetes.repo 등록
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

# 2. kubeadm, kubelet, kubectl 패키지 설치
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

# 2.5. init kubeadm
systemctl enable --now kubelet # 먼저 실행시켜야 하는듯?
kubeadm init
# cpu/mem 사양 무시
kubeadm init --ignore-preflight-errors=NumCPU  --ignore-preflight-errors=Mem
kubeadm init --ignore-preflight-errors=all

# 2.6. kubeadm 오류 발생 시 
sudo rm /etc/containerd/config.toml
sudo systemctl restart containerd
sudo kubeadm init

# 3. kubelet 실행
systemctl enable kubelet
systemctl start kubelet
# 3-1. kubelet 상태 확인
systemctl status kubelet

# 4. kubernetes 패키지 정보 확인
kubeadm version -o short
kubectl version
kubelet --version

# 4.1. docker daemon.json 편집
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF

```

## 4. Node 설정
> 각각 VM 혹은 Cloud 인스턴스에 설치해서 Master Node에 Worker Node들을 join시켜줘야함
### 4.1. Master Node 설정
```sh
# kubeadm init 
kubeadm reset # 초기화
kubeadm init --apiserver-advertise-address={K8S-master-IP} --pod-network-cidr=${사용할 ip주소 대역}
# public ip 적용
kubeadm init --apiserver-advertise-address={K8S-master-IP} --pod-network-cidr=${사용할 ip주소 대역} --control-plane-endpoint "PUBLIC_IP:PORT"
# example (내 경우)
kubeadm init --apiserver-advertise-address=192.168.56.10 --pod-network-cidr=172.16.0.0/16

kubeadm init --apiserver-advertise-address=172.31.35.102 --pod-network-cidr=172.16.0.0/16 --control-plane-endpoint "15.164.212.86:18000"

# cidr 권장 10.0.0.0/16
kubeadm init --apiserver-advertise-address=172.31.35.102 --pod-network-cidr=10.0.0.0/16 --control-plane-endpoint "15.164.212.86:6443" --apiserver-cert-extra-sans=172.31.35.102,15.164.212.86
# 결과물
kubeadm join 15.164.212.86:6443 --token 6xz0bw.2qh5ei8h2sr6cvjm --discovery-token-ca-cert-hash sha256:bac08866218028c921ac409e69ee14460266b7140de56a7d0776d65ecb206e06
# 2 제어노드 추가시
kubeadm join 15.164.212.86:6443 --token 6xz0bw.2qh5ei8h2sr6cvjm --discovery-token-ca-cert-hash sha256:bac08866218028c921ac409e69ee14460266b7140de56a7d0776d65ecb206e06 --control-plane 

# 마스터노드 공용 ip 매개변수 제공
kubeadm init --apiserver-advertise-address=172.31.35.102 --pod-network-cidr=172.16.0.0/16 --control-plane-endpoint "15.164.212.86:6443"

#example
kubeadm init --apiserver-advertise-address=172.31.35.102 --pod-network-cidr=172.16.0.0/16 --control-plane-endpoint "15.164.212.86:6443"

kubeadm init --apiserver-advertise-address=172.31.35.102 --pod-network-cidr=172.16.0.0/16 --control-plane-endpoint "15.164.212.86:6443"

# pulic root 접속(control plane)
kubeadm join 15.164.212.86:6443 --token 189wu5.z281lji6w2ijt7vn --discovery-token-ca-cert-hash sha256:5687ddd78a87aa6490ce86595ddc6931d95793c01f27836846b0102db1b84609 --control-plane

# public join 일반 (권장?)
kubeadm join 15.164.212.86:6443 --token 189wu5.z281lji6w2ijt7vn --discovery-token-ca-cert-hash sha256:5687ddd78a87aa6490ce86595ddc6931d95793c01f27836846b0102db1b84609

172.31.35.102 k8s-master
3.35.136.181 k8s-worker1 k8s-worker1
20.41.105.80 azure-worker k8s-worker2

# 자동생성
kubeadm join 172.31.35.102:6443 --token lpoxw7.w63kxmig48zin01n --discovery-token-ca-cert-hash sha256:967b6eac16c98385badbb091dba9aff7b3ce156d15ff125141b147eee66f4584

# [ERROR CRI]: container runtime is not running: output:... 에러 발생 시
sudo rm /etc/containerd/config.toml
sudo systemctl restart containerd
sudo kubeadm init

# k8s 서비스가 끊기거나 시작이 안될 경우 reset으로 초기화 시켜주고 다시 시작하면됨
kubeadm reset
kubeadm init

# Unable to connect to the server: x509: certificate is valid for ~ : 에러 발생 시
# (일반 사용자 권한에 추가 하는 방법)
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 에러해결 Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "kubernetes")
unset KUBECONFIG
export KUBECONFIG=/etc/kubernetes/admin.conf
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Weave CNI 플러그인 설치
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

# node 클러스터 참여 확인(worker 노드 설정 이후 확인)
kubectl get nodes
kubectl get pods --all-namespaces
```

### +TOKEN
```sh
# 2. Token 생성(확인)
kubeadm token list
# example
kubeadm token create
jdygfw.nwqd04ac9dv80lrc
# /example
# 토큰 생성
kubeadm token create

# 3. Hash 확인
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'

# 4. Join
kubeadm join <Kubernetes API Server:PORT> --token ${2. Token 값} --discovery-token-ca-cert-hash sha256:${3. Hash 값}

```

### 4.2. Worker Node 설정
```sh
# 1. kubeadm join
kubeadm reset
# # k8s 기본 open port : 6443
Join
kubeadm join <Kubernetes API Server:PORT> --token ${2. Token 값} --discovery-token-ca-cert-hash sha256:${3. Hash 값}
# example 
kubeadm join 192.168.56.10:6443 --token jdygfw.nwqd04ac9dv80lrc --discovery-token-ca-cert-hash sha256:a88c0efeaf9ad27dd5a4362ce395e40bd31e8c5634a60636874cb210cb94fa66
```

### 5. Worker Node 해제 및 에러 처리
```sh
# error 처리
# [ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1 
# 발생시
kubeadm reset
echo 1 > /proc/sys/net/ipv4/ip_forward

# 노드 삭제 전 drain
kubectl drain ip-172-31-13-164.ap-northeast-2.compute.internal --delete-local-data --force --ignore-daemonsets
# 노드 삭제
kubectl delete node ip-172-31-13-164.ap-northeast-2.compute.internal
```