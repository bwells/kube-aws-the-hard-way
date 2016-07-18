
wget https://github.com/coreos/etcd/releases/download/v3.0.1/etcd-v3.0.1-linux-amd64.tar.gz

tar -xvf etcd-v3.0.1-linux-amd64.tar.gz

sudo mkdir -p /srv/bin

sudo cp etcd-v3.0.1-linux-amd64/etcd* /srv/bin/

sudo mkdir -p /var/lib/etcd

cat > etcd.service <<"EOF"
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/srv/bin/etcd --name ETCD_NAME \
  --initial-advertise-peer-urls http://192.168.10.195:2380 \
  --listen-peer-urls http://192.168.10.195:2380 \
  --listen-client-urls http://192.168.10.195:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://192.168.10.195:2379 \
  --initial-cluster etcd0=http://192.168.10.195:2380 \
  --initial-cluster-token example-etcd-cluster-0 \
  --initial-cluster-state new \
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5
Type=notify

[Install]
WantedBy=multi-user.target
EOF


--cert-file=/etc/etcd/kubernetes.pem \
--key-file=/etc/etcd/kubernetes-key.pem \
--peer-cert-file=/etc/etcd/kubernetes.pem \
--peer-key-file=/etc/etcd/kubernetes-key.pem \
--trusted-ca-file=/etc/etcd/ca.pem \
--peer-trusted-ca-file=/etc/etcd/ca.pem \

