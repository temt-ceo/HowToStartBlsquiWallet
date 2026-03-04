# Develoer Guide

## 🛠 Blsqui CLI: The Sovereign Asset Manager

The Blsqui CLI provides developers with a secure interface to the Blsqui Protocol Infrastructure. In standard Web3 environments, “Blind Signing” creates a critical security vulnerability where users must authorize hex-code transactions.
<br><br>
Blsqui allows you to wrap complex Cadence interactions inside Sovereign Metadata. By registering your transaction logic with your official domain and a “Game Guide”, you can deliver human-readable transaction summaries. This ensures that a complex smart contract call is rendered in the wallet as a “Verified Institutional Action”, your players see a “Power-Up” or “Loot Drop” message and your custom artwork right in their wallet. This preserves the immersion of your game world and protects your Intellectual Property from external spoofing.
<br><br>

## Step1 Provision the Blsqui Environment

Install the Blsqui Engine.

```
sh -ci "$(curl -fsSL https://raw.githubusercontent.com/blsqui/blsqui-cli/master/install.sh)"
```

<br><br>

## Step2 Claim Your Domain Identity

```
blsqui-cli init —domain (Game’s landing page url) —email (same domain’s email address)
```
e.g.<br>
blsqui-cli init —domain https://aethel-dragon.com —email dev@aethel-dragon.com<br><br>
email domain must same as domain.
<br><br><br>

## Step3 Unlock Your Secret Project Key

Verify Enterprise Ownership<br>
Click one time use link, then project key is shown on the page. Download the file and change the password on the management screen.
<br><br>
Hello Team! A developer from your organization is claiming aethel-dragon.com on the Blsqui Protocol Registry. Please click here to verify ownership: https://blsqui.net/api/v1/verify?token=123abc

Click the button.
“Download blsqui-key.txt.”
<br><br><br>

## Step4 Sync Your Workspace

Initialize Permanent Authentication.

```
blsqui-cli config —file ./blsqui-key.txt
```

<br><br>

## Step5 Develop the Cadence Transaction Layer

Draft Your Game Move.

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

## Step6 Design the Player Guide (Metadata)

Define the Human-Readable Guide.

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

## Step7 Equip the Flow SDK

Integrate the Flow Protocol Toolkit.

```
sh -ci "$(curl -fsSL https://raw.githubusercontent.com/onflow/flow-cli/master/install.sh)"
```

<br><br>

## Step8 Generate a FLIX Template

```
flow flix generate ./cadence/transactions/buy_item.cdc \
  --pre-fill ./metadata.json \
  --save ./buy_item.template.json \
  --network mainmet
```

<br><br>

## Step9 Bind Visual IP to Transaction IDs.

Register Visual Assets to Protocol.

```
blsqui-cli upload ./template.json ./fire-sword.png
```

<br><br>

## Step10 Execute the Security Review

Request the Audit.

```
blsqui-cli update —template-id x355dfx —require-audit
```

<b>アップロードしたテンプレートの一覧を取得できます。(—allオプションをつけると審査状況と公開設定も表示されます。つけていない場合はtemplateIdとイメージ名を返します</b>

```
blsqui-cli list
blsqui-cli view —templateId “123xyz”
blsqui-cli view —templateId “123xyz” —image
```

<br><br>

## Step11 Promote to Global Registry

```
blsqui-cli update —template-id x355dfx —publish
```

This will launch your template to global registry. (審査が通っていない場合はdevelopment タブに表示されます)

URL: https://blsqui.net/registry/(your domain)
<br><br>

## Step12 Trigger Your First Transaction

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

一般公開設定をしていなくても表示されます。
<br><br>

#### サンプルトランザクションコード<br><br>

🎨 野菜室保存管理のトランザクションコード例: https://github.com/temt-ceo/HowToStartBlsquiWallet/blob/main/example/web3-game-frontend/SAMPLE_TRANSACTION1.md<br><br>

🎨 無人野菜販売所のトランザクションコード例: https://github.com/temt-ceo/HowToStartBlsquiWallet/blob/main/example/web3-game-frontend/SAMPLE_TRANSACTION2.md<br><br>

🎨 ライドシェアサービスのトランザクションコード例: https://github.com/temt-ceo/HowToStartBlsquiWallet/blob/main/example/web3-game-frontend/SAMPLE_TRANSACTION3.md<br><br>

🎨 シューティングeSportsゲームのトランザクションコード例: https://github.com/temt-ceo/HowToStartBlsquiWallet/blob/main/example/web3-game-frontend/SAMPLE_TRANSACTION4.md<br><br>

🎨 MMORPGのトランザクションコード例: https://github.com/temt-ceo/HowToStartBlsquiWallet/blob/main/example/web3-game-frontend/SAMPLE_TRANSACTION5.md<br><br>
