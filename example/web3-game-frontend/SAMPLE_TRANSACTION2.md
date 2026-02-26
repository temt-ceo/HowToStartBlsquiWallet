# 無人野菜販売所のトランザクションコード例

#### 野菜棚に野菜の在庫を登録

```
    transaction = `
      import VegeSeller from 0xb576a3926d239682

      transaction(id: String, add: [String], addAmount: [UInt8], remove: [String]) {
        prepare(signer: &Account) {
          VegeSeller.setVegeSellerInfo(id: id, add: add, addAmount: addAmount, remove: remove)
        }
        execute {
          log("success")
        }
      }
    `;
    txId = await mutate({
      cadence: transaction,
      args: (arg, t) => [
        arg(message.id.toString(), t.String),
        arg(message.add, t.Array(t.String)),
        arg(message.addAmount, t.Array(t.UInt8)),
        arg([], t.Array(t.String)),
      ],
      proposer: authFunctionForProposer,
      payer: authFunction,
      authorizations: [authFunction],
      limit: 999,
    });
```

#### 無人野菜販売所の野菜を購入

```
  const txId = await mutate({
    cadence: `
      import "VegeSeller"
      import "FlowToken"
      import "FungibleToken"

      transaction(id: String, buy: [String], price: UFix64) {
        prepare(signer: auth(BorrowValue) &Account) {
          let payment <- signer.storage.borrow<auth(FungibleToken.Withdraw) &FlowToken.Vault>(from: /storage/flowTokenVault)!.withdraw(amount: price) as! @FlowToken.Vault
          VegeSeller.buyVege(id: id, amount: <- payment, buy: buy)
        }
        execute {
          log("success")
        }
      }
    `,
    args: (arg, t) => [
      arg(id.toString(), t.String),
      arg(buy, t.Array(t.String)),
      arg(price, t.UFix64),
    ],
    proposer: authz,
    payer: authz,
    authorizations: [authz],
    limit: 999,
  });
```

#### スマートコントラクト

```
import "FlowToken"
import "FungibleToken"

access(all) contract VegeSeller {

  access(self) let FlowTokenVault: Capability<&{FungibleToken.Receiver}>
  access(self) let info: Info

  /* 構造体 */
  access(all) struct Info {

    access(contract) var data: {String: VegeData}

    /* setter */
    access(contract) fun set(id: String, add: [String], addAmount: [UInt8], remove: [String]) {
      if let saved = self.data[id] {
        for index, name in add {
          saved.set(category: name, amount: addAmount[index])
        }
        for name in remove {
          saved.unset(category: name)
        }
        // Save
        self.data[id] = saved
      } else {
        var data = VegeData()
        for index, name in add {
          data.set(category: name, amount: addAmount[index])
        }
        for name in remove {
          data.unset(category: name)
        }
        self.data[id] = data
      }
    }

    access(contract) fun buy(id: String, buy: [String]) {
      if let saved = self.data[id] {
        for name in buy {
          if (saved.data[name] == nil) {
            panic("The stock is empty.")
          }
          saved.set(category: name, amount: saved.data[name]! - 1)
        }
        // Save
        self.data[id] = saved
      } else {
        panic("No data.")
      }
    }

    init() {
      self.data = {}
    }
  }

  /* Info内の構造体 */
  access(all) struct VegeData {

    access(contract) var data: {String: UInt8}

    /* setter */
    access(contract) fun set(category: String, amount: UInt8) {
      self.data[category] = amount
    }

    access(contract) fun unset(category: String) {
      self.data[category] = nil
    }

    init() {
      self.data = {}
    }
  }

  // GET
  access(all) fun getVegeSellerInfo(id: String): VegeData? {
    return self.info.data[id]
  }
  // PUT
  access(all) fun setVegeSellerInfo(id: String, add: [String], addAmount: [UInt8], remove: [String]) {
    self.info.set(id: id, add: add, addAmount: addAmount, remove: remove)
  }
  // BUY
  access(all) fun buyVege(id: String, amount: @FlowToken.Vault, buy: [String]) {
    self.info.buy(id: id, buy: buy)
    self.FlowTokenVault.borrow()!.deposit(from: <- amount)
  }

  init() {
    self.info = Info()
    self.FlowTokenVault = self.account.capabilities.get<&{FungibleToken.Receiver}>(/public/flowTokenReceiver)
  }
}
```
