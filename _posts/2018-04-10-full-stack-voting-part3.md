---
layout: post
title: Full Stack Hello World Voting Ethereum Dapp Tutorial — Part 3
categories: 区块链
description: Full Stack Hello World Voting Ethereum Dapp Tutorial — Part 3
keywords: Ethereum, 区块链
---


在[Part 1](http://geeksai.com/2018/04/08/full-stack-voting-dapp/)中我们创建了一个简单的voting dapp，并在本地部署了该合约。在[Part 2](http://geeksai.com/2018/04/09/full-stack-voting-part2/)我们使用了Truffle Framework创建，编译，重新部署了投票应用，且将合约部署到了公共的Ethereum测试网络。本章节，我将丰富一下合约内容，以便覆盖更多的内容：

* 在合约中使用`struct`结构来组织数据，并将数据存储到区块链上

* 了解tokens，及使用

* 学习使用Ether支付

我们知道，在真正的票选中，每个有选举权的公民都有一票，且只有一票。在类似董事会投票的场景里，拥有股份越多的人，可投票数越多。所以，要想获得更多的投票数，就需要购买或持有更多的股份。

我将把之前的voting dapp调整为董事会投票的类型，这需要新增一个方法，即可以通过购买票数获取更多的投票权。另外，再增加一个可以查看投票人信息的方法。在Ethereum blockchain里，这里的票数通常被叫做tokens。

第一步，需要声明一个可以存储所需数据的变量，请看下面的代码：

```
// struct 结构
  struct voter {
    address voterAddress; // voter的身份地址
    uint tokensBought;    // 拥有的token数量
    uint[] tokensUsedPerCandidate; // 存储投票记录
  }

mapping (bytes32 => uint) public votesReceived;
mapping (address => voter) public voterInfo;

bytes32[] public candidateList;
uint public totalTokens; // 合约发布时规定的总token数量
uint public balanceTokens; // 可购买的token数量
uint public tokenPrice; // token的价格

```

因为我增加了`totalTokens，tokenPrice`这些字段，所以构造函数需要更改一下：

```
  function Voting(uint tokens, uint pricePerToken, bytes32[] candidateNames) public {
    candidateList = candidateNames;
    totalTokens = tokens;
    balanceTokens = tokens;
    tokenPrice = pricePerToken;
  }
```

接着修改`migrations`路径下的`1_deploy_contracts.js`文件：

```
deployer.deploy(Voting, 1000, web3.toWei('0.1', 'ether'), ['Sam', 'Leslie', 'Jetty', 'Arkila', 'Piu']);
```

到此，已经完成了初始化tokens，并设置了token的价格，下面看一下别人如何通过支付ether来购买tokens:

```
/* 购买tokens的方法，注意关键字‘payable’。该方法添加到合约中后，你的合约就可以接受别人支付ether来购买token了，这就是购买ERC2.0代币的过程*/

function buy() payable public returns (uint) {
    uint tokensToBuy = msg.value / tokenPrice;
    if (tokensToBuy > balanceTokens) throw;
    voterInfo[msg.sender].voterAddress = msg.sender;
    voterInfo[msg.sender].tokensBought += tokensToBuy;
    balanceTokens -= tokensToBuy;
    return tokensToBuy;
}
```

让我们来看一下voter和contract之间的交互流程：

![image.png](https://upload-images.jianshu.io/upload_images/1224641-4ddef27ab0b183e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Voting.sol

```
pragma solidity ^0.4.18; 

contract Voting {

  struct voter {
    address voterAddress; 
    uint tokensBought;    
    uint[] tokensUsedPerCandidate; 
  }



  mapping (address => voter) public voterInfo;

  mapping (bytes32 => uint) public votesReceived;

  bytes32[] public candidateList;

  uint public totalTokens; 
  uint public balanceTokens; 
  uint public tokenPrice; 


  function Voting(uint tokens, uint pricePerToken, bytes32[] candidateNames) public {
    candidateList = candidateNames;
    totalTokens = tokens;
    balanceTokens = tokens;
    tokenPrice = pricePerToken;
  }

  function totalVotesFor(bytes32 candidate) view public returns (uint) {
    return votesReceived[candidate];
  }

  function voteForCandidate(bytes32 candidate, uint votesInTokens) public {
    uint index = indexOfCandidate(candidate);
    require(index != uint(-1));

    if (voterInfo[msg.sender].tokensUsedPerCandidate.length == 0) {
      for(uint i = 0; i < candidateList.length; i++) {
        voterInfo[msg.sender].tokensUsedPerCandidate.push(0);
      }
    }

    // Make sure this voter has enough tokens to cast the vote
    uint availableTokens = voterInfo[msg.sender].tokensBought - totalTokensUsed(voterInfo[msg.sender].tokensUsedPerCandidate);
    require(availableTokens >= votesInTokens);

    votesReceived[candidate] += votesInTokens;

    // Store how many tokens were used for this candidate
    voterInfo[msg.sender].tokensUsedPerCandidate[index] += votesInTokens;
  }

  // Return the sum of all the tokens used by this voter.
  function totalTokensUsed(uint[] _tokensUsedPerCandidate) private pure returns (uint) {
    uint totalUsedTokens = 0;
    for(uint i = 0; i < _tokensUsedPerCandidate.length; i++) {
      totalUsedTokens += _tokensUsedPerCandidate[i];
    }
    return totalUsedTokens;
  }

  function indexOfCandidate(bytes32 candidate) view public returns (uint) {
    for(uint i = 0; i < candidateList.length; i++) {
      if (candidateList[i] == candidate) {
        return i;
      }
    }
    return uint(-1);
  }

  function buy() payable public returns (uint) {
    uint tokensToBuy = msg.value / tokenPrice;
    require(tokensToBuy <= balanceTokens);
    voterInfo[msg.sender].voterAddress = msg.sender;
    voterInfo[msg.sender].tokensBought += tokensToBuy;
    balanceTokens -= tokensToBuy;
    return tokensToBuy;
  }

  function tokensSold() view public returns (uint) {
    return totalTokens - balanceTokens;
  }

  function voterDetails(address user) view public returns (uint, uint[]) {
    return (voterInfo[user].tokensBought, voterInfo[user].tokensUsedPerCandidate);
  }

  function transferTo(address account) public {
    account.transfer(this.balance);
  }

  function allCandidates() view public returns (bytes32[]) {
    return candidateList;
  }

}
```

同时要配合修改一下index.html里的内容，以支持填入要投票的token数量，和查看投票人信息的功能：

* 投票时，你需要指定给候选人投多少票(tokens)

* 新增一个可购买token的section

* 增加查看voter的功能

* 候选人信息不再写死，通过调用合约方法获得

index.html

```
<!DOCTYPE html>
<html>
<head>
  <title>Hello World DApp</title>
  <link href='https://fonts.googleapis.com/css?family=Open+Sans:400,700' rel='stylesheet' type='text/css'>
  <link href='https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css' rel='stylesheet' type='text/css'>
  <style>
    .margin-top-3 {
      margin-top: 3em;
    }
  </style>
</head>
<body class="container">
  <h1>A Simple Hello World Voting Application</h1>
  <div class="col-sm-7 margin-top-3">
    <h2>Candidates</h2>
    <div class="table-responsive">
      <table class="table table-bordered">
        <thead>
          <tr>
            <th>Candidate</th>
            <th>Votes</th>
          </tr>
        </thead>
        <tbody id="candidate-rows">
        </tbody>
      </table>
    </div>
    <div class="container-fluid">
      <h2>Vote for Candidate</h2>
      <div id="msg"></div>
      <input type="text" id="candidate" placeholder="Enter the candidate name"/>
      <br>
      <br>
      <input type="text" id="vote-tokens" placeholder="Total no. of tokens to vote"/>
      <br>
      <br>
      <a href="#" onclick="voteForCandidate()" class="btn btn-primary">Vote</a>
    </div>
  </div>
  <div class="col-sm-offset-1 col-sm-4 margin-top-3">
    <div class="row">
      <h2>Token Stats</h2>
      <div class="table-responsive">
        <table class="table table-bordered">
          <tr>
            <td>Tokens For Sale</td>
            <td id="tokens-total"></td>
          </tr>
          <tr>
            <td>Tokens Sold</td>
            <td id="tokens-sold"></td>
          </tr>
          <tr>
            <td>Price Per Token</td>
            <td id="token-cost"></td>
          </tr>
          <tr>
            <td>Balance in the contract</td>
            <td id="contract-balance"></td>
          </tr>
        </table>
      </div>
    </div>
    <div class="row margin-top-3">
      <h2>Purchase Tokens</h2>
      <div class="col-sm-12">
        <div id="buy-msg"></div>
        <input type="text" id="buy" class="col-sm-8" placeholder="Number of tokens to buy"/>&nbsp;
        <a href="#" onclick="buyTokens()" class="btn btn-primary">Buy</a>
      </div>
    </div>
    <div class="row margin-top-3">
      <h2>Lookup Voter Info</h2>
      <div class="col-sm-12">
        <input type="text" id="voter-info", class="col-sm-8" placeholder="Enter the voter address" />&nbsp;
        <a href="#" onclick="lookupVoterInfo()" class="btn btn-primary">Lookup</a>
        <div class="voter-details row text-left">
          <div id="tokens-bought" class="margin-top-3 col-md-12"></div>
          <div id="votes-cast" class="col-md-12"></div>
        </div>
      </div>
    </div>
  </div>
</body>
<script src="https://code.jquery.com/jquery-3.1.1.slim.min.js"></script>
<script src="app.js"></script>
</html>
```

app.js

```
// Import the page's CSS. Webpack will know what to do with it.
import "../stylesheets/app.css";

// Import libraries we need.
import { default as Web3} from 'web3';
import { default as contract } from 'truffle-contract'
import voting_artifacts from '../../build/contracts/Voting.json'

var Voting = contract(voting_artifacts);

let candidates = {}

let tokenPrice = null;

window.voteForCandidate = function(candidate) {
  let candidateName = $("#candidate").val();
  let voteTokens = $("#vote-tokens").val();
  $("#msg").html("Vote has been submitted. The vote count will increment as soon as the vote is recorded on the blockchain. Please wait.")
  $("#candidate").val("");
  $("#vote-tokens").val("");

  /* Voting.deployed() returns an instance of the contract. Every call
   * in Truffle returns a promise which is why we have used then()
   * everywhere we have a transaction call
   */
  Voting.deployed().then(function(contractInstance) {
    contractInstance.voteForCandidate(candidateName, voteTokens, {gas: 140000, from: web3.eth.accounts[0]}).then(function() {
      let div_id = candidates[candidateName];
      return contractInstance.totalVotesFor.call(candidateName).then(function(v) {
        $("#" + div_id).html(v.toString());
        $("#msg").html("");
      });
    });
  });
}

/* The user enters the total no. of tokens to buy. We calculate the total cost and send it in
 * the request. We have to send the value in Wei. So, we use the toWei helper method to convert
 * from Ether to Wei.
 */

window.buyTokens = function() {
  let tokensToBuy = $("#buy").val();
  let price = tokensToBuy * tokenPrice;
  $("#buy-msg").html("Purchase order has been submitted. Please wait.");
  Voting.deployed().then(function(contractInstance) {
    contractInstance.buy({value: web3.toWei(price, 'ether'), from: web3.eth.accounts[0]}).then(function(v) {
      $("#buy-msg").html("");
      web3.eth.getBalance(contractInstance.address, function(error, result) {
        $("#contract-balance").html(web3.fromWei(result.toString()) + " Ether");
      });
    })
  });
  populateTokenData();
}

window.lookupVoterInfo = function() {
  let address = $("#voter-info").val();
  Voting.deployed().then(function(contractInstance) {
    contractInstance.voterDetails.call(address).then(function(v) {
      $("#tokens-bought").html("Total Tokens bought: " + v[0].toString());
      let votesPerCandidate = v[1];
      $("#votes-cast").empty();
      $("#votes-cast").append("Votes cast per candidate: <br>");
      let allCandidates = Object.keys(candidates);
      for(let i=0; i < allCandidates.length; i++) {
        $("#votes-cast").append(allCandidates[i] + ": " + votesPerCandidate[i] + "<br>");
      }
    });
  });
}

/* Instead of hardcoding the candidates hash, we now fetch the candidate list from
 * the blockchain and populate the array. Once we fetch the candidates, we setup the
 * table in the UI with all the candidates and the votes they have received.
 */
function populateCandidates() {
  Voting.deployed().then(function(contractInstance) {
    contractInstance.allCandidates.call().then(function(candidateArray) {
      for(let i=0; i < candidateArray.length; i++) {
        /* We store the candidate names as bytes32 on the blockchain. We use the
         * handy toUtf8 method to convert from bytes32 to string
         */
        candidates[web3.toUtf8(candidateArray[i])] = "candidate-" + i;
      }
      setupCandidateRows();
      populateCandidateVotes();
      populateTokenData();
    });
  });
}

function populateCandidateVotes() {
  let candidateNames = Object.keys(candidates);
  for (var i = 0; i < candidateNames.length; i++) {
    let name = candidateNames[i];
    Voting.deployed().then(function(contractInstance) {
      contractInstance.totalVotesFor.call(name).then(function(v) {
        $("#" + candidates[name]).html(v.toString());
      });
    });
  }
}

function setupCandidateRows() {
  Object.keys(candidates).forEach(function (candidate) { 
    $("#candidate-rows").append("<tr><td>" + candidate + "</td><td id='" + candidates[candidate] + "'></td></tr>");
  });
}

/* Fetch the total tokens, tokens available for sale and the price of
 * each token and display in the UI
 */
function populateTokenData() {
  Voting.deployed().then(function(contractInstance) {
    contractInstance.totalTokens().then(function(v) {
      $("#tokens-total").html(v.toString());
    });
    contractInstance.tokensSold.call().then(function(v) {
      $("#tokens-sold").html(v.toString());
    });
    contractInstance.tokenPrice().then(function(v) {
      tokenPrice = parseFloat(web3.fromWei(v.toString()));
      $("#token-cost").html(tokenPrice + " Ether");
    });
    web3.eth.getBalance(contractInstance.address, function(error, result) {
      $("#contract-balance").html(web3.fromWei(result.toString()) + " Ether");
    });
  });
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
  populateCandidates();

});
```

更新`2_deploy_contracts.js`代码，传入总token数量和token价格，并重新部署：

```
var Voting = artifacts.require("./Voting.sol");

module.exports = function(deployer) {
  deployer.deploy(Voting, 1000， web3.toWei('0.1', 'ether'), ['Sam', 'Leslie', 'Jetty', 'Arkila', 'Piu']);
};
```


```
truffle migrate
```

部署成功后，启动本地server:

```
npm run dev
```

开始愉快的投票吧。。。

![屏幕快照 2018-04-10 下午4.17.07.png](https://upload-images.jianshu.io/upload_images/1224641-f4bca79ecc490762.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

本节项目代码：[https://github.com/sam408130/ethereum_voting_dapp/tree/master/chapter3](https://github.com/sam408130/ethereum_voting_dapp/tree/master/chapter3)


原文地址： [Full Stack Hello World Voting Ethereum Dapp Tutorial, Mahesh Murthy](https://medium.com/@mvmurthy/full-stack-hello-world-voting-ethereum-dapp-tutorial-part-3-331c2712c9df)

