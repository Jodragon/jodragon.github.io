---
title: 以太坊上海升级-取款
date: 2023-02-21 20:46:22
tags:
- 以太坊
- 区块链
- 上海升级
categories:
- 区块链
---
### 相关文献：

https://zhejiang.launchpad.ethereum.org/en/withdrawals#excess-balance-withdrawals

https://docs.prylabs.network/docs/wallet/withdraw-validator

https://ethereum.org/zh/staking/withdrawals/#checking-an-account-for-withdrawals



### 撤回验证节点质押的ETH有两种方式：

1. **部分（收益）提取**：此选项可让您提取您的收益（即超过 32ETH 的部分）并继续验证。
2. **全额提取**：此选项可让您提取您的全部质押（32ETH）和收益，有效地清算您的验证节点并退出网络。

>  无法提取特定数量的 ETH
>
>  部分提取只能提取超过32ETH的部分，如果用户有slash，质押的ETH不足32，是无法提取的，但是可以全额提取。



### 注意事项

- **全额提取时验证者必须先退出验证才可以提款**：设置提款凭证或退出请求的顺序无所谓
- **提取不是立即完成的** :  全部或部分提款的处理速度为每个区块最多 16 个验证者。
- **全额提取是不可逆的**
- **部分提取不会退出您的验证器**
- **提款地址的设置是不可更改的**
- **slash或之前退出的验证者仍然可以提现**
- **智能合约地址可作为提现地址但不能触发功能**



### 提款需要做什么？

**如果生成质押文件deposit时填写了eth1的地址作为提款地址，用户不需要发起任何操作，等待网络升级后会自动把收益部分打入设置的eth1地址。就像爆块那样，共识层收益也会自动打到设置的地址里。**

**如果未设置eth1地址，那么eth网络会自动忽略你的验证节点，收益会存在您的验证节点账户。这种情况想要提款那就需要先设置eth1地址，之后会自动把收益部分打入设置的eth1地址，下面是操作步骤。**



### 一、设置eth1地址为提款地址

> 设置过提款地址的话是无法更改的，下面的流程只适用未设置提款地址的验证者。
>
> 此过程不消耗gas fee

##### 流程：

1、准备好助记词、提款地址和相关程序

2、离线生成bls_to_execution_changes-*.json签名文件

3、验证文件参数

4、在线将签名信息广播到ETH网络

5、查看是否设置成功



#### 准备

- **助记词**
- **可以访问的信标节点**
- **[安装 staking-deposit-cli 的稳定版本](https://github.com/ethereum/staking-deposit-cli/releases)**
- **[安装 prysmctl 的稳定版](https://github.com/prysmaticlabs/prysm/releases)**



#### 部分（收益）提取设置流程：

#### 1、签署请求设置您的以太坊取款地址（**BLS to Execution Change**）

```
python ./staking_deposit/deposit.py generate-bls-to-execution-change
```

##### 交互式过程:

1. 选择你的助记语言。
2. 执行此操作的**网络。**
3. 输入你的**助记词**
4. 将被要求提供您想要提取的取款密钥的索引。
5. 系统会询问您想要操作的验证器的**验证器** **索引。**
6. 您将被要求提供您的**提款凭证**
7. 您将被要求提供您希望用于接收提取资金的以太坊地址。**一旦上链，将无法更改**。

> 此过程涉及到助记词建议离线进行！！！
>
> 验证器索引和提款凭证可以在[Beaconcha.in](http://beaconcha.in/)上获取

```
//操作示例
python ./staking_deposit/deposit.py generate-bls-to-execution-change
Please choose your language ['1. العربية', '2. ελληνικά', '3. English', '4. Français', '5. Bahasa melayu', '6. Italiano', '7. 日本語', '8. 한국어', '9. Português do Brasil', '10. român', '11. Türkçe', '12. 简体中文']:  [English]: english

Please choose the (mainnet or testnet) network/chain name ['mainnet', 'goerli', 'sepolia', 'zhejiang']:  [mainnet]: zhejiang

Please enter your mnemonic separated by spaces (" "). Note: you only need to enter the first 4 letters of each word if you'd prefer.: 
bike shoe attitude violin fun life punch enhance attend bright voyage wheel clutch taxi high health siren jealous tell female upon firm manual wage

Please enter the index (key number) of the signing key you want to use with this mnemonic. [0]: 0

Please enter a list of the validator indices of your validator(s). Split multiple items with whitespaces or commas.: 8

Please enter a list of the old BLS withdrawal credentials of your validator(s). Split multiple items with whitespaces or commas.: 00a6bd30000296e9c9f5823b09e689ff0bc0b1bea1d256caab9a5f213a226b33

Please enter the 20-byte execution address for the new withdrawal credentials. Note that you CANNOT change it once you have set it on chain.: 0x9B984D5a03980D8dc0a24506c968465424c81DbE

**[Warning] you are setting an Eth1 address as your withdrawal address. Please ensure that you have control over this address.**

Repeat your execution address for confirmation.: 0x9B984D5a03980D8dc0a24506c968465424c81DbE

**[Warning] you are setting an Eth1 address as your withdrawal address. Please ensure that you have control over this address.**

Success!
Your SignedBLSToExecutionChange JSON file can be found at: /home/me/Desktop/code/python/staking-deposit-cli/bls_to_execution_changes
```

> 此过程会生成一个`bls_to_execution_changes-*.json`文件

##### 验证hash

```
//这个方法的结果是你的取款凭证，如果对得上证明参数是正确的
echo 0x$(echo -n '0xaf6b29b5c440170bc773b688119ff4b035466473096697dd7d3dce5e2bcdacf017d599b1dd23f5906dfb3e3c79207a45' | xxd -r -p | sha256sum | cut -d ' ' -f 1)	
```



#### 2、将您签名的请求提交到以太坊网络

此过程会使用到上一步生成的文件

```
prysmctl validator withdraw --beacon-node-host=<node-url> --path=<bls_to_execution_changes-*.json> 

prysmctl validator withdraw --beacon-node-host=localhost:3500 --path=/home/ubuntu/prysmctl/bls-change/bls_to_execution_change-1681812857.json --confirm --accept-terms-of-use

or

curl -X POST -H 'Content-type: application/json' -d @<@FILENAME DESTINATION> \
http://<BEACON_NODE_HTTP_API_URL>/eth/v1/beacon/pool/bls_to_execution_changes

curl -X POST -H 'Content-type: application/json' -d '@/home/ubuntu/withdraw-validator/staking_deposit-cli-d7b5304-linux-amd64/bls_to_execution_changes/bls_to_execution_change-1678889108.json' \
http://localhost:3500/eth/v1/beacon/pool/bls_to_execution_changes



./deposit --language=english generate-bls-to-execution-change \
--chain=goerli \
--mnemonic="razor ** skate punch poem *** pelican flat project popular ***** water ivory social beach **** photo random explain brown ahead tomato alien invest" \
--bls_withdrawal_credentials_list="0x00d621567c26382b332c9acce27baca16f99dda01149b15057cad86ba5738e9c" \
--validator_start_index=0 \
--validator_indices="393623" \
--execution_address="0x8Fba9B6F96f6faB0bE13A21CDc426D1531d126E7"
```



#### 3、确认提款地址是否设置成功

[像Beaconcha.in](http://beaconcha.in/)就包含跟踪取款的功能，还可以用本地节点的rpc进行确认。

```rust
curl -X 'GET' \  'http://YOUR_PRYSM_NODE_HOST:3500/eth/v1/beacon/states/head/validators/YOUR_VALIDATOR_INDEX' \  -H 'accept: application/json'
```



#### 二、全额提取设置流程：

全额提取流程与部分提取流程一样，区别是验证器已退出则全部取款。



#### 三、退出验证器：

```
prysmctl validator exit --wallet-dir=<path/to/wallet> --beacon-rpc-provider=<127.0.0.1:4000> 
```

##### 退出验证需要的时间：

想要退出验证需要验证者至少活跃256个epoch约27小时才可以提交申请，

退出验证需要排队，取决于当前排队的验证者数量。

每个epoch可以退出的验证者数量有当前网络活跃的验证者数量决定，目前每个epoch可以退出7个验证者，当验证者数量达到 524,288 时，将增加到8 个。

目前以太坊上有52w+节点，每天最多可以退 1800 个节点。假设有 10% 的节点要退，排队大概在 33天。

在验证者自愿进入退出状态的情况下，资金将在 256 个纪元（约 27 小时）后可提取。如果验证者被削减，这个延迟会延长到 4 周（2048 纪元*4 或 ~36 天）




### 四、部分提取或全部提取（已退出验证）的支付时间

ETH会维护一个提款队列，所有符合条件的验证者都在里边，他会无限循环的处理每个验证者的部分提取或全部提取。一个区块最多可以处理 16 笔提款。按照这个速度，每天可以处理 115,200 个验证者提款（假设没有遗漏区块）。

扩展此计算，我们可以估算处理给定数量的提款所需的时间：

| 取款数量 | 完成时间 |
| :------: | :------: |
| 400,000  |  3.5天   |
| 500,000  |  4.3天   |
| 600,000  |  5.2天   |
| 700,000  |  6.1天   |
| 800,000  |  7.0天   |

>  假如有500,000验证者排队，每个账户大概每4.3天发放一次收益。

>  随着网络上有更多验证者，速度会变慢。错过区块的增加可能会按比例减慢速度。

##### 全部提取的时间=退出验证器的时间+支付时间




#### 其他工具ethdo：https://github.com/wealdtech/ethdo/blob/master/docs/changingwithdrawalcredentials.md
