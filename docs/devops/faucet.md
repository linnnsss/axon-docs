---
title: Faucet
hide_title: true
sidebar_position: 4
---

import useBaseUrl from "@docusaurus/useBaseUrl";

# Faucet

Axon Faucet provides tokens for testing purposes. You can explore and experiment with their full range of capabilities in a safe and controlled environment. These tokens can be used for a variety of purposes, including transaction, staking, delegation, and more.

## Deployment

### 1. Copy axon-devops directory to the target machine

```shell
git clone https://github.com/axonweb3/axon-devops
cd axon-devops/axon-faucet
```

### 2. Edit the config file

- axon-devops/axon-faucet/config.yml

```yml
deploy_path: "/home/ckb/axon-faucet"
# Which path that the axon-faucet repo clone into
faucet_repo: "https://github.com/axonweb3/axon-faucet.git"
# The github address of axon-faucet 
faucet_branch: "master"
# The branch name of axon-faucet 
axon_faucet_rpc_url: http://xxxx.xxx.xxx.xxx:8000
# Http address of axon rpc
axon_faucet_claim_value: 1000000000000000000
# Faucet claim value
mongodb_password: mongodbpassword
# mongo db password that using by faucet
mongodb_url: mongodb://root:mongodbpassword@faucet-mongo:27017
# URL address of mongo db that using by faucet
```

### 3. Execute one-click deployment

```shell
cd axon-devops/axon-exeplorer
make start #start axon faucet
docker-compose ps # check axon faucet status
make down #stop axon faucet
```

### 4. Initialize a mnemonic phrase

```shell
curl http://localhost:8502/api/import-mnemonic?mnemonic=test%20test%20test%20test%20test%20test%20test%20test%20test%20test%20test%20junk
# Make sure to use unique mnemonic words in real deployment, NOT the sample words (these "test"s) here.
```

Once successfully deployed, open your browser and visit the faucet page <http://localhost:8502/> to claim your tokens.

<img alt="Untitled" src={useBaseUrl("img/devops/Axon Faucet interface.png")}  width="80%"/>
