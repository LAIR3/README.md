# layer 3 blockchain deployment kit<br />
instructions are somewhat specific to Ubuntu 22.04LTS<br />

# kurtosis logs directory error

Create Missing Directories:
It seems the directories /var/log/kurtosis/2024/31 and similar are missing. You can manually create these directories to resolve the issue.

```bash
sudo mkdir -p /var/log/kurtosis/2024/31
sudo chown -R $USER:docker /var/log/kurtosis
```
Verify Docker Permissions:
```bash
sudo chmod -R 755 /var/log/kurtosis
````
Restart Kurtosis and Docker:
```bash
kurtosis engine restart
sudo systemctl restart docker
```
requirements include but not limited to<br />


#  go1.21.6 install on Ubuntu Linux 22.04LTS for amd64 using bash<br />
```bash
#get wget
sudo apt install wget
#wget go
wget https://go.dev/dl/go1.21.6.linux-amd64.tar.gz
#remove existing go
rm -rf /usr/local/go
#extract verbosely with force
sudo tar -xvf go1.21.6.linux-amd64.tar.gz -C /usr/local/
#add go path to ./bashrc current user with default Ubuntu 22 bash shell
echo 'export PATH="$PATH:/usr/local/go/bin"' | sudo tee -a ./bashrc
#refresh your bash shell
source $HOME/.bashrc
```

# nodejs install<br />
```bash
nodejs -v
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install nodejs -y
nodejs -v
```

# docker install<br />
```bash
# docker install https://docs.docker.com/engine/install/ubuntu/
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
docker ps
# remove bdk-toolbox-postgres-pgvectorscale
docker rmi blockchaindeploymentkit/bdk-toolbox-postgres-pgvectorscale
docker system prune -a
docker compose version
```
modify pgvectorscale as docker build using toolbox.Dockerfile
```bash
docker build -t blockchaindeploymentkit/bdk-toolbox-postgres-pgvectorscale -f docker/toolbox.Dockerfile .
docker login
docker tag blockchaindeploymentkit/bdk-toolbox-postgres-pgvectorscale blockchaindeploymentkit/bdk-toolbox-postgres-pgvectorscale
docker push blockchaindeploymentkit/bdk-toolbox-postgres-pgvectorscale
```

```bash
docker network ls
docker network inspect NAME
```



# polygon-cli blockchain swiss army knife source build

```bash
snap install yq
sudo apt install bc protoc
git clone https://github.com/maticnetwork/polygon-cli.git
cd polygon-cli
make install
```
# foundry<br />
```bash
curl -L https://foundry.paradigm.xyz | bash
source $HOME/.bashrc
foundryup
cd BDK2
git config --global user.email "codephreak@dmg.finance"
git config --global user.name "Professor-Codephreak"
forge init --force
```

```bash
sudo apt install postgresql-client-common
```

foundry creates script and src folders

The LAIR3-BDK and LAIR3-BDK2 Layer 3 Blockchain Development Kits are designed to facilitate the deployment and management of advanced blockchain solutions, specifically supporting zkEVM Rollup and Validium technologies. Utilizing the Kurtosis SDK, polygon-cli, avalanche subnet-evm and ignite with Starlark. Blockchain Deployment Kit facilitates the creation and deployment of customized blockchain environments with enhanced observability and testing capabilities.

The primary goal is to to make blockchain deployment a sane operation accessiable to regular developers interested in dapplications development.

set postgresql master password
```bash
export POSTGRES_MASTER_PASSWORD="your_secure_password"
```

Set Validator PostgreSQL Password

Log into PostgreSQL
```bash
sudo -u postgres psql
ALTER USER validator1 WITH PASSWORD 'new_validator1_password';
\q
```

check postgresql installation
```bash
sudo apt install postgresql-client-common
psql -U master_user -h 127.0.0.1 -p 5432 -d master
\dx
SELECT * FROM pg_extension WHERE extname = 'vectorscale';
```
# reference links<br />
<a href="https://rollupjs.org/configuration-options/#output-manualchunks">manualchunks</a><br />
<a href="https://lighthouse-book.sigmaprime.io/">lighthouse</a><br />
<a href="https://docs.timescale.com/self-hosted/latest/install/installation-docker/">pgvectorscale timescale</a><br />

# zkevm_bridge logs ./lib/zkevm_bridge.star<br />
```bash
kurtosis service logs cdk-v1 zkevm-bridge-ui-001
```

# run the bridge UI standalone<br />
```bash
docker run -it --rm -p 8080:80 leovct/zkevm-bridge-ui:multi-network
generic instruction
docker run --name zkevm-bridge-ui -p 80:8080 -v /path/to/.env:/app/.env leovct/zkevm-bridge-ui:multi-network
```bash

# recomended diagnostics
```bash
sudo apt install netstat hardinfo
```

# run grapha as a standalone
```bash
docker run -d --name=grafana -p 3000:3000 grafana/grafana:latest
docker start grafana
open http://localhost:3000/login
docker logs grafana
docker stop graphana
```

# run the zkevm-prover standalone
docker run -it --rm -p 50071:50071 -p 50061:50061 hermeznetwork/zkevm-prover:v6.0.2 /bin/bash

# troubleshoot ###################################################################

list enclaves
```bash
kurtosis enclave ls
```

Inspect the Kurtosis enclave
```bash
kurtosis enclave inspect bdk-v2
```

Check the logs of the failing service
```bash
kurtosis service logs bdk-v2 zkevm-bridge-ui-001
```
Open a shell into the service container to manually inspect and debug
```bash
kurtosis service shell bdk-v2 zkevm-bridge-ui-001
```

# Environment Setup
Clone the Repository and Install Dependencies

```bash
git clone https://github.com/LAIR3/BDK.git
cd BDK
sh scripts/tool_check.sh
```

Clean Up Previous Environments

```bash
kurtosis clean --all
```

BDK as Kurtosis Enclave

```bash
kurtosis run --enclave bdk-v2 --args-file params.yml --image-download always .
```

Inspect Enclave for Dashboard Details

```bash
kurtosis enclave inspect bdk-v2
```

View Grafana Dashboard

```bash
open http://127.0.0.1:49701/dashboards
```

Prometheus Targets

```
open http://127.0.0.1:49651/targets
```

Fetch Service Logs

```bash
kurtosis service logs bdk-v2 zkevm-agglayer-001
```

Add Permissionless Node

```bash
yq -Y --in-place 'with_entries(if .key == "deploy_zkevm_permissionless_node" then .value = true elif .value | type == "boolean" then .value = false else . end)' params.yml
kurtosis run --enclave cdk-v1 --args-file params.yml --image-download always .
```

Sync External Permissionless Node

```bash
cp /path/to/external/genesis.json templates/permissionless-node/genesis.json
yq -Y --in-place 'with_entries(if .key == "deploy_zkevm_permissionless_node" then .value = true elif .value | type == "boolean" then .value = false else . end)' params.yml
kurtosis run --enclave cdk-v1 --args-file params.yml --image-download always .
```

Simple RPC Calls

```bash
export ETH_RPC_URL="$(kurtosis port print cdk-v1 zkevm-node-rpc-001 http-rpc)"
cast block-number
cast balance --ether 0xE34aaF64b29273B7D567FCFc40544c014EEe9970
```

Send Transactions

```bash
cast send --legacy --private-key 0x12d7de8621a77640c9241b2595ba78ce443d05e94090365ab3bb5e19df82c625 --value 0.01ether 0x0000000000000000000000000000000000000000
```

Load Testing with Polygon CLI
```bash
polycli loadtest --requests 500 --legacy --rpc-url $ETH_RPC_URL --verbosity 700 --rate-limit 5 --mode t --private-key 0x12d7de8621a77640c9241b2595ba78ce443d05e94090365ab3bb5e19df82c625
```

# Observability Tools

Prometheus
Prometheus captures all the metrics for the running services.
Accessible via: http://127.0.0.1:49678

Grafana
Dashboards for visualizing metrics.
Accessible via: http://127.0.0.1:49701

Panoptichain
Monitors on-chain metrics such as blocks, transactions, and smart contract calls.
Accessible via: http://127.0.0.1:49651



