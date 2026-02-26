# ライドシェアサービスのトランザクションコード例

#### 新しいドライバーの情報を保存する

```
    transaction = `
      import RideShare from 0xb576a3926d239682
      import FlowToken from 0x1654653399040a61
      import FungibleToken from 0xf233dcee88fe0abe

      transaction(driverId: UInt, driverAddress: Address, keys: [String], values: [String]) {
        prepare(signer: &Account) {
          let FlowTokenReceiver = getAccount(driverAddress)
            .capabilities
            .get<&{FungibleToken.Receiver}>(/public/flowTokenReceiver)

          RideShare.setDriverInfo(driverId: driverId, keys: keys, values: values, flow_vault_receiver: FlowTokenReceiver)
        }
        execute {
          log("success")
        }
      }
    `;
    txId = await mutate({
      cadence: transaction,
      args: (arg, t) => [
        arg(message.driverId, t.UInt),
        arg(message.driverAddress, t.Address),
        arg(message.keys, t.Array(t.String)),
        arg(message.values, t.Array(t.String)),
      ],
      proposer: authFunctionForProposer,
      payer: authFunction,
      authorizations: [authFunction],
      limit: 999,
    });
```

#### ライドシェアの注文を行う

```
  const txId = await mutate({
    cadence: `
      import "RideShare"
      import "FlowToken"
      import "FungibleToken"

      transaction(execTime: UFix64, driverId: UInt, start: String, goal: String, price: UFix64) {
        prepare(signer: auth(BorrowValue) &Account) {
          let payment <- signer.storage.borrow<auth(FungibleToken.Withdraw) &FlowToken.Vault>(from: /storage/flowTokenVault)!.withdraw(amount: price) as! @FlowToken.Vault
          RideShare.newOrder(payment: <- payment, execTime: execTime, driverId: driverId, start: start, goal: goal)
        }
        execute {
          log("success")
        }
      }
    `,
    args: (arg, t) => [
      arg(new Date(execDateTime).getTime() / 1000 + 0.0001, t.UFix64), // 少数でないとエラーになるので1ミリ秒を足す(Cadenceでは秒以下は少数)
      arg(driverId, t.UInt),
      arg(start, t.String),
      arg(goal, t.String),
      arg(Math.floor(price * 1000) / 1000, t.UFix64), // 少数でないとエラーになるが長すぎてもエラーになるので少数を特定桁数までとしている
    ],
    proposer: authz,
    payer: authz,
    authorizations: [authz],
    limit: 999,
  });
```

#### ドライバーに賃金を支払う

```
    transaction = `
      import RideShare from 0xb576a3926d239682

      transaction(driverId: UInt, wage: UFix64) {
        prepare(signer: auth(BorrowValue) &Account) {
          let RideShareAdmin = signer.storage.borrow<&RideShare.Admin>(from: /storage/RideShareAdmin)
            ?? panic("Could not borrow reference to the Administrator Resource.")
          RideShareAdmin.payWage(driverId: driverId, wage: wage)
        }
        execute {
          log("success")
        }
      }
    `;
    txId = await mutate({
      cadence: transaction,
      args: (arg, t) => [
        arg(message.driverId, t.UInt),
        arg(message.wage, t.UFix64),
      ],
      proposer: authFunctionForProposer,
      payer: authFunction,
      authorizations: [authFunction],
      limit: 999,
    });
```

#### ドライバーをレビューする

```
    transaction = `
      import RideShare from 0xb576a3926d239682

      transaction(driverId: UInt, keys: [String], values: [String]) {
        prepare(signer: &Account) {
          RideShare.setDriverInfo(driverId: driverId, keys: keys, values: values, flow_vault_receiver: nil)
        }
        execute {
          log("success")
        }
      }
    `;
    txId = await mutate({
      cadence: transaction,
      args: (arg, t) => [
        arg(message.driverId, t.UInt),
        arg(message.keys, t.Array(t.String)),
        arg(message.values, t.Array(t.String)),
      ],
      proposer: authFunctionForProposer,
      payer: authFunction,
      authorizations: [authFunction],
      limit: 999,
    });
```

#### スマートコントラクト

```
import "FlowToken"
import "FungibleToken"

access(all) contract RideShare {

  access(self) let FlowTokenVault: Capability<&{FungibleToken.Receiver}>
  access(self) let DriverFlowTokenVault: {UInt: Capability<&{FungibleToken.Receiver}>}
  access(self) let info: Info
  access(self) var totalDrivers: UInt // Driver ID

  /* 構造体 */
  access(all) struct Info {

    access(contract) var driverData: {UInt: DriverData}
    access(contract) var orderQueue: [Order]

    /* setter */
    access(contract) fun setDriver(id: UInt, keys: [String], values: [String]): UInt {
      if let saved = self.driverData[id] {
        for index, key in keys {
          saved.set(key: key, value: values[index])
        }
        self.driverData[id] = saved // 更新
        return id
      } else {
        // 新規登録
        RideShare.totalDrivers = RideShare.totalDrivers + 1
        var data = DriverData()
        for index, key in keys {
          data.set(key: key, value: values[index])
        }
        self.driverData[RideShare.totalDrivers] = data
        return RideShare.totalDrivers
      }
    }

    access(contract) fun setOrder(execTime: UFix64, driverId: UInt, start: String, goal: String, wage: UFix64) {
      var order = Order(execTime: execTime, driverId: driverId, start: start, goal: goal, wage: wage)
      self.orderQueue.append(order)

    }

    init() {
      self.driverData = {}
      self.orderQueue = []
    }
  }


  /* Info内の構造体 */
  access(all) struct DriverData {

    access(contract) var data: {String: String}

    /* setter */
    access(contract) fun set(key: String, value: String) {
      self.data[key] = value
    }

    init() {
      self.data = {}
    }
  }

  /* Info内の構造体 */
  access(all) struct Order {

    access(all) let execTime: UFix64
    access(all) let driverId: UInt
    access(all) let start: String
    access(all) let goal: String
    access(all) let wage: UFix64

    init(execTime: UFix64, driverId: UInt, start: String, goal: String, wage: UFix64) {
      self.execTime = execTime
      self.driverId = driverId
      self.start = start
      self.goal = goal
      self.wage = wage
    }
  }

  // GET
  access(all) fun getInfo(): Info {
    return self.info
  }
  // PUT
  access(all) fun setDriverInfo(driverId: UInt, keys: [String], values: [String], flow_vault_receiver: Capability<&{FungibleToken.Receiver}>?) {
    let _driverId = RideShare.info.setDriver(id: driverId, keys: keys, values: values)
    if (flow_vault_receiver != nil) {
      RideShare.DriverFlowTokenVault[_driverId] = flow_vault_receiver
    }
  }

  // PUT
  access(all) fun newOrder(payment: @FlowToken.Vault, execTime: UFix64, driverId: UInt, start: String, goal: String) {
    self.info.setOrder(execTime: execTime, driverId: driverId, start: start, goal: goal, wage: payment.balance * 0.97)
    self.FlowTokenVault.borrow()!.deposit(from: <- payment)
  }

  access(all) resource Admin {
    // ORDER COMPLETE
    access(all) fun payWage(driverId: UInt, wage: UFix64) {
      pre {
        RideShare.DriverFlowTokenVault[driverId] != nil: "Deposit destination is nil, so we can't send wage."
      }
      // Pay the wage.
      let wage <- RideShare.account.storage.borrow<auth(FungibleToken.Withdraw) &{FungibleToken.Provider}>(from: /storage/flowTokenVault)!.withdraw(amount: wage) as! @FlowToken.Vault
      RideShare.DriverFlowTokenVault[driverId]!.borrow()!.deposit(from: <- wage)
    }
  }

  init() {
    self.info = Info()
    self.FlowTokenVault = self.account.capabilities.get<&{FungibleToken.Receiver}>(/public/flowTokenReceiver)
    self.DriverFlowTokenVault = {}
    self.totalDrivers = 0
    self.account.storage.save( <- create Admin(), to: /storage/RideShareAdmin)
  }
}
```
