---
title: Explorer
hide_title: true
sidebar_position: 4
---

import useBaseUrl from "@docusaurus/useBaseUrl";

# Explorer
Axon uses [BlockScan](https://github.com/Magickbase/blockscan), an explorer tailored for Ethereum-compatible chains. By deploying BlockScan, you can access data from Axon-based chains and monitor their status.

## Deployment

Docker Compose is the current deployment method.

### Docker Compose

1. Clone BlockScan repository to your target machine:

``` shell
git clone --depth=1 https://github.com/Magickbase/blockscan.git
cd blockscan
```

2. Edit the config file located at `blockscan/dev.env`. Make the following adjustments:
   - The purpose of the dev.env file is to enable explorer to connect to your locally existing components, but if the axon node and explorer are in the same machine, you can also deploy directly using our docker-compose without any problems

```shell
ETHEREUM_JSONRPC_VARIANT=geth
# Http address of axon rpc,The default port is port 8000 of the host 
# Please change to your own axon rpc address port
ETHEREUM_JSONRPC_HTTP_URL=http://host.docker.internal:8000
# Http address of Axon RPC, defaults to port 8000 of the host
# Please change to your own axon rpc address port
ETHEREUM_JSONRPC_TRACE_URL=http://host.docker.internal:8000
# The default is the postgresql address deployed in docker-compose
# If you have your own postgresql, change it
DATABASE_URL=postgresql://postgres:postgres123@db:5432/blockscan?ssl=false
```

3. Execute one-click deployment:

```shell
cd ./bolockscan
sudo docekr-compose up -d
```

4. Access the explorer at:

```shell
# Replace this address with the actual URL where your BlockScan explorer is deployed
http://127.0.0.1:4020
```

<img alt="Untitled" src={useBaseUrl("Axon Explorer interface.png")}  width="80%"/>
