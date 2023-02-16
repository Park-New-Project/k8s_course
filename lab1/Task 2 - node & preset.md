# Task 2 - node & preset  

### 3개의 노드에 필요한 설정과 k8s 설치 전에 해야할 설정을 진행
#
1. 각 노드에서 hostname을 변경합니다.

Master Node
```
sudo -i
sudo hostnamectl set-hostname k8s-master
sudo -i
```
Worker1 Node
```
sudo -i
sudo hostnamectl set-hostname k8s-worker1
sudo -i
```
Worker2 Node
```
sudo -i
sudo hostnamectl set-hostname k8s-worker2
sudo -i
```


`이후 과정은 Master, Worker1, Worker2 모든 노드에 공통으로 진행합니다`

2. swap 해제
```
swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

3. 방화벽 해제
```
sudo ufw disable
```

4. 필요 패키지 설치
```
sudo apt-get update -y
```
```
sudo apt-get install -y curl ethtool ebtables socat conntrack
```

5. 모듈 로드
```
cat <<EOF | sudo tee /etc/modules-load.d/crio.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```
6. sysctl 파라미터 설정
```
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
```

7. 버전 변수 설정
```
export OS=xUbuntu_18.04 # OS 버전
export VERSION=1.24     # cri-o 버전
```

8. CRI-O 설치
```
echo "deb [signed-by=/usr/share/keyrings/libcontainers-archive-keyring.gpg] https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
echo "deb [signed-by=/usr/share/keyrings/libcontainers-crio-archive-keyring.gpg] https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list

mkdir -p /usr/share/keyrings
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | gpg --dearmor -o /usr/share/keyrings/libcontainers-archive-keyring.gpg
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/Release.key | gpg --dearmor -o /usr/share/keyrings/libcontainers-crio-archive-keyring.gpg

apt-get update
apt-get install cri-o cri-o-runc
```

```
systemctl daemon-reload
systemctl enable crio --now
systemctl status crio
```

키보드의 Q 키 또는 <Ctrl + C> 를 입력하면 다시 쉘을 받아옵니다.
