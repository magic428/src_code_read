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

