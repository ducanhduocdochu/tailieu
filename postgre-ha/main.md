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

Sau khi restart ec2
sudo mkdir -p /var/run/postgresql
sudo chown postgres:postgres /var/run/postgresql
sudo chmod 2775 /var/run/postgresql

sudo systemctl reset-failed patroni
sudo systemctl restart patroni
sudo systemctl status patroni --no-pager -l

sudo tee /etc/tmpfiles.d/postgresql.conf >/dev/null <<'EOF'
d /var/run/postgresql 2775 postgres postgres -
EOF

sudo systemd-tmpfiles --create

//// fix lỗi stoped thay vì stream
Mở leader 
sudo tee -a /var/lib/postgresql/15/main/pg_hba.conf >/dev/null <<'EOF'
host replication replicator 172.31.22.35/32 md5
host replication replicator 172.31.23.63/32 md5
EOF

chạy config 
sudo -u postgres psql -c "select pg_reload_conf();"
verify
sudo -u postgres egrep -n "replication|172.31" /var/lib/postgresql/15/main/pg_hba.conf
reinit
sudo patronictl -c /etc/patroni/patroni.yml reinit cluster_1 ip-172-31-22-35.ap-southeast-2.compute.internal --force
sudo patronictl -c /etc/patroni/patroni.yml reinit cluster_1 ip-172-31-23-63.ap-southeast-2.compute.internal --force


/// ha proxy
apt-get update
apt install haproxy -y
mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.org
vi /etc/haproxy/haproxy.cfg

global
    maxconn 100                # Maximum number of concurrent connections

defaults
    log global                 # Use global logging configuration
    mode tcp                   # TCP mode for PostgreSQL connections
    retries 2                  # Number of retries before marking a server as failed
    timeout client 30m         # Maximum time to wait for client data
    timeout connect 4s         # Maximum time to establish connection to server
    timeout server 30m         # Maximum time to wait for server response
    timeout check 5s           # Maximum time to wait for health check response

listen stats                # Statistics monitoring 
    mode http              # The protocol for web-based stats UI
    bind *:7000            # Port to listen to on all network interfaces
    stats enable           # Statistics reporting interface
    stats uri /stats       # URL path for the stats page
    stats auth percona:myS3cr3tpass    # Username:password authentication

listen primary
    bind *:5000                        # Port for write connections
    option httpchk /primary 
    http-check expect status 200  
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions  # Server health check parameters
    server node1 node1:5432 maxconn 100 check port 8008 
    server node2 node2:5432 maxconn 100 check port 8008  
    server node3 node3:5432 maxconn 100 check port 8008 

listen standbys
    balance roundrobin     # Round-robin load balancing for read connections
    bind *:5001            # Port for read connections
    option httpchk /replica 
    http-check expect status 200  
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions  # Server health check parameters
    server node1 node1:5432 maxconn 100 check port 8008  
    server node2 node2:5432 maxconn 100 check port 8008  
    server node3 node3:5432 maxconn 100 check port 8008

sudo systemctl restart haproxy
