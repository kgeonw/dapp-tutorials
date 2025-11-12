# 10주차 단계별 실습 가이드

## 학습 목표
10주차에서는 React 프론트엔드를 단계별로 구축하며 블록체인 READ 작업을 학습합니다.

---

##  사전 준비: 프로젝트 생성 및 설정

### 0-1: 프로젝트 디렉토리 생성

```bash
# 프로젝트 디렉토리 생성
mkdir edu_dan_nft
cd edu_dan_nft
```

### 0-2: Hardhat 프로젝트 초기화

```bash
# npm 프로젝트 초기화
npm init -y

# Hardhat v2 및 필요한 패키지 설치 (버전 고정)
npm install --save-dev hardhat@^2.22.0 @nomicfoundation/hardhat-toolbox@^5.0.0

# Hardhat 초기화
npx hardhat init
```

**프롬프트 응답**:
- "What do you want to do?" → `Create a JavaScript project` 선택 (화살표 키로 이동 후 Enter)
- "Hardhat project root:" → 그냥 Enter (현재 디렉토리 사용)
- "Do you want to add a .gitignore?" → Enter (Yes)


**중요**:
- Hardhat v3를 설치한 경우 의존성 충돌이 발생할 수 있습니다
- 위 명령어는 안정적인 Hardhat v2를 설치합니다

### 0-3: 필요한 추가 패키지 설치

```bash
# Ethers.js v6 (프론트엔드용)
npm install ethers

# 개발 도구
npm install --save-dev @openzeppelin/contracts
```

### 0-4: Hardhat 설정 파일 수정

**파일**: `hardhat.config.js`

```javascript
require("@nomicfoundation/hardhat-toolbox");

module.exports = {
  solidity: "0.8.20",
  networks: {
    hardhat: {
      chainId: 31337
    },
    localhost: {
      url: "http://127.0.0.1:8545",
      chainId: 31337
    }
  }
};
```

### 0-5: 스마트 컨트랙트 작성

**파일 생성**: `contracts/Step8_CompleteERC721.sol`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract Step8_CompleteERC721 is ERC721, ERC721URIStorage, ERC721Enumerable, Ownable {
    uint256 private _nextTokenId;

    constructor() ERC721("EducationNFT", "EDUNFT") Ownable(msg.sender) {}

    function mint(address to, string memory uri) public onlyOwner {
        uint256 tokenId = _nextTokenId++;
        _safeMint(to, tokenId);
        _setTokenURI(tokenId, uri);
    }

    function tokensOfOwner(address owner) external view returns (uint256[] memory) {
        uint256 tokenCount = balanceOf(owner);
        uint256[] memory tokenIds = new uint256[](tokenCount);

        for (uint256 i = 0; i < tokenCount; i++) {
            tokenIds[i] = tokenOfOwnerByIndex(owner, i);
        }

        return tokenIds;
    }

    // Required overrides
    function _update(address to, uint256 tokenId, address auth)
        internal
        override(ERC721, ERC721Enumerable)
        returns (address)
    {
        return super._update(to, tokenId, auth);
    }

    function _increaseBalance(address account, uint128 value)
        internal
        override(ERC721, ERC721Enumerable)
    {
        super._increaseBalance(account, value);
    }

    function tokenURI(uint256 tokenId)
        public
        view
        override(ERC721, ERC721URIStorage)
        returns (string memory)
    {
        return super.tokenURI(tokenId);
    }

    function supportsInterface(bytes4 interfaceId)
        public
        view
        override(ERC721, ERC721Enumerable, ERC721URIStorage)
        returns (bool)
    {
        return super.supportsInterface(interfaceId);
    }
}
```

### 0-6: 배포 스크립트 작성

```bash
# 파일 생성
mkdir scripts
touch scripts/deploy.js
```


**파일 생성**: `scripts/deploy.js`

```javascript
const hre = require("hardhat");

async function main() {
  console.log("컨트랙트 배포 중...");

  const Step8 = await hre.ethers.getContractFactory("Step8_CompleteERC721");
  const contract = await Step8.deploy();

  await contract.waitForDeployment();

  const address = await contract.getAddress();
  console.log(`Step8_CompleteERC721 배포 완료: ${address}`);

  // 배포자 주소 출력
  const [deployer] = await hre.ethers.getSigners();
  console.log(`배포자 주소: ${deployer.address}`);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

### 0-7: Hardhat 로컬 노드 실행

**터미널 1번** (계속 실행 유지):
```bash
npx hardhat node --port 18545
```

출력 예시:
```
Started HTTP and WebSocket JSON-RPC server at http://127.0.0.1:8545/

Accounts
========
Account #0: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 (10000 ETH)
Private Key: 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
...
```

**중요**: 이 터미널은 닫지 마세요!

### 0-8: 컨트랙트 배포

**터미널 2번** (새 터미널 열기):
```bash
npx hardhat run scripts/deploy.js --network localhost
```

출력 예시:
```
컨트랙트 배포 중...
Step8_CompleteERC721 배포 완료: 0x5FbDB2315678afecb367f032d93F642f64180aa3
배포자 주소: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
```

**중요**: `0x5FbDB2315678afecb367f032d93F642f64180aa3` 주소를 복사해두세요!

### 0-9: React 프로젝트 생성

```bash
# 프로젝트 루트에서 실행
npx create-react-app frontend
cd frontend
```

### 0-10: 프론트엔드 패키지 설치

```bash
# Ethers.js 설치
npm install ethers
```

### 0-11: 프론트엔드 디렉토리 구조 생성

```bash
# frontend 디렉토리에서 실행
mkdir -p src/components
mkdir -p src/utils
mkdir -p src/abis
```

### 0-12: 컨트랙트 ABI 복사

```bash
# 프로젝트 루트로 이동
cd ..

# ABI 파일 복사
cp artifacts/contracts/Step8_CompleteERC721.sol/Step8_CompleteERC721.json frontend/src/abis/
```

### 0-13: 환경 변수 파일 생성

```bash
# 파일 생성
touch frontend/.env.local
```

```env
REACT_APP_CONTRACT_ADDRESS=0x5FbDB2315678afecb367f032d93F642f64180aa3
REACT_APP_NETWORK_ID=31337
REACT_APP_NETWORK_NAME=localhost
```

**주의**: `REACT_APP_CONTRACT_ADDRESS`에는 0-8 단계에서 복사한 주소를 입력하세요!

### 0-14: 기본 스타일 파일 생성

**파일**: `frontend/src/App.css`

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.App {
  min-height: 100vh;
  background: #1a1d2e;
  padding: 30px;
}

/* 헤더 스타일 - 왼쪽 제목, 오른쪽 버튼 */
.app-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
  padding: 0;
}

.app-header h1 {
  font-size: 32px;
  margin: 0;
  color: #7289da;
  font-weight: 600;
}

.app-header p {
  display: none;
}

.app-header .header-info {
  display: none;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
}

/* 카드 스타일 - 투명한 배경 */
.card {
  background: transparent;
  border-radius: 0;
  padding: 0;
  margin-bottom: 30px;
  box-shadow: none;
}

.card h2 {
  margin-bottom: 20px;
  color: #e0e0e0;
  font-size: 18px;
  font-weight: 500;
}

.card h3 {
  margin-bottom: 12px;
  color: #e0e0e0;
  font-size: 16px;
}

/* 일반 텍스트 색상 */
.card p,
.card label,
.card small {
  color: #b0b0b0;
}

/* 버튼 스타일 */
button {
  background: #7289da;
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 6px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.2s;
  font-family: inherit;
}

button:hover:not(:disabled) {
  background: #5b6eae;
}

button:active:not(:disabled) {
  background: #4e5d94;
}

button:disabled {
  background: #4a4d5e;
  cursor: not-allowed;
  opacity: 0.5;
}

/* 입력 필드 */
input,
textarea {
  width: 100%;
  padding: 12px;
  border: 1px solid #2c2f48;
  border-radius: 6px;
  font-size: 14px;
  margin-bottom: 12px;
  background: #252837;
  color: #ffffff;
  font-family: inherit;
}

input:focus,
textarea:focus {
  outline: none;
  border-color: #7289da;
  background: #2a2d42;
}

input::placeholder,
textarea::placeholder {
  color: #72767d;
}

/* 정보 박스 */
.info-box {
  background: #c23f38;
  padding: 16px;
  border-radius: 6px;
  margin-bottom: 16px;
  border-left: none;
}

.info-box p {
  margin: 4px 0;
  color: #ffffff;
  line-height: 1.6;
  font-size: 14px;
}

.info-box strong {
  color: #ffffff;
}

.info-box ul {
  margin: 8px 0 8px 20px;
}

.info-box li {
  margin: 4px 0;
  line-height: 1.6;
}

.info-box code {
  background: rgba(0, 0, 0, 0.2);
  padding: 2px 6px;
  border-radius: 4px;
  font-family: 'Courier New', monospace;
  font-size: 13px;
  color: #ffffff;
}

.error {
  background: #c23f38;
}

.success {
  background: #43b581;
}

.warning {
  background: #faa61a;
}

/* 로딩 스피너 */
.loading {
  display: inline-block;
  width: 20px;
  height: 20px;
  border: 3px solid rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  border-top-color: white;
  animation: spin 1s ease-in-out infinite;
  vertical-align: middle;
  margin-right: 8px;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

/* NFT 그리드 */
.nft-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 16px;
  margin-top: 16px;
}

.nft-card {
  background: #252837;
  padding: 16px;
  border-radius: 6px;
  text-align: center;
  transition: transform 0.2s;
  border: 1px solid #2c2f48;
}

.nft-card:hover {
  transform: translateY(-4px);
  border-color: #7289da;
}

.nft-card p {
  color: #ffffff;
}

/* 트랜잭션 상태 */
.tx-status {
  margin-top: 16px;
  padding: 16px;
  background: #43b581;
  border-radius: 6px;
  border-left: none;
}

.tx-status h3 {
  color: #ffffff;
  margin-bottom: 12px;
}

.tx-status p {
  margin: 8px 0;
  color: #ffffff;
}

.tx-status a {
  color: #7289da;
  text-decoration: none;
  font-weight: 600;
}

.tx-status a:hover {
  text-decoration: underline;
}

.tx-status code {
  background: rgba(0, 0, 0, 0.2);
  padding: 4px 8px;
  border-radius: 4px;
  font-family: 'Courier New', monospace;
  font-size: 13px;
  color: #ffffff;
}

/* 푸터 */
footer {
  text-align: center;
  padding: 40px 20px;
  color: #72767d;
  font-size: 14px;
}

footer p {
  opacity: 0.8;
}

/* 구분선 숨기기 */
.container > div[style*="dashed"] {
  display: none;
}

/* 반응형 디자인 */
@media (max-width: 768px) {
  .app-header h1 {
    font-size: 24px;
  }

  .card {
    padding: 0;
  }

  .nft-grid {
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  }
}
```

### 0-15: contractConfig.js 유틸리티 생성

**파일 생성**: `frontend/src/utils/contractConfig.js`

```javascript
import contractABI from '../abis/Step8_CompleteERC721.json';

// 환경 변수에서 컨트랙트 주소 가져오기
export const CONTRACT_ADDRESS = process.env.REACT_APP_CONTRACT_ADDRESS;
export const NETWORK_ID = parseInt(process.env.REACT_APP_NETWORK_ID || '31337');
export const NETWORK_NAME = process.env.REACT_APP_NETWORK_NAME || 'localhost';

// 컨트랙트 ABI
export const CONTRACT_ABI = contractABI.abi;

// 네트워크 설정
export const NETWORK_CONFIG = {
  31337: {
    name: 'Localhost',
    rpcUrl: 'http://127.0.0.1:8545',
    chainId: 31337,
    nativeCurrency: {
      name: 'ETH',
      symbol: 'ETH',
      decimals: 18
    }
  }
};

// Etherscan 링크 생성 (로컬 네트워크는 null 반환)
export function getEtherscanLink(type, data) {
  if (NETWORK_ID === 31337) {
    return null; // 로컬 네트워크는 Etherscan이 없음
  }
  return null;
}

// OpenSea 링크 생성 (로컬 네트워크는 null 반환)
export function getOpenSeaLink(tokenId) {
  if (NETWORK_ID === 31337) {
    return null; // 로컬 네트워크는 OpenSea가 없음
  }
  return null;
}
```

### 0-16: networkHelper.js 유틸리티 생성

**파일 생성**: `frontend/src/utils/networkHelper.js`

```javascript
import { NETWORK_CONFIG, NETWORK_ID, NETWORK_NAME } from './contractConfig';

// 현재 네트워크가 올바른지 확인
export async function checkNetwork() {
  if (!window.ethereum) {
    throw new Error('MetaMask가 설치되지 않았습니다');
  }

  const chainId = await window.ethereum.request({ method: 'eth_chainId' });
  const currentChainId = parseInt(chainId, 16);

  return {
    isCorrect: currentChainId === NETWORK_ID,
    currentChainId,
    expectedChainId: NETWORK_ID,
    expectedNetworkName: NETWORK_NAME,
  };
}

// 네트워크 확인 (전환 없이 에러만 발생)
export async function ensureCorrectNetwork() {
  const networkInfo = await checkNetwork();

  if (!networkInfo.isCorrect) {
    const targetNetwork = NETWORK_CONFIG[NETWORK_ID];
    throw new Error(
      `잘못된 네트워크입니다. MetaMask에서 "${targetNetwork.name}" 네트워크로 전환해주세요. ` +
      `(현재: Chain ID ${networkInfo.currentChainId}, 필요: Chain ID ${networkInfo.expectedChainId})`
    );
  }

  return true;
}
```

### 0-17: React 개발 서버 실행

**터미널 3번** (새 터미널 열기):
```bash
cd frontend
npm start
```

브라우저가 자동으로 `http://localhost:3000`을 엽니다.

### ✅ 사전 준비 체크포인트

완료했는지 확인하세요:
- [ ] Hardhat 로컬 노드가 터미널 1에서 실행 중
- [ ] 컨트랙트가 배포되었고 주소를 복사했음
- [ ] `frontend/.env.local` 파일에 컨트랙트 주소 입력
- [ ] React 앱이 브라우저에서 실행 중
- [ ] MetaMask가 설치되어 있음

### 💡 MetaMask 네트워크 추가

MetaMask에서 Localhost 네트워크를 수동으로 추가해야 합니다:

1. MetaMask 열기
2. 네트워크 드롭다운 클릭
3. "네트워크 추가" 클릭
4. "수동으로 네트워크 추가" 클릭
5. 다음 정보 입력:
   - 네트워크 이름: `Localhost 8545`
   - 새 RPC URL: `http://127.0.0.1:8545`
   - 체인 ID: `31337`
   - 통화 기호: `ETH`
6. "저장" 클릭

### 💡 MetaMask에 테스트 계정 추가

Hardhat이 제공하는 테스트 계정을 MetaMask에 추가합니다:

1. MetaMask에서 "계정 가져오기" 클릭
2. 개인 키 입력:
   ```
   0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
   ```
3. "가져오기" 클릭

이 계정은 10,000 ETH를 가지고 있으며, 컨트랙트를 배포한 소유자 계정입니다.

---

##  준비 단계: 프로젝트 구조 이해

### 현재 프로젝트 구조
```
frontend/src/
├── components/           # React 컴포넌트
│   ├── ConnectWallet.jsx  # 지갑 연결 (이번 단계에서 만듭니다)
│   └── TotalSupply.jsx    # totalSupply 조회 (이번 단계에서 만듭니다)
├── utils/                # 유틸리티
│   └── contractConfig.js  # 컨트랙트 설정 (이미 완성됨)
├── abis/                 # 컨트랙트 ABI
│   └── Step8_CompleteERC721.json  # (이미 복사됨)
├── App.js               # 메인 앱 (이번 단계에서 수정합니다)
└── index.js             # 엔트리 포인트 (수정하지 않습니다)
```

### 학습 흐름
```
Step 1: 기본 UI 구조
  ↓
Step 2: MetaMask 지갑 연결
  ↓
Step 3: Provider 설정
  ↓
Step 4: 컨트랙트 읽기 (totalSupply)
```

---

##  단계 1: 기본 UI 구조 만들기

### 목표
- App.js의 기본 구조 이해
- 헤더와 컨테이너 레이아웃 만들기

### 파일: `App.js`

```javascript
import React from 'react';
import './App.css';

function App() {
  return (
    <div className="App">
      {/* 헤더 */}
      <header className="app-header">
        <h1>기념 NFT 민팅 DApp</h1>
      </header>

      {/* 메인 컨텐츠 */}
      <div className="container">
        <div className="card">
          <h2>환영합니다!</h2>
          <p>1주차 실습을 시작합니다.</p>
        </div>
      </div>
    </div>
  );
}

export default App;
```

### ✅ 체크포인트
- [ ] 브라우저에서 헤더가 보입니까?
- [ ] 보라색 그라데이션 배경이 보입니까?
- [ ] "환영합니다!" 카드가 보입니까?

---

##  단계 2: MetaMask 지갑 연결

### 목표
- `window.ethereum` 객체 이해
- `eth_requestAccounts` 메서드 사용
- React State로 계정 관리

### 2-1: ConnectWallet 컴포넌트 생성

**파일 생성**: `components/ConnectWallet.jsx`

```javascript
import React, { useState } from 'react';

function ConnectWallet({ onAccountChange }) {
  const [account, setAccount] = useState('');
  const [error, setError] = useState('');

  // TODO 2-1: 지갑 연결 함수 작성
  const connectWallet = async () => {
    try {
      // 1단계: MetaMask 확인
      if (!window.ethereum) {
        throw new Error('MetaMask를 설치해주세요!');
      }

      // 2단계: 계정 연결 요청
      const accounts = await window.ethereum.request({
        method: 'eth_requestAccounts'
      });

      // 3단계: 첫 번째 계정 저장
      const account = accounts[0];
      setAccount(account);
      onAccountChange(account);

    } catch (err) {
      setError(err.message);
    }
  };

  return (
    <div>
      {!account ? (
        <button onClick={connectWallet} style={{
          background: '#7289da',
          color: 'white',
          border: 'none',
          padding: '10px 20px',
          borderRadius: '8px',
          cursor: 'pointer',
          fontSize: '14px',
          fontWeight: '600'
        }}>
          지갑 연동하기
        </button>
      ) : (
        <div style={{
          background: '#43b58120',
          border: '1px solid #43b581',
          padding: '10px 15px',
          borderRadius: '8px',
          fontSize: '12px',
          color: '#43b581'
        }}>
          <strong>연결됨:</strong> {account.slice(0, 6)}...{account.slice(-4)}
        </div>
      )}

      {error && (
        <div style={{
          background: '#f0444420',
          border: '1px solid #f04444',
          padding: '10px 15px',
          borderRadius: '8px',
          fontSize: '12px',
          color: '#f04444',
          marginTop: '10px'
        }}>
          {error}
        </div>
      )}
    </div>
  );
}

export default ConnectWallet;
```

**주요 변경사항:**
- `card` 클래스 제거: 헤더에 버튼만 깔끔하게 표시
- 연결 후 주소를 짧게 표시 (0x1234...abcd 형식)
- 인라인 스타일로 헤더에 맞는 디자인 적용

### 2-2: App.js에 ConnectWallet 추가

이제 App.js를 수정하여 **헤더에 ConnectWallet 버튼**을 배치하고, **지갑 연결 전/후에 다른 UI**를 표시합니다.

```javascript
import React, { useState } from 'react';
import './App.css';
import ConnectWallet from './components/ConnectWallet';

function App() {
  const [account, setAccount] = useState('');

  const handleAccountChange = (newAccount) => {
    setAccount(newAccount);
    console.log('연결된 계정:', newAccount);
  };

  return (
    <div className="App">
      {/* 헤더: 왼쪽 제목, 오른쪽 지갑 연결 버튼 */}
      <header className="app-header">
        <h1>기념 NFT 민팅 DApp</h1>
        <ConnectWallet onAccountChange={handleAccountChange} />
      </header>

      {/* 메인 컨텐츠 */}
      <div className="container">
        {!account ? (
          // 지갑 연결 전: 안내 카드 표시
          <div className="card">
            <h2>환영합니다!</h2>
            <div className="info-box" style={{
              background: '#7289da20',
              border: '1px solid #7289da',
              textAlign: 'center',
              padding: '40px 20px'
            }}>
              <p style={{ fontSize: '18px', marginBottom: '20px', color: '#e0e0e0' }}>
                NFT DApp을 사용하려면 먼저 지갑을 연결해주세요
              </p>
              <p style={{ fontSize: '14px', color: '#b0b0b0' }}>
                오른쪽 상단의 "지갑 연동하기" 버튼을 클릭하세요
              </p>
            </div>

            <div className="info-box" style={{
              marginTop: '20px',
              background: '#43b58120',
              border: '1px solid #43b581'
            }}>
              <p style={{ color: '#e0e0e0' }}><strong>시작하기 전에:</strong></p>
              <ul style={{ marginLeft: '20px', marginTop: '12px', lineHeight: '1.8', color: '#b0b0b0' }}>
                <li>MetaMask 브라우저 확장 프로그램이 설치되어 있어야 합니다</li>
                <li>로컬 Hardhat 네트워크(localhost:18545)에 연결되어야 합니다</li>
                <li>테스트용 ETH가 있는 계정을 사용하세요</li>
              </ul>
            </div>
          </div>
        ) : (
          // 지갑 연결 후: 기능 컴포넌트들이 여기에 추가됩니다
          <div className="card">
            <h2>지갑이 연결되었습니다!</h2>
            <p>다음 단계에서 기능을 추가할 예정입니다.</p>
          </div>
        )}
      </div>
    </div>
  );
}

export default App;
```

### ✅ 체크포인트
- [ ] "MetaMask 연결" 버튼이 보입니까?
- [ ] 버튼 클릭 시 MetaMask 팝업이 열립니까?
- [ ] 연결 후 계정 주소가 표시됩니까?
- [ ] 콘솔에 계정 주소가 출력됩니까?

### 💡 학습 포인트
```javascript
window.ethereum이란?
→ MetaMask가 브라우저에 주입하는 객체
→ 이것을 통해 지갑과 상호작용합니다

eth_requestAccounts란?
→ 사용자에게 계정 연결을 요청하는 메서드
→ MetaMask 팝업을 띄웁니다

왜 accounts[0]을 사용하나요?
→ MetaMask는 여러 계정을 가질 수 있습니다
→ 현재 선택된 첫 번째 계정을 가져옵니다
```

---

##  단계 3: Provider 설정

### 목표
- Ethers.js 라이브러리 이해
- Provider 개념 학습
- Contract 인스턴스 생성

### 3-1: contractConfig.js 확인

**파일**: `utils/contractConfig.js` (이미 생성됨)

```javascript
import contractABI from '../abis/Step8_CompleteERC721.json';

// 환경 변수에서 컨트랙트 주소 가져오기
export const CONTRACT_ADDRESS = process.env.REACT_APP_CONTRACT_ADDRESS;
export const NETWORK_ID = parseInt(process.env.REACT_APP_NETWORK_ID || '31337');

// 컨트랙트 ABI
export const CONTRACT_ABI = contractABI.abi;
```

### 💡 학습 포인트
```javascript
ABI란?
→ Application Binary Interface
→ 스마트 컨트랙트의 "사용 설명서"
→ 어떤 함수가 있고, 어떻게 호출하는지 정의

CONTRACT_ADDRESS란?
→ 배포된 컨트랙트의 주소
→ .env.local 파일에서 읽어옵니다

왜 환경 변수를 사용하나요?
→ 네트워크마다 주소가 다르기 때문
→ 코드를 수정하지 않고 설정만 변경 가능
```

### 3-2: .env.local 확인

**파일**: `frontend/.env.local` (이미 생성됨)

```env
REACT_APP_CONTRACT_ADDRESS=0x5FbDB2315678afecb367f032d93F642f64180aa3
REACT_APP_NETWORK_ID=31337
REACT_APP_NETWORK_NAME=localhost
```

### ✅ 체크포인트
- [ ] `.env.local` 파일이 존재합니까?
- [ ] 컨트랙트 주소가 설정되어 있습니까?
- [ ] `utils/contractConfig.js` 파일이 존재합니까?

---

##  단계 4: 컨트랙트 읽기 (totalSupply)

### 목표
- Provider를 사용한 블록체인 읽기
- Contract 인스턴스 생성
- View 함수(totalSupply) 호출

### 4-1: TotalSupply 컴포넌트 생성

**파일 생성**: `components/TotalSupply.jsx`

```javascript
import React, { useState, useEffect } from 'react';
import { ethers } from 'ethers';
import { CONTRACT_ADDRESS, CONTRACT_ABI } from '../utils/contractConfig';

function TotalSupply({ account }) {
  const [totalSupply, setTotalSupply] = useState(0);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState('');

  // TODO 4-1: account가 변경될 때마다 totalSupply 조회
  useEffect(() => {
    if (account && window.ethereum) {
      fetchTotalSupply();
    }
  }, [account]);

  // TODO 4-2: totalSupply 조회 함수
  const fetchTotalSupply = async () => {
    try {
      setIsLoading(true);
      setError('');

      // 1단계: Provider 생성 (읽기 전용)
      const provider = new ethers.BrowserProvider(window.ethereum);

      // 2단계: Contract 인스턴스 생성
      const contract = new ethers.Contract(
        CONTRACT_ADDRESS,  // 컨트랙트 주소
        CONTRACT_ABI,      // ABI
        provider           // Provider (읽기 전용)
      );

      // 3단계: totalSupply() 호출
      const supply = await contract.totalSupply();

      // 4단계: BigInt를 Number로 변환
      setTotalSupply(Number(supply));

      console.log('✅ totalSupply:', Number(supply));

    } catch (err) {
      console.error('오류:', err);
      setError('총 발행량을 불러오는데 실패했습니다.');
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div className="card">
      <h2>총 NFT 발행량</h2>

      {!account ? (
        <div className="info-box warning">
          <p>지갑을 먼저 연결해주세요.</p>
        </div>
      ) : error ? (
        <div className="info-box error">
          <p>{error}</p>
        </div>
      ) : (
        <div>
          <div className="info-box">
            <p style={{
              fontSize: '48px',
              fontWeight: 'bold',
              textAlign: 'center'
            }}>
              {isLoading ? '...' : totalSupply}
            </p>
            <p style={{ textAlign: 'center', color: '#666' }}>
              개의 NFT가 발행되었습니다
            </p>
          </div>

          <button onClick={fetchTotalSupply} disabled={isLoading}>
            🔄 새로고침
          </button>
        </div>
      )}
    </div>
  );
}

export default TotalSupply;
```

### 4-2: App.js에 TotalSupply 추가

이제 지갑 연결 후에 TotalSupply 컴포넌트를 표시하도록 App.js를 수정합니다.

```javascript
import React, { useState } from 'react';
import './App.css';
import ConnectWallet from './components/ConnectWallet';
import TotalSupply from './components/TotalSupply';

function App() {
  const [account, setAccount] = useState('');
  const [refreshTrigger, setRefreshTrigger] = useState(0);

  const handleAccountChange = (newAccount) => {
    setAccount(newAccount);
    console.log('연결된 계정:', newAccount);
  };

  return (
    <div className="App">
      {/* 헤더: 왼쪽 제목, 오른쪽 지갑 연결 버튼 */}
      <header className="app-header">
        <h1>기념 NFT 민팅 DApp</h1>
        <ConnectWallet onAccountChange={handleAccountChange} />
      </header>

      {/* 메인 컨텐츠 */}
      <div className="container">
        {!account ? (
          // 지갑 연결 전: 안내 카드 표시
          <div className="card">
            <h2>환영합니다!</h2>
            <div className="info-box" style={{
              background: '#7289da20',
              border: '1px solid #7289da',
              textAlign: 'center',
              padding: '40px 20px'
            }}>
              <p style={{ fontSize: '18px', marginBottom: '20px', color: '#e0e0e0' }}>
                NFT DApp을 사용하려면 먼저 지갑을 연결해주세요
              </p>
              <p style={{ fontSize: '14px', color: '#b0b0b0' }}>
                오른쪽 상단의 "지갑 연동하기" 버튼을 클릭하세요
              </p>
            </div>

            <div className="info-box" style={{
              marginTop: '20px',
              background: '#43b58120',
              border: '1px solid #43b581'
            }}>
              <p style={{ color: '#e0e0e0' }}><strong>시작하기 전에:</strong></p>
              <ul style={{ marginLeft: '20px', marginTop: '12px', lineHeight: '1.8', color: '#b0b0b0' }}>
                <li>MetaMask 브라우저 확장 프로그램이 설치되어 있어야 합니다</li>
                <li>로컬 Hardhat 네트워크(localhost:18545)에 연결되어야 합니다</li>
                <li>테스트용 ETH가 있는 계정을 사용하세요</li>
              </ul>
            </div>
          </div>
        ) : (
          // 지갑 연결 후: 모든 기능 표시
          <>
            {/* totalSupply 표시 */}
            <TotalSupply account={account} refreshTrigger={refreshTrigger} />
          </>
        )}
      </div>
    </div>
  );
}

export default App;
```

**주요 변경사항:**
- `refreshTrigger` state 추가: 나중에 NFT 민팅 후 자동으로 데이터를 새로고침하기 위해 사용
- 지갑 연결 후에만 TotalSupply 컴포넌트 표시

### ✅ 체크포인트
- [ ] 지갑 연결 후 totalSupply가 표시됩니까?
- [ ] "새로고침" 버튼이 작동합니까?
- [ ] 콘솔에 totalSupply 값이 출력됩니까?
- [ ] 현재 값이 0입니까? (아직 민팅하지 않았으므로)

### 💡 학습 포인트
```javascript
Provider란?
→ 블록체인에 대한 읽기 전용 연결
→ 가스비가 들지 않습니다
→ 누구나 사용 가능합니다

Contract 인스턴스란?
→ 스마트 컨트랙트와 상호작용하기 위한 JavaScript 객체
→ 필요한 것: 주소, ABI, Provider

View 함수란?
→ 블록체인 상태를 읽기만 하는 함수
→ 가스비가 들지 않습니다
→ totalSupply(), balanceOf() 등

왜 BigInt를 Number로 변환하나요?
→ Solidity의 uint256은 JavaScript의 Number보다 큽니다
→ Ethers.js는 BigInt로 반환합니다
→ UI에 표시하려면 Number로 변환해야 합니다
```

---

## 🔧 문제 해결

### "MetaMask가 설치되지 않았습니다" 오류
→ MetaMask를 설치하고 페이지를 새로고침하세요.

### totalSupply가 표시되지 않음
→ `.env.local` 파일의 컨트랙트 주소를 확인하세요.

### "Cannot read property 'request' of undefined"
→ window.ethereum이 없습니다. MetaMask를 설치하세요.

### 페이지가 새로고침되지 않음
→ React 개발 서버를 재시작하세요.
