本篇教程讲述了怎样自己动手搭建一个EOSIO 的Testnet，采用Docker进行部署，直接将[../docker-compose.yml](https://github.com/winlin/eostoolkit/blob/master/docker-compose.yml)进行改动下即可。
注意: 建议使用 winlin/eos:VERSION的镜像，里面默认已经安装 vim、net-tools工具  

### 1.创建一个新目录用来启动Testnet的起始节点
```
mkdir firstnode
```
### 2.启动该节点
```
cd firstnode
sudo docker-compose up -d
```
### 3.进入nodeosd Docker 并进行相应初始化
```
sudo docker-compose exec nodeosd bash
alias cleos='cleos -u http://localhost:$NODSYSPORT --wallet-url http://localhost:$WALLETPORT'
```
*也可以将 cleos 的 alias 添加到 /root/.bashrc 中，避免每次进入docker bash都要执行*
*NODSYSPORT、WALLETPORT 是docker-compose.yml中声明的环境变量*

### 4.创建钱包并导入默认创建的eosio账户的 producer key (config.ini文件中)
```
cleos wallet create
cleos wallet import xxxxxxxxxx
```
#### 5.加载系统相应合约
```
cleos set contract eosio /opt/eosio/bin/data-dir/contracts/eosio.bios -p eosio
```
#### 6.创建 eosio.token、eosio.msig 账号
```
cleos create key && cleos create key
cleos wallet import OWNER_PRIKEY
cleos wallet import ACTIVE_PRIKEY
cleos create account eosio eosio.token OWNER_PUBKEY ACTIVE_PUBKEY
cleos create account eosio eosio.msig OWNER_PUBKEY ACTIVE_PUBKEY
```
### 7.部署 eosio.token、eosio.msig 合约
```
cleos set contract eosio.token /opt/eosio/bin/data-dir/contracts/eosio.token
cleos set contract eosio.msig /opt/eosio/bin/data-dir/contracts/eosio.msig
```
### 8.创建EOS Token 并且发放到 eosio 账号
```
cleos push action eosio.token create '{"issuer":"eosio", "maximum_supply":"1000000000.0000 EOS", , "can_freeze": 0, "can_recall": 0, "can_whitelist": 0}' -p eosio.token
cleos push action eosio.token issue '{"to":"eosio","quantity":"1000000000.0000 EOS","memo":"issue"}' -p eosio
#查看账户余额
cleos get currency balance eosio.token eosio
```
### 9.部署 eosio.system 合约
```
cleos set contract eosio /opt/eosio/bin/data-dir/contracts/eosio.system
```
### 10.eosio.msig 账户设置成特权账户
```
cleos push action eosio setpriv '{"account": "eosio.msig", "is_priv": 1}' -p eosio
```

### 10.创建其他BP账号,账号名称要求12个字节
```
cleos system newaccount --stake-net "1000000.0000 EOS" --stake-cpu "1000000.0000 EOS" --buy-ram 102400  eosio bp1111111111 OWNER_PUBKEY -p eosio
cleos system newaccount --stake-net "1000000.0000 EOS" --stake-cpu "1000000.0000 EOS" --buy-ram 102400  eosio bp2222222222 OWNER_PUBKEY -p eosio
```
### 11.将其他BP账号注册成块生产者
```
cleos system regproducer bp1111111111 OWNER_PUBKEY http://xxx.xxx
cleos system regproducer bp2222222222 OWNER_PUBKEY http://xxx.xxx
```
### 12.创建投票账户
```
cleos system newaccount --stake-net "10.0000 EOS" --stake-cpu "10.0000 EOS" --buy-ram "0.200 EOS"  eosio voter1111111 OWNER_PUBKEY -p eosio
cleos system newaccount --stake-net "10.0000 EOS" --stake-cpu "10.0000 EOS" --buy-ram "0.200 EOS"  eosio voter2222222 OWNER_PUBKEY -p eosio
cleos system newaccount --stake-net "10.0000 EOS" --stake-cpu "10.0000 EOS" --buy-ram "0.200 EOS"  eosio voter3333333 OWNER_PUBKEY -p eosio
cleos system newaccount --stake-net "10.0000 EOS" --stake-cpu "10.0000 EOS" --buy-ram "0.200 EOS"  eosio voter4444444 OWNER_PUBKEY -p eosio
```
### 13.向投票账户发放共超过3亿的EOS，要让其他BP出块就需要一共有超过1.5亿的EOS被质押(cpu/ram 单个超过1.5亿)
```
cleos push action eosio.token transfer '{"from":"eosio","to":"voter1111111","quantity":"80000000.0000 EOS","memo":"transfer"}' -p eosio
cleos push action eosio.token transfer '{"from":"eosio","to":"voter2222222","quantity":"80000000.0000 EOS","memo":"transfer"}' -p eosio
cleos push action eosio.token transfer '{"from":"eosio","to":"voter3333333","quantity":"100000000.0000 EOS","memo":"transfer"}' -p eosio
cleos push action eosio.token transfer '{"from":"eosio","to":"voter4444444","quantity":"100000000.0000 EOS","memo":"transfer"}' -p eosio
#查看账户余额
cleos get currency balance eosio.token voter1111111
cleos get currency balance eosio.token voter2222222
```
### 14.投票账户进行托管并投票
```
#托管
cleos system delegatebw voter1111111 voter1111111 "40000000.0000 EOS"  "40000000.0000 EOS" --transfer
cleos system delegatebw voter2222222 voter2222222 "40000000.0000 EOS"  "40000000.0000 EOS" --transfer
cleos system delegatebw voter3333333 voter3333333 "50000000.0000 EOS"  "50000000.0000 EOS" --transfer
cleos system delegatebw voter4444444 voter4444444 "50000000.0000 EOS"  "50000000.0000 EOS" --transfer
#投票
cleos system voteproducer prods voter1111111 bp1111111111 bp2222222222
cleos system voteproducer prods voter2222222 bp1111111111 bp2222222222
cleos system voteproducer prods voter3333333 bp1111111111 bp2222222222
cleos system voteproducer prods voter4444444 bp1111111111 bp2222222222
#查看投票账户信息
cleos get table eosio eosio voters
#查看连上区块生产者
cleos system listproducers
```
### 15.创建secondnode、thirdnode 目录并初始化配置文件
```
mkdir secondnode thirdnode
cp firstnode/config.ini firstnode/docker-compose.yml fisetnode/genesis.json secondname/
cp firstnode/config.ini firstnode/docker-compose.yml fisetnode/genesis.json thirdnode/
```
### 16.修改secondnode、thirdnode配置文件
修改的部分如下：
* docker-compose.yml: 
    * 其中两个端口需要分配成未使用的
    * volumes 的本地路径要调整下
* config.ini:
    * p2p-listen-endpoint 
    * http-server-address 
    * private-key
    * producer-name
    * p2p-peer-address

做完上面操作之后 eosio 会停止出块，由新创建的BP进行出块，对应的示例配置文件可以在当前目录下查看.

也可以参考 CryptoKylin 的启动流程：
https://github.com/cryptokylin/CryptoKylin-Testnet/blob/master/boot.md

*本过程参考了Zhaoyu [<<register-producer-and-vote-dawn-4.0.md>>](https://gist.github.com/JohnnyZhao/147636a325118ccc51da48e9e8e68de7)以及[EOS.HOST](https://eos.host/)团队的提供的友情支持。*


