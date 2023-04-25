---
title: 以太坊节点搭建教程
date: 2022-11-25 11:25:25
tags: 
- ETH
- 以太坊
- 节点搭建
categories:
- ETH
---

# 前期准备：

###### 本文采用geth+prysm+mev-boost+web3signer+aws Secrets Manager+Grafana&Prometheus架构搭建。

- geth：执行层节点。
- prysm：共识层节点和validator。
- mev-boost：使用mev构建区块。
- web3signer：用来管理密钥远程签名。
- aws Secrets Manager：存储密钥。
- Grafana&Prometheus：监控服务器及节点状态。



###### 机器要求：

- **操作系统**：64 位 Linux
- **CPU**：8核 @ 2.8+ GHz
- **内存**：32GB内存
- **存储**：2.5TB 可用空间的 SSD

> 本教程是在一台机器下构建的，实际使用根据情况拆分。



###### 端口：

| 端口/协议       | 防火墙规则           | 原因/注意事项                                                |
| --------------- | -------------------- | ------------------------------------------------------------ |
| `8545/TCP`      | 阻止所有流量。       | 这是执行节点的查询 API 的 JSON-RPC 端口。您（和应用程序）可以使用此端口检查执行节点状态，查询执行层链数据，甚至提交交易。这个端口一般不应该暴露给外界。 |
| `3500/TCP`      | 阻止所有流量。       | 这是信标节点查询 API 的 JSON-RPC 端口。您（和应用程序）可以使用此端口检查信标节点状态并查询共识层链数据。这个端口一般不应该暴露给外界。 |
| `8551/TCP`      | 阻止所有流量。       | 您的信标节点使用此端口连接到执行节点的引擎 API 。只有当您的本地信标节点连接到远程执行节点时，才应允许通过此端口的入站和出站流量。 |
| `4000/TCP`      | 阻止所有流量。       | 您的验证器使用此端口通过gRPC连接到您的信标节点。只有当您的本地验证器连接到远程信标节点时，才应允许通过此端口的入站和出站流量。 |
| `*/UDP+TCP`     | 允许出站流量。       | 为了发现对等点，Prysm 的信标节点通过随机端口拨出。允许来自任何端口的出站 TCP/UDP 流量将有助于 Prysm 找到对等点。 |
| `13000/TCP`     | 允许入站和出站流量。 | 在我们发现对等点之后，我们通过这个端口拨叫它们为libp2p建立一个持续的连接，并且所有 gossip/p2p 请求和响应都将通过该连接流动。 |
| `12000/UDP`     | 允许入站和出站流量。 | 你的信标节点公开这个 UDP 端口，以便其他以太坊节点可以发现你的节点，请求链数据，并提供链数据。 |
| `30303/TCP+UDP` | 允许入站和出站流量。 | `30303/TCP`是您的执行节点的侦听器端口，`30303/UDP`而是它的发现端口。此规则允许您的执行节点连接到其他对等节点。请注意，某些客户端`30301`默认使用。 |



## 搭建步骤：



### 1.安装prysm

创建一个文件夹`ethereum`，然后在其中创建两个子文件夹：`consensus`和`execution`。

```
$ cd consensus
$ mkdir prysm && cd prysm
$ curl https://raw.githubusercontent.com/prysmaticlabs/prysm/master/prysm.sh --output prysm.sh && chmod +x prysm.sh
```



#### 生成 JWT 

共识节点和执行节点之间的 HTTP 连接需要使用JWT 令牌进行身份验证。

```
$ ./prysm.sh beacon-chain generate-auth-secret
```



### 2.安装geth

```
$ cd execution
# 在下载页面找到对应版本的geth，https://geth.ethereum.org/downloads
$ wget https://gethstore.blob.core.windows.net/builds/geth-linux-amd64-1.11.6-ea9e62ca.tar.gz
$ tar xvf geth-linux-amd64-1.11.6-ea9e62ca.tar.gz
# 运行geth，
$ geth --mainnet --http --http.api eth,net,engine,admin --authrpc.jwtsecret /path/to/jwt.hex 
```

> 执行层需要同步区块，可能需要很长时间

```
# 执行层常用命令
# 查看当前区块高度
$ curl --location --request POST 'localhost:8545' --header 'Content-Type: application/json' --data-raw '{ "jsonrpc":"2.0", "method":"eth_blockNumber", "params":[], "id":83 }'
# 查看是否同步完成
$ curl --location --request POST 'localhost:8545' --header 'Content-Type: application/json' --data-raw '{ "jsonrpc":"2.0", "method":"eth_syncing", "params":[], "id":83 }'
```



### 3.运行信标链

```
$ cd prysm
# --suggested-fee-recipient 用来接收执行层收益，要替换成你自己的以太坊地址
$ prysm.sh beacon-chain --execution-endpoint=http://localhost:8551 --jwt-secret=path/to/jwt.hex --suggested-fee-recipient=0x01234567722E6b0000012BFEBf6177F1D2e9758D9
```

```
# 共识层常用命令
# 共识层同步状态
curl http://localhost:3500/eth/v1/node/syncing
# 共识层健康状态
curl http://localhost:8080/healthz
# 共识层连接状态
curl http://localhost:3500/eth/v1alpha1/node/eth1/connections
```



### 4.运行验证器

```
# --validators-external-signer-url --validators-external-signer-public-keys用来配置web3signer
$ prysm.sh validator --validators-external-signer-url=http://localhost:9000 --validators-external-signer-public-keys=http://localhost:9000/api/v1/eth2/publicKeys --web --beacon-rpc-provider=localhost:4000
```



### 5.安装mev-boost

```
安装 mev-boost

$ cd ~
$ wget https://github.com/flashbots/mev-boost/releases/download/v1.4.0/mev-boost_1.4.0_linux_amd64.tar.gz

# 提取存档。全局安装 mev-boost 并删除下载残留物。
$ tar xvf mev-boost_1.4.0_linux_amd64.tar.gz

# 创建一个 systemd 服务文件来存储服务配置，该文件告诉 systemd 以 mevboost 用户身份运行 mev-boost。
$ sudo nano /etc/systemd/system/mevboost.service

# 将以下内容粘贴到文件中以在主网上运行 mev-boost。您必须用一个或多个现有继电器替换https://example.com此配置。我们有一份您可以探  # 索的继电器列表。完成后退出并保存（Ctrl+ X、、、Y）Enter。

[Unit]
Description=mev-boost (Mainnet)
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=mevboost
Group=mevboost
Restart=always
RestartSec=5
ExecStart=mev-boost \
    -mainnet \
    -relay-check \
    -relays https://example.com

[Install]
WantedBy=multi-user.target

# 您可以将多个以逗号分隔的中继添加到-relays标志中，如下所示：-relays https://relay1,https://relay2。


# 然后使用以下命令启动服务并检查状态以确保其正常运行。
$ sudo systemctl daemon-reload
$ sudo systemctl start mevboost
$ sudo systemctl status mevboost
```

> 启动mev-boost后在共识层节点增加--http-mev-relay=http://localhost:18550，在验证器节点增加--enable-builder --suggested-fee-recipient=<address>。



### 6.安装web3signer

```
# 需要提前准备aws密钥。
# 下载web3signer
$ wget https://artifacts.consensys.net/public/web3signer/raw/names/web3signer.tar.gz/versions/23.3.1/web3signer-23.3.1.tar.gz
$ tar -zxvf web3signer-23.3.1.tar.gz
# 启动web3signer
$ bin/web3signer --http-host-allowlist=* eth2 --aws-secrets-enabled=true --aws-secrets-access-key-id=AKI******Q2 --aws-secrets-secret-access-key=Ad*************WiI --aws-secrets-region=ap-southeast-1 --key-manager-api-enabled=true --slashing-protection-enabled=false
```

>  运行成功后需要在验证器客户端配置--validators-external-signer-url --validators-external-signer-public-keys。

```
# 常用命令
# 重新加载密钥
$ curl -X POST http://localhost:9000/reload
# key列表
$ curl -X GET http://localhost:9000/eth/v1/keystores
# 添加key
$ curl -X POST http://127.0.0.1:9000/eth/v1/keystores --header "Content-Type: application/json" --data '<keystore>'
# 删除key
$ curl -H "Content-Type: application/json" -X DELETE -d '{"pubkeys":["0xb66af903a56c792253f55a247d2d5e428c6735fe2aaff53cc54dd5a9ddc70dfe790565361744459546f3c0c6995e8dce"]}'  http://127.0.0.1:9000/eth/v1/keystores
```



### 7.安装prometheus

```
$ sudo wget https://github.com/prometheus/prometheus/releases/download/v2.39.1/prometheus-2.39.1.linux-amd64.tar.gz
$ tar prometheus-2.39.1.linux-amd64.tar.gz

# 找到prometheus.yml文件并替换为以下内容：
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).
# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093
# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"
# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  - job_name: 'validator'
    static_configs:
      - targets: ['localhost:8081']
  - job_name: 'beacon node'
    static_configs:
      - targets: ['localhost:8080']
  - job_name: 'slasher'
    static_configs:
      - targets: ['localhost:8082']
      
# 运行prometheus
$ ./prometheus --web.listen-address=0.0.0.0:9095
```

> prometheus后台：http://localhost:9090/graph



### 8.安装grafana

```
# 下载grafana
https://grafana.com/grafana/download
# 运行grafana
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl status grafana-server

# 打开grafana后台http://localhost:3000，默认账户密码是admin。
# 添加上一步的数据源和仪表板就可以查看节点状态了
# 仪表板模版：https://docs.prylabs.network/assets/grafana-dashboards/small_amount_validators.json
```

> 本文所有的组件都可以添加到grafana，这里就不多说了。



## 总结

至此整套以太坊节点就搭建完成了，等待执行层和共识层同步完成，就可以尝试质押一个validator，然后把密钥导入aws运行了，然后可以在grafana仪表板查看各种状态。







## 参考文档：

https://grafana.com/docs/

https://prometheus.io/docs/

https://docs.prylabs.network/docs/

https://docs.web3signer.consensys.net/




