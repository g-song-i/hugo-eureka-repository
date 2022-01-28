---
title: Truffle을 이용하여 이더리움 블록체인에 스마트 컨트랙 배포하기 (1)
description: Truffle 툴을 통해 이더리움 ropsten 네트워크에 블록체인 스마트 컨트랙을 배포하는 방법
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

이번 포스트부터는 Truffle이라는 툴을 통해 이더리움 ropsten 네트워크에 저희의 블록체인 컨트랙을 배포하는 방법을 포스트 하려고 합니다.
</br>
그 전에 제가 블록체인을 공부하면서 모호했던 부분들을 정리해보려고 해요. 
</br>
1. Contract이 왜 필요한가? Contract이 블록과 다른 것인가?
2. Truffle은 왜 필요한가?
3. 이더리움 네트워크에 어떻게 내 컨트랙을 배포하며, 어떻게 접근해서 사용하는가?
</br>

먼저 첫번째 의문에 대한 답이에요. 제가 개인적으로 공부하고 이해한 사실을 서술한 것이기 때문에 객관적인 사실과 다르거나 틀린 내용이 있을 수 있어요. 우선 우리가 만약 특정한 계정을 가지고 블록체인에 참여하는 노드라고 합시다. 우리가 만약 메타 마스크 같은 지갑(그러니까 계정)을 가지고 있다면, 직접 받는 사람을 써서 트랜잭션을 만들 수 있어요. 그러나 사람들이 애플리케이션을 사용할 때, 트랜잭션의 발행이 필요할 때마다 자신들이 일일이 계정(지갑)을 만들어 상대편에게 트랜잭션을 보낼 수 없을 겁니다. 그건 블록체인에 대한 지식이 없는 일반 사용자 입장에서는 어려운 일이고, 그걸 대신해주는 게 애플리케이션이니까요. 
</br>
</br>
그러면, 필요한 내용을 담아 조건을 충족할 경우 트랜잭션을 발행할 수 있도록 하는 코드를 짜게 됩니다. 이게 Contract이에요. 물론 컨트랙이 직접 트랜잭션을 발행하진 않습니다. 저 같은 경우는 Js에서 설치할 수 있는 Web3 Js 모듈을 통해 컨트랙에 접근해 트랜잭션을 발행합니다. 컨트랙은 트랜잭션의 내용을 담는 것이라고 생각하면 되겠습니다.  그럼 컨트랙은 블록과는 다른 걸까요? 답은 아닙니다. 컨트랙도 블록으로 블록체인에 붙게 됩니다. 특정한 해시값을 가지며, 이 해시 값으로 접근이 가능합니다.
</br>
</br>
다음으로 트러플이 필요한 이유입니다. 우선 Truffle은 npm 명령어를 통해 설치할 수 있어요. 일단 트러플은 컨트랙을 배포해주는 툴이라고 생각하면 편할 것 같아요. 만약 리믹스를 통해 배포할 것이다. 그러면 트러플은 필요가 없습니다. 그런데 배포할 컨트랙이 많을 경우, 한 번에 배포 가능한 트러플이 조금 더 편할 것 같아요. 프로젝트 자체를 한꺼번에 볼 수 있어서 그런 점도 개인적으로는 편한 것 같습니다.
</br>
</br>
마지막으로 이더리움 네트워크에 어떻게 내 컨트랙을 배포하며, 어떻게 접근할까요? truffle 모듈로 프로젝트를 초기화하게 되면, configuration 파일이 생성됩니다. 여기서 네트워크를 지정해주면 해당 truffle 명령어로 해당 네트워크에 컨트랙을 배포할 수 있습니다. 위에서, 컨트랙을 배포하면 블록이 되므로 특정한 해시값을 가지고, 이 해시 값으로 접근이 가능하다고 얘기했습니다. 
</br>
</br>
일단 Node js와 truffle 이 설치되어 있다고 가정하겠습니다. npm도 설치되어 있어야 하는데 윈도우에서는 Node js를 설치하면 자동적으로 설치가 됩니다. truffle은 npm 명령어로 설치할 수 있습니다. 저는 AWS에서 할당받은 ubuntu instace에서 처음에 블록체인을 돌리다가, 메모리가 초과되어 멈추는 일이 빈번하게 발생해서 그냥 로컬 컴퓨터에서 배포하였습니다. 저는 infura에서 public key를 받아 public test network에 배포할 거기 때문에 local에서 해도 괜찮았어요! 먼저 node -v, npm -v, truffle -v로 얘네들이 설치되어 있는지 확인할 수 있습니다.
</br>
</br>
자, 우선 cmd에서 해당 프로젝트 경로로 이동한 뒤 truffle init 명령어를 입력해주세요. 그럼 cmd에서는 아래와 같은 화면을 확인할 수 있습니다. 
</br>
</br>
<p align="center"><img src="https://user-images.githubusercontent.com/57793091/151473178-b90e60a9-9f46-42f9-9ae0-66584e116cd6.png"></p>
</br>
</br>
실제 프로젝트는 이렇게 초기화가 됩니다.
</br>
</br>
<p align="center"><img src="https://user-images.githubusercontent.com/57793091/151473549-23d34188-2f41-4eca-9b28-86768abbd653.png"></p>
</br>
</br>
저는 편집기로 vs code를 이용했어요. 마켓 플레이스에서 solidity를 받을 수 있습니다. 프로젝트 구조만 살짝 볼게요. contracts라는 디렉토리에는 sol 파일을 생성합니다. migrations 디렉터리에는 sol 파일을 모듈로 exports 해서 배포할 수 있도록 하는 js코드를 작성할 거예요. 지금 내부에 들어있는 Migration 파일은 기존에 생성되어 있는 파일입니다. 그리고 truffle-config 파일은 배포할 네트워크를 설정하는 파일이에요. 오늘은 프로젝트 구조를 보이려고 test로 했지만, 다음부터는 제가 직접 짠 코드로 포스트 하려고 합니다. 바로 다음 포스트는 배포를 미리 준비하기 위해, truffle-config를 설정하는 방법을 먼저 포스트 할 겁니다.
</br>
</br>
<p align="center"><img src="https://user-images.githubusercontent.com/57793091/151473656-4663e3d6-ab72-4ddb-b9d5-9188795aba20.png"></p>
</br>
</br>