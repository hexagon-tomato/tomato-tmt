
````markdown


このリポジトリは **Foundry** と **OpenZeppelin** を使って  
ERC20 トークン **TOMATO (TMT)** を作成し、テストネット **Sepolia** にデプロイする手順をまとめたものです。    
=======
# TOMATO (TMT) ERC20 Token on Sepolia

このリポジトリは **Foundry** と **OpenZeppelin** を使って  
ERC20 トークン **TOMATO (TMT)** を作成し、テストネット **Sepolia** にデプロイする手順をまとめたものです。  
---

## ゴール

- TOMATO (TMT) トークンを作る  
- Sepolia テストネットにデプロイする  
- デプロイ後にトークン情報を確認できる  
- 作成した**TOMATO (TMT)トークン**は次のリンクから確認できます。
 [View on Etherscan (Sepolia)]
(https://sepolia.etherscan.io/token/0x99f81904A33b5a40E4EAF8758a0c2FbAB2E658E5)
=======
---

## 1. 事前準備

### 必要なもの
- Windows + Git Bash または WSL  
  （Linux / macOS ユーザーは通常の bash でOK）
- Node.js LTS（`node -v` で確認）
- Foundry（`forge`, `cast`, `anvil` が使える状態）

---

## 2. プロジェクト作成

```bash
#Foundry プロジェクトの雛形を作成
forge init my-project
#カレントディレクトリを my-project に移動する
cd my-project
#.git/ フォルダが作成されて、バージョン管理ができるようになる。
git init

=======
mkdir amm-origin && cd amm-origin
forge init amm-origin
cd amm-origin
git init
````

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

### Foundry プロジェクトの設定ファイル `foundry.toml`を作成

```bash
cat << 'EOF' > foundry.toml
=======
### `foundry.toml`

```toml
>>>>>>> d4136da (Add README at repo root)
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
solc_version = "0.8.24"
remappings = [
  "@openzeppelin/=lib/openzeppelin-contracts/"
]
EOF
#src = "src" 
#コントラクト（Solidity ファイル）のソースコードを置くフォルダを src/ にする

#out = "out"
#コンパイル結果（ABI やバイトコードなど）を出力するフォルダ

#libs = ["lib"]
#外部ライブラリを配置するフォルダを lib/ に指定

#remappings
#インポートのパス変換ルール
#Solidity のコード内で import "@openzeppelin/contracts/token/ERC20/ERC20.sol"; と書いたとき、
#lib/openzeppelin-contracts/contracts/token/ERC20/ERC20.solを参照するようになる

=======
```

---

## 4. コントラクト`src/TOMATO.sol`を作成

```bash

cat << 'EOF' > src/TOMATO.sol
=======
## 4. コントラクト作成

`src/TOMATO.sol`

```solidity
>>>>>>> d4136da (Add README at repo root)
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract TOMATO is ERC20 {
    constructor() ERC20("TOMATO", "TMT") {
        _mint(msg.sender, 1_000_000 ether);
    }
}
EOF
```

: '
1. // SPDX-License-Identifier: MIT

意味: このコードのライセンスを指定している

Solidity では SPDX ライセンス表記が必須

2. pragma solidity ^0.8.24;

意味: Solidity コンパイラのバージョン指定

^0.8.24 → 0.8.24 以上、0.9.0 未満 のバージョンで動く

3. import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

意味: OpenZeppelin が提供する ERC20 の標準実装を読み込む

4. contract TOMATO is ERC20

意味: TOMATO というコントラクトを定義し、OpenZeppelin の ERC20 を継承する

継承により、ERC20 トークンの基本機能（転送、残高確認、承認など）が自動的に使える

5. constructor() ERC20("TOMATO", "TMT")

意味: デプロイ時に実行される「コンストラクタ」

ERC20("TOMATO", "TMT") → 親クラスのコンストラクタを呼び出し、トークン名とシンボルを設定

名前: "TOMATO"

シンボル: "TMT"

6. _mint(msg.sender, 1_000_000 ether);

意味: デプロイした人（msg.sender）に 100万 TOMATO トークンを発行する

ether は ERC20 の 最小単位（10^18 wei 相当） を扱うための書き方

つまり「100万 * 10^18」単位のトークンを発行する

これは実質的に 100万 TOMATO トークン
'

=======
```

---

## 5. テストコード

`test/TOMATO.t.sol`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "forge-std/Test.sol";
import "../src/TOMATO.sol";

contract TOMATOTest is Test {
    TOMATO token;

    function setUp() public {
        token = new TOMATO();
    }

    function testName() public {
        assertEq(token.name(), "TOMATO");
    }

    function testSymbol() public {
        assertEq(token.symbol(), "TMT");
    }

    function testTotalSupply() public {
        assertEq(token.totalSupply(), 1_000_000 ether);
    }
}
```

実行:

```bash
forge test -vv
```

---

## 6. デプロイスクリプト & .env

### `.env`

```ini
SEPOLIA_RPC_URL=https://sepolia.infura.io/v3/<YOUR_INFURA_PROJECT_ID>
PRIVATE_KEY=0x<YOUR_PRIVATE_KEY>
```

### デプロイスクリプト`script/DeployTOMATO.s.sol`

```bash
cat << 'EOF' > script/DeployTOMATO.s.sol
=======
ETHERSCAN_API_KEY=<任意>
```

### `script/DeployTOMATO.s.sol`

```solidity
>>>>>>> d4136da (Add README at repo root)
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
EOF
```
---
## 7. デプロイ手順（Sepolia）

=======
```

---

## 7. デプロイ手順（Sepolia）

### \<YOUR\_ADDRESS> とは？

* `.env` の **PRIVATE\_KEY に対応する自分のウォレットアドレス（EOA）**
* faucet や残高確認に使う公開アドレス

確認コマンド:

```bash
source .env
cast wallet address $PRIVATE_KEY
#実行すると<YOUR\_ADDRESS>が返ってくる。
```

### \<YOUR\_ADDRESS> とは？
* `.env` の **PRIVATE\_KEY に対応する自分のウォレットアドレス（EOA）**
* faucet や残高確認に使う公開アドレス

### 残高確認（省略可能）
=======
```

### 残高確認

```bash
cast balance <YOUR_ADDRESS> --rpc-url $SEPOLIA_RPC_URL
```

### シミュレーション（省略可能）
=======
### シミュレーション

```bash
forge script script/DeployTOMATO.s.sol:DeployTOMATO \
  --rpc-url $SEPOLIA_RPC_URL \
  -vvvv
```

### 本番デプロイ

```bash
forge script script/DeployTOMATO.s.sol:DeployTOMATO --rpc-url "$SEPOLIA_RPC_URL" --broadcast --chain 11155111 -vvvv
```
chain 11155111：Sepolia テストネットのチェーンID

コマンド実行結果：

![Screenshot]("C:\Users\tell5\Pictures\Screenshots\comand.png")

Sepoliaテストネットにデプロイしたトランザクション詳細：

![Sepolia Transaction Screenshot]("C:\Users\tell5\Pictures\Screenshots\ethscan.png")
---

## 8. デプロイ確認
 <ADDR>はコントラクトアドレスに置き換えて。
 
=======
forge script script/DeployTOMATO.s.sol:DeployTOMATO \
  --rpc-url $SEPOLIA_RPC_URL \
  --broadcast \
  --chain 11155111 \
  -vvvv
```
---

## 8. デプロイ確認

>>>>>>> d4136da (Add README at repo root)
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

### 送金

```bash
cast send <ADDR> "transfer(address,uint256)" <TO_ADDR> $(cast --to-uint256 10e18) \
  --rpc-url $SEPOLIA_RPC_URL \
  --private-key $PRIVATE_KEY
```

### MetaMask 表示

MetaMask → 「トークンをインポート」 → コントラクトアドレス `<ADDR>` を入力

---

## 11. トラブルシューティング

* **`--rpc-url` が空** → `source .env` を忘れている
* **秘密鍵形式** → `0x` を付ける、改行や空白を削除
* **totalSupply 表示エラー** → `cast --to-unit <wei> ether` を使う
* **残高不足** → Sepolia faucet から ETH 補充

---

##  ファイル構成（完成形）
=======
```
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

##  ライセンス


このリポジトリのコードは **MIT License** のもとで公開されています。
誰でも自由に利用・改変・再配布が可能ですが、利用は自己責任でお願いします。

---

##  用語集

```markdown
##  Forge・OpenZeppelin・ABI の関係性

### OpenZeppelin とは
Ethereum 向けの **定番ライブラリ集** です。  
ERC20 や NFT など、よく使う機能（transfer, balanceOf など）を **安全に定義済みの部品** として提供しています。  
→ これを継承することで、自分で一から関数を書かずに、安全な機能をそのまま利用できます。

### Forge とは
[Foundry](https://book.getfoundry.sh/) に含まれる **開発用の工房ツール** です。  
スマートコントラクトを **ビルド（コンパイル）・テスト・デプロイ** できます。  
Forge でビルドすると、部品（OpenZeppelin）を組み合わせて「完成品（バイトコード）」と「取扱説明書（ABI）」を自動で作ってくれます。

主なコマンド例:
- `forge build` : コードをコンパイル（成果物と ABI を生成）
- `forge test`  : テストを自動実行
- `forge script`: デプロイスクリプトを実行してブロックチェーンに展開

### ABI とは
ABI (Application Binary Interface) は、**スマートコントラクトの「取扱説明書」** です。  
「どんな関数があり、どんなデータを渡せば呼び出せるのか」を人間やアプリに伝えます。  
これがないと、ウォレットやフロントエンドからコントラクトを正しく操作できません。

#### ABI はどこに生成される？
`forge build` を実行すると、`out/` ディレクトリに生成される **`.json` ファイル** の中に含まれています。

例: `TOMATO.sol` をビルドした場合
```

out/
└─ TOMATO.sol/
├─ TOMATO.json       # ← この中に "abi": \[...] が入っている
└─ TOMATO.dbg.json   # デバッグ用情報

````




