---
title: Cardano(Ada)节点搭建
date: 2022-06-13 11:15:18
tags: 
- 区块链
- ada
- cardano
- 节点搭建
categories:
- 区块链
---

ada在我来看是比较神奇的一个币，市值经常排在第三位也就是btc和eth之后，但是跟以太坊相比可以说没什么生态。

ada的文档和节点的操作流程跟其他链比起来真的是太不人性化了！

##### 下面是ada节点的搭建流程：



### 最低硬件要求

在硬件方面，您应该确保具备以下条件：

- 10 GB 内存
- 处理器至少有两个核心
- 24 GB 硬盘空间
- 良好的网络连接和每小时约 1 GB 的带宽
- 公共 IP4 地址
- 端口3001

请注意，处理器速度不是运行节点的重要因素。



### 节点搭建

下载二进制包

https://github.com/input-output-hk/cardano-node#linux-executable

准备配置文件

```
cp -p cardano-node ~/.local/bin/
cp -p cardano-cli ~/.local/bin/
cardano-cli --version
cardano-node --version
mkdir -p config
cd config
curl -O -J https://hydra.iohk.io/build/7370192/download/1/mainnet-config.json
curl -O -J https://hydra.iohk.io/build/7370192/download/1/mainnet-byron-genesis.json
curl -O -J https://hydra.iohk.io/build/7370192/download/1/mainnet-shelley-genesis.json
curl -O -J https://hydra.iohk.io/build/7370192/download/1/mainnet-alonzo-genesis.json
curl -O -J https://hydra.iohk.io/build/7370192/download/1/mainnet-topology.json

```

出块节点要保证只与中继节点连接，通过配置topology.json 

```
//出块节点
{
  "Producers": [
    {
      "addr": "<RELAY IP ADDRESS>",
      "port": <PORT>,
      "valency": 1
    }
  ]
}
```

```
//中继节点
{
  "Producers": [
    {
      "addr": "<BLOCK-PRODUCING IP ADDRESS>",
      "port": <PORT>,
      "valency": 1
    },
    {
      "addr": "<IP ADDRESS>",
      "port": <PORT>,
      "valency": 1
    },
    {
      "addr": "<IP ADDRESS>",
      "port": <PORT>,
      "valency": 1
    }
  ]
}
```



运行节点

```
cardano-node run \
   --topology /data/ada/config/mainnet-topology.json \
   --database-path /data/ada/data \
   --socket-path /data/ada/data/node.socket \
   --host-addr 0.0.0.0 \
   --port 3001 \
   --config /data/ada/config/mainnet-config.json
//启动生产节点
cardano-node run \
   --topology /data/ada/config/mainnet-topology.json \
   --database-path /data/ada/data \
   --socket-path /data/ada/data/node.socket \
   --host-addr 0.0.0.0 \
   --port 3001 \
   --config /data/ada/config/mainnet-config.json
	 --shelley-kes-key /data/ada/keys/kes.skey \
	 --shelley-vrf-key /data/ada/keys/vrf.skey \
	 --shelley-operational-certificate /data/ada/keys/node.cert
```



配置node-socket环境变量

```
//.bashrc
export CARDANO_NODE_SOCKET_PATH="/data/ada/data/node.socket"
```



查看追块信息

```
//--mainnet | --testnet-magic 1097911063
cardano-cli query tip --mainnet
```



#### 升级步骤

下载最新二进制包

https://github.com/input-output-hk/cardano-node#linux-executable

```
停止节点
替换包
cp -p cardano-node ~/.local/bin/
cp -p cardano-cli ~/.local/bin/
检查版本
cardano-cli --version
cardano-node --version
重启节点
```





### 注册stake pool

1，生成操作证书
https://github.com/input-output-hk/cardano-node/blob/master/doc/stake-pool-operations/node_keys.md

2，在链上注册stake  addr

https://github.com/input-output-hk/cardano-node/blob/master/doc/stake-pool-operations/register_key.md

3，注册stake pool（包括手续费押金等）

https://github.com/input-output-hk/cardano-node/blob/master/doc/stake-pool-operations/register_stakepool.md#generate-stake-pool-registration-certificate

成为stake pool成本大概需要501个ada币（500押金和0.4手续费[上边两个注册操作需要手续费]）。

##### 提取奖励

https://github.com/input-output-hk/cardano-node/blob/master/doc/stake-pool-operations/withdraw-rewards.md



#### 交易示例

```
//获取主网参数
cardano-cli query protocol-parameters \
  --mainnet \
  --out-file protocol.json
```

```
//查询账户余额
cardano-cli query utxo \
  --address $(cat /data/ada/keys/paymentwtihstake.addr) \
  --mainnet
```

```
//--tx-in引用上边查到的utxo
cardano-cli transaction build-raw \
--tx-in 26e11321a1f543f03fb0b6361b8637329f396dcc1f72db4d545240e1255fab2a#0 \
--tx-out DdzFFzCqrhtDFWtaaoG2yA6BUGs4jVHEF5FtqXZ9QZZAqkhi8mx25F9VfpPhkfshcn24EPUN3mRD67bwzKTPFBXCVo52uTsJh2woWcjp+0 \
--tx-out $(cat /data/ada/keys/paymentwtihstake.addr)+0 \
--invalid-hereafter 0 \
--fee 0 \
--out-file tx.draft
```

```
//计算交易费用
cardano-cli transaction calculate-min-fee \
--tx-body-file tx.draft \
--tx-in-count 1 \
--tx-out-count 2 \
--witness-count 1 \
--byron-witness-count 0 \
--mainnet \
--protocol-params-file protocol.json
```

```
//计算余额
expr <UTXO BALANCE> - <AMOUNT TO SEND> - <TRANSACTION FEE>
```

```
//获取slot
cardano-cli query tip --mainnet
```

```
//--invalid-hereafter 填上条命令获取的slot加200
cardano-cli transaction build-raw \
--tx-in 26e11321a1f543f03fb0b6361b8637329f396dcc1f72db4d545240e1255fab2a#0 \
--tx-out DdzFFzCqrhtDFWtaaoG2yA6BUGs4jVHEF5FtqXZ9QZZAqkhi8mx25F9VfpPhkfshcn24EPUN3mRD67bwzKTPFBXCVo52uTsJh2woWcjp+20000000 \
--tx-out $(cat /data/ada/keys/paymentwtihstake.addr)+9822575 \
--invalid-hereafter 50841283 \
--fee 177425 \
--out-file tx.raw
```

```
//签名
cardano-cli transaction sign \
--tx-body-file tx.raw \
--signing-key-file /data/ada/keys/payment.skey \
--mainnet \
--out-file tx.signed
```

```
//广播
cardano-cli transaction submit \
--tx-file tx.signed \
--mainnet
```

### 查看kes

```
curl http://localhost:12798/metrics | grep -si cardano_node_metrics | grep -i KES
```



### 矿池源数据信息

{
  "name": "Huobi",
  "description": "As a world-leading blockchain industry player, Huobi Group was founded in 2013 to make breakthroughs in core blockchain technology and integrate it with other industries.",
  "ticker": "Huobi",
  "homepage": "https://www.huobi.com"
  }



### 查询预计出块信息

```
cardano-cli query leadership-schedule --genesis /data/ada/config/mainnet-shelley-genesis.json --mainnet --vrf-signing-key-file /data/ada/keys/vrf.skey --current --cold-verification-key-file /data/ada/keys/cold.vkey
```



### 查询权益快照信息

```
cardano-cli query stake-snapshot --stake-pool-id 2a999d0685a1dbddfae38e754d6380a6398151130a7a2f8065b0705d --main net
```



### log等级

```
minSeverity: Debug(Debug, Info, Notice and Warning and Error)
```



### 注册矿池信息

```
cardano-cli stake-pool metadata-hash --pool-metadata-file poolMetaData.json 
```

```
cardano-cli stake-pool registration-certificate \
--cold-verification-key-file /data/ada/keys/cold.vkey \
--vrf-verification-key-file /data/ada/keys/vrf.vkey \
--pool-pledge 0 \
--pool-cost 340000000 \
--pool-margin 0 \
--pool-reward-account-verification-key-file /data/ada/keys/reward_stake.vkey \
--pool-owner-stake-verification-key-file /data/ada/keys/stake.vkey \
--mainnet \
--pool-relay-ipv4 47.245.35.85 \
--pool-relay-port 3001 \
--metadata-url https://inj.2k7rqfvt5ahuc.hbpcrypto.com/cardano/huobi.json \
--metadata-hash a1c13e4511099ca63f1af6533208c58a1891ea696a27e93e12e93bd76902fbda \
--out-file pool-registration.cert
```

```
cardano-cli stake-address delegation-certificate \
    --stake-verification-key-file /data/ada/keys/stake.vkey \
    --cold-verification-key-file /data/ada/keys/cold.vkey \
    --out-file delegation.cert
```

```
cardano-cli query utxo \
    --address $(cat /data/ada/keys/paymentwtihstake.addr) \
    --mainnet
```

```
cardano-cli transaction build-raw \
    --tx-in 697eddea14e29952ecc65d85870da7d8de7142c598867d147bdb775dadfcd783#0 \
    --tx-out $(cat /data/ada/keys/paymentwtihstake.addr)+0 \
    --invalid-hereafter 0 \
    --fee 0 \
    --out-file tx.draft \
    --certificate-file pool-registration.cert \
    --certificate-file delegation.cert
```

```
cardano-cli query protocol-parameters \
    --mainnet \
    --out-file protocol.json
```

```
cardano-cli transaction calculate-min-fee \
    --tx-body-file tx.draft \
    --tx-in-count 1 \
    --tx-out-count 1 \
    --witness-count 3 \
    --byron-witness-count 0 \
    --mainnet \
    --protocol-params-file protocol.json
```

```
protocol.json中查找"poolDeposit": 500000000
expr <UTXO BALANCE> - <DEPOSIT> - <TRANSACTION FEE>
```

```
cardano-cli query tip --mainnet
```

```
cardano-cli transaction build-raw \
    --tx-in 697eddea14e29952ecc65d85870da7d8de7142c598867d147bdb775dadfcd783#0 \
    --tx-out $(cat /data/ada/keys/paymentwtihstake.addr)+6470224 \
    --invalid-hereafter 61608827 \
    --fee 196917 \
    --out-file tx.raw \
    --certificate-file pool-registration.cert \
    --certificate-file delegation.cert
```

```
cardano-cli transaction sign \
    --tx-body-file tx.raw \
    --signing-key-file /data/ada/keys/payment.skey \
    --signing-key-file /data/ada/keys/stake.skey \
    --signing-key-file /data/ada/keys/cold.skey \
    --mainnet \
    --out-file tx.signed
```

```
cardano-cli transaction submit \
    --tx-file tx.signed \
    --mainnet
```

```
cardano-cli stake-pool id --cold-verification-key-file /data/ada/keys/cold.vkey --output-format "hex"
```

```
cardano-cli query ledger-state --mainnet | grep publicKey | grep 2a999d0685a1dbddfae38e754d6380a6398151130a7a2f8065b0705d
```



### 把stake地址注册到链上

```
cardano-cli stake-address registration-certificate \
--stake-verification-key-file /data/ada/keys/stake.vkey \
--out-file stake.cert
```

```
cardano-cli query utxo \
    --address $(cat /data/ada/keys/paymentwtihstake.addr) \
    --mainnet
```

```
cardano-cli transaction build-raw \
--tx-in 1bfe34527736e7b7cc0ad6c95790ee99f4f67650f7d83f64bd494e2369f3129c#1 \
--tx-out $(cat /data/ada/keys/paymentwtihstake.addr)+0 \
--invalid-hereafter 0 \
--fee 0 \
--out-file tx.draft \
--certificate-file stake.cert
```

```
cardano-cli query protocol-parameters \
    --mainnet \
    --out-file protocol.json
```

```
cardano-cli transaction calculate-min-fee \
--tx-body-file tx.draft \
--tx-in-count 1 \
--tx-out-count 1 \
--witness-count 2 \
--byron-witness-count 0 \
--mainnet \
--protocol-params-file protocol.json
```

```
protocol.json中查找"stakeAddressDeposit": 2000000
expr <UTXO BALANCE> - <DEPOSIT> - <TRANSACTION FEE>
```

```
cardano-cli transaction build-raw \
--tx-in 1bfe34527736e7b7cc0ad6c95790ee99f4f67650f7d83f64bd494e2369f3129c#1 \
--tx-out $(cat /data/ada/keys/paymentwtihstake.addr)+7643874 \
--invalid-hereafter 51854359 \
--fee 178701 \
--out-file tx.raw \
--certificate-file stake.cert
```

```
cardano-cli transaction sign \
--tx-body-file tx.raw \
--signing-key-file /data/ada/keys/payment.skey \
--signing-key-file /data/ada/keys/stake.skey \
--mainnet \
--out-file tx.signed
```



### 生成密钥

```
cardano-cli address key-gen \
--verification-key-file payment.vkey \
--signing-key-file payment.skey
```

```
cardano-cli stake-address key-gen \
--verification-key-file stake.vkey \
--signing-key-file stake.skey
```

```
cardano-cli address build \
--payment-verification-key-file payment.vkey \
--stake-verification-key-file stake.vkey \
--out-file paymentwithstake.addr \
--mainnet
```

```
cardano-cli stake-address build \
--stake-verification-key-file stake.vkey \
--out-file stake.addr \
--mainnet
```

```
cardano-cli node key-gen \
--cold-verification-key-file cold.vkey \
--cold-signing-key-file cold.skey \
--operational-certificate-issue-counter-file cold.counter
```

```
cardano-cli node key-gen-VRF \
--verification-key-file vrf.vkey \
--signing-key-file vrf.skey
```

```
cardano-cli node key-gen-KES \
--verification-key-file kes.vkey \
--signing-key-file kes.skey
```

  

### 生成操作证书

```
cat /data/ada/config/mainnet-shelley-genesis.json | grep KESPeriod
> "slotsPerKESPeriod": 3600,
```

```
cardano-cli query tip --mainnet
{
    "epoch": 259,
    "hash": "dbf5104ab91a7a0b405353ad31760b52b2703098ec17185bdd7ff1800bb61aca",
    "slot": 26633911,
    "block": 5580350
}
```

```
expr 26633911 / 3600
> 7398
```

```
//--kes-period 填写上一步计算结果
cardano-cli node issue-op-cert \
--kes-verification-key-file /data/ada/keys/kes.vkey \
--cold-signing-key-file /data/ada/keys/cold.skey \
--operational-certificate-issue-counter /data/ada/keys/cold.counter \
--kes-period 514 \
--out-file node.cert
```



### 交易

```
//获取主网参数
cardano-cli query protocol-parameters \
  --mainnet \
  --out-file protocol.json
```

```
//查询账户余额
cardano-cli query utxo \
  --address $(cat /data/ada/keys/paymentwtihstake.addr) \
  --mainnet
```

```
//--tx-in引用上边查到的utxo
cardano-cli transaction build-raw \
--tx-in 169978a6524bb01408d01b743e9df64d189ee6ba0080f780174b611e88d300af#1 \
--tx-out addr1q836p8u4dkgeq9m9l6xeayutlavg3d673t283259rrrktrpqj274ddmph7trcdxp5wj0ucfvga2gtwuuz7q825k6ucesc66u8m+0 \
--tx-out $(cat /data/ada/keys/paymentwtihstake.addr)+0 \
--invalid-hereafter 0 \
--fee 0 \
--out-file tx.draft
```

```
//计算交易费用
cardano-cli transaction calculate-min-fee \
--tx-body-file tx.draft \
--tx-in-count 1 \
--tx-out-count 2 \
--witness-count 1 \
--byron-witness-count 0 \
--mainnet \
--protocol-params-file protocol.json
```

```
//计算余额
expr <UTXO BALANCE> - <AMOUNT TO SEND> - <TRANSACTION FEE>
```

```
//获取slot
cardano-cli query tip --mainnet
```

```
//--invalid-hereafter 填上条命令获取的slot加200
cardano-cli transaction build-raw \
--tx-in 169978a6524bb01408d01b743e9df64d189ee6ba0080f780174b611e88d300af#1 \
--tx-out addr1q836p8u4dkgeq9m9l6xeayutlavg3d673t283259rrrktrpqj274ddmph7trcdxp5wj0ucfvga2gtwuuz7q825k6ucesc66u8m+440000000 \
--tx-out $(cat /data/ada/keys/paymentwtihstake.addr)+6864058 \
--invalid-hereafter 53909058 \
--fee 176589 \
--out-file tx.raw
```

```
//签名
cardano-cli transaction sign \
--tx-body-file tx.raw \
--signing-key-file /data/ada/keys/payment.skey \
--mainnet \
--out-file tx.signed
```

```
//广播
cardano-cli transaction submit \
--tx-file tx.signed \
--mainnet
```



### 领取收益

```
cardano-cli query stake-address-info \
> --mainnet \
> --address $(cat /data/ada/keys/stake.addr)
```

```
cardano-cli query utxo \
>     --address $(cat /data/ada/keys/paymentwtihstake.addr) \
>     --mainnet
```

```
cardano-cli transaction build-raw \
--tx-in a18d290b6077602b4e6b1f6ac8466dbf7c9c5d52147c55ed1a7d80810da7e0ee#0 \
--tx-out $(cat /data/ada/keys/paymentwtihstake.addr)+0 \
--withdrawal $(cat /data/ada/keys/stake.addr)+0 \
--invalid-hereafter 0 \
--fee 0 \
--out-file withdraw_rewards.draft
```

```
cardano-cli transaction calculate-min-fee \
--tx-body-file withdraw_rewards.draft  \
--tx-in-count 1 \
--tx-out-count 1 \
--witness-count 2 \
--byron-witness-count 0 \
--mainnet \
--protocol-params-file protocol.json
```

```
expr 194054070 - 171089 + 550000000
```

```
cardano-cli transaction build-raw \
--tx-in a18d290b6077602b4e6b1f6ac8466dbf7c9c5d52147c55ed1a7d80810da7e0ee#0 \
--tx-out $(cat /data/ada/keys/paymentwtihstake.addr)+454217236 \
--withdrawal $(cat /data/ada/keys/stake.addr)+447145853 \
--invalid-hereafter 53909058 \
--fee 178613 \
--out-file withdraw_rewards.raw
```

```
cardano-cli transaction sign \
--tx-body-file withdraw_rewards.raw  \
--signing-key-file /data/ada/keys/payment.skey \
--signing-key-file /data/ada/keys/stake.skey \
--mainnet \
--out-file withdraw_rewards.signed
```

```
cardano-cli transaction submit \
--tx-file withdraw_rewards.signed \
--mainnet
```







### 中继节点拓扑配置

```
  
  ,
    {
      "addr": "178.128.62.251",
      "port": 6000,
      "valency": 2
    },
    {
      "addr": "178.62.4.233",
      "port": 3001,
      "valency": 2
    },
    {
      "addr": "185.244.192.94",
      "port": 3001,
      "valency": 2
    },
    {
      "addr": "207.244.230.71",
      "port": 3001,
      "valency": 2
    },
    {
      "addr": "82.95.66.203",
      "port": 3001,
      "valency": 2
    },
    {
      "addr": "104.248.9.122",
      "port": 6000,
      "valency": 2
    },
    {
      "addr": "118.27.31.4",
      "port": 6000,
      "valency": 2
    },
    {
      "addr": "87.107.166.86",
      "port": 3001,
      "valency": 2
    },
    {
      "addr": "54.221.127.96",
      "port": 3001,
      "valency": 2
    },
    {
      "addr": "142.93.233.165",
      "port": 3001,
      "valency": 2
    },
    {
      "addr": "50.116.38.76",
      "port": 6000,
      "valency": 2
    },
    {
      "addr": "89.40.72.100",
      "port": 6000,
      "valency": 2
    },
    {
      "addr": "5.89.17.242",
      "port": 6000,
      "valency": 2
    },
    {
      "addr": "20.203.153.163",
      "port": 6000,
      "valency": 2
    },
    {
      "addr": "66.94.100.70",
      "port": 6000,
      "valency": 2
    },
    {
      "addr": "131.153.199.82",
      "port": 6000,
      "valency": 2
    },
    {
      "addr": "54.232.189.129",
      "port": 6000,
      "valency": 2
    },
    {
      "addr": "3.17.19.242",
      "port": 3000,
      "valency": 2
    },
    {
      "addr": "134.209.209.178",
      "port": 3146,
      "valency": 2
    }


```



### 总结

搭建节点还好，但是矿池注册等操作真的是一点不友好。



### 参考文档

- 官网: https://cardano.org/
- 白皮书: https://why.cardano.org/zh-cn/introduction/motivation/
- 文档: https://docs.cardano.org/ |https://developers.cardano.org/docs/get-started/
- 钱包：https://yoroi-wallet.com/#/｜https://adalite.io/
- 浏览器: https://cardanoscan.io/
- chrome钱包插件: https://chrome.google.com/webstore/detail/yoroi/ffnbelfdoeiohenkjibnmadjiehjhajb
- github https://github.com/input-output-hk/cardano-node
- Telegram: [*t.me/CardanoDevelopers*](https://t.me/CardanoDevelopersOfficial)
- 博客: https://iohk.io/en/blog/posts/page-1/
