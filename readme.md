# CORD Sparknet Deployment

The CORD Sparknet comprises of different types of nodes - boot node, full node, validator node and archive node. Each type of node has different function within the network infrastructure. This document provides a single page instruction set to create one or all different kinds of the nodes and associate them with the Sparknet instance of CORD network.

The ***archive node*** maintains all the past blocks and the states. This makes it convenient to query the past state of the chain at any point in time. A ***full node*** prunes historical states eg. except for the state of the genesis block all states of finalised blocks older than a configurable number. This means that it requires much less space than an archived node. 

Before we begin it is necessary to understand that the generation of the node key is mandatory only for boot nodes. If a node key file is not provided as a parameter then cord will generate a random node key.

# Recommended Resources

<aside>
💡 OS: Ubuntu 22.04

</aside>

## Validator Node

- Ideal: 32 GB RAM, NVMe SSD of 256GB
- Minimum: 16 GB RAM, SSD Storage of 256GB

<aside>
💡 AWS instance type: c6i.4xlarge

</aside>

<aside>
 GCP instance type: n2-standard-8

</aside>

## Archive Node

- Ideal: 32 GB RAM, NVMe SSD of 512 GB
- Minimum: 16 GB RAM, SSD Storage of 512 GB

## Full Nodes and Boot Nodes

- Minimum: 2vCPU, 4 GB RAM, 10 GB Storage

<aside>
💡 GCP Instance type: e2-small

</aside>

<aside>
💡 AWS Instance type: t2.micro

</aside>

# Boot Node

## Using systemd for Boot Node

1. Update and Upgrade packages

```bash
sudo apt update -y && sudo apt upgrade -y
```

1. The CORD project makes pre-built binaries available which remove the need to compile and saves time during deployment. Download the compiled CORD binary using `wget` commands as below.

```bash
# using wget
sudo wget -O /usr/local/bin/cord https://github.com/dhiway/cord/releases/download/125b662/cord-develop-x86_64-ubuntu-22.04 && sudo chmod +x /usr/local/bin/cord
```

1. Create a `data` directory in `root` directory

```bash
sudo mkdir /data
```

1. Generate Node Key

```
sudo cord key generate-node-key --file /data/node.key
```

1. Create cord.service file

```bash
sudo vim /etc/systemd/system/cord.service
```

This file contains the commands which will be run on server boot/restart

```bash
[Unit]
Description=cord

[Service]
User=ubuntu
ExecStart=sudo cord --name <node-name> --base-path /data/ --chain spark --node-key-file /data/node.key --port 30333 --rpc-port 9933 --prometheus-port 9615 --rpc-methods=Safe --rpc-cors all --no-hardware-benchmarks --state-pruning 100 --blocks-pruning 100 --offchain-worker never --prometheus-external
Restart=always
RestartSec=120

[Install]
WantedBy=multi-user.target
```

1. Reload the daemon

```bash
sudo systemctl daemon-reload
```

1. To enable this to autostart on reboot use the following command

```bash
sudo systemctl enable cord.service
```

1. Start the cord process

```bash
sudo systemctl start cord
```

1. Check cord status

```bash
sudo systemctl status cord
```

1. Check the log

```bash
sudo tail -f /var/log/syslog
```

## Using Docker for Boot Node

<aside>
💡 Install Docker :[https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

- **Post-installation steps for Docker 👇**
    
    source : [https://docs.docker.com/engine/install/linux-postinstall/](https://docs.docker.com/engine/install/linux-postinstall/)
    
    To create the docker group and add your user:
    
    1. Create the `docker` group.
        
        ```bash
        sudo groupadd docker
        ```
        
    2. Add your user to the `docker` group.
        
        ```bash
        sudo usermod -aG docker $USER
        ```
        
    3. Log out and log back in so that your group membership is re-evaluated. You can also run the following command to activate the changes to groups.
        
        ```bash
        newgrp docker
        ```
        
    4. Verify that you can run `docker` commands without `sudo`.
        
        ```bash
        docker run hello-world
        ```
        
</aside>

1. Update and Upgrade packages

```bash
sudo apt update -y && sudo apt upgrade -y
```

1. Now, pull CORD image from docker hub

```bash
docker pull dhiway/cord:develop 
```

 3. make an empty folder named data in home directory

```bash
mkdir /data
```

<aside>
💡 Make sure **data** directory have **rwx** (read,write,execute) permissions. To give all the permissions use **`sudo chmod 777 /data`**

</aside>

3.1  Change the ownership of **data** directory

```bash
sudo chown $USER /data
```

1. Create Validator Account 

```bash
docker run dhiway/cord:develop key generate -n cord --words 24 --scheme Sr25519
```

<aside>
💡 **Note:** Please ensure that you save a copy of this output in a secure location. At this time, we will need to share your SS58 address with Sparknet Admin ([amar@dhiway.com](mailto:amar@dhiway.com) or [satish@dhiway.com](mailto:satish@dhiway.com)) as part of the validator node deployment process, which will place your node in a queue to become a validator. Ongoing development within the CORD project will allow this to become the responsibility of the on-chain governance process.

</aside>

1. Generate Node key and store in **data** directory

```bash
sudo docker run -v /data:/data --rm dhiway/cord:develop key generate-node-key --file /data/node.key
```

6. Run Cord (Here the **<name-you-see-in-telemetry>** should be changed to name of your choice)

```bash
sudo docker run --detach --restart unless-stopped -v /data:/data --name cord dhiway/cord:develop --name <name> --base-path /data/ --chain spark --node-key-file /data/node.key --port 30333 --rpc-port 9933 --prometheus-port 9615 --rpc-methods=Safe --rpc-cors all --no-hardware-benchmarks --state-pruning 100 --blocks-pruning 100 --offchain-worker never --prometheus-external
```

1. generate the session keys(The **secret** mentioned here is the **secret seed** from Account generation”step 4”)

```bash
sudo docker run -v /data:/data -v $(pwd):/home/ubuntu --rm dhiway/cord:develop key generate-session-keys -d /data --chain spark --suri "<secret>"
```

1. Check logs 

```bash
sudo docker logs <container-id>
```

# Validator Node

## Using systemd for Validator Node

1. Update and Upgrade packages

```bash
sudo apt update -y && sudo apt upgrade -y
```

1. The CORD project makes pre-built binaries available which remove the need to compile and saves time during deployment. Download the compiled CORD binary using wget or curl commands as below.

```bash
# using wget
sudo wget -O /usr/local/bin/cord https://github.com/dhiway/cord/releases/download/125b662/cord-develop-x86_64-ubuntu-22.04 && sudo chmod +x /usr/local/bin/cord
```

1. Create a `data` directory in `root` directory

```bash
sudo mkdir /data
```

1. Generate Node Key

```
sudo cord key generate-node-key --file /data/node.key

```

1. Create Validator Account

```bash
sudo cord key generate -n cord --words 24 --scheme Sr25519
```

<aside>
💡 **NOTE: Make sure you copy the output in a safe place. At the moment we have to share the SS58 Address to Sparknet Admin(amar@dhiway.com or satish@dhiway.com). This validator node deployment process makes your node ready in a queue to become a validator. Ongoing development in the CORD project will enable this as a responsibility of the on-chain governance process.**

</aside>

Example Output

```bash
Secret phrase:       life bachelor card luggage where believe awful tongue retreat planet trip soul fork kiss announce virtual client pact tomato try auto trial rare upper
  Network ID:        cord
  Secret seed:       0x97338113516f0553195ebcafd0bfc13c985339546a6bd030752ab322d21cfd44
  Public key (hex):  0x56d99259e2ece1ed8d6d06f802c2a5a5252cd7ad78e71537ad34897a3847237a
  Account ID:        0x56d99259e2ece1ed8d6d06f802c2a5a5252cd7ad78e71537ad34897a3847237a
  Public key (SS58): 3vo1FHxrrEBmnRVvoHpabfhos1MxBgyzyrjqzAsgnMCEEQQN
  SS58 Address:      3vo1FHxrrEBmnRVvoHpabfhos1MxBgyzyrjqzAsgnMCEEQQN
```

1. Run CORD in one terminal: Make sure to give a good name

```bash
cord --validator \
--name <node-name> \
--base-path /data/ \
--chain spark \
--node-key-file /data/node.key \
--port 30333 \
--rpc-port 9933 \
--prometheus-port 9615 \
--rpc-methods=Safe \
--rpc-cors all \
--no-hardware-benchmarks \
--state-pruning 100 \
--blocks-pruning 100 \
--offchain-worker never \
--prometheus-external \
--bootnodes /dns4/sparknet-bn1.cord.network/tcp/30333/p2p/12D3KooWRsz18nHnHFxAjkGiCE6auBG5Y8LCgi2MmPTWUnvVYwxe /dns4/sparknet-bn2.cord.network/tcp/30333/ws/p2p/12D3KooWKMvTXGEWLWFVKJg3GFibapB9dmN91pmxdEwRpMvJ7jja /dns4/sparknet-bn3.cord.network/tcp/30333/ws/p2p/12D3KooWSv1Cxpa93CbGJPZsSbEbn9BAHvfkL9DyjgXAMGHcUHZ8
```

1. Open another terminal and generate session keys using the commands below

```bash
cord generate-session-keys -d /data/ \
--chain spark \
--suri "<secret seed>"
```

Example:

```bash
# The secret key is from step 5 example
cord key generate-session-keys -d /home/ubuntu/data/ \
--chain spark \
--suri "0x97338113516f0553195ebcafd0bfc13c985339546a6bd030752ab322d21cfd44"
```

1. Stop the running Node

```bash
Ctrl + C
```

1. Create cord.service file

```bash
sudo vim /etc/systemd/system/cord.service
```

This file contains the commands which will be run on server boot/restart

```bash
[Unit]
Description=cord

[Service]
User=ubuntu
ExecStart=sudo cord --name <node-name> --validator --base-path /data/ --chain spark --node-key-file /data/node.key --port 30333 --rpc-port 9933 --prometheus-port 9615 --rpc-methods=Safe --rpc-cors all --no-hardware-benchmarks --state-pruning 100 --blocks-pruning 100 --offchain-worker never --prometheus-external --bootnodes /dns4/sparknet-bn1.cord.network/tcp/30333/p2p/12D3KooWRsz18nHnHFxAjkGiCE6auBG5Y8LCgi2MmPTWUnvVYwxe /dns4/sparknet-bn2.cord.network/tcp/30333/ws/p2p/12D3KooWKMvTXGEWLWFVKJg3GFibapB9dmN91pmxdEwRpMvJ7jja /dns4/sparknet-bn3.cord.network/tcp/30333/ws/p2p/12D3KooWSv1Cxpa93CbGJPZsSbEbn9BAHvfkL9DyjgXAMGHcUHZ8
Restart=always
RestartSec=120

[Install]
WantedBy=multi-user.target
```

1. Reload the daemon

```bash
sudo systemctl daemon-reload
```

1. To enable this to autostart on reboot use the following command

```bash
sudo systemctl enable cord.service
```

1. Start the cord process

```bash
sudo systemctl start cord
```

1. Check cord status

```bash
sudo systemctl status cord
```

1. Check the log

```bash
sudo tail -f /var/log/syslog
```

## Using Docker for Validator Node

<aside>
💡 **Install Docker :** [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

- **Post-installation steps for Docker 👇**
    
    source : [https://docs.docker.com/engine/install/linux-postinstall/](https://docs.docker.com/engine/install/linux-postinstall/)
    
    To create the docker group and add your user:
    
    1. Create the `docker` group.
        
        ```bash
        sudo groupadd docker
        ```
        
    2. Add your user to the `docker` group.
        
        ```bash
        sudo usermod -aG docker $USER
        ```
        
    3. Log out and log back in so that your group membership is re-evaluated. You can also run the following command to activate the changes to groups.
        
        ```bash
        newgrp docker
        ```
        
    4. Verify that you can run `docker` commands without `sudo`.
        
        ```bash
        docker run hello-world
        ```
        
</aside>

1. Update and Upgrade packages

```bash
sudo apt update -y && sudo apt upgrade -y
```

1. Now, pull CORD image from docker hub

```bash
docker pull dhiway/cord:develop 
```

 3. Make an empty folder named **data** in your root directory

```bash
sudo mkdir /data
```

<aside>
💡 Make sure **data** directory have **rwx** (read,write,execute) permissions. To give all the permissions use **`sudo chmod 777 /data`**

</aside>

3.1  Change the ownership of **data** directory

```bash
sudo chown $USER /data
```

1. Create Validator Account

```bash
sudo docker run dhiway/cord:develop key generate -n cord --words 24 --scheme Sr25519
```

<aside>
💡 **Note:** Please ensure that you save a copy of this output in a secure location. At this time, we will need to share your SS58 address with Sparknet Admin ([amar@dhiway.com](mailto:amar@dhiway.com) or [satish@dhiway.com](mailto:satish@dhiway.com)) as part of the validator node deployment process, which will place your node in a queue to become a validator. Ongoing development within the CORD project will allow this to become the responsibility of the on-chain governance process.

</aside>

1. Generate Node key and store in ‘data’ directory

```bash
sudo docker run -v $(pwd)/data:/data --rm dhiway/cord:develop key generate-node-key --file /data/node.key
```

1. Run Cord (Here the **<name-you-see-in-telemetry>** should be changed to name of your choice)

```bash
sudo docker run --detach --restart unless-stopped -v /data:/data --name cord dhiway/cord:develop --name **<name-you-see-in-telemetry>** --validator --base-path /data/ --chain spark --node-key-file /data/node.key --port 30333 --rpc-port 9933 --prometheus-port 9615 --rpc-methods=Safe --rpc-cors all --no-hardware-benchmarks --state-pruning 100 --blocks-pruning 100 --offchain-worker never --prometheus-external --bootnodes /dns4/sparknet-bn1.cord.network/tcp/30333/p2p/12D3KooWRsz18nHnHFxAjkGiCE6auBG5Y8LCgi2MmPTWUnvVYwxe /dns4/sparknet-bn2.cord.network/tcp/30333/ws/p2p/12D3KooWKMvTXGEWLWFVKJg3GFibapB9dmN91pmxdEwRpMvJ7jja /dns4/sparknet-bn3.cord.network/tcp/30333/ws/p2p/12D3KooWSv1Cxpa93CbGJPZsSbEbn9BAHvfkL9DyjgXAMGHcUHZ8
```

1. Generate the session keys(The **secret** mentioned here is the **secret seed** from Account generation”step 3”)

```bash
sudo docker run -v $(pwd)/data:/data -v $(pwd):/home/ubuntu --rm dhiway/cord:develop key generate-session-keys -d /data --chain spark --suri "<secret>"
```

1. Check logs 

```bash
sudo docker logs <container-id>
```

# Full Node

## Using systemd for Full Node

1. Update and Upgrade packages

```bash
sudo apt update -y && sudo apt upgrade -y
```

1. Download the compiled binary and rename it

```bash
# using wget
sudo wget -O /usr/local/bin/cord https://github.com/dhiway/cord/releases/download/125b662/cord-develop-x86_64-ubuntu-22.04 && sudo chmod +x /usr/local/bin/cord
```

1. Create a `data` directory in `home` directory

```bash
mkdir /data
```

1. Generate Node Key

```bash
sudo cord key generate-node-key --file /data/node.key
```

1. Create a systemd file

```bash
sudo vim /etc/systemd/system/cord.service
```

The content of the systemd file

```bash
[Unit]
Description=cord

[Service]
User=ubuntu
ExecStart=sudo cord --name <node-name> --validator --base-path /data/ --chain spark --node-key-file /data/node.key --port 30333 --rpc-port 9933 --prometheus-port 9615 --rpc-methods=Safe --rpc-cors all --no-hardware-benchmarks --state-pruning 100 --blocks-pruning 100 --offchain-worker never --prometheus-external --bootnodes /dns4/sparknet-bn1.cord.network/tcp/30333/p2p/12D3KooWRsz18nHnHFxAjkGiCE6auBG5Y8LCgi2MmPTWUnvVYwxe /dns4/sparknet-bn2.cord.network/tcp/30333/ws/p2p/12D3KooWKMvTXGEWLWFVKJg3GFibapB9dmN91pmxdEwRpMvJ7jja /dns4/sparknet-bn3.cord.network/tcp/30333/ws/p2p/12D3KooWSv1Cxpa93CbGJPZsSbEbn9BAHvfkL9DyjgXAMGHcUHZ8
Restart=always
RestartSec=120

[Install]
WantedBy=multi-user.target
```

1. Reload the daemon

```bash
sudo systemctl daemon-reload
```

1. Enable systemd

```bash
sudo systemctl enable cord.service
```

1. Start the cord process

```bash
sudo systemctl start cord
```

1. Check cord status

```bash
sudo systemctl status cord
```

1. Check the log

```bash
sudo tail -f /var/log/syslog
```

## Using Docker for Full Node

<aside>
💡 **Install Docker :** [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

- **Post-installation steps for Docker 👇**
    
    source : [https://docs.docker.com/engine/install/linux-postinstall/](https://docs.docker.com/engine/install/linux-postinstall/)
    
    To create the docker group and add your user:
    
    1. Create the `docker` group.
        
        ```bash
        sudo groupadd docker
        ```
        
    2. Add your user to the `docker` group.
        
        ```bash
        sudo usermod -aG docker $USER
        ```
        
    3. Log out and log back in so that your group membership is re-evaluated. You can also run the following command to activate the changes to groups.
        
        ```bash
        newgrp docker
        ```
        
    4. Verify that you can run `docker` commands without `sudo`.
        
        ```bash
        docker run hello-world
        ```
        
</aside>

1. Update and Upgrade packages

```bash
sudo apt update -y && sudo apt upgrade -y
```

1. Now, pull image from dockerhub

```bash
sudo docker pull dhiway/cord:develop 
```

 3.  Make an empty folder named **data** in root directory

```bash
sudo mkdir /data
```

<aside>
💡 Make sure **data** directory have **rwx** (read,write,execute) permissions. To give all the permissions use **`sudo chmod 777 /data`**

</aside>

3.1  Change the ownership of **data** directory

```bash
sudo chown $USER /data
```

1. Generate Node key and store in `data` directory

```bash
sudo docker run -v /data:/data --rm dhiway/cord:develop key generate-node-key --file /data/node.key
```

1. Run Cord (Here the **<name-you-see-in-telemetry>** should be changed to name of your choice)

```bash
sudo docker run --detach --restart unless-stopped -v /data:/data --name cord dhiway/cord:develop --name <name> --base-path /data/ --chain spark --node-key-file /data/node.key --port 30333 --rpc-port 9933 --prometheus-port 9615 --rpc-methods=Safe --rpc-cors all --no-hardware-benchmarks --state-pruning 100 --blocks-pruning 100 --offchain-worker never --prometheus-external --bootnodes /dns4/sparknet-bn1.cord.network/tcp/30333/p2p/12D3KooWRsz18nHnHFxAjkGiCE6auBG5Y8LCgi2MmPTWUnvVYwxe /dns4/sparknet-bn2.cord.network/tcp/30333/ws/p2p/12D3KooWKMvTXGEWLWFVKJg3GFibapB9dmN91pmxdEwRpMvJ7jja /dns4/sparknet-bn3.cord.network/tcp/30333/ws/p2p/12D3KooWSv1Cxpa93CbGJPZsSbEbn9BAHvfkL9DyjgXAMGHcUHZ8
```

1. Check logs 

```bash
sudo docker logs <container-id>
```

# Archive Node

## Using systemd for Archive Node

1. Update and Upgrade packages

```bash
sudo apt update -y && sudo apt upgrade -y
```

1. Download the compiled binary

```bash
# using wget
sudo wget -O /usr/local/bin/cord https://github.com/dhiway/cord/releases/download/125b662/cord-develop-x86_64-ubuntu-22.04 && sudo chmod +x /usr/local/bin/cord
```

1. Create `data` directory in `root` directory

```bash
mkdir /data
```

1. Generate Node Key

```bash
cord key generate-node-key --file /data/node.key

```

1. Create a Systemd file

```bash
sudo vim /etc/systemd/system/cord.service
```

The content of the systemd file

```bash
[Unit]
Description=cord

[Service]
User=ubuntu
ExecStart=sudo cord --name <node-name> --pruning=archive --base-path /data/ --chain spark --node-key-file /data/node.key --port 30333 --rpc-port 9933 --prometheus-port 9615 --rpc-methods=Safe --rpc-cors all --no-hardware-benchmarks --state-pruning 100 --blocks-pruning 100 --offchain-worker never --prometheus-external --bootnodes /dns4/sparknet-bn1.cord.network/tcp/30333/p2p/12D3KooWRsz18nHnHFxAjkGiCE6auBG5Y8LCgi2MmPTWUnvVYwxe /dns4/sparknet-bn2.cord.network/tcp/30333/ws/p2p/12D3KooWKMvTXGEWLWFVKJg3GFibapB9dmN91pmxdEwRpMvJ7jja /dns4/sparknet-bn3.cord.network/tcp/30333/ws/p2p/12D3KooWSv1Cxpa93CbGJPZsSbEbn9BAHvfkL9DyjgXAMGHcUHZ8
Restart=always
RestartSec=120

[Install]
WantedBy=multi-user.target
```

1. Reload the daemon

```bash
sudo systemctl daemon-reload
```

1. Enable cord process

```bash
sudo systemctl enable cord
```

1. Start the cord process

```bash
sudo systemctl start cord
```

1. Check cord process status

```bash
sudo systemctl status cord
```

1. Check the log

```bash
sudo tail -f /var/log/syslog
```

## Using Docker for Archive Node

<aside>
💡 **Install Docker :** [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

- **Post-installation steps for Docker 👇**
    
    source : [https://docs.docker.com/engine/install/linux-postinstall/](https://docs.docker.com/engine/install/linux-postinstall/)
    
    To create the docker group and add your user:
    
    1. Create the `docker` group.
        
        ```bash
        sudo groupadd docker
        ```
        
    2. Add your user to the `docker` group.
        
        ```bash
        sudo usermod -aG docker $USER
        ```
        
    3. Log out and log back in so that your group membership is re-evaluated. You can also run the following command to activate the changes to groups.
        
        ```bash
        newgrp docker
        ```
        
    4. Verify that you can run `docker` commands without `sudo`.
        
        ```bash
        docker run hello-world
        ```
        
</aside>

1. Update and Upgrade packages

```bash
sudo apt update -y && sudo apt upgrade -y
```

1. Pull Cord image from docker hub

```bash
docker pull dhiway/cord:develop 
```

 3.  make an empt folder named data in home directory

```bash
sudo mkdir /data
```

<aside>
💡 Make sure **data** directory have **rwx** (read,write,execute) permissions. To give all the permissions use **`sudo chmod 777 /data`**

</aside>

3.1  Change the ownership of **data** directory

```bash
sudo chown $USER /data
```

1. Generate Node key and store in ‘data’ directory

```bash
docker run -v $(pwd)/data:/data --rm dhiway/cord:develop key generate-node-key --file /data/node.key
```

1. Run Cord (Here the **<name-you-see-in-telemetry>** should be changed to name of your choice)

```bash
sudo docker run --detach --restart unless-stopped -v /data:/data --name cord dhiway/cord:develop --name <name-you-see-in-telemetry> --base-path /data/ --chain spark --node-key-file /data/node.key --port 30333 --rpc-port 9933 --prometheus-port 9615 --rpc-methods=Safe --rpc-cors all --no-hardware-benchmarks --state-pruning 100 --blocks-pruning 100 --offchain-worker never --prometheus-external --bootnodes /dns4/sparknet-bn1.cord.network/tcp/30333/p2p/12D3KooWRsz18nHnHFxAjkGiCE6auBG5Y8LCgi2MmPTWUnvVYwxe /dns4/sparknet-bn2.cord.network/tcp/30333/ws/p2p/12D3KooWKMvTXGEWLWFVKJg3GFibapB9dmN91pmxdEwRpMvJ7jja /dns4/sparknet-bn3.cord.network/tcp/30333/ws/p2p/12D3KooWSv1Cxpa93CbGJPZsSbEbn9BAHvfkL9DyjgXAMGHcUHZ8
```

1. Check logs 

```bash
sudo docker logs <container-id>
```
