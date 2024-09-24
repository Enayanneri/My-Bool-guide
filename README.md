# My-Bool-guide
### **Prerequisites**

1. **Hardware Requirements**:
   - **CPU**: 4+ cores (x86_64 architecture)
   - **RAM**: 8GB+
   - **Storage**: 256GB SSD (preferably NVMe for speed)
   - **Network**: 100 Mbps bandwidth or higher

2. **Operating System**:
   - Ubuntu 20.04 LTS or higher is recommended.

3. **Software Requirements**:
   - `go` version 1.19+
   - `git`, `make`, `gcc` installed.

---

### **Server Setup**

1. **Update your system**:

   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Install dependencies**:

   ```bash
   sudo apt install build-essential git curl jq -y
   ```

3. **Install Go**:

   ```bash
   wget https://golang.org/dl/go1.19.7.linux-amd64.tar.gz
   sudo tar -xvf go1.19.7.linux-amd64.tar.gz
   sudo mv go /usr/local
   ```

   Add Go to your system’s environment variables:

   ```bash
   echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.bashrc
   source ~/.bashrc
   ```

   Verify installation:

   ```bash
   go version
   ```

---

### **Clone the Bool Network Repository**

1. **Clone the repository**:

   ```bash
   git clone https://github.com/BoolNetwork/bool.git
   cd bool
   ```

2. **Checkout the latest stable version**:

   ```bash
   git checkout tags/v0.1.0
   ```

3. **Build the node**:

   ```bash
   make install
   ```

   Verify the installation:

   ```bash
   boold version
   ```

---

### **Initialize the Node**

1. **Initialize your node with a custom moniker (node name)**:

   ```bash
   boold init <your-node-moniker> --chain-id bool-mainnet
   ```

2. **Download the genesis file**:

   ```bash
   curl -O https://bool.network/genesis.json
   mv genesis.json ~/.boold/config/
   ```

3. **Configure seeds and peers** in the `config.toml` file:

   ```bash
   sed -i.bak -e "s/seeds =.*/seeds = \"<seed-nodes>\"/" ~/.boold/config/config.toml
   ```

   Replace `<seed-nodes>` with the seed node list provided in the official repository or documentation.

---

### **Create a Validator Wallet**

1. **Create a new key (wallet)**:

   ```bash
   boold keys add <your-wallet-name>
   ```

   Save the mnemonic phrase securely as you will need it later for transactions.

2. **Request testnet tokens** (optional, if using testnet):

   You can request tokens from the Bool faucet (if available) or use your own tokens for the mainnet.

---

### **Start the Node**

1. **Configure the service file** to start the node as a system service.

   Create a new service file:

   ```bash
   sudo nano /etc/systemd/system/boold.service
   ```

   Add the following content:

   ```bash
   [Unit]
   Description=Bool Network Node
   After=network-online.target

   [Service]
   User=$USER
   ExecStart=$(which boold) start
   Restart=on-failure
   LimitNOFILE=4096

   [Install]
   WantedBy=multi-user.target
   ```

   Reload the systemd manager and start the node:

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable boold
   sudo systemctl start boold
   ```

2. **Check logs** to ensure the node is syncing:

   ```bash
   journalctl -u boold -f
   ```

---

### **Create a Validator**

Once your node is fully synced (you can check sync status with `boold status`), create a validator:

1. **Submit a create-validator transaction**:

   ```bash
   boold tx staking create-validator \
     --amount=1000000ubool \
     --pubkey=$(boold tendermint show-validator) \
     --moniker="<your-validator-name>" \
     --chain-id=bool-mainnet \
     --commission-rate="0.10" \
     --commission-max-rate="0.20" \
     --commission-max-change-rate="0.01" \
     --min-self-delegation="1" \
     --from=<your-wallet-name> \
     --fees=500ubool
   ```

   Replace `<your-wallet-name>` with the name of the wallet you created earlier.

2. **Confirm validator status**:

   You can check your validator’s status using:

   ```bash
   boold query staking validator $(boold keys show <your-wallet-name> --bech val -a)
   ```

---

### **Monitoring and Maintenance**

1. **Check validator status**:

   ```bash
   boold status
   ```

2. **Check for missed blocks**:

   ```bash
   boold query slashing signing-info $(boold tendermint show-validator)
   ```

3. **Upgrade the node** as new releases are made available:

   Follow the official documentation for upgrading the node to newer versions.

