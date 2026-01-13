wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb
sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb
sudo apt update
sudo percona-release setup ppg-15
sudo apt install percona-ppg-server-15 -y

sudo apt install python3-pip python3-dev binutils -y
sudo apt install percona-patroni -y
sudo apt install percona-pgbackrest -y
sudo apt install etcd etcd-server etcd-client -y
sudo systemctl stop {etcd,patroni,postgresql}
sudo systemctl disable {etcd,patroni,postgresql}

sudo rm -rf /var/lib/postgresql/15/main

- config server 1: vi /etc/etcd/etcd.conf.yaml, 

name: 'server-ps-1'
initial-cluster-token: postgresql_cluster_ducanh
initial-cluster-state: new
initial-cluster: server-ps-1=http://172.31.19.96:2380
data-dir: /var/lib/etcd
initial-advertise-peer-urls: http://172.31.19.96:2380
listen-peer-urls: http://0.0.0.0:2380
advertise-client-urls: http://172.31.19.96:2379
listen-client-urls: http://0.0.0.0:2379

systemctl start etcd
systemctl enable etcd

etcdctl --endpoints=http://172.31.19.96:2379 member list

etcdctl --endpoints=http://172.31.19.96:2379
  member add server-ps-2 --peer-urls=http://3.104.35.50:2380

- config server add: vi /etc/etcd/etcd.conf.yaml

name: 'server-ps-2'
initial-cluster-token: postgresql_cluster_ducanh
initial-cluster-state: existing
initial-cluster: server-ps-1=http://172.31.19.96:2380,server-ps-2=http://3.104.35.50:2380
data-dir: /var/lib/etcd
initial-advertise-peer-urls: http://3.104.35.50:2380
listen-peer-urls: http://0.0.0.0:2380
advertise-client-urls: http://3.104.35.50:2379
listen-client-urls: http://0.0.0.0:2379

systemctl start etcd
systemctl enable etcd
