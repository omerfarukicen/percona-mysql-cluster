# percona-mysql-cluster
percona-mysql-cluster

# 1. Server1 and Server2 Install
```
sudo apt-get update -y  && sudo apt-get upgrade -y
&& sudo apt-get dist-upgrade -y && sudo apt-get install wget -y
&& wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb
&& sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb
&& sudo percona-release setup pxc57 && sudo apt-get update -y && sudo apt install percona-xtradb-cluster-full-57 -y
```
# 2. Host Config(Server1-Server2)
/etc/hosts 
```
192.168.122.61 p1
192.168.122.63 p2

```
# 3. Firewall Disable
```
sudo ufw allow 3306/tcp && sudo ufw allow 4444/tcp && sudo ufw allow 4567/tcp && sudo ufw allow 4568/tcp && sudo ufw allow 2222/tcp && sudo ufw allow 22/tcp
```
Firewall restart
```
sudo ufw disable
sudo ufw enable

```

AppArmor Disable
```
sudo systemctl stop apparmor
```
# 4. Node1 Start

```
sudo systemctl stop mysql
```
Edit my.conf  ->
```
sudo nano /etc/mysql/my.cnf
```
```
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/percona-xtradb-cluster.conf.d/
[mysqld]
wsrep_provider=/usr/lib/libgalera_smm.so
wsrep_cluster_name=democluster
wsrep_cluster_address=gcomm://192.168.122.63,192.168.122.62
wsrep_node_name=p3
wsrep_node_address=192.168.122.63
wsrep_sst_method=xtrabackup-v2
wsrep_sst_auth=sstuser:sstpassword
pxc_strict_mode=DISABLE
```

Start Node1
```
 sudo /etc/init.d/mysql bootstrap-pxc
```

Create sstuser
```
mysql -uroot -p -e "create user sstuser@localhost identified by 'sstpassword'"
mysql -uroot -p -e "grant reload, replication client, process, lock tables on *.* to sstuser@localhost"
mysql -uroot -p -e "flush privileges"
```
Monitoring
```
mysql -u root -p
show global status where variable_name in ('wsrep_provider_name','wsrep_cluster_status','wsrep_cluster_size','wsrep_evs_state','wsrep_local_state_comment');
```
# 5. Node2 Start

```
 sudo systemctl stop mysql
```
Edit my.conf  ->
```
sudo nano /etc/mysql/my.cnf
```
```
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/percona-xtradb-cluster.conf.d/
[mysqld]
wsrep_provider=/usr/lib/libgalera_smm.so
wsrep_cluster_name=democluster
wsrep_cluster_address=gcomm://192.168.122.63,192.168.122.62
wsrep_node_name=p2
wsrep_node_address=192.168.122.62
wsrep_sst_method=xtrabackup-v2
wsrep_sst_auth=sstuser:sstpassword
pxc_strict_mode=DISABLE
```
Start 
```
 sudo systemctl start mysql
```

# 5. Proxy SQL Kurulumu

wget -nv -O /etc/apt/trusted.gpg.d/proxysql-2.6.x-keyring.gpg 'https://repo.proxysql.com/ProxySQL/proxysql-2.6.x/repo_pub_key.gpg'
sudo echo deb https://repo.proxysql.com/ProxySQL/proxysql-2.6.x/$(lsb_release -sc)/ ./ | sudo tee /etc/apt/sources.list.d/proxysql.list
sudo systemctl start proxysql
sudo ufw allow 6033/tcp


