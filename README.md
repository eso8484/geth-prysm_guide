# geth-prysm_guide
How to setup your self hosted RPC (Requires high disk space up to 500GB initial)

## 🖥️ Hardware Requirements

| **OS**              | **RAM**   | **CPU**     | **Disk**                          |
|---------------------|-----------|-------------|-----------------------------------|
| Ubuntu 20.04 or later | 8–16 GB | 4–6 cores   | 550 GB SSD (can grow up to 1 TB) |

---
### Step 1: One-liner to fetch the installer
```bash
bash <(curl -fsSL https://raw.githubusercontent.com/Iziedking/geth-prysm_guide/main/install_sepolia_node.sh)
```
### Step 2: Check Sync status
Use ctrl + C to exit logs and then run next command each time to check sync status
```bash
bash <(curl -fsSL https://raw.githubusercontent.com/Iziedking/geth-prysm_guide/main/sync.sh)
```


---

### Update your existing geth RPC node

Ethereum upgrades like Fusaka require running the latest stable client versions. Your Docker setup makes this straightforward here is a Quick Update Steps:

1. Pull Latest Images (from your node directory, ~/ethereum):
if you followed my guide earlier above, its easy just run the following commands
This updates Geth to the latest stable (v1.16.4) and Prysm to the latest release (v6.1.2).

```bash
  cd ~/ethereum
```

```bash
  rm -rf docker-compose.yml
```
```bash
  nano docker-compose.yml
```
```bash
services:
  geth:
    image: ethereum/client-go:v1.16.4
    container_name: geth
    network_mode: host
    restart: unless-stopped
    ports:
      - 30303:30303
      - 30303:30303/udp
      - 8545:8545
      - 8546:8546
      - 8551:8551
    volumes:
      - /root/ethereum/execution:/data
      - /root/ethereum/jwt.hex:/data/jwt.hex
    command:
      - --sepolia
      - --http
      - --http.api=eth,net,web3
      - --http.addr=0.0.0.0
      - --authrpc.addr=0.0.0.0
      - --authrpc.vhosts=*
      - --authrpc.jwtsecret=/data/jwt.hex
      - --authrpc.port=8551
      - --syncmode=snap
      - --datadir=/data
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  prysm:
    image: gcr.io/prysmaticlabs/prysm/beacon-chain:v6.1.2
    container_name: prysm
    network_mode: host
    restart: unless-stopped
    depends_on:
      - geth
    ports:
      - 4000:4000
      - 3500:3500
    volumes:
      - /root/ethereum/consensus:/data
      - /root/ethereum/jwt.hex:/data/jwt.hex
    command:
      - --sepolia
      - --accept-terms-of-use
      - --datadir=/data
      - --disable-monitoring
      - --rpc-host=0.0.0.0
      - --execution-endpoint=http://127.0.0.1:8551
      - --jwt-secret=/data/jwt.hex
      - --rpc-port=4000
      - --grpc-gateway-corsdomain=*
      - --grpc-gateway-host=0.0.0.0
      - --grpc-gateway-port=3500
      - --min-sync-peers=3
      - --checkpoint-sync-url=https://checkpoint-sync.sepolia.ethpandaops.io
      - --genesis-beacon-api-url=https://checkpoint-sync.sepolia.ethpandaops.io
      - --subscribe-all-data-subnets
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```
save with ctrl + o hit Enter
then ctrl + X to exit 

pull latest image:

```bash
  docker compose pull
```
```bash
  docker compose down
```
```bash
  docker compose up -d
```

Monitor sync: 

```bash
  bash <(curl -fsSL https://raw.githubusercontent.com/Iziedking/geth-prysm_guide/main/sync.sh)
```
 rpc node syncs within few mins

Verify Versions
Geth: 
```bash 
docker exec -it geth geth version
```
Prysm:
```bash
  curl -s http://localhost:3500/eth/v1/node/version | jq -r '.data.version'
```


if you face any issues or have improvements and contributions to make you can dm me [here](https://x.com/Iziedking) 
   
