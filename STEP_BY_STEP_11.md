# Week 2: NFT 민팅 DApp - 단계별 실습 가이드

## 목차

- [학습 개요](#학습-개요)
- [필수 환경 설정 (먼저 읽기)](#-필수-환경-설정-먼저-읽으세요)
- [5. Signer 개념 및 구현](#5-signer-개념-및-구현)
- [6. NFT 민팅 기능 구현](#6-nft-민팅-기능-구현)
- [7. 트랜잭션 처리 및 이벤트](#7-트랜잭션-처리-및-이벤트)
- [8. NFT 갤러리 구현](#8-nft-갤러리-구현)
- [최종 체크리스트](#최종-체크리스트-)
- [문제 해결](#문제-해결-)

---

## 학습 개요

### Week 2에서 배울 내용

Week 1에서는 블록체인에서 **데이터를 읽는(READ)** 방법을 배웠습니다. Week 2에서는 블록체인에 **데이터를 쓰는(WRITE)** 방법을 학습합니다.

**핵심 개념:**
- **Signer**: 트랜잭션을 서명하는 주체
- **Gas**: 트랜잭션 실행 비용
- **Minting**: NFT 생성
- **Events**: 블록체인 이벤트 리스닝
- **Gallery**: NFT 목록 표시

---

## 🔧 필수 환경 설정 (먼저 읽으세요!)

> ⚠️ **중요:** 아래 단계를 건너뛰지 마세요! 이 설정 없이는 튜토리얼을 진행할 수 없습니다.

### 0-1. 현재 프로젝트 상태 확인

먼저 프로젝트에 필요한 파일들이 있는지 확인합니다:

```bash
# 터미널에서 프로젝트 루트 디렉토리로 이동
cd /path/to/edu_dan_nft

# 필수 파일 확인
ls -la frontend/src/components/
```

**확인 사항:**
- `ConnectWallet.jsx` ✅ (Week 1에서 생성)
- `TotalSupply.jsx` ✅ (Week 1에서 생성)
- `MintNFT.jsx` ❓ (Week 2에서 생성 필요)
- `MyNFTs.jsx` ❓ (Week 2에서 생성 필요)

### 0-2. package.json 스크립트 추가

루트 디렉토리의 `package.json` 파일을 엽니다:

```bash
# 파일 열기 (VSCode 사용 시)
code package.json
```

`"scripts"` 섹션에 다음 두 줄을 추가합니다:

**수정 전:**
```json
{
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  }
}
```

**수정 후:**
```json
{
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "node": "npx hardhat node",
    "deploy:local": "npx hardhat run scripts/deploy.js --network localhost"
  }
}
```

저장하고 닫습니다.

### 0-3. hardhat.config.js 포트 확인

`hardhat.config.js` 파일을 열고 localhost 포트가 **8545**인지 확인합니다:

```javascript
module.exports = {
  solidity: "0.8.20",
  networks: {
    hardhat: {
      chainId: 31337
    },
    localhost: {
      url: "http://127.0.0.1:8545",  // 반드시 8545여야 함!
      chainId: 31337
    }
  }
};
```

만약 `18545`로 되어 있다면 `8545`로 수정하세요.

### 0-4. Hardhat 로컬 노드 실행

```bash
# 터미널 1 (이 터미널은 계속 실행 상태로 유지)
npm run node
```

**예상 출력:**
```
Started HTTP and WebSocket JSON-RPC server at http://127.0.0.1:8545/

Account #0: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 (10000 ETH)
Private Key: 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
...
```

> 💡 **중요:** Account #0의 주소를 메모하세요. 이 계정이 컨트랙트 Owner가 됩니다.

### 0-5. 컨트랙트 배포

**새 터미널을 열고** (터미널 2):

```bash
# 프로젝트 루트에서
npm run deploy:local
```

**예상 출력:**
```
컨트랙트 배포 중...
Step8_CompleteERC721 배포 완료: 0x5FbDB2315678afecb367f032d93F642f64180aa3
배포자 주소: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
```

배포된 **컨트랙트 주소**를 메모하세요.

### 0-6. .env.local 파일 확인

`frontend/.env.local` 파일을 열어 컨트랙트 주소가 올바른지 확인합니다:

```bash
# 파일 열기
code frontend/.env.local
```

**내용:**
```
PORT=13000
REACT_APP_CONTRACT_ADDRESS=0x5FbDB2315678afecb367f032d93F642f64180aa3
REACT_APP_NETWORK_ID=31337
REACT_APP_NETWORK_NAME=localhost
```

- `REACT_APP_CONTRACT_ADDRESS`가 방금 배포한 주소와 일치하는지 확인
- 다르다면 배포된 주소로 업데이트

### 0-7. MetaMask 네트워크 설정

1. **MetaMask 확장 프로그램 열기**

2. **네트워크 드롭다운 클릭** → "네트워크 추가" → "네트워크를 수동으로 추가"

3. **다음 정보 입력:**
   ```
   네트워크 이름: Localhost 8545
   새 RPC URL: http://127.0.0.1:8545
   체인 ID: 31337
   통화 기호: ETH
   ```

4. **"저장" 클릭**

5. **네트워크 드롭다운에서 "Localhost 8545" 선택**

### 0-8. 테스트 계정 Import

1. MetaMask에서 **계정 아이콘 클릭** → "계정 가져오기"

2. **유형:** 비공개 키

3. **비공개 키 입력:**
   ```
   0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
   ```

4. **"가져오기" 클릭**

5. 계정 이름을 "Hardhat #0" 등으로 변경 (선택사항)

> ⚠️ **경고:** 이 키는 공개된 테스트 키입니다. 실제 네트워크에서는 절대 사용하지 마세요!

### 0-9. 환경 설정 체크리스트

모든 항목이 체크되어야 다음 단계로 진행할 수 있습니다:

- [ ] package.json에 `node`, `deploy:local` 스크립트 추가됨
- [ ] hardhat.config.js의 localhost 포트가 8545임
- [ ] Hardhat 로컬 노드 실행 중 (터미널 1)
- [ ] 컨트랙트 배포 완료 (배포 주소 메모함)
- [ ] frontend/.env.local에 올바른 컨트랙트 주소 설정됨
- [ ] MetaMask에 Localhost 8545 네트워크 추가됨
- [ ] MetaMask에 Hardhat Account #0 import됨
- [ ] MetaMask에서 Localhost 8545 네트워크 선택됨

모든 항목이 체크되면 다음 섹션으로 진행하세요! 👇

---

## 5. Signer 개념 및 구현

### 학습 목표

이 단계를 완료하면:
- Provider와 Signer의 차이를 이해할 수 있습니다
- Signer 객체를 생성할 수 있습니다
- 왜 트랜잭션에 서명이 필요한지 이해할 수 있습니다

### 개념 설명 💡

#### Provider vs Signer

**비유로 이해하기:**
```
Provider = 도서관 열람증
- 책을 읽을 수 있음 (READ)
- 책을 빌리거나 바꿀 수 없음
- 무료

Signer = 신용카드 + 서명
- 물건을 살 수 있음 (WRITE)
- 거래마다 서명 필요
- 비용(가스비) 지불
```

**코드로 비교:**

```javascript
// Provider로 할 수 있는 것 (READ - 무료)
const provider = new ethers.BrowserProvider(window.ethereum);
const contract = new ethers.Contract(address, abi, provider);
const supply = await contract.totalSupply(); // 작동

// Signer로 할 수 있는 것 (WRITE - 가스비 필요)
const signer = await provider.getSigner();
const contract = new ethers.Contract(address, abi, signer);
const tx = await contract.mint(address, tokenURI); // 작동
```

#### 왜 Signer가 필요한가?

블록체인에서 데이터를 변경하려면:
1. **신원 증명**: 누가 이 작업을 요청했는지 증명
2. **권한 확인**: 이 작업을 할 권한이 있는지 확인
3. **비용 지불**: 가스비를 누가 낼 것인지 명시

**핵심 차이:**

| 항목 | Provider | Signer |
|------|----------|--------|
| 읽기 (READ) | ✅ 가능 | ✅ 가능 |
| 쓰기 (WRITE) | ❌ 불가능 | ✅ 가능 |
| 가스비 | 불필요 | 필요 |
| 서명 | 불가능 | 가능 |
| 개인키 | 불필요 | 필요 (MetaMask 관리) |

### 체크포인트 ✅

다음 개념을 이해했는지 확인하세요:

1. **Provider와 Signer의 차이**
   - [ ] Provider는 READ만, Signer는 READ+WRITE
   - [ ] Signer는 가스비를 지불함
   - [ ] Signer는 개인키로 트랜잭션을 서명함

2. **Signer 생성 방법**
   - [ ] `provider.getSigner()`로 생성
   - [ ] 필요할 때마다 생성하는 것이 안전함

---

## 6. NFT 민팅 기능 구현

### 학습 목표

이 단계를 완료하면:
- 스마트 컨트랙트의 쓰기 함수를 호출할 수 있습니다
- 트랜잭션을 생성하고 전송할 수 있습니다
- 가스 개념을 이해하고 가스비를 추정할 수 있습니다
- 트랜잭션 상태를 관리할 수 있습니다

### 개념 설명 💡

#### 트랜잭션이란?

**트랜잭션 = 블록체인 상태를 변경하는 요청**

```javascript
// READ (트랜잭션 X, 무료, 즉시)
const supply = await contract.totalSupply();

// WRITE (트랜잭션 O, 가스비, 대기시간)
const tx = await contract.mint(address, tokenURI);
await tx.wait(); // 블록에 포함될 때까지 대기
```

#### 가스(Gas)란?

**가스 = 블록체인 작업의 계산 비용**

```
가스비 = Gas Used × Gas Price

예시:
- Gas Used: 85,000 (민팅에 사용된 가스)
- Gas Price: 20 Gwei (사용자가 지불할 가격)
- 총 비용: 85,000 × 20 Gwei = 0.0017 ETH
```

**왜 가스비가 필요한가?**
1. **네트워크 유지**: 검증자(Validator)에게 보상
2. **스팸 방지**: 무분별한 트랜잭션 방지
3. **자원 할당**: 제한된 블록 공간 효율적 사용

#### 우리 컨트랙트의 mint 함수

```solidity
// Step8_CompleteERC721.sol (49-54번째 줄)
function mint(address to, string memory uri) public onlyOwner returns (uint256) {
    uint256 tokenId = _nextTokenId++;
    _safeMint(to, tokenId);
    _setTokenURI(tokenId, uri);
    return tokenId;
}
```

**중요한 점:**
- ⚠️ `onlyOwner`: 컨트랙트 소유자만 호출 가능
- 파라미터 2개: `address to`, `string uri`
- 반환값: `uint256 tokenId` (하지만 프론트엔드에서는 이벤트로 확인)

### 실습: MintNFT 컴포넌트 생성 🛠️

> 💡 **이제 직접 코드를 작성합니다!**

#### 6-1. MintNFT.jsx 파일 생성

터미널에서:

```bash
# 파일 생성
touch frontend/src/components/MintNFT.jsx

# 파일 열기 (VSCode)
code frontend/src/components/MintNFT.jsx
```

#### 6-2. 기본 구조 작성

다음 코드를 복사하여 붙여넣습니다:

```javascript
import React, { useState } from 'react';
import { ethers } from 'ethers';
import { CONTRACT_ADDRESS, CONTRACT_ABI, getEtherscanLink } from '../utils/contractConfig';
import { ensureCorrectNetwork } from '../utils/networkHelper';

function MintNFT({ account, onMintSuccess }) {
  // 상태 관리
  const [recipientAddress, setRecipientAddress] = useState('');
  const [tokenURI, setTokenURI] = useState('');
  const [isMinting, setIsMinting] = useState(false);
  const [txHash, setTxHash] = useState('');
  const [mintedTokenId, setMintedTokenId] = useState(null);
  const [error, setError] = useState('');

  // 내 주소로 채우기
  const fillMyAddress = () => {
    setRecipientAddress(account);
  };

  // 샘플 URI 채우기
  const fillSampleURI = () => {
    setTokenURI('https://ipfs.io/ipfs/QmeSjSinHpPnmXmspMjwiXyN6zS4E9zccariGR3jxcaWtq/1');
  };

  // 폼 제출 처리
  const handleSubmit = (e) => {
    e.preventDefault();

    // 입력 검증
    if (!recipientAddress || !tokenURI) {
      setError('모든 필드를 입력해주세요.');
      return;
    }

    if (!ethers.isAddress(recipientAddress)) {
      setError('유효하지 않은 이더리움 주소입니다.');
      return;
    }

    mintNFT();
  };

  // NFT 민팅 함수 (여기서 구현할 예정)
  const mintNFT = async () => {
    // TODO: 다음 단계에서 구현
  };

  return (
    <div className="card">
      <h2>NFT 민팅 (2주차: WRITE 작업)</h2>

      {!account ? (
        <div className="info-box warning">
          <p>지갑을 먼저 연결해주세요.</p>
        </div>
      ) : (
        <form onSubmit={handleSubmit}>
          {/* 수신자 주소 입력 */}
          <div>
            <label>수신자 주소:</label>
            <input
              type="text"
              value={recipientAddress}
              onChange={(e) => setRecipientAddress(e.target.value)}
              placeholder="0x..."
              style={{ marginRight: '10px' }}
            />
            <button type="button" onClick={fillMyAddress}>
              내 주소로
            </button>
          </div>

          {/* 토큰 URI 입력 */}
          <div>
            <label>토큰 URI (메타데이터 JSON):</label>
            <input
              type="text"
              value={tokenURI}
              onChange={(e) => setTokenURI(e.target.value)}
              placeholder="https://ipfs.io/ipfs/..."
              style={{ marginRight: '10px' }}
            />
            <button type="button" onClick={fillSampleURI}>
              샘플 URI
            </button>
          </div>

          {/* 에러 표시 */}
          {error && <div className="error">{error}</div>}

          {/* 민팅 버튼 */}
          <button type="submit" disabled={isMinting}>
            {isMinting ? '민팅 중...' : 'NFT 민팅하기'}
          </button>
        </form>
      )}

      {/* 트랜잭션 결과 표시 */}
      {txHash && (
        <div className="tx-status">
          <h3>민팅 성공!</h3>
          {mintedTokenId !== null && (
            <p>토큰 ID: #{mintedTokenId}</p>
          )}
          <p>
            트랜잭션:{' '}
            {getEtherscanLink('tx', txHash) ? (
              <a href={getEtherscanLink('tx', txHash)} target="_blank" rel="noreferrer">
                {txHash.substring(0, 10)}...
              </a>
            ) : (
              <span>{txHash.substring(0, 10)}...</span>
            )}
          </p>
        </div>
      )}
    </div>
  );
}

export default MintNFT;
```

저장합니다 (Ctrl+S 또는 Cmd+S).

#### 6-3. mintNFT 함수 구현 (핵심!)

이제 `mintNFT` 함수를 구현합니다. 주석 `// TODO: 다음 단계에서 구현` 부분을 다음 코드로 교체합니다:

```javascript
const mintNFT = async () => {
  try {
    setIsMinting(true);
    setError('');
    setTxHash('');
    setMintedTokenId(null);

    // 올바른 네트워크로 전환 확인
    await ensureCorrectNetwork();

    // 1단계: Provider와 Signer 생성
    const provider = new ethers.BrowserProvider(window.ethereum);
    const signer = await provider.getSigner();

    console.log('Signer 주소:', await signer.getAddress());

    // 2단계: Contract 인스턴스 생성 (Signer와 함께)
    const contract = new ethers.Contract(
      CONTRACT_ADDRESS,
      CONTRACT_ABI,
      signer // Signer 사용!
    );

    // 3단계: 가스 추정 (선택사항, 하지만 유용함)
    try {
      const gasEstimate = await contract.mint.estimateGas(
        recipientAddress,
        tokenURI
      );
      console.log('예상 가스:', gasEstimate.toString());
    } catch (gasError) {
      console.warn('가스 추정 실패:', gasError);
      // onlyOwner 체크
      if (gasError.message.includes('OwnableUnauthorizedAccount')) {
        setError('컨트랙트 소유자만 민팅할 수 있습니다.');
        setIsMinting(false);
        return;
      }
    }

    // 4단계: mint() 함수 호출 및 트랜잭션 전송
    console.log('트랜잭션 전송 중...');
    const tx = await contract.mint(recipientAddress, tokenURI);

    console.log('트랜잭션 전송 완료!');
    console.log('TX Hash:', tx.hash);
    setTxHash(tx.hash);

    // 5단계: 트랜잭션 확인 대기 (7에서 자세히)
    console.log('블록 확인 대기 중...');
    const receipt = await tx.wait();

    console.log('트랜잭션 확인 완료!');
    console.log('블록 번호:', receipt.blockNumber);

    // 이벤트 파싱 (섹션 7에서 추가할 예정)
    // TODO: Transfer 이벤트에서 tokenId 추출

    // 부모 컴포넌트(App.jsx)에 성공 알림
    if (onMintSuccess) {
      onMintSuccess();
    }

  } catch (err) {
    console.error('민팅 오류:', err);

    // 에러 처리
    if (err.code === 'ACTION_REJECTED') {
      setError('사용자가 트랜잭션을 거부했습니다.');
    } else if (err.message.includes('OwnableUnauthorizedAccount')) {
      setError('컨트랙트 소유자만 NFT를 민팅할 수 있습니다.');
    } else {
      setError(err.message || '민팅에 실패했습니다.');
    }
  } finally {
    setIsMinting(false);
  }
};
```

저장합니다.

**코드 설명:**

```javascript
// 트랜잭션 라이프사이클
// 1. 함수 호출
const tx = await contract.mint(recipientAddress, tokenURI);
// 이 시점: 트랜잭션 전송됨, 하지만 아직 블록에 포함 안됨

// 2. tx 객체
console.log(tx.hash); // 트랜잭션 해시
console.log(tx.from); // 보낸 사람
console.log(tx.to);   // 받는 사람 (컨트랙트 주소)

// 3. 블록 확인 대기
const receipt = await tx.wait();
// 이 시점: 블록에 포함됨!
```

#### 6-4. App.js에 MintNFT 추가

`frontend/src/App.js` 파일을 엽니다:

```bash
code frontend/src/App.js
```

**1단계:** Import 추가 (파일 상단)

```javascript
import MintNFT from './components/MintNFT';
```

**2단계:** handleMintSuccess 함수 추가 (handleAccountChange 아래)

```javascript
const handleMintSuccess = () => {
  // 민팅 성공 시 refreshTrigger 증가
  setRefreshTrigger(prev => prev + 1);
  console.log('민팅 성공! 갤러리 새로고침');
};
```

**3단계:** MintNFT 컴포넌트 사용 (TotalSupply 아래)

기존:
```javascript
<>
  {/* totalSupply 표시 */}
  <TotalSupply account={account} refreshTrigger={refreshTrigger} />
</>
```

수정 후:
```javascript
<>
  {/* totalSupply 표시 */}
  <TotalSupply account={account} refreshTrigger={refreshTrigger} />

  {/* NFT 민팅 */}
  <MintNFT account={account} onMintSuccess={handleMintSuccess} />
</>
```

저장합니다.

#### 6-5. 프론트엔드 실행 및 테스트

터미널 2 (또는 새 터미널)에서:

```bash
cd frontend
npm start
```

브라우저가 자동으로 열리고 http://localhost:13000 으로 이동합니다.

**테스트 순서:**

1. **지갑 연결**
   - "지갑 연동하기" 버튼 클릭
   - MetaMask에서 Hardhat #0 계정 선택
   - "연결" 클릭

2. **NFT 민팅**
   - "내 주소로" 버튼 클릭
   - "샘플 URI" 버튼 클릭
   - "NFT 민팅하기" 버튼 클릭
   - MetaMask 팝업에서 가스비 확인
   - "확인" 클릭

3. **콘솔 확인 (F12)**
   ```
   Signer 주소: 0xf39Fd...
   예상 가스: 85000
   트랜잭션 전송 중...
   트랜잭션 전송 완료!
   TX Hash: 0x...
   블록 확인 대기 중...
   트랜잭션 확인 완료!
   블록 번호: 2
   ```

4. **UI 확인**
   - "민팅 중..." 표시됨
   - "민팅 성공!" 메시지
   - 트랜잭션 해시 표시

### 체크포인트 ✅

- [ ] MintNFT.jsx 파일 생성
- [ ] mintNFT 함수 구현
- [ ] App.js에 MintNFT 컴포넌트 추가
- [ ] 프론트엔드 실행 성공
- [ ] 지갑 연결 성공
- [ ] NFT 민팅 테스트 성공
- [ ] 콘솔에서 로그 확인
- [ ] UI에서 성공 메시지 확인

---

## 7. 트랜잭션 처리 및 이벤트

### 학습 목표

이 단계를 완료하면:
- 트랜잭션 receipt를 이해하고 활용할 수 있습니다
- 블록체인 이벤트를 파싱할 수 있습니다
- Transfer 이벤트에서 tokenId를 추출할 수 있습니다
- 사용자에게 민팅 결과를 표시할 수 있습니다

### 개념 설명

#### Transaction Receipt란?

**Receipt = 트랜잭션 실행 결과 영수증**

```javascript
const receipt = await tx.wait();

console.log(receipt);
// {
//   blockNumber: 2,
//   blockHash: "0xabc...",
//   transactionHash: "0x123...",
//   gasUsed: 85000n,
//   logs: [ ... ],  // 이벤트 로그
//   status: 1  // 1 = 성공, 0 = 실패
// }
```

#### 블록체인 이벤트란?

**이벤트 = 스마트 컨트랙트가 발생시키는 로그**

```solidity
// ERC721 표준에 정의된 Transfer 이벤트
event Transfer(address indexed from, address indexed to, uint256 indexed tokenId);

// mint() 실행 시 자동으로 발생
function mint(address to, string memory uri) public {
    uint256 tokenId = _nextTokenId++;
    _safeMint(to, tokenId); // 여기서 Transfer 이벤트 발생!
    _setTokenURI(tokenId, uri);
}
```

**이벤트의 용도:**
1. **모니터링**: 컨트랙트 활동 추적
2. **알림**: 프론트엔드에 상태 변화 전달
3. **저장 비용 절감**: 스토리지 대신 로그 사용
4. **검색**: indexed 파라미터로 효율적 검색

#### Transfer 이벤트 구조

```javascript
event Transfer(address indexed from, address indexed to, uint256 indexed tokenId);

// 민팅 시 발생하는 이벤트:
// from: 0x0000000000000000000000000000000000000000 (영주소 = 새로 생성)
// to: 0x1234... (받는 사람)
// tokenId: 0, 1, 2, ... (새로 생성된 NFT ID)
```

### 실습: 이벤트 파싱 구현

#### 7-1. MintNFT.jsx 수정

`frontend/src/components/MintNFT.jsx` 파일을 다시 엽니다:

```bash
code frontend/src/components/MintNFT.jsx
```

`mintNFT` 함수에서 `// TODO: Transfer 이벤트에서 tokenId 추출` 주석을 찾아 다음 코드로 교체합니다:

```javascript
// 이벤트에서 토큰 ID 추출
// Transfer 이벤트를 찾아서 tokenId를 가져옵니다
const transferEvent = receipt.logs.find((log) => {
  try {
    const parsed = contract.interface.parseLog(log);
    return parsed && parsed.name === 'Transfer';
  } catch (e) {
    return false;
  }
});

if (transferEvent) {
  const parsed = contract.interface.parseLog(transferEvent);
  const tokenId = parsed.args.tokenId;
  setMintedTokenId(Number(tokenId));
  console.log('민팅된 토큰 ID:', Number(tokenId));
}
```

저장합니다.

**코드 설명:**

```javascript
// 1. receipt.logs: 모든 이벤트 로그 배열
console.log('총 로그 개수:', receipt.logs.length);

// 2. 각 로그를 파싱 시도
receipt.logs.forEach((log) => {
  try {
    const parsed = contract.interface.parseLog(log);
    console.log('이벤트 이름:', parsed.name);
    console.log('이벤트 인자:', parsed.args);
  } catch (e) {
    // 다른 컨트랙트의 이벤트일 수 있음
  }
});

// 3. Transfer 이벤트 찾기
const transferEvent = receipt.logs.find((log) => {
  try {
    const parsed = contract.interface.parseLog(log);
    return parsed && parsed.name === 'Transfer';
  } catch (e) {
    return false;
  }
});

// 4. tokenId 추출
if (transferEvent) {
  const parsed = contract.interface.parseLog(transferEvent);

  console.log('from:', parsed.args.from);  // 0x0000... (민팅)
  console.log('to:', parsed.args.to);      // 수신자 주소
  console.log('tokenId:', parsed.args.tokenId); // 토큰 ID

  // BigInt를 Number로 변환
  const tokenId = Number(parsed.args.tokenId);
  console.log('민팅된 NFT ID:', tokenId);
}
```

#### 7-2. 테스트

프론트엔드가 실행 중이라면 자동으로 리로드됩니다. 다시 NFT를 민팅해보세요:

1. "내 주소로" 클릭
2. "샘플 URI" 클릭
3. "NFT 민팅하기" 클릭
4. MetaMask에서 확인

**콘솔 출력 (F12):**
```
Signer 주소: 0xf39Fd...
예상 가스: 85000
트랜잭션 전송 중...
트랜잭션 전송 완료!
TX Hash: 0x...
블록 확인 대기 중...
트랜잭션 확인 완료!
블록 번호: 2
민팅된 토큰 ID: 0    ← 새로 추가됨!
```

**UI 확인:**
```
민팅 성공!
토큰 ID: #0             ← 새로 추가됨!
트랜잭션: 0xabc...
```

#### 7-3. 트랜잭션 라이프사이클 전체 흐름

```
사용자 클릭 "NFT 민팅하기"
   ↓
[1] contract.mint(address, tokenURI) 호출
   ↓
MetaMask 팝업 → 사용자 승인
   ↓
[2] tx 객체 반환
   - tx.hash: 트랜잭션 해시
   - 상태: Pending (멤풀에 대기)
   - UI: "민팅 중..." 표시
   ↓
[3] tx.wait() 호출
   - 블록에 포함될 때까지 대기
   - 로컬: 즉시 (1-2초)
   - Sepolia: 10-30초
   ↓
[4] receipt 반환
   - receipt.blockNumber: 블록 번호
   - receipt.gasUsed: 실제 사용된 가스
   - receipt.logs: 이벤트 로그들
   - 상태: Confirmed
   ↓
[5] 이벤트 파싱
   - receipt.logs에서 Transfer 이벤트 찾기
   - tokenId 추출
   ↓
[6] UI 업데이트
   - "민팅 성공!"
   - "토큰 ID: #0"
   - Etherscan 링크
   ↓
[7] 부모 컴포넌트에 알림
   - onMintSuccess() 호출
   - totalSupply 새로고침
```

### 체크포인트 ✅

- [ ] 이벤트 파싱 코드 추가
- [ ] NFT 민팅 재테스트
- [ ] 콘솔에서 "민팅된 토큰 ID: 0" 확인
- [ ] UI에서 "토큰 ID: #0" 표시 확인
- [ ] totalSupply가 1 증가 확인

---

## 8. NFT 갤러리 구현

### 학습 목표

이 단계를 완료하면:
- 사용자가 소유한 모든 NFT를 조회할 수 있습니다
- 커스텀 헬퍼 함수 `tokensOfOwner()`를 사용할 수 있습니다
- NFT 메타데이터를 표시할 수 있습니다
- OpenSea에서 NFT를 확인할 수 있습니다

### 개념 설명

#### ERC721Enumerable vs 커스텀 헬퍼 함수

**표준 방식 (ERC721Enumerable):**

```solidity
// 표준 함수들
function balanceOf(address owner) public view returns (uint256)
function tokenOfOwnerByIndex(address owner, uint256 index) public view returns (uint256)
```

```javascript
// 프론트엔드에서 사용 (비효율적)
const balance = await contract.balanceOf(account);
const tokenIds = [];

for (let i = 0; i < balance; i++) {
  const tokenId = await contract.tokenOfOwnerByIndex(account, i);
  tokenIds.push(tokenId);
}
// 문제: N개의 NFT가 있으면 N번 호출!
```

**커스텀 헬퍼 함수 (우리 컨트랙트):**

```solidity
// Step8_CompleteERC721.sol (73-82번째 줄)
function tokensOfOwner(address owner) external view returns (uint256[] memory) {
    uint256 tokenCount = balanceOf(owner);
    uint256[] memory tokenIds = new uint256[](tokenCount);

    for (uint256 i = 0; i < tokenCount; i++) {
        tokenIds[i] = tokenOfOwnerByIndex(owner, i);
    }

    return tokenIds;
}
```

```javascript
// 프론트엔드에서 사용 (효율적!)
const tokenIds = await contract.tokensOfOwner(account);
// 한 번 호출로 모든 토큰 ID 가져옴!
```

**성능 비교:**
```
10개 NFT 조회:
- 표준 방식: 11번 호출 (balanceOf 1번 + tokenOfOwnerByIndex 10번)
- 커스텀 방식: 1번 호출 (tokensOfOwner 1번)

속도:
- 표준 방식: ~3-5초
- 커스텀 방식: ~0.5초
```

### 실습: MyNFTs 컴포넌트 생성

#### 8-1. MyNFTs.jsx 파일 생성

터미널에서:

```bash
# 파일 생성
touch frontend/src/components/MyNFTs.jsx

# 파일 열기
code frontend/src/components/MyNFTs.jsx
```

#### 8-2. 전체 코드 작성

다음 코드를 복사하여 붙여넣습니다:

```javascript
import React, { useState, useEffect } from 'react';
import { ethers } from 'ethers';
import { CONTRACT_ADDRESS, CONTRACT_ABI, getOpenSeaLink } from '../utils/contractConfig';
import { ensureCorrectNetwork } from '../utils/networkHelper';

function MyNFTs({ account, refreshTrigger }) {
  const [myTokens, setMyTokens] = useState([]);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState('');

  // account나 refreshTrigger가 바뀌면 자동으로 NFT 목록 새로고침
  useEffect(() => {
    if (account && window.ethereum) {
      fetchMyNFTs();
    } else {
      setMyTokens([]);
    }
    // eslint-disable-next-line
  }, [account, refreshTrigger]);

  // NFT 조회 함수
  const fetchMyNFTs = async () => {
    try {
      setIsLoading(true);
      setError('');

      // 올바른 네트워크로 전환 확인
      await ensureCorrectNetwork();

      // Provider 생성 (READ만 필요하므로 Signer 불필요)
      const provider = new ethers.BrowserProvider(window.ethereum);
      const contract = new ethers.Contract(
        CONTRACT_ADDRESS,
        CONTRACT_ABI,
        provider // Provider만으로 충분 (READ 작업)
      );

      // tokensOfOwner() 함수 호출
      // 한 번에 모든 tokenId 가져옴!
      const tokenIds = await contract.tokensOfOwner(account);

      console.log('내 NFT 토큰 IDs:', tokenIds);

      // 각 토큰의 URI 가져오기 (병렬 처리)
      const tokensWithURI = await Promise.all(
        tokenIds.map(async (tokenId) => {
          try {
            const uri = await contract.tokenURI(tokenId);
            return {
              tokenId: Number(tokenId),
              uri: uri,
            };
          } catch (err) {
            console.error(`토큰 ${tokenId} URI 조회 실패:`, err);
            return {
              tokenId: Number(tokenId),
              uri: null,
            };
          }
        })
      );

      setMyTokens(tokensWithURI);

    } catch (err) {
      console.error('NFT 목록 조회 오류:', err);
      setError('NFT 목록을 불러오는데 실패했습니다.');
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div className="card">
      <h2>내 NFT 컬렉션</h2>

      {!account ? (
        <div className="info-box warning">
          <p>지갑을 먼저 연결해주세요.</p>
        </div>
      ) : error ? (
        <div className="info-box error">
          <p>{error}</p>
          <button onClick={fetchMyNFTs}>다시 시도</button>
        </div>
      ) : isLoading ? (
        <div className="info-box">
          <p>NFT 목록 불러오는 중...</p>
        </div>
      ) : myTokens.length === 0 ? (
        <div className="info-box">
          <p>아직 소유한 NFT가 없습니다.</p>
        </div>
      ) : (
        <div>
          <div className="info-box">
            <p>총 {myTokens.length}개의 NFT를 소유하고 있습니다.</p>
          </div>

          {/* NFT 그리드 */}
          <div className="nft-grid">
            {myTokens.map((token) => (
              <div key={token.tokenId} className="nft-card">
                {/* NFT ID 표시 */}
                <div style={{
                  background: 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)',
                  padding: '40px 20px',
                  fontSize: '32px',
                  fontWeight: 'bold',
                  color: '#fff',
                  borderRadius: '8px 8px 0 0',
                  textAlign: 'center'
                }}>
                  #{token.tokenId}
                </div>

                <div style={{ padding: '15px' }}>
                  <p style={{ marginBottom: '8px', fontWeight: 'bold' }}>Token #{token.tokenId}</p>

                  {/* URI 미리보기 */}
                  {token.uri && (
                    <p style={{
                      fontSize: '12px',
                      wordBreak: 'break-all',
                      color: '#888',
                      marginBottom: '12px'
                    }}>
                      {token.uri.substring(0, 30)}...
                    </p>
                  )}

                  {/* OpenSea 링크 */}
                  {getOpenSeaLink(token.tokenId) && (
                    <a
                      href={getOpenSeaLink(token.tokenId)}
                      target="_blank"
                      rel="noreferrer"
                      style={{
                        display: 'inline-block',
                        padding: '8px 16px',
                        background: '#2081e2',
                        color: '#fff',
                        borderRadius: '4px',
                        textDecoration: 'none',
                        fontSize: '14px'
                      }}
                    >
                      OpenSea에서 보기
                    </a>
                  )}
                </div>
              </div>
            ))}
          </div>

          <button onClick={fetchMyNFTs} style={{ marginTop: '20px' }}>
            새로고침
          </button>
        </div>
      )}
    </div>
  );
}

export default MyNFTs;
```

저장합니다.

**코드 설명:**

```javascript
// 1. tokensOfOwner() 호출 - 한 번에 모든 ID
const tokenIds = await contract.tokensOfOwner(account);
// 결과: [0n, 1n, 2n] (BigInt 배열)

// 2. 각 토큰의 URI 가져오기
const uri = await contract.tokenURI(tokenId);
// 결과: "https://ipfs.io/ipfs/..."

// 3. Promise.all로 병렬 처리
const tokensWithURI = await Promise.all(
  tokenIds.map(async (tokenId) => {
    const uri = await contract.tokenURI(tokenId);
    return { tokenId: Number(tokenId), uri };
  })
);
// 모든 tokenURI를 동시에 가져옴 (빠름!)
```

#### 8-3. App.js에 MyNFTs 추가

`frontend/src/App.js` 파일을 엽니다:

```bash
code frontend/src/App.js
```

**1단계:** Import 추가 (파일 상단)

```javascript
import MyNFTs from './components/MyNFTs';
```

**2단계:** MyNFTs 컴포넌트 사용 (MintNFT 아래)

기존:
```javascript
<>
  {/* totalSupply 표시 */}
  <TotalSupply account={account} refreshTrigger={refreshTrigger} />

  {/* NFT 민팅 */}
  <MintNFT account={account} onMintSuccess={handleMintSuccess} />
</>
```

수정 후:
```javascript
<>
  {/* totalSupply 표시 */}
  <TotalSupply account={account} refreshTrigger={refreshTrigger} />

  {/* NFT 민팅 */}
  <MintNFT account={account} onMintSuccess={handleMintSuccess} />

  {/* 내 NFT 컬렉션 */}
  <MyNFTs account={account} refreshTrigger={refreshTrigger} />
</>
```

저장합니다.

#### 8-4. 테스트

브라우저에서 http://localhost:13000 을 다시 로드합니다.

**결과:**
- 이미 민팅한 NFT들이 "내 NFT 컬렉션"에 표시됨
- 각 NFT 카드에 Token ID와 URI 미리보기 표시

**새 NFT 민팅 후:**
1. "내 주소로" 클릭
2. "샘플 URI" 클릭
3. "NFT 민팅하기" 클릭
4. MetaMask에서 확인

**자동 새로고침 확인:**
- 민팅 성공 후 자동으로 "내 NFT 컬렉션"에 새 NFT 추가됨
- totalSupply도 자동으로 증가

### 병렬 처리 vs 순차 처리

#### 순차 처리 (느림)

```javascript
const tokensWithURI = [];
for (const tokenId of tokenIds) {
  const uri = await contract.tokenURI(tokenId);
  tokensWithURI.push({ tokenId, uri });
}
// 10개 NFT: 약 3-5초
```

#### 병렬 처리 (빠름)

```javascript
const tokensWithURI = await Promise.all(
  tokenIds.map(async (tokenId) => {
    const uri = await contract.tokenURI(tokenId);
    return { tokenId, uri };
  })
);
// 10개 NFT: 약 0.5-1초
```

### 체크포인트 ✅

- [ ] MyNFTs.jsx 파일 생성
- [ ] 전체 코드 작성 완료
- [ ] App.js에 MyNFTs 컴포넌트 추가
- [ ] 기존 NFT가 갤러리에 표시됨
- [ ] 새 NFT 민팅 후 자동 추가 확인
- [ ] "새로고침" 버튼 작동 확인
- [ ] 콘솔에서 "내 NFT 토큰 IDs" 로그 확인

---

## 최종 체크리스트 ✅

### 기능 테스트

Week 2의 모든 기능이 정상 작동하는지 확인하세요:

**환경 설정:**
- [ ] Hardhat 로컬 노드 실행 중
- [ ] 컨트랙트 배포 완료
- [ ] `.env.local` 파일에 컨트랙트 주소 설정됨
- [ ] 프론트엔드 실행 중
- [ ] MetaMask 네트워크 설정 (Localhost 8545)
- [ ] 테스트 계정 import

**기본 기능 (Week 1):**
- [ ] MetaMask 지갑 연결
- [ ] 계정 주소 표시
- [ ] totalSupply 읽기

**민팅 기능 (Week 2):**
- [ ] MintNFT.jsx 파일 존재
- [ ] "내 주소로" 버튼 작동
- [ ] "샘플 URI" 버튼 작동
- [ ] NFT 민팅 버튼 클릭
- [ ] MetaMask 팝업에서 가스비 확인
- [ ] 트랜잭션 승인
- [ ] "민팅 중..." 로딩 표시
- [ ] 민팅 완료 후 Token ID 표시
- [ ] 트랜잭션 해시 링크 작동

**갤러리 기능:**
- [ ] MyNFTs.jsx 파일 존재
- [ ] 민팅 후 자동으로 갤러리 업데이트
- [ ] NFT 카드 그리드 표시
- [ ] Token ID 정확함
- [ ] URI 미리보기 표시
- [ ] "새로고침" 버튼 작동

**에러 처리:**
- [ ] 지갑 연결 안된 상태에서 안내 메시지
- [ ] 잘못된 주소 입력 시 에러 메시지
- [ ] 트랜잭션 거부 시 적절한 메시지
- [ ] onlyOwner 제한 에러 메시지

### 코드 이해도 체크

다음 질문에 답할 수 있나요?

**Signer:**
- [ ] Provider와 Signer의 차이를 설명할 수 있나요?
- [ ] 왜 필요할 때마다 Signer를 생성하나요?
- [ ] Signer는 어떻게 생성하나요?

**트랜잭션:**
- [ ] 트랜잭션이란 무엇인가요?
- [ ] 가스비가 왜 필요한가요?
- [ ] `tx.wait()`의 역할은 무엇인가요?

**이벤트:**
- [ ] 블록체인 이벤트란 무엇인가요?
- [ ] Transfer 이벤트의 구조를 설명할 수 있나요?
- [ ] 이벤트 로그를 어떻게 파싱하나요?

**NFT 조회:**
- [ ] `tokensOfOwner()`의 장점은 무엇인가요?
- [ ] 왜 READ 작업에는 Provider만 사용하나요?
- [ ] `Promise.all`을 사용하는 이유는?

---

## 문제 해결

### 자주 발생하는 에러 및 해결방법

#### 1. "insufficient funds for gas"

**원인:** 지갑에 ETH가 부족합니다.

**해결:**
```bash
# 로컬 개발 환경
# Hardhat 로컬 노드는 자동으로 테스트 ETH를 제공합니다.
# MetaMask의 계정이 Hardhat의 기본 계정인지 확인하세요.

# Sepolia 테스트넷
# Faucet에서 테스트 ETH 받기:
# https://sepoliafaucet.com/
# https://www.alchemy.com/faucets/ethereum-sepolia
```

#### 2. "OwnableUnauthorizedAccount"

**원인:** 컨트랙트 소유자가 아닌 계정으로 민팅 시도

**해결:**
1. 컨트랙트를 배포한 계정 확인
2. MetaMask에서 해당 계정(Hardhat #0)으로 전환
3. 또는 다른 계정으로 다시 배포

#### 3. "Cannot find module './components/MintNFT'"

**원인:** MintNFT.jsx 파일이 없거나 위치가 잘못됨

**해결:**
1. `frontend/src/components/MintNFT.jsx` 파일이 존재하는지 확인
2. 파일 이름이 정확한지 확인 (대소문자 구분)
3. 프론트엔드 서버 재시작

#### 4. "network mismatch"

**원인:** MetaMask 네트워크와 설정이 다름

**해결:**
```javascript
// .env.local 확인
REACT_APP_NETWORK_ID=31337  // Localhost
REACT_APP_NETWORK_NAME=localhost

// MetaMask에서 "Localhost 8545" 네트워크 선택
```

#### 5. NFT가 갤러리에 표시 안됨

**원인:** 네트워크 오류 또는 컨트랙트 이슈

**해결:**
1. 브라우저 콘솔(F12)에서 에러 확인
2. "새로고침" 버튼 클릭
3. 페이지 새로고침 (F5)
4. MyNFTs.jsx 파일이 정확히 생성되었는지 확인

#### 6. "npm run node: command not found"

**원인:** package.json에 스크립트가 없음

**해결:**
섹션 0-2로 돌아가서 package.json 스크립트 추가

#### 7. Hardhat 노드 포트 오류

**원인:** hardhat.config.js의 포트가 잘못됨

**해결:**
섹션 0-3으로 돌아가서 포트를 8545로 수정

---

## 다음 단계

### 축하합니다!

Week 2를 완료했습니다! 이제 여러분은:
- ✅ Provider와 Signer의 차이를 이해합니다
- ✅ 블록체인에 데이터를 쓸 수 있습니다
- ✅ NFT를 민팅할 수 있습니다
- ✅ 트랜잭션을 관리할 수 있습니다
- ✅ 이벤트를 파싱할 수 있습니다
- ✅ 완전한 NFT DApp을 만들었습니다!

### 추가 실습

다음을 시도해보세요:

1. **여러 개 민팅**
   - 다양한 tokenURI로 여러 NFT 민팅
   - totalSupply 증가 확인

2. **다른 계정 테스트**
   - Hardhat Account #1 import
   - Owner가 아닌 계정으로 민팅 시도
   - 예상: "OwnableUnauthorizedAccount" 에러

3. **코드 수정**
   - 콘솔 로그 추가
   - 에러 메시지 커스터마이징
   - UI 스타일 변경

4. **실시간 이벤트 리스닝**
   ```javascript
   // MintNFT.jsx에 추가
   useEffect(() => {
     const provider = new ethers.BrowserProvider(window.ethereum);
     const contract = new ethers.Contract(ADDRESS, ABI, provider);

     contract.on('Transfer', (from, to, tokenId) => {
       console.log(`NFT #${tokenId} transferred!`);
     });

     return () => contract.removeAllListeners();
   }, []);
   ```

---

**Happy Coding! 🚀**

이제 여러분은 완전한 NFT DApp 개발자입니다!
