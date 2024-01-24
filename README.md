# Avail Full Node Setup Guide in Binanries and Docker
## System Requirements
| Component | Minimum | Recommended |
|-----------|---------|-------------|
| RAM | 4GB | 8GB |
| CPU (amd64/x86 architecture) | 2 core | 	4 core |
| Storage (SSD) | 20-40 GB | 200-300 GB |
**OS Recommended Ubuntu 22.04**
## Update 11/25/2023
Run Light Node v1.7.5-rc9 with one command
```bash
bash <(wget -qO- https://raw.githubusercontent.com/mooser1/full-node-avail/main/auto-run-avail-light-node.sh)
```
## Update 11/21/2023
Run full node v1.8.0.4 with one command
```bash
bash <(wget -qO- https://raw.githubusercontent.com/hiephtdev/huong-dan-chay-full-node-avail/main/auto-run-avail-full-node.sh)
```
## English
### Part 1: Using Binaries on Ubuntu 22.04

1. Set up the environment by copying and executing the following commands:

    ```bash
    sudo apt-get -y update &&
    sudo apt-get -y install build-essential &&
    sudo apt-get -y install --assume-yes git clang curl libssl-dev protobuf-compiler && 
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh &&
    source ~/.cargo/env &&
    rustup default stable &&
    rustup update &&
    rustup update nightly && 
    rustup target add wasm32-unknown-unknown --toolchain nightly
    ```

2. Build the latest version of the Avail project (v1.8.0.3):

    ```bash
    mkdir -p $HOME/avail-node &&
    cd $HOME/avail-node &&
    git clone https://github.com/availproject/avail.git &&
    cd avail &&
    mkdir -p output &&
    mkdir -p $HOME/avail-node/data &&
    git checkout v1.9.0.0 &&
    cargo run --locked --release -- --chain goldberg -d ./output
    ```
     <img src="https://github.com/hiephtdev/huong-dan-chay-full-node-avail/blob/main/build.png">
    Wait for the process to complete, then press Ctrl + C.

3. Create a system service for more stable startup (**Before creating the service, please read all the notes below, then proceed with the command**)

    ```bash
    sudo touch /etc/systemd/system/availd.service
    sudo nano /etc/systemd/system/availd.service
    ```

    Then, paste the following command into the file **(Change [HOME_PATH] in the command below before pasting it into the service. Please read more below)**:

    ```
    [Unit] 
    Description=Avail Validator
    After=network.target
    StartLimitIntervalSec=0

    [Service] 
    User=root 
    ExecStart=[HOME_PATH]/avail-node/avail/target/release/data-avail --base-path [HOME_PATH]/avail-node/data --chain goldberg --port 30333 --rpc-port 9944 --prometheus-port 9615 --prometheus-external --validator --name "mysticwho-node"
    Restart=always 
    RestartSec=120

    [Install] 
    WantedBy=multi-user.target
    ```

    **In the above command, please note the following information:**
   
    -  `[HOME_PATH]` type the command $HOME and copy the path to replace in `[HOME_PATH]` above as shown in the example below. Replace `[HOME_PATH]` with `/home/tuan` like this: `[HOME_PATH]/avail-node/avail/target/release/data-avail` will become `/home/tuan/avail-node/avail/target/release/data-avail`, `[HOME_PATH]/avail-node/data` will become `/home/tuan/avail-node/data`.
   
       <img src="https://github.com/hiephtdev/huong-dan-chay-full-node-avail/blob/main/home.png">
       
    - `--name` is the name of the node.
    - Ports `30333, 9944, 9615` must be opened in the firewall. If you are using a VPS, configure it to allow TCP/UDP connections through these ports. If you're using a VPS, please make sure the port is open from the provider's side.
After editing, press Ctrl + O and then Enter, then press Ctrl + X to exit.

5. Enable and start the service:

    ```bash
    systemctl enable availd.service && systemctl start availd.service
    ```

6. Check the service status:

    ```bash
    systemctl status availd.service
    ```
<img src="https://github.com/hiephtdev/huong-dan-chay-full-node-avail/blob/main/service-status.png">

6. View logs while running with the following command:

    ```bash
    journalctl -f -u availd
    ```
7. Remove availd.service to recreate from scratch
   - Run the command to stop and disable the running service
   ```bash
    systemctl stop availd.service && systemctl disable availd.service
   ```
   - Delete the service file in systemd
   ```bash
    rm /etc/systemd/system/availd.service
   ```
   - Reload systemd
   ```bash
    systemctl daemon-reload && systemctl reset-failed 
   ```
8. In the event of updating the availd.service file, run the following command to reload the service:
```bash
systemctl daemon-reload
```
### Part 2: Using Docker on Ubuntu 22.04

1. Install Docker by running the following commands:

```bash
sudo apt-get update &&
sudo apt-get -y install ca-certificates curl gnupg &&
sudo install -m 0755 -d /etc/apt/keyrings &&
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg &&
sudo chmod a+r /etc/apt/keyrings/docker.gpg &&
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null &&
sudo apt-get update &&
sudo apt-get -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin &&
sudo apt-get -y install docker-compose &&
sudo usermod -aG docker $USER &&
newgrp docker
```

2. Run the following command to create data storage for the node:
```bash
mkdir -p $HOME/avail-node/data/keystore &&
mkdir -p $HOME/avail-node/data/state
```
3. Run the Docker container:
```bash
docker run -v $HOME/avail-node/data/state:/da/state:rw -v $HOME/avail-node/data/keystore:/da/keystore:rw -e DA_CHAIN=goldberg -e DA_NAME=goldberg-docker-avail-Node -p 0.0.0.0:30333:30333 -p 9615:9615 -p 9944:9944 -d --restart unless-stopped availj/avail:v1.9.0.0
```
In this step, make sure to replace `DA_NAME=goldberg-docker-avail-Node` with your node's name. Also, ensure that ports `30333, 9944, 9615` are opened in the firewall. If you are using a VPS, configure it to allow TCP/UDP connections through these ports.


To check your node, visit [https://telemetry.avail.tools/](https://telemetry.avail.tools/). Your node will be displayed after the synchronization process is complete and the node starts running.
<img src="https://github.com/hiephtdev/huong-dan-chay-full-node-avail/blob/main/check-tool.png">

#### Contact

Telegram: [https://t.me/cryptomoose](https://t.me/cryptomoose)

X: [https://twitter.com/cryptomooseog](https://twitter.com/cryptomooseog)

Discord: moose#1741


