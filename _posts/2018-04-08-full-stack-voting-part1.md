---
layout: post
title: Full Stack Hello World Voting Ethereum Dapp Tutorial — Part 1
categories: 区块链
description: Full Stack Hello World Voting Ethereum Dapp Tutorial — Part 1
keywords: Ethereum, 区块链
---

原文地址： [Full Stack Hello World Voting Ethereum Dapp Tutorial, Mahesh Murthy](https://medium.com/@mvmurthy/full-stack-hello-world-voting-ethereum-dapp-tutorial-part-1-40d2d0d807c2)

[上一篇文章](http://geeksai.com/2018/04/07/ethereum-for-web-developer/)中，整体介绍了Ethereum platform和web application的对比。作为程序员，最好的学习方法便是在新的框架下写一个Hello World工程，来熟悉整个框架的开发，测试，部署环境及方法。本篇文章将通过一个投票的应用，介绍基于web的Dapp开发流程。

这个应用很简单，会初始化几个候选人，所有人都可以给他们投票，并将每个人的总票数显示出来。在开发过程中，不会细说语法问题，只是初步熟悉一下Ethereum Dapp的逻辑编写，打包编译，及部署流程。

在目前阶段，我尽量不用成熟的框架，只用最原始的语言逻辑来呈现开发过程。在你使用框架之前，请尽可能深的解读框架内容。

## 目的

* 创建一个开发环境
* 了解编写一个contract，编译和部署在开发环境的流程
* 在node环境下使用部署好的contract与blockchain交互
* 基于部署好的contract，在网页中完成投票进度的展示和投票动作



投票应用的结构图如下： 

![voting app](https://upload-images.jianshu.io/upload_images/1224641-1d4d651427d3729e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1. 创建开发环境

开发环境下，我们使用ganache创建的虚拟链上开发和部署，下篇文章将介绍以太坊上真正的blockchain。下面是安装和使用方法：

```
npm install ganache-cli web3@0.20.2
node_modules/.bin/ganache-cli
```

![[图片上传中...(image-721dc3-1523189167231)]
](https://upload-images.jianshu.io/upload_images/1224641-a01d32b14bf823f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过上图可以看出，ganache-cli帮我们创建了10个账户，每个账户下都有100个eth

### 2. voting contract

我将使用Solidity语言编写contract，它是一个面向对象的编程语言，语法类似于javascript。我们将编写一个contract叫做Voting，它会包含一个构造函数，输入是候选人的数组。还有两个方法，一个是返回输入候选人的得票数，另一个是投票的动作。

注意：当你将代码部署到区块链上以后，构造函数会被执行，且只执行一次。和web应用不同的是，重新部署的代码会覆盖旧的的代码，在blockchain上的部署是不可逆的。例如你更新了contract，并重新部署，旧的contract任然会存在于blockchain中，和起绑定的数据将无法修改。新部署的contract会由构造函数生成新的实例。

代码如下：

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

将上面的代码复制后，保存到文件”Voting.sol“中，接下来我们将代码编译，并部署到ganache blockchain上。

为了编译上面的代码，需要先安装solc：

```
npm install solc
```

部署过程在node concole中进行，下篇文章会介绍使用truffle framework管理和部署代码。在此阶段，尽量去理解每一步执行的作用。上篇文章介绍到使用web3.js可以通过RPC跟blockchain进行交互，接下来将介绍如何使用web3.js实现部署和交互的过程。

首先切换到node环境，解释实例化solc和web3:

```
node
> Web3 = require('web3')
> web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
```

为保证通讯正常，web3 object可以正常跟blockchain通信，输入下面的指令查看blockchain上的所有账户信息，你会看到以下信息：

```
> web3.eth.accounts

['0x9c02f5c68e02390a3ab81f63341edc1ba5dbb39e',
'0x7d920be073e92a590dc47e4ccea2f28db3f218cc',
'0xf8a9c7c65c4d1c0c21b06c06ee5da80bd8f074a9',
'0x9d8ee8c3d4f8b1e08803da274bdaff80c2204fc6',
'0x26bb5d139aa7bdb1380af0e1e8f98147ef4c406a',
'0x622e557aad13c36459fac83240f25ae91882127c',
'0xbf8b1630d5640e272f33653e83092ce33d302fd2',
'0xe37a3157cb3081ea7a96ba9f9e942c72cf7ad87b',
'0x175dae81345f36775db285d368f0b1d49f61b2f8',
'0xc26bda5f3370bdd46e7c84bdb909aead4d8f35f3']
```

读取Voting.sol的内容，使用solc编译代码：

```
> code = fs.readFileSync('Voting.sol').toString()
> solc = require('solc')
> compiledCode = solc.compile(code)
```

编译成功后，将contract打印出来(在console中输入compiledCode)，有两个字段比较重要，需要重点理解：

* compileCode.contracts[':Voting'].bytecode
   这是我们将Voting.sol中的内容编译后的bytecode文本，也是将要部署到blockchain上的内容。

* compileCode.contracts[':Voting'].interface
   这里返回的是合约的接口说明abi，Application Binary Interface。它是合约的二进制接口，可以通俗的理解为合约的接口说明。当合约被编译后，那么它的api也就确定了。

以下代码完成了代码的部署，首先创建一个contract object(下面的VotingContract)，它将用于在blockchain上实例化合约和部署：

```
> abiDefinition = JSON.parse(compiledCode.contracts[':Voting'].interface)
> VotingContract = web3.eth.contract(abiDefinition)
> byteCode = compiledCode.contracts[':Voting'].bytecode
> deployedContract = VotingContract.new(['Rama','Nick','Jose'],{data: byteCode, from: web3.eth.accounts[0], gas: 4700000})
> deployedContract.address
> contractInstance = VotingContract.at(deployedContract.address)
```

`VotingCongtract.new` 方法实现了部署动作，第一个参数是候选人名单，来看一下其他参数：

* data
   data对应的内容是合约的二进制编码

*  from
    部署到区块链上时，需要指定是谁在部署，这里使用了我们ganache blockchain为我们创建的第一个地址，这里地址表示用户的身份，即hash后的公钥(详细信息请查阅钱包相关信息)。

* gas
 旷工费用，即在跟blockchain交互时(记录数据)，需要支付给旷工一定的费用来完成区块的创建。如果你想将合约部署到现实的以太坊区块链上时，你需要指定gas费用，这个费用将从你指定的账户里扣除。

现在我们已经将合约部署到了ganache blockchain上，并获取了合约的实例(contractInstance)。以太坊上部署了成千上万个合约，哪个才是你的合约呢？答案是`deployedContract.address`。 当你要通过合约执行操作时，你需要先获取合约的部署地址以及合约abi，然后实例化合约进行方法调用。

### 3. 在node console中使用contract

```
> contractInstance.totalVotesFor.call('Rama')
{ [String: '0'] s: 1, e: 0, c: [ 0 ] }
> contractInstance.voteForCandidate('Rama', {from: web3.eth.accounts[0]})
'0xdedc7ae544c3dde74ab5a0b07422c5a51b5240603d31074f5b75c0ebc786bf53'
> contractInstance.voteForCandidate('Rama', {from: web3.eth.accounts[0]})
'0x02c054d238038d68b65d55770fabfca592a5cf6590229ab91bbe7cd72da46de9'
> contractInstance.voteForCandidate('Rama', {from: web3.eth.accounts[0]})
'0x3da069a09577514f2baaa11bc3015a16edf26aad28dffbcd126bde2e71f2b76f'
> contractInstance.totalVotesFor.call('Rama').toLocaleString()
'3'
```

试着执行上面的代码，给Rama投票。每完成一次投票，都会返回一个地址信息，这个地址信息是blockchain中的transaction id，它证明了刚才的投票过程，你可以在任何事件，任何地点，根据这个id在以太坊中查新这次transaction信息。注意，这条transaction是不可修改的，这也是区块链技术的一个重要特点。

### 4. 在网页中进行投票

到此，part1的内容已经讲述过半，现在需要编写一个简单的web页面，使得在页面上进行投票和查看票数的操作。

index.html

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
  </div>
  <input type="text" id="candidate" />
  <a href="#" onclick="voteForCandidate()" class="btn btn-primary">Vote</a>
</body>
<script src="https://cdn.rawgit.com/ethereum/web3.js/develop/dist/web3.js"></script>
<script src="https://code.jquery.com/jquery-3.1.1.slim.min.js"></script>
<script src="./index.js"></script>
</html>
```

index.js

```
web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
abi = JSON.parse('[{"constant":false,"inputs":[{"name":"candidate","type":"bytes32"}],"name":"totalVotesFor","outputs":[{"name":"","type":"uint8"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"candidate","type":"bytes32"}],"name":"validCandidate","outputs":[{"name":"","type":"bool"}],"payable":false,"type":"function"},{"constant":true,"inputs":[{"name":"","type":"bytes32"}],"name":"votesReceived","outputs":[{"name":"","type":"uint8"}],"payable":false,"type":"function"},{"constant":true,"inputs":[{"name":"x","type":"bytes32"}],"name":"bytes32ToString","outputs":[{"name":"","type":"string"}],"payable":false,"type":"function"},{"constant":true,"inputs":[{"name":"","type":"uint256"}],"name":"candidateList","outputs":[{"name":"","type":"bytes32"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"candidate","type":"bytes32"}],"name":"voteForCandidate","outputs":[],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"contractOwner","outputs":[{"name":"","type":"address"}],"payable":false,"type":"function"},{"inputs":[{"name":"candidateNames","type":"bytes32[]"}],"payable":false,"type":"constructor"}]')
VotingContract = web3.eth.contract(abi);
// 请把这里的合约地址替换成你部署合约时所得到的地址信息
contractInstance = VotingContract.at('0x2a9c1d265d06d47e8f7b00ffa987c9185aecf672');
candidates = {"Rama": "candidate-1", "Nick": "candidate-2", "Jose": "candidate-3"}

function voteForCandidate() {
  candidateName = $("#candidate").val();
  contractInstance.voteForCandidate(candidateName, {from: web3.eth.accounts[0]}, function() {
    let div_id = candidates[candidateName];
    $("#" + div_id).html(contractInstance.totalVotesFor.call(candidateName).toString());
  });
}

$(document).ready(function() {
  candidateNames = Object.keys(candidates);
  for (var i = 0; i < candidateNames.length; i++) {
    let name = candidateNames[i];
    let val = contractInstance.totalVotesFor.call(name).toString()
    $("#" + candidates[name]).html(val);
  }
});
```

index.js中通过abi和合约地址实例化了一个`VotingContract`，变可以使用合约中的方法，打开`index.html`，你可以看到以下内容：

![image.png](https://upload-images.jianshu.io/upload_images/1224641-4137aba75a0bac94.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

去体验一下投票吧。。。

下一节，在part2中，我将把合约部署到以太坊公共的测试网络中，这样全世界的人在测试网络下变可以使用你的Voting Dapp。同时，下一节将使用[truffle framework](https://github.com/ConsenSys/truffle)进行合约的开发，不再使用node console一步步的执行，敬请期待。。。


本节项目代码：https://github.com/sam408130/ethereum_voting_dapp/tree/master/chapter1

