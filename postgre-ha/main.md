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

export NODE_NAME=`hostname -f`
export NODE_IP=`getent hosts $(hostname -f) | awk '{ print $1 }' | grep -v grep | grep -v '127.0.1.1'`
DATA_DIR="/var/lib/postgresql/15/main"
PG_BIN_DIR="/usr/lib/postgresql/15/bin"
NAMESPACE="percona_lab"
SCOPE="cluster_1"

echo "
namespace: ${NAMESPACE}
scope: ${SCOPE}
name: ${NODE_NAME}

restapi:
    listen: 0.0.0.0:8008
    connect_address: ${NODE_IP}:8008

etcd3:
    host: ${NODE_IP}:2379

bootstrap:
  # this section will be written into Etcd:/<namespace>/<scope>/config after initializing new cluster
  dcs:
      ttl: 30
      loop_wait: 10
      retry_timeout: 10
      maximum_lag_on_failover: 1048576

      postgresql:
          use_pg_rewind: true
          use_slots: true
          parameters:
              wal_level: replica
              hot_standby: "on"
              wal_keep_segments: 10
              max_wal_senders: 5
              max_replication_slots: 10
              wal_log_hints: "on"
              logging_collector: 'on'
              max_wal_size: '10GB'
              archive_mode: "on"
              archive_timeout: 600s
              archive_command: "cp -f %p /home/postgres/archived/%f"

      pg_hba: # Add following lines to pg_hba.conf after running 'initdb'
      - host replication replicator 127.0.0.1/32 trust
      - host replication replicator 0.0.0.0/0 md5
      - host all all 0.0.0.0/0 md5
      - host all all ::0/0 md5
      recovery_conf:
            restore_command: cp /home/postgres/archived/%f %p

  # some desired options for 'initdb'
  initdb: # Note: It needs to be a list (some options need values, others are switches)
      - encoding: UTF8
      - data-checksums


postgresql:
    cluster_name: cluster_1
    listen: 0.0.0.0:5432
    connect_address: ${NODE_IP}:5432
    data_dir: ${DATA_DIR}
    bin_dir: ${PG_BIN_DIR}
    pgpass: /tmp/pgpass0
    authentication:
        replication:
            username: replicator
            password: replPasswd
        superuser:
            username: postgres
            password: qaz123
    parameters:
        unix_socket_directories: "/var/run/postgresql/"
    create_replica_methods:
        - basebackup
    basebackup:
        checkpoint: 'fast'

    watchdog:
      mode: required # Allowed values: off, automatic, required
      device: /dev/watchdog
      safety_margin: 5

tags:
    nofailover: false
    noloadbalance: false
    clonefrom: false
    nosync: false
" | sudo tee /etc/patroni/patroni.yml

sudo mkdir -p /var/run/postgresql
sudo chown postgres:postgres /var/run/postgresql
sudo chmod 2775 /var/run/postgresql

systemctl enable patroni
systemctl start patroni
