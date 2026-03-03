# Develoer Guide

## 🛠 Blsqui CLIの操作方法

Blsqui CLIはデベロッパーがBlsqui Protocol インフラストラクチャーを操作する為のコマンドラインインターフェースです。

トランザクションコードは普段ゲームをする人にはこれによって何が起こるのか分からない怖いコードです。
そこで、Blsqui プロトコルにトランザクションコードとゲームガイドを一緒にしてIDを発行し、監査を経ることで、ゲームをする人にこれは「戦士の能力を仲間に貸し出し、魔法剣で攻撃」など信用できるメッセージをウォレットに表示できます。
<br><br>
## Step1 :wrench: Install the Blsqui CLI

```
sh -ci "$(curl -fsSL https://raw.githubusercontent.com/blsqui/blsqui-cli/master/install.sh)"
```
<br><br>

## Step2 :sparkles: Register game domain.

```
blsqui-cli init —domain (Game’s landing page url) —email (same domain’s email address)
```

e.g.
blsqui-cli init —domain https://aethel-dragon.com —email dev@aethel-dragon.com

email domain must same as domain.
<br><br><br>

## Step3 Click one time use link, then project key is shown on the page. Download the file and change the password on the management screen.

Hello Team! A developer from your organization is claiming aethel-dragon.com on the Blsqui Protocol Registry. Please click here to verify ownership: https://blsqui.net/api/v1/verify?token=123abc

Click the button.
“Download blsqui-key.txt.”
<br><br><br>

## Step4 :gear: Configure Blsqui CLI with project key.

```
blsqui-cli config —file ./blsqui-key.txt
```
<br><br>

## Step5 トランザクションコードを考えます。

./cadence/transactions/buy_item.cdcに以下のようなトランザクションコードがあったとします。

```
import FungibleToken from 0xFungibleToken
import FlowToken from 0xFlowToken
import NonFungibleToken from 0xNonFungibleToken
import GameItems from 0xGameItwms

transaction(itemID: UInt64, price: UFix64) {
  let payment: @FungibleToken.Vault
  let collectionRef: &GameItems.Collection

  prepare(signer: AuthAccount) {
    // 1. Withdraw FLOW from the player
    let vaultRef = signer.borrow<&FlowToken.Vault>(from: /storage/flowTokenVault)
      ?? panic(“Could not borrow FLOW vault”)
    self.payment <- vaultRef.withdraw(amount: price)

    // 2. Borrow the player’s Item Collection
    self.collectionRef = signer.borrow<&GameItems.Collection>(from: GameItems.CollectionStoragePath)
      ?? panic(“Could not borrow GameItems collection”)
  }

  execute {
    // 3. Trade the money for the item
    GameItems.purchaseItem(item: itemID, payment: <-self.payment, recipient: self.collectionRef)
  }
}
```
<br><br>

## Step6 Prepare the Metadata(メタデータ(そのトランザクションに一致するゲームガイド)を考えJSONで作成する)

metadata.jsonを作成してゲームガイドを記入します。

```
{
  "f_type": “InteractionTemplate”,
  "f_version": “1.1.0”,
  "data": {
    "message": {
      "title": {
        “en”: “Purchase Game Item”,
        "ja": “ゲームアイテムを購入”,
      },
      "description": {
        “en”: “Buy 1x {itemName} for {price} FLOW from the your-game shop.”
        “ja”: “your-game shopで{itemName}を一つ、{price} FLOWで購入。”
      }
    },
    “category”: “GAME_ACTION”
  }
}
```
<br><br>

## Step7 Install the Flow CLI

```
sh -ci "$(curl -fsSL https://raw.githubusercontent.com/onflow/flow-cli/master/install.sh)"
```
<br><br>

## Step8 :gear: Generate the FLIX Template.

```
flow flix generate ./cadence/transactions/buy_item.cdc \
  --pre-fill ./metadata.json \
  --save ./buy_item.template.json \
  --network mainmet
```
<br><br>

## Step9 🚀 Upload your buy_item.template.json. You can attach one image.

```
blsqui-cli upload ./template.json ./fire-sword.png
```
<br><br>

## Step10 :gear: 審査依頼を出します。

```
blsqui-cli update —template-id x355dfx —require-audit
```

<b>アップロードしたテンプレートの一覧を取得できます。(—allオプションをつけると審査状況と公開設定も表示されます)</b>

```
blsqui-cli list # (templateIdとイメージ名を返す)
```

```
blsqui-cli view —templateId “123xyz”
blsqui-cli view —templateId “123xyz” —image
```
<br><br>

## Step11 📈 一般公開設定をします。

```
blsqui-cli update —template-id x355dfx —publish
```

ブラウジングページに表示されます。これによって世界に公開された信用のおけるトランザクションコードである事が証明されます。(審査が通っていない場合はdevelopment タブに表示されます)

URL: https://blsqui.net/registry/(your domain)
<br><br>

## Step12 ゲームを実行

<b>テンプレートキーをmutateファンクションにセットするとウォレットが呼び出されます。一般公開設定をしていなくても表示されます。画像を登録していれば画像も表示されます。</b>

```
important { mutate} from
await fcl.mutate({
  template: “5a2b3c…”, // template ID
  args: (arg. t) => [
    arg("1240", t.UInt64),
    arg(“10.0”, t.UFix64)
  ]
})
```

<br><br>

#### サンプルトランザクションコード<br><br>

🎨 野菜室保存管理のトランザクションコード例: https://github.com/temt-ceo/HowToStartBlsquiWallet/blob/main/example/web3-game-frontend/SAMPLE_TRANSACTION1.md<br><br>

🎨 無人野菜販売所のトランザクションコード例: https://github.com/temt-ceo/HowToStartBlsquiWallet/blob/main/example/web3-game-frontend/SAMPLE_TRANSACTION2.md<br><br>

🎨 ライドシェアサービスのトランザクションコード例: https://github.com/temt-ceo/HowToStartBlsquiWallet/blob/main/example/web3-game-frontend/SAMPLE_TRANSACTION3.md<br><br>

🎨 シューティングeSportsゲームのトランザクションコード例: https://github.com/temt-ceo/HowToStartBlsquiWallet/blob/main/example/web3-game-frontend/SAMPLE_TRANSACTION4.md<br><br>

🎨 MMORPGのトランザクションコード例: https://github.com/temt-ceo/HowToStartBlsquiWallet/blob/main/example/web3-game-frontend/SAMPLE_TRANSACTION5.md<br><br>
