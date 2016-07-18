sudo mkdir -p /var/lib/kubernetes

wget https://storage.googleapis.com/kubernetes-release/release/v1.3.0/bin/linux/amd64/kube-apiserver
wget https://storage.googleapis.com/kubernetes-release/release/v1.3.0/bin/linux/amd64/kube-controller-manager
wget https://storage.googleapis.com/kubernetes-release/release/v1.3.0/bin/linux/amd64/kube-scheduler
wget https://storage.googleapis.com/kubernetes-release/release/v1.3.0/bin/linux/amd64/kubectl

chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl

sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /srv/bin/

wget https://raw.githubusercontent.com/kelseyhightower/kubernetes-the-hard-way/master/authorization-policy.jsonl

sudo mv authorization-policy.jsonl /var/lib/kubernetes/

cat > kube-apiserver.service <<"EOF"
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service]
ExecStart=/srv/bin/kube-apiserver \
  --admission-control=NamespaceLifecycle,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota \
  --advertise-address=192.168.10.195 \
  --allow-privileged=true \
  --apiserver-count=1 \
  --authorization-mode=ABAC \
  --authorization-policy-file=/var/lib/kubernetes/authorization-policy.jsonl \
  --bind-address=0.0.0.0 \
  --enable-swagger-ui=true \
  --insecure-bind-address=0.0.0.0 \
  --etcd-servers=http://localhost:2379 \
  --service-cluster-ip-range=192.168.5.0/24 \
  --service-node-port-range=30000-32767 \
  --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \
  --token-auth-file=/var/lib/kubernetes/token.csv \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

# --etcd-cafile=/var/lib/kubernetes/ca.pem \
# --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \
# --service-account-key-file=/var/lib/kubernetes/kubernetes-key.pem \
# --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \

sudo mv kube-apiserver.service /etc/systemd/system/

sudo systemctl daemon-reload
sudo systemctl enable kube-apiserver
sudo systemctl start kube-apiserver


cat > kube-controller-manager.service <<"EOF"
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service]
ExecStart=/srv/bin/kube-controller-manager \
  --allocate-node-cidrs=true \
  --cluster-cidr=192.168.0.0/16 \
  --cluster-name=kubernetes \
  --leader-elect=true \
  --master=http://192.168.10.195:8080 \
  --service-cluster-ip-range=192.168.5.0/24 \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

# --root-ca-file=/var/lib/kubernetes/ca.pem \
# --service-account-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \

sudo mv kube-controller-manager.service /etc/systemd/system/

sudo systemctl daemon-reload
sudo systemctl enable kube-controller-manager
sudo systemctl start kube-controller-manager


cat > kube-scheduler.service <<"EOF"
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service]
ExecStart=/srv/bin/kube-scheduler \
  --leader-elect=true \
  --master=http://192.168.10.195:8080 \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF


sudo mv kube-scheduler.service /etc/systemd/system/

sudo systemctl daemon-reload
sudo systemctl enable kube-scheduler
sudo systemctl start kube-scheduler

