# 野菜室保存管理のトランザクションコード例

#### 野菜室に保存した時間を保存する

```
    transaction = `
      import CookStocker from 0xb576a3926d239682

      transaction(id: String, add: [String], remove: [String]) {
        prepare(signer: &Account) {
          CookStocker.setCookStockerInfo(id: id, add: add, remove: remove)
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
        arg(message.remove, t.Array(t.String)),
      ],
      proposer: authFunctionForProposer,
      payer: authFunction,
      authorizations: [authFunction],
      limit: 999,
    });
```

#### ブロックチェーンに保存した野菜室の情報を削除する

```
    transaction = `
      import CookStocker from 0xb576a3926d239682

      transaction(id: String) {
        prepare(signer: &Account) {
          CookStocker.deleteCookStockerInfo(id: id)
        }
        execute {
          log("success")
        }
      }
    `;
    txId = await mutate({
      cadence: transaction,
      args: (arg, t) => [arg(message.id.toString(), t.String)],
      proposer: authFunctionForProposer,
      payer: authFunction,
      authorizations: [authFunction],
      limit: 999,
    });
```

#### スマートコントラクト

```
access(all) contract CookStocker {

  access(self) let info: Info

  /* 構造体 */
  access(all) struct Info {

    access(contract) var data: {String: VegeData}

    /* setter */
    access(contract) fun set(id: String, add: [String], remove: [String]) {
      if let saved = self.data[id] {
        for name in add {
          saved.set(category: name, bought_at: getCurrentBlock().timestamp)
        }
        for name in remove {
          saved.unset(category: name)
        }
        // Save
        self.data[id] = saved
      } else {
        var data = VegeData()
        for name in add {
          data.set(category: name, bought_at: getCurrentBlock().timestamp)
        }
        for name in remove {
          data.unset(category: name)
        }
        self.data[id] = data
      }
    }

    access(contract) fun delete(id: String) {
      self.data[id] = nil
    }

    init() {
      self.data = {}
    }
  }

  /* Info内の構造体 */
  access(all) struct VegeData {

    access(contract) var data: {String: UFix64?}

    /* setter */
    access(contract) fun set(category: String, bought_at: UFix64) {
      self.data[category] = bought_at
    }

    access(contract) fun unset(category: String) {
      self.data[category] = nil
    }

    init() {
      self.data = {}
    }
  }

  // GET
  access(all) fun getCookStockerInfo(id: String): VegeData? {
    return self.info.data[id]
  }
  // PUT
  access(all) fun setCookStockerInfo(id: String, add: [String], remove: [String]) {
    self.info.set(id: id, add: add, remove: remove);
  }
  // DELETE
  access(all) fun deleteCookStockerInfo(id: String) {
    self.info.delete(id: id)
  }

  init() {
    self.info = Info()
  }
}
```
