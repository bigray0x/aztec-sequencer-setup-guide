# aztec-sequencer-setup-guide

![Image 5-1-25 at 9 42 PM](https://github.com/user-attachments/assets/54e3126a-f82d-4f97-9874-377c52a27df9)

Aztec Network is a private ZK-rollup on Ethereum, enabling decentralized applications to access privacy and scale. Aztec’s rollup is secured by its industry-standard PLONK proving mechanism used by the leading zero-knowledge scaling projects.

Team has said running this node isn’t incentivized but it’s light weight and we can get contributions roles by running it so I’ll get involved.

## Pre-requisites :

- Pc or VPS
- docker 
- at least 5GB of ram.

## Node Guide :

This will work on Mac and Linux based devices.

### Step 1 : Install dependencies 

MacOS

- Install Homebrew if needed
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
- Install Docker & Socat
```bash
brew install socat
brew install --cask docker
```

Linux OS
```bash
sudo apt update -y && sudo apt upgrade -y
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update -y && sudo apt upgrade -y

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
- Test Docker
```bash
sudo docker run hello-world
```
```bash
sudo systemctl enable docker
sudo systemctl restart docker
```
```bash
sudo apt update
sudo apt install -y curl git socat docker.io docker-compose
sudo systemctl enable --now docker
```

### Step 2 : Install Aztec Cli

```bash
bash -i <(curl -s https://install.aztec.network)
```

MacOS

```bash
 echo 'export PATH="$HOME/.aztec/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
```
or for Bash

```bash
echo 'export PATH="$HOME/.aztec/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc
```

Linux OS

```bash
echo 'export PATH="$HOME/.aztec/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc
```
Test the cli functionality 

```bash
aztec --version
```
```bash
aztec-up alpha-testnet
```

Output should be in this format 0.8.5 or so.

### Step 3 : Get your public IP and expose your ports

```bash
curl ifconfig.me
```

Save the Ip gotten.

Open your ports 
```bash
# Firewall
ufw allow 22
ufw allow ssh
ufw enable
```
```bash
# Sequencer
ufw allow 40400
ufw allow 8080
```

### Step 4 : create and env file to store sensitive details

Get your own Sepolia RPC URL & Sepolia BEACON URL: Register in Ankr, Fund it with any amount of USDT via your wallet, Create a project, get your normal sepolia rpc and beacon sepolia rpc. (Reliable)

Link - https://www.ankr.com/rpc/?utm_referral=5J4259FS4V

You can create a Sepolia RPC URL in Alchemy and Use this https://rpc.drpc.org/eth/sepolia/beacon as free BEACON RPC. (Limited)

```bash
nano ~/.aztec-sequencer.env
```

Paste this into the file and edit with your private RPCs and beacon or use public.

```bash
# Your Infura Execution RPC (sepolia)
ETHEREUM_HOSTS=rpc.ankr.com/eth_sepolia

# Reliable Public Beacon Chain (Consensus) Endpoint
L1_CONSENSUS_HOST_URLS=lodestar-sepolia.chainsafe.io

# Replace with your values
P2P_IP=your.public.ip.address
SEQUENCER_VALIDATOR_PRIVATE_KEY=YourPrivateKey
SEQUENCER_COINBASE=0xYourPublicWalletAddress
```
Save the file with control x + y + enter.


### Step 5 : Start The Sequencer Node

```bash
screen -S Aztec
```
Load the .env file 

```bash
source ~/.aztec-sequencer.env
```
```bash
aztec start --node --archiver --sequencer \
  --network alpha-testnet \
  --l1-rpc-urls "$ETHEREUM_HOSTS" \
  --l1-consensus-host-urls "$L1_CONSENSUS_HOST_URLS" \
  --sequencer.validatorPrivateKey "$SEQUENCER_VALIDATOR_PRIVATE_KEY" \
  --sequencer.coinbase "$SEQUENCER_COINBASE" \
  --p2p.p2pIp "$P2P_IP" \
  --p2p.maxTxPoolSize 1000000000 \
  --sequencer.governanceProposerPayload 0x54F7fe24E349993b363A5Fa1bccdAe2589D5E5Ef
```
![Image 5-1-25 at 8 56 PM](https://github.com/user-attachments/assets/fb9372d1-4446-4c04-b2f8-5406a0ac0c09)

### Step 6 : Register as an apprentice on discord

Wait 10-15mins for your sequencer node to come up then proceed with this step.

Go to the Aztec discord operators channel :

[discord.com/aztec](https://discord.com/channels/1144692727120937080/1367196595866828982)

You can continue the guide with this commands if you need help.

### Step A: Get the latest proven block number:

```bash
curl -s -X POST -H 'Content-Type: application/json' \
-d '{"jsonrpc":"2.0","method":"node_getL2Tips","params":[],"id":67}' \
http://localhost:8080 | jq -r ".result.proven.number"
```

- Save this block number for the next steps
- Example of the output: 20834

### Step B: Generate your sync proof

```bash
curl -s -X POST -H 'Content-Type: application/json' \
-d '{"jsonrpc":"2.0","method":"node_getArchiveSiblingPath","params":["BLOCK_NUMBER","BLOCK_NUMBER"],"id":67}' \
http://localhost:8080 | jq -r ".result"
```

Replace 2x BLOCK_NUMBER with your number

### Step C: Register with Discord

Type the following command in this Discord server: /operator start
After typing the command, Discord will display option fields that look like this:
address: Your validator address (Ethereum Address)
block-number: Block number for verification (Block number from Step 1)
proof: Your sync proof (step 2 result)

Then you'll get your Apprentice Role.

### Step 7 : Register as a validator 

```bash
aztec add-l1-validator \
  --l1-rpc-urls "$ETHEREUM_HOSTS" \
  --private-key "$SEQUENCER_VALIDATOR_PRIVATE_KEY" \
  --attester "$SEQUENCER_COINBASE" \
  --proposer-eoa "$SEQUENCER_COINBASE" \
  --staking-asset-handler 0xF739D03e98e23A7B65940848aBA8921fF3bAc4b2 \
  --l1-chain-id 1 
```
![Image 5-1-25 at 9 04 PM (1)](https://github.com/user-attachments/assets/ed683c0c-251e-4049-9479-fd4ec86ee640)

### Facing Known Errors like

-> ```The computed genesis archive tree root 0x1f9a1f495b0a8f12ebc07e1bea931ea1e2b6f862b6da9d5395ab11c5374ccabb does not match the expected genesis archive tree root 0x0e95630d12892327952c4a86062c78fa1efb960050efe6b75eff48342d581d96 for the rollup deployed at 0x8d1cc702453fa889f137dbd5734cdb7ee96b6ba0```

you need to remove the aztec cli and reinstall and restart the node using step 2.

or

-> ```"unable to get blob sidecar block" or "archiver Error syncing archiver: No blob bodies found for block" or "blob-sink-client Unexpected end of JSON input"```

you need to change your beacon rpc to a valid one or check if it's maxxed out.

or

--> ```no URL was provided to transport```

you need to source your .env file again inside the screen and restart the node or check if the .env file is in same path because it contains both beacon and rpc URLs needed to run the node.

### possible rpc solutions

this solution helps you add up to 5 RPCs so your node can always fall back on others and stay reliable even if it's free.

- open the .env file and replace the variables with this new line

  ```
  screen -D -r Aztec && nano ~/.aztec-sequencer.env
  ```
  
replace the first two lines with up to 5 RPCs and Beacons.

```
export L1_CONSENSUS_HOST_URLS=rpc_1,rpc_2,rpc_3,rpc_4,rpc_5
export ETHEREUM_HOSTS=rpc_1,rpc_2,rpc_3,rpc_4,rpc_5

```

then restart the node

 ```bash
source ~/.aztec-sequencer.env
```
```bash
aztec start --node --archiver --sequencer \
  --network alpha-testnet \
  --l1-rpc-urls "$ETHEREUM_HOSTS" \
  --l1-consensus-host-urls "$L1_CONSENSUS_HOST_URLS" \
  --sequencer.validatorPrivateKey "$SEQUENCER_VALIDATOR_PRIVATE_KEY" \
  --sequencer.coinbase "$SEQUENCER_COINBASE" \
  --p2p.p2pIp "$P2P_IP" \
  --p2p.maxTxPoolSize 1000000000 \
  --sequencer.governanceProposerPayload 0x54F7fe24E349993b363A5Fa1bccdAe2589D5E5Ef
```

you can use free rpcs from different platforms to prevent ip address restrictions

examples:

alchemy.com rpc,
ankr.com rpc,
drpc.org rpc

That’s all for now to get it up and running.

If you want more simpler guides check my GitHub.

