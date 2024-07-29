# layer 3 blockchain deployment kit

requirements include but not limited to<br />
golang<br />
nodejs<br />
docker<br />
kurtosis-cli<br />
polygon-cli<br />


The LAIR3-BDK and LAIR3-BDK2 Layer 3 Blockchain Development Kits are designed to facilitate the deployment and management of advanced blockchain solutions, specifically supporting zkEVM Rollup and Validium technologies. Utilizing the Kurtosis SDK and Starlark integration, this kit enables the creation and deployment of customized blockchain environments with enhanced observability and testing capabilities.

The primary goal is to integrate Kurtosis-CDK with zkEVM solutions, ensuring the entire process is documented thoroughly in Markdown format for GitHub.

Command Overview
The following is a list of essential commands and operations available within the LAIR3-BDK toolkit, specifically for setting up and managing your blockchain deployment.

Environment Setup
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
kurtosis service logs cdk-v1 zkevm-agglayer-001
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



