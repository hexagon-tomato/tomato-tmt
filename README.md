# 🍅 TOMATO (TMT) ERC20 Token on Sepolia

このリポジトリでは **Foundry** と **OpenZeppelin** を使って  
ERC20 トークン **TOMATO (TMT)** を作成し、テストネット **Sepolia** にデプロイする手順をまとめています。

作成した **TOMATO (TMT) トークン** は以下から確認できます。  

👉 [Etherscan で確認](https://sepolia.etherscan.io/token/0x99f81904A33b5a40E4EAF8758a0c2FbAB2E658E5)

---

## 🎯 ゴール

- TOMATO (TMT) トークンを作成する  
- Sepolia テストネットにデプロイする  
- デプロイ後にトークン情報を Etherscan で確認できる  

---

## 1. 事前準備

- **Windows + Git Bash** または **WSL**  
  （Linux / macOS の場合は通常の bash でOK）
- **Node.js LTS** （`node -v` で確認）
- **Foundry** （`forge`, `cast`, `anvil` が利用可能な状態）

---

## 2. プロジェクト作成

```bash
# Foundry プロジェクトの雛形を作成
forge init my-project

# プロジェクトディレクトリへ移動
cd my-project

# Git リポジトリを初期化
git init
```

---

## 3. OpenZeppelin ライブラリ導入

```bash
forge install OpenZeppelin/openzeppelin-contracts@v5.0.2
```

### 成功確認

```bash
ls lib | grep openzeppelin-contracts
forge remappings | grep openzeppelin
```

### Foundry 設定ファイル `foundry.toml`

```bash
cat << 'EOF' > foundry.toml
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
solc_version = "0.8.24"
remappings = [
  "@openzeppelin/=lib/openzeppelin-contracts/"
]
EOF
```

---

## 4. コントラクト作成

`src/TOMATO.sol`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract TOMATO is ERC20 {
    constructor() ERC20("TOMATO", "TMT") {
        _mint(msg.sender, 1_000_000 ether);
    }
}
```

---

## 📖 コード解説

1. **SPDX ライセンス表記**  
   コードのライセンスを指定（必須）
2. **pragma solidity ^0.8.24**  
   Solidity コンパイラのバージョンを指定
3. **import**  
   OpenZeppelin の ERC20 標準実装を利用
4. **contract TOMATO is ERC20**  
   ERC20 を継承したコントラクト
5. **constructor()**  
   デプロイ時にトークン名とシンボルを設定
6. **_mint(msg.sender, 1_000_000 ether)**  
   デプロイアドレスに 100万 TOMATO を発行

---

## 5. テスト実行

```bash
forge test -vv
```

---

## 6. デプロイスクリプト & 環境変数

### `.env`

```ini
SEPOLIA_RPC_URL=https://sepolia.infura.io/v3/<YOUR_INFURA_PROJECT_ID>
PRIVATE_KEY=0x<YOUR_PRIVATE_KEY>
```

### `script/DeployTOMATO.s.sol`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "forge-std/Script.sol";
import "../src/TOMATO.sol";

contract DeployTOMATO is Script {
    function run() external {
        uint256 deployerKey = vm.envUint("PRIVATE_KEY");
        vm.startBroadcast(deployerKey);
        new TOMATO();
        vm.stopBroadcast();
    }
}
```

---

## 7. デプロイ手順（Sepolia）

ウォレットアドレス確認:

```bash
source .env
cast wallet address $PRIVATE_KEY
# => <YOUR_ADDRESS>
```

残高確認:

```bash
cast balance <YOUR_ADDRESS> --rpc-url $SEPOLIA_RPC_URL
```

シミュレーション:

```bash
forge script script/DeployTOMATO.s.sol:DeployTOMATO \
  --rpc-url $SEPOLIA_RPC_URL \
  -vvvv
```

本番デプロイ:

```bash
forge script script/DeployTOMATO.s.sol:DeployTOMATO \
  --rpc-url "$SEPOLIA_RPC_URL" \
  --broadcast \
  --chain 11155111 \
  -vvvv
```

---

## 8. デプロイ確認

```bash
cast code <ADDR> --rpc-url $SEPOLIA_RPC_URL | wc -c
cast call <ADDR> "name()(string)"    --rpc-url $SEPOLIA_RPC_URL
cast call <ADDR> "symbol()(string)"  --rpc-url $SEPOLIA_RPC_URL
cast call <ADDR> "decimals()(uint8)" --rpc-url $SEPOLIA_RPC_URL

SUPPLY=$(cast call <ADDR> "totalSupply()(uint256)" --rpc-url $SEPOLIA_RPC_URL | awk '{print $1}')
cast --to-unit "$SUPPLY" ether
# => 1000000
```

---

## 9. （任意）Etherscan 検証

```bash
forge verify-contract <ADDR> TOMATO \
  --chain 11155111 \
  --etherscan-api-key $ETHERSCAN_API_KEY
```

---

## 10. （任意）送金とウォレット表示

送金:

```bash
cast send <ADDR> "transfer(address,uint256)" <TO_ADDR> $(cast --to-uint256 10e18) \
  --rpc-url $SEPOLIA_RPC_URL \
  --private-key $PRIVATE_KEY
```

MetaMask で表示:  
「トークンをインポート」 → コントラクトアドレス `<ADDR>` を入力

---

## 11. トラブルシューティング

- **`--rpc-url` が空** → `source .env` を忘れている
- **秘密鍵形式** → `0x` を付ける、改行や空白を削除
- **totalSupply 表示エラー** → `cast --to-unit <wei> ether` を使う
- **残高不足** → Sepolia faucet から ETH 補充

---

## 📸 実行例スクリーンショット

### デプロイコマンド実行結果
![Screenshot](./docs/command.png)

### Sepolia トランザクション詳細
![Sepolia Transaction Screenshot](./docs/ethscan.png)

---

## 📂 プロジェクト構成

```text
amm-origin/
├─ foundry.toml
├─ .gitignore
├─ .env
├─ lib/
│  └─ openzeppelin-contracts/...
├─ src/
│  └─ TOMATO.sol
├─ test/
│  └─ TOMATO.t.sol
└─ script/
   └─ DeployTOMATO.s.sol
```

---

## 📜 ライセンス

MIT License  
誰でも自由に利用・改変・再配布が可能ですが、利用は自己責任でお願いします。

---

## 📖 用語集

### OpenZeppelin  
Ethereum 向けの **定番ライブラリ集**。  
ERC20 や NFT など、安全に実装済みの部品を提供。

### Forge（Foundry）  
スマートコントラクト開発ツール。  
ビルド・テスト・デプロイが可能。

主なコマンド:
- `forge build` : コードをコンパイルし ABI を生成
- `forge test`  : テストを実行
- `forge script`: デプロイを実行

### ABI (Application Binary Interface)  
スマートコントラクトの「取扱説明書」。  
関数や入出力の仕様を記載。ウォレットやフロントエンドが利用。

#### ABI の場所
`forge build` 実行後、`out/` に生成される JSON に含まれる。

例: `TOMATO.sol` をビルドした場合

```text
out/
└─ TOMATO.sol/
   ├─ TOMATO.json       # ← この中に "abi": [...] がある
   └─ TOMATO.dbg.json   # デバッグ情報
```
