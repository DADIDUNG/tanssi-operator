Run an Operator Using Docker
description: Learn how to set up and run an operator (aka validator) for Tanssi network using Docker to participate in the protocol, secure networks, and earn rewards.
icon: simple-docker
---

# Run an Operator in Tanssi Using Docker

## Introduction 

In this guide, you'll learn how to spin up a Tanssi operator using the official image release with [Docker](https://www.docker.com){target=\_blank} on Linux systems.

### Pull the Docker Image 

A Docker image is built and published in every release, containing all the necessary dependencies a Tanssi operator requires and the binary file itself.

A Docker image combines the binary corresponding to the latest stable release of the [client node](/learn/framework/architecture/#architecture){target=\_blank}, along with the Tanssi orchestrator specification file.

The following command to pull the Docker image:

```bash
docker pull moondancelabs/dancelight-chain:3
```

### Set Up the Data Directory 

Running an operator requires syncing with the Tanssi chain and storing its state.

Run the following command to create the directory where your node will store the databases containing blocks and chain states:

```bash
mkdir /var/lib/tanssi-data
```

Set the folder's ownership to the account that will run the Docker image to ensure writing permission:

```bash
chown INSERT_DOCKER_USER /var/lib/tanssi-data
```

Or run the following command if you want to run the node with the current logged-in user:

```bash
sudo chown -R $(id -u):$(id -g) /var/lib/tanssi-data
```

!!! note
    The directory is a parameter in the Docker start-up command. If you decide to create the directory elsewhere, update the command accordingly.

### Generate the Node Key 

To generate and store on disk the session keys that will be referenced on the start-up command, run the following command:

```bash
docker run --network="host" -v "/var/lib/tanssi-data:/data" \
-u $(id -u ${USER}):$(id -g ${USER}) \
{{ networks.dancelight.operator_docker_image }} key generate-node-key --file /data/node-key
```

## Start Your Node 
To spin up your node, you must run the Docker image with the `docker run` command. 

Replace `INSERT_YOUR_TANSSI_NODE_NAME` with a human-readable name and set `YOUR_IP_ADDRESS` with your public IP address.
```docker run --name tanssi -d -p 9944:9944 -p 30333:30333 -v "/var/lib/tanssi-data:/data" \
-u $(id -u ${USER}):$(id -g ${USER}) \
--entrypoint "/chain-network/tanssi-relay" \
moondancelabs/dancelight-chain:3 \
--chain=dancelight \
--base-path=/data/ \
--unsafe-force-node-key-generation \
--database=paritydb \
--name=$YOURNODENAME \
--listen-addr=/ip4/0.0.0.0/tcp/30333 \
--public-addr=/ip4/$YOURIP/tcp/30333 \
--state-pruning=archive \
--blocks-pruning=archive \
--rpc-cors=all \
--rpc-methods=unsafe \
--unsafe-rpc-external \
--rpc-max-connections=100 \
--telemetry-url='wss://telemetry.polkadot.io/submit/ 0' \
--validator
docker update --restart=unless-stopped tanssi
```
## Syncing Your Node 

The first time your node spins up, the syncing process displays lots of log information from the node configuration and the chain blocks being synced. Some errors are expected to be displayed at the beginning of the process, disappearing once the chain gets synced to the last block.

When the syncing process is finished, your node is ready for the next steps.
## Setup Account & Map Session Keys
✅ Generate session keys:
```curl http://127.0.0.1:9944 -H \
"Content-Type:application/json;charset=utf-8" -d \
  '{
    "jsonrpc":"2.0",
    "id":1,
    "method":"author_rotateKeys",
    "params": []
  }'
```
##Copy the "result" hex string. ✅ Map keys to your account:
![111](https://github.com/user-attachments/assets/10463f28-02f4-4164-b56c-25cf5e312882)
1. Go to https://polkadot.js.org/apps/?rpc=wss://dancelight.tanssi-api.network#/extrinsics
2. Navigate to Developer → Extrinsics
3. Select your validator account
4. Module: session
5. Extrinsic: setKeys
6. Keys: paste the session keys
7. Proof: 0x
8. Submit the transaction → sign with your wallet (DOT wallet)
✅ Verify mapping:

1. Go to Developer → Chain state
2. Module: session
3. Query: keyOwner
4. Key type: gran
5. Bytes: first 66 characters of your session key (e.x., 0x......)
6. Click + → your validator account should be displayed.
## Register in Symbiotic (via MetaMask & Etherscan)
1. Go to Contract (Sepolia): https://sepolia.etherscan.io/address/0x6F75a4ffF97326A00e52662d82EA4FdE86a2C548#writeContract
![image](https://github.com/user-attachments/assets/1c42d1b1-e554-4deb-a4ac-fd6fb805120a)
2. Connect Wallet

    Click “Connect to Web3”
    Select MetaMask (make sure MetaMask is on Sepolia network)
✅ Confirm connection.

3. Register Operator
![image](https://github.com/user-attachments/assets/16560043-6d19-4057-8a1c-58f3230556d1)
    Find registerOperator() function
    Click “Write”
    Confirm the transaction in MetaMask
4. Check Registration Status
![image](https://github.com/user-attachments/assets/12787b2c-e583-40b8-b13f-319d25a4647e)
    Go to “Read Contract” tab
    Find isEntity()
    Enter your wallet address
    Click “Query” → Returns true = registration successful.
## Opt In to Tanssi (via MetaMask & Etherscan)
1. Opt In to Vault
👉 Open contract: https://sepolia.etherscan.io/address/0x95CC0a052ae33941877c9619835A233D21D57351#writeContract
![image](https://github.com/user-attachments/assets/e119cf3c-5ed7-4e76-aa91-b3feca31f17c)
✅ Click “Connect to Web3” → choose MetaMask (Sepolia network)

✅ Find optin(address) function

    → Enter vault address: ```0xB94f8852443FB4faB18363D22a45cA64a8CF4482```

✅ Click “Write” → confirm transaction in MetaMask

    Done ✅
2. Check Opt In Status
👉 Open contract: https://sepolia.etherscan.io/address/0x95CC0a052ae33941877c9619835A233D21D57351#readContract
![image](https://github.com/user-attachments/assets/ca0673f0-1f29-4468-a5fd-f00863faba9d)
✅ Find isOptedIn(address who, address where)

    → who: your wallet address → where: 0xB94f8852443FB4faB18363D22a45cA64a8CF4482 → Click “Query”

✅ Result true = Opted in
3. Opt In to Tanssi Network
👉 Open contract: https://sepolia.etherscan.io/address/0x58973d16FFA900D11fC22e5e2B6840d9f7e13401#writeContract
✅ Connect Web3 (MetaMask, Sepolia)

✅ Find optin(address) function

    → Enter network address: ```0xdaD051447C4452e15B35B7F831ceE8DEb890f1a4```

✅ Click “Write” → confirm transaction in MetaMask

    Done ✅
4. Check the Registration Status:
👉 Open contract: https://sepolia.etherscan.io/address/0x58973d16FFA900D11fC22e5e2B6840d9f7e13401#readContract
![image](https://github.com/user-attachments/assets/c0771c04-c80c-408d-bcdd-46cf9a49dc2d)
✅ Find isOptedIn(address who, address where)

    → who: your wallet address → where: 0xdaD051447C4452e15B35B7F831ceE8DEb890f1a4 → Click “Query”

✅ Result true = Opted in
## Deposit
1. Wrap ETH to stETH 👉 Go to Lido Sepolia (testnet):
    ```https://stake-sepolia.testnet.fi/```

✅ Swap desired amount of Sepolia ETH → stETH

✅ Confirm transaction in MetaMask
2. Approve Vault to Spend stETH 👉 Open Collateral contract on Etherscan: https://sepolia.etherscan.io/address/0x3e3FE7dBc6B4C189E7128855dD526361c49b40Af#writeContract
![image](https://github.com/user-attachments/assets/b145f107-a59d-4bc4-8466-00d6661267c9)
✅ Connect Web3

✅ Find approve(address spender, uint256 amount)

→ spender: ```0xB94f8852443FB4faB18363D22a45cA64a8CF4482``` → amount: (your deposit amount in wei)

✅ Click “Write”, confirm in MetaMask
3. Deposit to Vault 👉 Open Vault contract on Etherscan: https://sepolia.etherscan.io/address/0xB94f8852443FB4faB18363D22a45cA64a8CF4482#writeProxyContract
✅ Connect Web3

✅ Find deposit(address operator, uint256 amount)

    → operator: your wallet address → amount: deposit amount (in wei)

✅ Click “Write”, confirm in MetaMask

🎉 Done! You’re opted in and have deposited collateral to the Vault for Tanssi (Sepolia testnet).

--------------------------------------------FINISH RUN TANSSI OPERATOR TESTNET-----------------------------------------------
