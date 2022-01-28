---
title: Truffle을 이용하여 이더리움 블록체인에 스마트 컨트랙 배포하기 (2)
description: Truffle project 내부의 truffle-config 파일에서 이더리움 ropsten 테스트 네트워크를 설정하는 방법
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

이번 포스팅에서는 truffle project 내부의 truffle-config 파일에서 이더리움 rospsten 테스트 네트워크로 네트워크를 설정하는 방법을 포스팅하려고 합니다. 우선 infura에서 회원가입 - 로그인 - 프로젝트를 생성해주세요. 프로젝트 이름은 상관 없습니다. infura 링크를 걸어 놓을게요. 
</br>
    https://infura.io
</br>
</br>
그럼 이제 infura에서 사용할 프로젝트를 클릭하고 SETTINGS 페이지로 들어가 주세요. 그러면 해당 페이지의 PROJECT ID가 나옵니다. 우리는 이걸 사용할거에요. ENDPOINTS는 접근 가능한 주소입니다. 주소 뒷부분에는 PROJECT ID가 그대로 들어가요. 저희는 테스트 네트워크 중에서 가장 크고, 흔히 알려진 ropsten 네트워크를 사용할 거에요. ENDPOINTS 옆에 드롭 다운 메뉴를 펼치면 main net을 포함한 다른 테스트 네트워크 주소를 확인할 수 있습니다.
</br>
</br>
![infura key 확인](https://user-images.githubusercontent.com/57793091/151474083-13dcaa49-bebc-4be8-ba1f-a4a20f1c3cdf.jpg)
</br>
</br>
다음으로는 메타마스크 계정이 있어야해요. 메타마스크 계정을 생성했다고 가정하고 그 다음부터 해볼게요. 먼저 제 계정 이름 밑에 계정 해시 값을 사용합니다. 그리고 계정 바로 밑 세부사항 - 개인키 내보내기를 누르고 비밀번호를 입력하면 개인키를 확인할 수 있습니다. 이것도 사용할 거에요!
</br>
</br>
![메타마스크 계정 확인](https://user-images.githubusercontent.com/57793091/151474155-af60be80-1cb9-4b81-9dcc-8902a43e42eb.jpg)
</br>
</br>
이제 오른쪽 위에 네트워크 ?test 라고 되어 있는 드롭 다운 메뉴를 펼쳐보겠습니다. 저는 이미 네트워크를 구성해 놓아서 test라고 해 놓은거에요. 제일 아래 사용자 정의 RPC를 선택해주세요.
</br>
</br>
![메타마스크 네트워크 지정](https://user-images.githubusercontent.com/57793091/151474211-05430e85-ad66-4eb1-bc35-0b1b72a3ef2e.jpg)
</br>
</br>
네트워크 추가 버튼을 누르시고, 네트워크 이름을 설정하면 됩니다. 이 때 네트워크 이름은 자신만 보는거라 어떻게 설정하셔도 상관 없어요. 아래 RPC URL 에는 아까 infura에서 복사한 ENDPOINTS 를 넣어 줍니다. 빨간 줄이 제 Project ID 에요. ChainID 속성은 사실 잘 모르겠는데, 저는 truffle-config.js에서 네트워크 아이디를 3으로 해놔서 그냥 일단 3으로 했습니다. 그리고 저장했을때, 연결이 됩니다. 연결이 실패가 되지 않으면 자신의 infura에 연결 된거에요. 이 연결한 계정을 해당 프로젝트에 사용하겠습니다.
</br>
</br>
![infura주소로 네트워크 설정](https://user-images.githubusercontent.com/57793091/151474337-854997de-b32b-4988-979e-91f686274a7f.jpg)
</br>
</br>
그리고 여러분이 처음 메타마스크 계정을 생성하면 당연하게 이더가 0만큼 있을거에요. 보통 gas라고 하는 수수료는 wei단위(이더보다 한참 작은 단위)로 payment가 되는데, 그래도 테스트용 이더가 필요하기 때문에, 우리는 faucet이라는 곳에서 이더를 받을 거에요. faucet은 테스트용 네트워크에 임의로 이더를 충전해주는 사이트인데, 각각의 테스트 네트워크마다 faucet 주소가 다 다르기 때문에 이 점 유의하시기 바랍니다. faucet의 주소를 올려 놓을게요. faucet에서 이더를 받으면 트랜잭션을 발행해서 이더가 오는 것이기 때문에 조금 시간이 걸려요. 바로 안 온다고 계속 요청하지 맙시다. faucet은 기본적으로 24시간 당 한 번 요청이 가능해요.

https://faucet.ropsten.be/
</br>
</br>
이제 전에 생성한 트러플 프로젝트에서 truffle-config로 네트워크 설정을 해볼게요. 기본적으로 모든 변수 및 네트워크는 주석처리 되어있습니다. 위에 HDWalletProvider 부터 mnemonic 까지는 주석처리 되어있어요. 저희는 트랜잭션 발행자(프로젝트 배포자)가 저희의 계정이므로 니모닉은 사용하지 않을 예정입니다. 그래서 주석을 풀지 않았아요. 대신에 저희의 메타마스크 계정과 계정의 개인키를 써줍니다. 그리고 truffle-hdwallet-provider 라는 모듈은 사용하시는 데스크탑 환경에 설치되어 있어야해요. 아마 npm 명령어로 설치 했던 것 같네요! infura 키에도 프로젝트 ID를 넣어주세요.
</br>
</br>
네트워크에 development 는 테스트용입니다. 루프백 IP가 적혀 있는 걸 보아 local에서 실행 가능한 환경입니다. 이 때 이 실제 테스트에서 develope 네트워크를 사용하시려면 인바운드 규칙에 8545 포트를 추가하셔야 될 거에요.
</br>
</br>
![트러플 설정](https://user-images.githubusercontent.com/57793091/151474464-28267f0c-e2e2-41d1-b65a-a50edda89a83.jpg)
</br>
</br>
더 아래로 내려보면, ropsten 네트워크가 있습니다. 주석을 풀어줍니다. 저 내용이 기본적으로 전부 적혀있어요. 저희가 수정할 부분은 provider 부분에 파라미터인데요. 원래는 니모닉이 적혀있는데, 저는 저희의 계정으로 배포할 것이므로 위에 적었던 privateKeys 파라미터를 바꿔줍니다. 주소 부분은 아마 건들일 필요없이 저렇게 그대로 적혀있었던 것 같아요. 자 이제 네트워크 구성을 끝마치고 배포할 준비를 했습니다. 다음 포스트에서는 간단히 제가 코딩한 솔리디티 파일을 보여드리고, 배포하는 명령어를 알아보도록 하겠습니다.
</br>
</br>
니모닉을 이용하는 방법도 크게 어렵지는 않은데, 테스트 환경인 가나슈나 혹은 truffle develop 명령어를 실행하면 나오는 니모닉을 복사해서 해당 프로젝트 내에 .secret 파일을 만들어서 붙여 넣으면 됩니다.
</br>
</br>
![트러플 설정2](https://user-images.githubusercontent.com/57793091/151474551-868bc485-92ac-4637-a3e0-c8da868a2599.png)
</br>