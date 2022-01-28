---
title: Truffle을 이용하여 이더리움 블록체인에 스마트 컨트랙 배포하기 (4)
description: web3 js 모듈을 통해 해당 트랜잭션에 접근해 정보를 삽입하고, 가져오는 방법
toc: true
authors:
  - songi
tags:
  - Blockchain
  - Ethereum
  - Truffle
  - Smart contract
categories:
series: BlockChain Used Goods Trade Platform
date: '2020-05-16'
lastmod: '2020-05-16'
draft: false
---
</br>

## Content

이제 블록체인 마지막 포스트가 되겠네요. 해당 프로젝트의 다음 포스트부터는 multer 라이브러리로 이미지 업로드하는 방법과 안드로이드에서 어려움을 겪었던 구현 등을 포스팅할게요.
</br>
</br>
이번 포스트에서는 js에서 web3 js 모듈을 이용하여 컨트랙에 접근/ 사용하는 방법을 포스팅하려고 해요. 우선 물건을 삽입하고, 가져오는 코드를 보여드리려고 합니다. 우선 파일 가장 위에, 사용하려는 모듈을  const 변수로 선언해 놓았어요.
</br>

```javascript
const infuraKey = "자신의 인퓨라 키★";
const Web3 = require('web3');
const url = `https://ropsten.infura.io/v3/${infuraKey}`;
const web3 = new Web3(new Web3.providers.HttpProvider(url));
const mysql = require('mysql');
const bodyParser = require('body-parser');
const express = require('express');
var Tx = require('ethereumjs-tx').Transaction;

const rootAccount = "자신의 메타마스크 계정★";
const privateKey = new Buffer.from('자신의 메타마스크 비밀 키★', 'hex');
```
</br>
Tx는 계정에 서명을 해주기 위해서 사용하는 모듈입니다. ethereumjs-tx는 npm 명령어로 설치해주어야 해요. 비밀 키도 buffer 형태로 선언해줍니다.
</br>
</br>

```javascript
function insertBlock(transactionNumber, registerNumber, sellerId, buyerId, completeTime) {

		var transactionContract = new web3.eth.Contract([{

			"constant": true,
			"inputs": [
				{
					"internalType": "uint256",
					"name": "_transactionNumber",
					"type": "uint256"
				}
			],
			"name": "getTransactionInfo",
			"outputs": [
				{
					"internalType": "uint256",
					"name": "",
					"type": "uint256"
				},
				{
					"internalType": "uint256",
					"name": "",
					"type": "uint256"
				},
				{
					"internalType": "string",
					"name": "",
					"type": "string"
				},
				{
					"internalType": "string",
					"name": "",
					"type": "string"
				},
				{
					"internalType": "string",
					"name": "",
					"type": "string"
				}
			],
			"payable": false,
			"stateMutability": "view",
			"type": "function"
		},
			{
				"constant": false,
				"inputs": [
					{
						"internalType": "uint256",
						"name": "_transactionNumber",
						"type": "uint256"
					},
					{
						"internalType": "uint256",
						"name": "_registerNumber",
						"type": "uint256"
					},
					{
						"internalType": "string",
						"name": "_sellerId",
						"type": "string"
					},
					{
						"internalType": "string",
						"name": "_buyerId",
						"type": "string"
					},
					{
						"internalType": "string",
						"name": "_completeTime",
						"type": "string"
					}
				],
				"name": "setTransactionInfo",
				"outputs": [],
				"payable": false,
				"stateMutability": "nonpayable",
				"type": "function"
			}

		],"본인의 컨트랙 주소★");

		web3.eth.getTransactionCount(rootAccount, (err, txCount) => {

			const txObject = {
				nonce: web3.utils.toHex(txCount),
				gasLimit: web3.utils.toHex(800000),
				gasPrice: web3.utils.toHex(web3.utils.toWei('10', 'gwei')),
				to: "본인의 컨트랙 주소★",
				data: transactionContract.methods.setTransactionInfo(transactionNumber, registerNumber, sellerId, buyerId, completeTime).encodeABI()
			}

			const tx = new Tx(txObject, {chain : 'ropsten', hardfork: 'petersburg'});
			tx.sign(privateKey);

			const serializedTx = tx.serialize()
			const raw = '0x' + serializedTx.toString('hex')

			web3.eth.sendSignedTransaction(raw, (err, txHash) => {
				console.log('err:', err, 'txHash:', txHash)
				// Use this txHash to find the contract on Etherscan!
			});
		});



		//transactionContract
		transactionContract.methods.setTransactionInfo(transactionNumber, registerNumber, sellerId, buyerId, completeTime).send({ from: "보내는 계정★"})
			.on("receipt", function (receipt) {

				consolei.log(receipt);
			})
			.on("error", function (err){ 

				console.log(err);
			});
	}
```
</br>
abi는 컨트랙의 json 형태예요. 저는 remix에 코드를 넣고 abi를 추출하였습니다. web3.eth.Contract(abi, 컨트랙주소) 형태로 컨트랙을 불러와 var 형태로 선언해줍니다. 아래 web3.eth.getTransactionCount 는 블로그마다 조금씩 다를 거예요. 그런데 처음에는 Tx를 사용하지 않아서, 다음으로는 sendTransaction이라는 함수가 안되어서, 이런 이유들로 다음의 코드가 결론적으로 나왔던 것 같아요. 코딩한 지 살짝 오래되어서 엄청 고생했는데도 기억이 잘 안 나네요... 머쓱
</br>
</br>
어쨌든, sendTransaction이 안되어서 sendSignedTransaction을 따로 만들어 주었어요. tx는 비밀 키에 서명을 해주는 역할인데, tx를 선언할 때 chain과 hardfork를 명시하게 되어있어요. hardfork는 web3 api 사용 설명서에 보면 여러 단어가 있습니다. 원하는 걸 사용하시면 될 거 같아요. web3 api 설명서 링크를 첨부할게요.
</br>

https://web3js.readthedocs.io/en/v1.2.7/

제일 아래에 transactionContract는 제가 위에서 선언해서 var에 넣어놓은 함수인데요, transactionContrac.methods라는 메서드를 통해 내부 함수를 추출합니다. setTransactionInfo는 제가 만든 솔리디티 함수예요. 보낼 때는 receipt 혹은 err 메시지를 받아 콘솔에 로그로 확인하는데, 트랜잭션을 보내는 과정이 성공했다면 receipt를 확인할 수 있습니다. 
</br>
</br>
위가 연결하는 코드의 전문인데요. 작성할 땐 엄청 고생했는데 코드 전문을 보여드리니까 설명할 게 별로 없네요ㅠㅠ