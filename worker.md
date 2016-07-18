sudo mkdir -p /var/lib/kubernetes


sudo sh -c 'echo "[Unit]
Description=Docker Application Container Engine
Documentation=http://docs.docker.io

[Service]
ExecStart=/usr/bin/docker daemon \
  --iptables=false \
  --ip-masq=false \
  --host=unix:///var/run/docker.sock \
  --log-level=error \
  --storage-driver=overlay
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/docker.service'


sudo systemctl daemon-reload
sudo systemctl enable docker
sudo systemctl start docker

sudo docker version

sudo mkdir -p /opt/cni

wget http://storage.googleapis.com/kubernetes-release/network-plugins/cni-c864f0e1ea73719b8f4582402b0847064f9883b0.tar.gz

sudo tar -xvf cni-c864f0e1ea73719b8f4582402b0847064f9883b0.tar.gz -C /opt/cni

wget http://storage.googleapis.com/kubernetes-release/release/v1.3.0/bin/linux/amd64/kubectl
wget http://storage.googleapis.com/kubernetes-release/release/v1.3.0/bin/linux/amd64/kube-proxy
wget http://storage.googleapis.com/kubernetes-release/release/v1.3.0/bin/linux/amd64/kubelet

chmod +x kubectl kube-proxy kubelet

sudo mkdir -p /srv/bin

sudo mv kubectl kube-proxy kubelet /srv/bin/

sudo mkdir -p /var/lib/kubelet/

sudo sh -c 'echo "apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority: /var/lib/kubernetes/ca.pem
    server: http://10.240.0.20:8080
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubelet
  name: kubelet
current-context: kubelet
users:
- name: kubelet
  user:
    token: chAng3m3" > /var/lib/kubelet/kubeconfig'<Paste>

sudo sh -c 'echo "[Unit]
Description=Kubernetes Kubelet
Documentation=http://github.com/GoogleCloudPlatform/kubernetes
After=docker.service
Requires=docker.service

[Service]
ExecStart=/srv/bin/kubelet \
  --allow-privileged=true \
  --api-servers=http://192.168.10.195:8080 \
  --cluster-dns=192.168.2.1 \
  --cluster-domain=cluster.local \
  --configure-cbr0=true \
  --container-runtime=docker \
  --docker=unix:///var/run/docker.sock \
  --network-plugin=kubenet \
  --kubeconfig=/var/lib/kubelet/kubeconfig \
  --reconcile-cidr=true \
  --serialize-image-pulls=false \
  --v=2

Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/kubelet.service'


# --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \
# --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \
# --cloud-provider=aws \

sudo systemctl daemon-reload
sudo systemctl enable kubelet
sudo systemctl start kubelet

sudo sh -c 'echo "[Unit]
Description=Kubernetes Kube Proxy
Documentation=http://github.com/GoogleCloudPlatform/kubernetes

[Service]
ExecStart=/srv/bin/kube-proxy \
  --master=http://192.168.10.195:8080 \
  --kubeconfig=/var/lib/kubelet/kubeconfig \
  --proxy-mode=iptables \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/kube-proxy.service'
