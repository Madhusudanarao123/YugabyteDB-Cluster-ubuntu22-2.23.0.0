# YugabyteDB-Cluster-ubuntu22-2.23.0.0
Yugabyte cluster setup.

YugabyteDB Cluster Setup Guide (3 Nodes, Ubuntu 22, v2.23.0.0)
Nodes
Master: 10.0.0.1
Replica 1: 10.0.0.2
Replica 2: 10.0.0.3
Preparation on All Nodes
Update System & Install Tools
bash
apt update
apt install curl wget unzip -y
Download YugabyteDB
bash
wget https://downloads.yugabyte.com/releases/2.23.0.0/yugabyte-2.23.0.0-b710-linux-x86_64.tar.gz
# Alternative:
wget https://downloads.yugabyte.com/releases/2024.1.2.0/yugabyte-2024.1.2.0-b77-linux-x86_64.tar.gz
Extract & Install
bash
tar xvfz yugabyte-2.23.0.0-b710-linux-x86_64.tar.gz
cd yugabyte-2.23.0.0/
./bin/post_install.sh
Directory & SSL Certificate Setup
Prepare Data & Certs Directory

bash
mkdir -p /DISK01/yugadata/certs
Start Yugabyted (Standalone)

bash
./bin/yugabyted start --advertise_address=<node_ip> --base_dir=/DISK01/yugadata
Repeat on all nodes.

Generate SSL Certs (Master Only)

bash
./bin/yugabyted cert generate_server_certs --base_dir=/DISK01/yugadata/certs --hostnames=10.0.0.1,10.0.0.2,10.0.0.3
# Certs will be in /DISK01/yugadata/certs/generate_server_certs/
Distribute certs to /DISK01/yugadata/certs on all nodes.
Cluster Configuration
Stop Yugabyted on All Nodes
bash
./bin/yugabyted stop --base_dir=/DISK01/yugadata
Start Master Node (with SSL and flags)
bash
./bin/yugabyted start --secure --advertise_address=10.0.0.1 --base_dir=/DISK01/yugadata \
--fault_tolerance=zone \
--master_flags="use_node_to_node_encryption=true,use_client_to_server_encryption=true,certs_dir=/DISK01/yugadata/certs" \
--tserver_flags="use_node_to_node_encryption=true,use_client_to_server_encryption=true,certs_dir=/DISK01/yugadata/certs,ysql_hba_conf_csv={hostssl all all all md5 clientcert=1}"
Start Replica Nodes
bash
# Replica 1
./bin/yugabyted start --secure --advertise_address=10.0.0.2 --base_dir=/DISK01/yugadata \
--fault_tolerance=zone --join=10.0.0.1 \
--master_flags="use_node_to_node_encryption=true,use_client_to_server_encryption=true,certs_dir=/DISK01/yugadata/certs" \
--tserver_flags="use_node_to_node_encryption=true,use_client_to_server_encryption=true,certs_dir=/DISK01/yugadata/certs,ysql_hba_conf_csv={hostssl all all all md5 clientcert=1}"

# Replica 2
./bin/yugabyted start --secure --advertise_address=10.0.0.3 --base_dir=/DISK01/yugadata \
--fault_tolerance=zone --join=10.0.0.1 \
--master_flags="use_node_to_node_encryption=true,use_client_to_server_encryption=true,certs_dir=/DISK01/yugadata/certs" \
--tserver_flags="use_node_to_node_encryption=true,use_client_to_server_encryption=true,certs_dir=/DISK01/yugadata/certs,ysql_hba_conf_csv={hostssl all all all md5 clientcert=1}"
Cluster Management
Check Status
bash
./bin/yugabyted status --base_dir=/DISK01/yugadata/
Start/Stop Yugabyted
bash
./bin/yugabyted start --base_dir=/DISK01/yugadata/
./bin/yugabyted stop --base_dir=/DISK01/yugadata/
Key Points
SSL certificates must be set up before cluster configuration.
Use --join only when starting a replica node during cluster setup.
Use master/tserver flags to enforce SSL and certificate use.
The cluster will show all three nodes only after all have joined with the master.
