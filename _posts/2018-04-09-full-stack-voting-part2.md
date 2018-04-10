---
layout: post
title: Full Stack Hello World Voting Ethereum Dapp Tutorial — Part 2
categories: 区块链
description: Full Stack Hello World Voting Ethereum Dapp Tutorial — Part 2
keywords: Ethereum, 区块链
---

原文地址： [Full Stack Hello World Voting Ethereum Dapp Tutorial, Mahesh Murthy](https://medium.com/@mvmurthy/full-stack-hello-world-voting-ethereum-dapp-tutorial-part-2-30b3d335aa1f)

在[Part 1](http://geeksai.com/2018/04/08/full-stack-voting-dapp/)中，我在ganache blockchain的开发环境上创建了一个简单的投票Dapp，本节我将把Dapp部署到真正的blockchain上。Ethereum有多个公共的test blockchain，一条main blockchain。

* Test Blockchain
  有很多测试，开发用的以太坊区块链可供我们选择，例如Ropsten, Rinkeby, Kovan。可以理解为测试网络，网络中使用的Ether都是假的。

* Main Blockchain
  主网络只有一个，抄币时交易的Ether，钱包应用中的Ether都使用的主网。

#### 本篇文章覆盖的内容有：

* 安装geth ，一个客户端应用，用户同步区块链数据，并且可以在本地运行Ethereum节点。

* 安装Ethereum Dapp开发框架Truffle，方面我们进行合约的开发，编译和打包。

* 优化Voting合约。

* 在Rinkeby测试网络中编译和部署Voting合约。

* 使用truffle console调用合约，并在网页中使用。


##### 1. 安装geth和同步blockchain

```
brew tap ethereum/ethereum
brew install ethereum
```

如果你使用Windows或者Ubuntu，可以在[这里]([https://github.com/ethereum/go-ethereum/wiki/Building-Ethereum](https://github.com/ethereum/go-ethereum/wiki/Building-Ethereum)
)查阅安装方法。

成功安装geth后，运行下面的代码：

```
geth --rinkeby --syncmode "fast" --rpc --rpcapi db,eth,net,web3,personal --cache=1024  --rpcport 8545 --rpcaddr 127.0.0.1 --rpccorsdomain "*"
```
  
执行后，geth会连接测试网络中的节点，并同步blockchain信息到本地，整个过程会持续一段时间，大概40分钟左右。

在console里你可以看到下面的信息，即同步blockchain的进度：

![屏幕快照 2018-04-09 下午8.00.26.png](https://upload-images.jianshu.io/upload_images/1224641-3d3b1946cfa31748.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 2. 安装Truffle Framework

使用npm安装：

```
npm install -g truffle
```

##### 3. 使用Truffle创建Voting的contract

第一步，使用truffle的命令，创建一个Dapp工程：

```
mkdir voting
cd voting
npm install -g webpack
truffle unbox webpack

> ls
README.md               contracts               node_modules            test                    webpack.config.js       truffle.js
app                     migrations              package.json  

> ls app/
index.html  javascripts  stylesheets

> ls contracts/
ConvertLib.sol  MetaCoin.sol  Migrations.sol

> ls migrations/
1_initial_migration.js  2_deploy_contracts.js
```

Truffle为我们创建好了必要的路径和文件，同时初始化了一个默认的Dapp。这里我们不需要这个初始化工程，可以把`ConvertLib.sol`和`MetaCoin.sol`删掉。

这里有个`migrations`的路径，它的作用可以和数据库表结构的更新结合理解，migration的过程就是将最新的合约内容部署到区块链中。



```
pragma solidity ^0.4.18;
// 指定solidity版本

contract Voting {
  // mapping 方法生成一个字典，key为bytes32类型，value为uint8类型
  mapping (bytes32 => uint8) public votesReceived;
  
  
  bytes32[] public candidateList;
  
  // 构造函数，初始化时传入候选人名单
  function Voting(bytes32[] candidateNames) public {

  	candidateList = candidateNames; 
  }
  
  // 传入候选人姓名，返回得票数 
  function totalVotesFor(bytes32 candidate) view public returns (uint8) {
  	require(validCandidate(candidate));
  	return votesReceived[candidate];
  }
  
  // 给候选人投票
  function voteForCandidate(bytes32 candidate) public {
  	require(validCandidate(candidate));
  	votesReceived[candidate] += 1;
  }
  
  // 验证传入的候选人是否有效
  function validCandidate(bytes32 candidate) view public returns (bool) {
  	for(uint i = 0; i < candidateList.length; i++) {
  	  if (candidateList[i] == candidate) {
  	  	return true;
  	  }
  	}
  	return false;
  }
}
```
在`contracts`路径下，新建一个文件`Voting.sol`，并将上面的合约代码复制进去（并没有改动）。

```
> ls contracts/
Migrations.sol  Voting.sol
```

接下来将下面的代码复制到`2_deploy_contracts.js`文件中：

```
var Voting = artifacts.require("./Voting.sol");

module.exports = function(deployer) {
  deployer.deploy(Voting, ['Sam', 'Leslie', 'Jetty', 'Arkila', 'Piu'], {gas: 6700000});
};

```

你也可以在`truffle.js`文件中设置gas费用：

```
require('babel-register')
module.exports = {
  networks: {
    development: {
      host: 'localhost',
      port: 8545,
      network_id: '*',
      gas: 470000
    }
  }
}
```

把`app/javascripts/app.js`中的内容替换成如下：

```
// Import the page's CSS. Webpack will know what to do with it.
import "../stylesheets/app.css";

import { default as Web3} from 'web3';
import { default as contract } from 'truffle-contract'


import voting_artifacts from '../../build/contracts/Voting.json'

var Voting = contract(voting_artifacts);

let candidates = {"Sam": "candidate-1", "Leslie": "candidate-2", "Jetty": "candidate-3", "Arkila": "candidate-4", "Piu": "candidate-5"}

window.voteForCandidate = function(candidate) {
  let candidateName = $("#candidate").val();
  try {
    $("#msg").html("Vote has been submitted. The vote count will increment as soon as the vote is recorded on the blockchain. Please wait.")
    $("#candidate").val("");

    /* Voting.deployed() returns an instance of the contract. Every call
     * in Truffle returns a promise which is why we have used then()
     * everywhere we have a transaction call
     */
    Voting.deployed().then(function(contractInstance) {
      contractInstance.voteForCandidate(candidateName, {gas: 140000, from: web3.eth.accounts[0]}).then(function() {
        let div_id = candidates[candidateName];
        return contractInstance.totalVotesFor.call(candidateName).then(function(v) {
          $("#" + div_id).html(v.toString());
          $("#msg").html("");
        });
      });
    });
  } catch (err) {
    console.log(err);
  }
}

$( document ).ready(function() {
  if (typeof web3 !== 'undefined') {
    console.warn("Using web3 detected from external source like Metamask")
    // Use Mist/MetaMask's provider
    window.web3 = new Web3(web3.currentProvider);
  } else {
    console.warn("No web3 detected. Falling back to http://localhost:8545. You should remove this fallback when you deploy live, as it's inherently insecure. Consider switching to Metamask for development. More info here: http://truffleframework.com/tutorials/truffle-and-metamask");
    // fallback - use your fallback strategy (local node / hosted node + in-dapp id mgmt / fail)
    window.web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
  }

  Voting.setProvider(web3.currentProvider);
  let candidateNames = Object.keys(candidates);
  for (var i = 0; i < candidateNames.length; i++) {
    let name = candidateNames[i];
    Voting.deployed().then(function(contractInstance) {
      contractInstance.totalVotesFor.call(name).then(function(v) {
        $("#" + candidates[name]).html(v.toString());
      });
    })
  }
});
```

同时替换`app/index.html`里面的内容：

```

<!DOCTYPE html>
<html>
<head>
  <title>Hello World DApp</title>
  <link href='https://fonts.googleapis.com/css?family=Open+Sans:400,700' rel='stylesheet' type='text/css'>
  <link href='https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css' rel='stylesheet' type='text/css'>
</head>
<body class="container">
  <h1>A Simple Hello World Voting Application</h1>
  <div id="address"></div>
  <div class="table-responsive">
    <table class="table table-bordered">
      <thead>
        <tr>
          <th>Candidate</th>
          <th>Votes</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>Rama</td>
          <td id="candidate-1"></td>
        </tr>
        <tr>
          <td>Nick</td>
          <td id="candidate-2"></td>
        </tr>
        <tr>
          <td>Jose</td>
          <td id="candidate-3"></td>
        </tr>
      </tbody>
    </table>
    <div id="msg"></div>
  </div>
  <input type="text" id="candidate" />
  <a href="#" onclick="voteForCandidate()" class="btn btn-primary">Vote</a>
</body>
<script src="https://cdn.rawgit.com/ethereum/web3.js/develop/dist/web3.js"></script>
<script src="https://code.jquery.com/jquery-3.1.1.slim.min.js"></script>
<script src="app.js"></script>
</html>
```

##### 4. 部署到Rinkeby网络

在部署之前，你需要给账户里充电ether。从创建账户开始：

```
> ruffle console
truffle(development)> web3.personal.newAccount('verystrongpassword')
'0xb4831d00165b2faa495e209e58be1549ab94d27d'

truffle(development)> web3.eth.getBalance('0xb4831d00165b2faa495e209e58be1549ab94d27d')
{ [String: '0'] s: 1, e: 0, c: [ 0 ] }

truffle(development)> web3.personal.unlockAccount('0xb4831d00165b2faa495e209e58be1549ab94d27d', 'verystrongpassword', 15000)

// 记得把verystrongpassword替换成一个比较安全的密码 
```

在上一章节中，我在`node console`中初始化了一个web3 object。在`truffle console`里直接使用就可以了。我们现在有了一个钱包地址，但是balance是0。

你可以在Rinkeby网络中获取一些测试用的ether：[https://faucet.rinkeby.io/](https://faucet.rinkeby.io/)

按照网站里的步骤执行完成后，你可以在rinkeby.etherscan.io 中查看balance，如果这个时候你执行`web3.eth.getBalance`看到余额任然 为0，这表示`geth`没有同步完测试网络的所有区块。

本地同步区块完成后，你的账户上应该有了一定的ether，下面开始部署：

note: 在migrage之前，一定要unlock你的账户，不然会出现认证问题

```
➜  voting truffle migrate
Compiling ./contracts/Migrations.sol...
Compiling ./contracts/Voting.sol...
Writing artifacts to ./build/contracts

Using network 'development'.

Running migration: 1_initial_migration.js
  Deploying Migrations...
  ... 0x5687c59779ec6cf1432974327b6fd8cd31e4d1f038d6abc1e1452599b2f3c56c
  Migrations: 0x03b11bdc336a395899675a1545de58e8ccd8f6b5
Saving successful migration to network...
  ... 0x1f09b4895d20598450e030114830f742f6f013e35bde8cf8f179b2dbb240065c
Saving artifacts...
Running migration: 2_deploy_contracts.js
  Deploying Voting...
  ... 0x0c60551172065b2d92054845073a7c4c68d49234d0fa85e0c3aa4f3d6699307f
  Voting: 0x2bedf107cf41ae7fd526d884f3df69ebf5e7e2d2
Saving successful migration to network...
  ... 0x17141ba56a0d7c4d1f8768f8c760a6a40f8183c11722e1acff830741c881c4ad
Saving artifacts...
➜  voting 
```

migrate过程大概需要一分钟。

##### 5. 使用voting contract

```
➜  voting npm run dev

> truffle-init-webpack@0.0.2 dev /Users/sam/Documents/workspace/blockchain/voting
> webpack-dev-server

Project is running at http://localhost:8080/
webpack output is served from /
Hash: 91cb5e9cc32c8b8b3ee0
Version: webpack 2.7.0
Time: 2033ms
     Asset     Size  Chunks                    Chunk Names
    app.js  1.67 MB       0  [emitted]  [big]  main
index.html  1.24 kB          [emitted]         
...
webpack: Compiled successfully.
```

启动本地服务，在浏览器中打开localhost:8080，就可以开始投票了：

![屏幕快照 2018-04-09 下午10.51.53.png](https://upload-images.jianshu.io/upload_images/1224641-c74bf2608934d294.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果你能看到上面的界面，说明你已经成功在公共测试网络上完成了一个Ethereum application的开发和部署，恭喜你。。。

你可以登录https://rinkeby.etherscan.io/ ，使用账户地址查看所有的transaction。

![屏幕快照 2018-04-09 下午10.55.28.png](https://upload-images.jianshu.io/upload_images/1224641-30c933283fe942f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

本节项目代码：https://github.com/sam408130/ethereum_voting_dapp/tree/master/chapter2

