Bitcoin Core Example
=====================================

[![Build Status](https://travis-ci.org/bitcoin/bitcoin.svg?branch=master)](https://travis-ci.org/bitcoin/bitcoin)

https://bitcoincore.org

## 参考资料  

https://legacy.gitbook.com/book/programmingblockchain/programmingblockchain/details

https://github.com/bitcoinbook/bitcoinbook

https://github.com/wbnns/bitcoin-documentation

开发者文档; https://bitcoin.org/en/developer-documentation   
中文文档: https://github.com/wbnns/bitcoincore.org  
维基百科: https://en.bitcoin.it/wiki/Main_Page  
https://en.bitcoin.it/wiki/Category:Technical  
https://en.bitcoin.it/wiki/Bitcoin_Core_0.11_(ch_1):_Overview
比特币工作原理: https://bitcoin.org/zh_CN/how-it-works  

https://github.com/Bitmessage/PyBitmessage

## unix 编译  

> https://www.jianshu.com/p/5d397bfc61d7   

```bash
sudo apt-get update
sudo apt-get install build-essential libtool autotools-dev autoconf automake libssl-dev libboost-all-dev libdb-dev libdb++-dev pkg-config libevent-dev git-core

./autogen.sh
./configure --without-gui --with-incompatible-bdb
make
```

make 可能时间比较久，用上面的配置大概花了差不多20分钟, 成功后在 src 目录下面生成三个可执行文件：  

    bitcoind
    bitcoin-tx
    bitcoin-cli

在 bitcoind 里面包括了 Bitcoin 所有的核心的功能，生成了这三个文件后就可以开始玩 Bitcoin 了。
我们可以通过 bitcoin-cli 或者 RPC 协议来调用 bitcoind 里面的功能。  

下面我来看如何生成有界面的 gui，即 Bitcoin-qt. 首先要安装依赖库：  

```bash
sudo apt-get install libqt4-dev pkg-config  libprotobuf-dev  protobuf-compiler

./autogen.sh
./configure -with-gui --with-incompatible-bdb
make
```

编译成功后会有 src/qt 目录下面生成 bitcoin-qt. 

至此，我们已经成功编译出了 Bitcoin 钱包了， 只要把里面的参数进行修改，我们完全可以发布一个新的币种了，这也是我后面要研究的。

## windows 编译  

参考文档: https://github.com/bitcoin/bitcoin/blob/master/doc/build-windows.md   

## 先启动 bitcoind 

```bash
./bitcoind  

# 运行 cli 命令: bitcoin-cli getblockheader

./bitcoin-cli getblockheader
```

    error code: -1
    error message:
    getblockheader "hash" ( verbose )

    If verbose is false, returns a string that is serialized, hex-encoded data for blockheader 'hash'.
    If verbose is true, returns an Object with information about blockheader <hash>.

    Arguments:
    1. "hash"          (string, required) The block hash
    2. verbose           (boolean, optional, default=true) true for a json object, false for the hex encoded data

    Result (for verbose = true):
    {
    "hash" : "hash",     (string) the block hash (same as provided)
    "confirmations" : n,   (numeric) The number of confirmations, or -1 if the block is not on the main chain
    "height" : n,          (numeric) The block height or index
    "version" : n,         (numeric) The block version
    "versionHex" : "00000000", (string) The block version formatted in hexadecimal
    "merkleroot" : "xxxx", (string) The merkle root
    "time" : ttt,          (numeric) The block time in seconds since epoch (Jan 1 1970 GMT)
    "mediantime" : ttt,    (numeric) The median block time in seconds since epoch (Jan 1 1970 GMT)
    "nonce" : n,           (numeric) The nonce
    "bits" : "1d00ffff", (string) The bits
    "difficulty" : x.xxx,  (numeric) The difficulty
    "chainwork" : "0000...1f3"     (string) Expected number of hashes required to produce the current chain (in hex)
    "nTx" : n,             (numeric) The number of transactions in the block.
    "previousblockhash" : "hash",  (string) The hash of the previous block
    "nextblockhash" : "hash",      (string) The hash of the next block
    }

    Result (for verbose=false):
    "data"             (string) A string that is serialized, hex-encoded data for block 'hash'.

    Examples:
    > bitcoin-cli getblockheader "00000000c937983704a73af28acdec37b049d214adbda81d7e2a3dd146f6ed09"
    > curl --user myusername --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "getblockheader", "params": ["00000000c937983704a73af28acdec37b049d214adbda81d7e2a3dd146f6ed09"] }' -H 'content-type: text/plain;' http://127.0.0.1:8332/

它会提示参数错误, 并提供一个 Example, 我们尝试运行:  

```
./bitcoin-cli getblockheader "00000000c937983704a73af28acdec37b049d214adbda81d7e2a3dd146f6ed09"
```
    {
    "hash": "00000000c937983704a73af28acdec37b049d214adbda81d7e2a3dd146f6ed09",
    "confirmations": 276769,
    "height": 1000,
    "version": 1,
    "versionHex": "00000001",
    "merkleroot": "fe28050b93faea61fa88c4c630f0e1f0a1c24d0082dd0e10d369e13212128f33",
    "time": 1232346882,
    "mediantime": 1232344831,
    "nonce": 2595206198,
    "bits": "1d00ffff",
    "difficulty": 1,
    "chainwork": "000000000000000000000000000000000000000000000000000003e903e903e9",
    "nTx": 1,
    "previousblockhash": "0000000008e647742775a230787d66fdf92c46a48c896bfbc85cdc8acc67e87d",
    "nextblockhash": "00000000a2887344f8db859e372e7e4bc26b23b9de340f725afbf2edb265b4c6"
    }

可以看到区块的头信息获取正确.   

## bitcoind 的入口  

src/bitcoind.cpp 文件中的 `int main(int argc, char* argv[])` 函数.  

`static bool AppInit(int argc, char* argv[]);` 函数;   

bitcoind 服务启动的输出信息:   

```
2018-09-26T02:17:28Z Bitcoin Core version v0.17.99.0-4dac24d-dirty (release build)
2018-09-26T02:17:28Z InitParameterInteraction: parameter interaction: -whitelistforcerelay=1 -> setting -whitelistrelay=1
2018-09-26T02:17:28Z Assuming ancestors of block 0000000000000000002e63058c023a9a1de233554f28c7b21380b6c9003f36a8 have valid signatures.
2018-09-26T02:17:28Z Setting nMinimumChainWork=0000000000000000000000000000000000000000028822fef1c230963535a90d
2018-09-26T02:17:28Z Using the 'sse4(1way),sse41(4way)' SHA256 implementation
2018-09-26T02:17:28Z Using RdRand as an additional entropy source
2018-09-26T02:17:28Z Default data directory /home/klm/.bitcoin
2018-09-26T02:17:28Z Using data directory /home/klm/.bitcoin
2018-09-26T02:17:28Z Config file: /home/klm/.bitcoin/bitcoin.conf (not found, skipping)
2018-09-26T02:17:28Z Using at most 125 automatic connections (1024 file descriptors available)
2018-09-26T02:17:28Z Using 16 MiB out of 32/2 requested for signature cache, able to store 524288 elements
2018-09-26T02:17:28Z Using 16 MiB out of 32/2 requested for script execution cache, able to store 524288 elements
2018-09-26T02:17:28Z Using 4 threads for script verification
2018-09-26T02:17:28Z scheduler thread start
2018-09-26T02:17:28Z HTTP: creating work queue of depth 16
2018-09-26T02:17:28Z No rpcpassword set - using random cookie authentication.
2018-09-26T02:17:28Z Generated RPC authentication cookie /home/klm/.bitcoin/.cookie
2018-09-26T02:17:28Z HTTP: starting 4 worker threads
2018-09-26T02:17:28Z Using wallet directory /home/klm/.bitcoin/wallets
2018-09-26T02:17:28Z init message: Verifying wallet(s)...
2018-09-26T02:17:28Z Using BerkeleyDB version Berkeley DB 5.3.28: (September  9, 2013)
2018-09-26T02:17:28Z Using wallet wallet.dat
2018-09-26T02:17:28Z BerkeleyEnvironment::Open: LogDir=/home/klm/.bitcoin/wallets/database ErrorFile=/home/klm/.bitcoin/wallets/db.log
2018-09-26T02:17:28Z Cache configuration:
2018-09-26T02:17:28Z * Using 2.0MiB for block index database
2018-09-26T02:17:28Z * Using 8.0MiB for chain state database
2018-09-26T02:17:28Z * Using 440.0MiB for in-memory UTXO set (plus up to 286.1MiB of unused mempool space)
2018-09-26T02:17:28Z init message: Loading block index...
2018-09-26T02:17:28Z Opening LevelDB in /home/klm/.bitcoin/blocks/index
2018-09-26T02:17:28Z Opened LevelDB successfully
2018-09-26T02:17:28Z Using obfuscation key for /home/klm/.bitcoin/blocks/index: 0000000000000000
2018-09-26T02:17:28Z LoadBlockIndexDB: last block file = 0
2018-09-26T02:17:28Z LoadBlockIndexDB: last block file info: CBlockFileInfo(blocks=0, size=0, heights=0...0, time=1970-01-01...1970-01-01)
2018-09-26T02:17:28Z Checking all blk files are present...
2018-09-26T02:17:28Z Initializing databases...
2018-09-26T02:17:28Z Pre-allocating up to position 0x1000000 in blk00000.dat
2018-09-26T02:17:28Z Opening LevelDB in /home/klm/.bitcoin/chainstate
2018-09-26T02:17:28Z Opened LevelDB successfully
2018-09-26T02:17:28Z Wrote new obfuscate key for /home/klm/.bitcoin/chainstate: c546ca0bdbdaad84
2018-09-26T02:17:28Z Using obfuscation key for /home/klm/.bitcoin/chainstate: c546ca0bdbdaad84
2018-09-26T02:17:28Z init message: Rewinding blocks...
2018-09-26T02:17:28Z  block index              26ms
2018-09-26T02:17:28Z init message: Loading wallet...
2018-09-26T02:17:28Z [default wallet] nFileVersion = 179900
2018-09-26T02:17:28Z [default wallet] Keys: 0 plaintext, 0 encrypted, 0 w/ metadata, 0 total. Unknown wallet records: 0
2018-09-26T02:17:28Z [default wallet] Performing wallet upgrade to 169900
2018-09-26T02:17:30Z [default wallet] keypool added 2000 keys (1000 internal), size=2000 (1000 internal)
2018-09-26T02:17:30Z [default wallet] Wallet completed loading in            1186ms
2018-09-26T02:17:30Z [default wallet] setKeyPool.size() = 2000
2018-09-26T02:17:30Z [default wallet] mapWallet.size() = 0
2018-09-26T02:17:30Z [default wallet] mapAddressBook.size() = 0
2018-09-26T02:17:30Z UpdateTip: new best=000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f height=0 version=0x00000001 log2_work=32.000022 tx=1 date='2009-01-03T18:15:05Z' progress=0.000000 cache=0.0MiB(0txo)
2018-09-26T02:17:30Z Failed to open mempool file from disk. Continuing anyway.
2018-09-26T02:17:30Z mapBlockIndex.size() = 1
2018-09-26T02:17:30Z nBestHeight = 0
2018-09-26T02:17:30Z torcontrol thread start
2018-09-26T02:17:30Z Bound to [::]:8333
2018-09-26T02:17:30Z Bound to 0.0.0.0:8333
2018-09-26T02:17:30Z init message: Loading P2P addresses...
2018-09-26T02:17:30Z ERROR: DeserializeFileDB: Failed to open file /home/klm/.bitcoin/peers.dat
2018-09-26T02:17:30Z Invalid or missing peers.dat; recreating
2018-09-26T02:17:30Z init message: Loading banlist...
2018-09-26T02:17:30Z ERROR: DeserializeFileDB: Failed to open file /home/klm/.bitcoin/banlist.dat
2018-09-26T02:17:30Z Invalid or missing banlist.dat; recreating
2018-09-26T02:17:30Z init message: Starting network threads...
2018-09-26T02:17:30Z net thread start
2018-09-26T02:17:30Z addcon thread start
2018-09-26T02:17:30Z dnsseed thread start
2018-09-26T02:17:30Z init message: Done loading
2018-09-26T02:17:30Z opencon thread start
2018-09-26T02:17:30Z Loading addresses from DNS seeds (could take a while)
2018-09-26T02:17:30Z msghand thread start
2018-09-26T02:17:30Z New outbound peer connected: version: 70015, blocks=543095, peer=0
2018-09-26T02:17:33Z New outbound peer connected: version: 70015, blocks=543095, peer=1
2018-09-26T02:17:34Z 254 addresses found from DNS seeds
2018-09-26T02:17:34Z dnsseed thread exit
2018-09-26T02:17:37Z New outbound peer connected: version: 70015, blocks=543095, peer=2
2018-09-26T02:17:38Z New outbound peer connected: version: 70015, blocks=543095, peer=3
2018-09-26T02:17:43Z New outbound peer connected: version: 70015, blocks=543095, peer=4
2018-09-26T02:17:43Z New outbound peer connected: version: 70015, blocks=543095, peer=5
2018-09-26T02:17:44Z New outbound peer connected: version: 70015, blocks=543095, peer=6
2018-09-26T02:17:49Z New outbound peer connected: version: 70015, blocks=543095, peer=7
2018-09-26T02:17:57Z Pre-allocating up to position 0x100000 in rev00000.dat
2018-09-26T02:17:57Z UpdateTip: new best=00000000839a8e6886ab5951d76f411475428afc90947ee320161bbf18eb6048 height=1 version=0x00000001 log2_work=33.000022 tx=2 date='2009-01-09T02:54:25Z' progress=0.000000 cache=0.0MiB(1txo)
2018-09-26T02:17:57Z UpdateTip: new best=000000006a625f06636b8bb6ac7b960a8d03705d1ace08b1a19da3fdcc99ddbd height=2 version=0x00000001 log2_work=33.584985 tx=3 date='2009-01-09T02:55:44Z' progress=0.000000 cache=0.0MiB(2txo)
2018-09-26T02:17:57Z UpdateTip: new best=0000000082b5015589a3fdf2d4baff403e6f0be035a5d9742c1cae6295464449 height=3 version=0x00000001 log2_work=34.000022 tx=4 date='2009-01-09T03:02:53Z' progress=0.000000 cache=0.0MiB(3txo)
2018-09-26T02:17:57Z UpdateTip: new best=000000004ebadb55ee9096c9a2f8880e09da59c0d68b1c228da88e48844a1485 height=4 version=0x00000001 log2_work=34.32195 tx=5 date='2009-01-09T03:16:28Z' progress=0.000000 cache=0.0MiB(4txo
012-10-30T05:13:37Z' progress=0.024554 cache=405.1MiB(3014399txo)
2018-09-26T02:33:05Z UpdateTip: new best=000000000000005dbf9da64b7584b9504f8d2912005ff1efc74e8e3542103818 height=205661 version=0x00000002 log2_work=68.940971 tx=8431838 date='2012-10-30T05:20:07Z' progress=0.024555 cache=405.1MiB(3014488txo)
2018-09-26T02:33:05Z UpdateTip: new best=00000000000000b0f50275dceb7041dd035b927c0f8684e55e43df698fc17504 height=205662 version=0x00000001 log2_work=68.941007 tx=8431884 date='2012-10-30T05:24:13Z' progress=0.024555 cache=405.1MiB(3014515txo)

> Ctrl+C 中断   

^C2018-09-26T02:59:59Z tor: Thread interrupt
2018-09-26T02:59:59Z Shutdown: In progress...
2018-09-26T02:59:59Z torcontrol thread exit
2018-09-26T02:59:59Z addcon thread exit
2018-09-26T02:59:59Z opencon thread exit
2018-09-26T02:59:59Z net thread exit
2018-09-26T02:59:59Z UpdateTip: new best=00000000000000023f8a0ababb74343ea64ec0ba317bc3697ee541084fa7fa83 height=274296 version=0x00000002 log2_work=74.685011 tx=29010237 date='2013-12-11T08:09:25Z' progress=0.084481 cache=145.3MiB(636883txo)
2018-09-26T02:59:59Z msghand thread exit
2018-09-26T02:59:59Z scheduler thread interrupt
2018-09-26T02:59:59Z Dumped mempool: 4e-06s to copy, 0.003224s to dump
2018-09-26T03:00:01Z [default wallet] Releasing wallet
2018-09-26T03:00:01Z Shutdown: done
```
