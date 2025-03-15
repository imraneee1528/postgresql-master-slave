# full set up of postgresql  master slave step by step

Master PostgreSQL Server  -  172.16.10.220

Slave  PostgreSQL Server  -  172.16.10.119

# Install both server postgresql 

sudo apt -y update

sudo apt -y install nano wget

# Add GPG key

wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# Add repository
 
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# update and install now with version

sudo apt update

sudo apt -y install postgresql-14 postgresql-client-14

# Now Restart and Enable Postgresql

sudo service postgresql start

sudo service postgresql status

sudo systemctl is-enabled postgresql

# Configuring Master Database 172.16.10.220

sudo -i -u postgres

# Create replication new user with limited connections 

CREATE USER foo REPLICATION LOGIN CONNECTION LIMIT 1 ENCRYPTED PASSWORD 'foopass';

# list check role 

\du

# create slot for replication

SELECT * FROM pg_create_physical_replication_slot('ocean');

# check slot name 

SELECT slot_name, slot_type, active FROM pg_replication_slots;


# maximum number of connections allowed to the replication user.

ALTER ROLE replication CONNECTION LIMIT -1;

# open  configuration files 

sudo vi /etc/postgresql/14/main/postgresql.conf

# change below configurations like this 

listen_addresses = 'localhost,172.16.10.220'

wal_level = replica

max_wal_senders = 10

wal_keep_segments = 64

wal_log_hints = on

max_replication_slots = 10

hot_standby = on 

primary_conninfo = 'host=172.16.10.220  port=5432 user=foo password=foopass'

primary_slot_name = 'ocean'

# comment below configuration

synchronous_commit = on

synchronous_standby_names = '*'

# open pg_hba.conf file 

sudo vi /etc/postgresql/14/main/pg_hba.conf

# add below line 

host    replication     foo     172.16.10.119/0   md5

# Restart postgresql service 

sudo service postgresql restart

sudo service postgresql status

# Configuring Slave Database (172.16.10.119)

sudo service postgresql stop

sudo service postgresql status

# backup and delete basebacup

sudo  cp -rp  /var/lib/postgresql/14/main/ /tmp/backup-base-postgres


sudo rm -rf  /var/lib/postgresql/14]/main/*


# take basebackup  in postgresql user 

pg_basebackup -h 172.16.10.220 -U foo -p 5432 -D /var/lib/postgresql/14/main/  -Fp -Xs -P -R


# open postgresql.conf file 

sudo vi /etc/postgresql/14/main/postgresql.conf

# change configuration like this 

listen_addresses = 'localhost,172.16.10.119'
wal_level = replica
max_wal_senders = 10
wal_keep_segments = 64
hot_standby = on

# open pg_hba.conf file 

sudo vi /etc/postgresql/12/main/pg_hba.conf

# add below line in configurations file 

host    replication     replication     172.16.10.220/0   md5# Restart postgresql service 

# restart service 

sudo service postgresql restart

sudo service postgresql status






























