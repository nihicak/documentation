# **Subtensor Setup**

References:
* [Bittensor Doc](https://docs.bittensor.com/)
* [Bittensor Github](https://github.com/opentensor/bittensor)
* [Subtensor Github](https://github.com/opentensor/subtensor)

## System used for this setup

* Ubuntu 22.04
* 4 vCPU
* 16 GB RAM
* SSD (80 GB free space)
* Python 3.10.X

## Setup
### Install essentials
The following command installs the essentials for most mining operation. Simply run and go through the steps.

Bittensor installation may take awhile with slow download speed, you can skip it if you are not running a miner or plan to use it.
```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/nihicak/bittensor/master/scripts/install.sh)"
```

### Run subtensor (mainnet lite node)
Pull from subtensor github
```bash
git clone https://github.com/opentensor/subtensor.git
```
```bash
cd subtensor
```
Ensure at main branch
```bash
git checkout main
```
Switch to stable version (optional)
```bash
git switch -d 0c77062a10cadfbab8507b7dccb2e1ea49848e58
```
(you can use which ever editor you are comfortable with, this example uses vim)
```bash
vim docker-compose.yml
```
1. hit `i` to start editing
2. go to line *11*
3. change `image: ghcr.io/opentensor/subtensor:latest` to `image: ghcr.io/opentensor/subtensor:v1.1.1-prelease`
4. hit `esc` to end editing
5. enter `:wq` and hit `return` to save and exit

Next step, you can run subtensor using [docker](https://github.com/nihicak/documentation/blob/main/subtensor.md#using-docker) or by [building the binary](https://github.com/nihicak/documentation/blob/main/subtensor.md#using-build-binary)

## Using docker
Pull subtensor image and run
```bash
sudo ./scripts/run/subtensor.sh -e docker --network mainnet --node-type lite
```

---
**Useful docker commands**

- Auto restart
```bash
docker update --restart unless-stopped subtensor-mainnet-lite
```
- View log
```bash
docker logs -f --tail 10 subtensor-mainnet-lite
```
- Restart / start / stop
```bash
docker restart subtensor-mainnet-lite
```
```bash
docker start subtensor-mainnet-lite
```
```bash
docker stop subtensor-mainnet-lite
```
- Clear all data
```bash
docker container stop subtensor-mainnet-lite && \
docker system prune -a -f && \
docker volume prune -a -f
```

## Using build binary
**Building the binary**

Install essentials
```bash
sudo apt-get update && \
sudo apt install build-essential && \
sudo apt-get install -y clang && \
sudo apt-get install -y curl && \
sudo apt-get install -y git && \
sudo apt-get install -y make && \
sudo apt install --assume-yes git clang curl libssl-dev protobuf-compiler && \
sudo apt install --assume-yes git clang curl libssl-dev llvm libudev-dev make protobuf-compiler
```
Install Rust
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
- Hit enter when prompt to choose option
```bash
source ~/.cargo/env
```
Update Rust
```bash
rustup default stable && \
rustup update && \
rustup target add wasm32-unknown-unknown && \
rustup toolchain install nightly && \
rustup target add --toolchain nightly wasm32-unknown-unknown
```
Build subtensor binary
```bash
cargo build --release --features=runtime-benchmarks
```
- Build time may take up to 20 minutes. Take a break, go have lunch or grab a coffee

Ensure data is clean
```bash
rm -rf /tmp/blockchain
```

Run subtensor binary with pm2
```bash
pm2 start ./target/release/node-subtensor \
--name subtensor -- \
--base-path /tmp/blockchain \
--chain ./raw_spec.json \
--rpc-external --rpc-cors all \
--bootnodes /ip4/13.58.175.193/tcp/30333/p2p/12D3KooWDe7g2JbNETiKypcKT1KsCEZJbTzEHCn8hpd4PHZ6pdz5 \
--sync warp
```

---
**Useful pm2 commands**

- List processes
```bash
pm2 status
```
- View log
```bash
pm2 log --lines 100 subtensor
```
- Restart / start / stop / delete
```bash
pm2 restart subtensor
```
```bash
pm2 start subtensor
```
```bash
pm2 stop subtensor
```
```bash
pm2 del subtensor
```

## Network requirements
* Subtensor needs access to the public internet
* Subtensor runs on ipv4
* Subtensor listens on the following ports:
  1) 9944 - Websocket. Use for retrieving chain data. This port can be made private by removing `--rpc-external --rpc-cors all`.
  2) 9933 - RPC. This port is opened, but not used.
  3) 30333 - p2p socket. This port accepts connections from other subtensor nodes. Make sure your firewall(s) allow incoming traffic to this port.
* It is assumed your default outgoing traffic policy is ACCEPT. If not, make sure outbound traffic to port 30333 is allowed.

## Test subtensor
Wait for subtensor to connect with peers, finish downloading and **starts syncing** (when the log runs like crazy)

Use `btcli` to retrieve data (assuming you have installed Bittensor via the essential setup)

- Locally
```bash
btcli s list --subtensor.network local
```

- Remotely (for personal devices, install bittensor via `pip install bittensor`, strongly recommend to use a virtual env)
```bash
btcli s list --subtensor.network <server_ip:9944>
```

[Documentation for btcli](https://docs.bittensor.com/btcli)
