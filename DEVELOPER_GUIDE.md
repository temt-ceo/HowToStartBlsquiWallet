# デベロッパーガイド

### Step1 Install the Flow CLI

```
sh -ci "$(curl -fsSL https://raw.githubusercontent.com/onflow/flow-cli/master/install.sh)"
```
<hr>
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

### Step2 Prepare the Metadata(メタデータをJSONで作成する)

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

### Step3 Flow CLIでGenerate the FLIX Templateを作成する

```
flow flix generate ./cadence/transactions/buy_item.cdc \
  --pre-fill ./metadata.json \
  --save ./buy_item.template.json \
  --network mainmet
```

### Step4 Templateをレジストリに保存する

Upload your buy_item.template.json

### Step5 ゲームを実行

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

参考までにサンプルトランザクションコードを載せておきます。<br><br>

🎨 野菜室保存管理のトランザクションコード例はこちら: https://github.com/temt-ceo/HowToStartBlsquiWallet/blob/main/example/web3-game-frontend/SAMPLE_TRANSACTION1.md<br><br>

🎨 無人野菜販売所のトランザクションコード例はこちら: https://github.com/temt-ceo/HowToStartBlsquiWallet/blob/main/example/web3-game-frontend/SAMPLE_TRANSACTION2.md<br><br>

🎨 ライドシェアサービスのトランザクションコード例はこちら: https://github.com/temt-ceo/HowToStartBlsquiWallet/blob/main/example/web3-game-frontend/SAMPLE_TRANSACTION3.md<br><br>

🎨 シューティングeSportsゲームのトランザクションコード例はこちら: https://github.com/temt-ceo/HowToStartBlsquiWallet/blob/main/example/web3-game-frontend/SAMPLE_TRANSACTION4.md<br><br>

🎨 MMORPGのトランザクションコード例はこちら: https://github.com/temt-ceo/HowToStartBlsquiWallet/blob/main/example/web3-game-frontend/SAMPLE_TRANSACTION5.md<br><br>
