# シューティングeSportsゲームのトランザクションコード例

#### ゲームプレイヤーのリソースをストレージに保存する

```
  const txId = await mutate({
    cadence: `
      import "OragaESports"
      import "FlowToken"
      import "FungibleToken"

      transaction(nickname: String) {
        prepare(signer: auth(Storage, Capabilities) &Account) {
          let FlowTokenReceiver = signer.capabilities.get<&{FungibleToken.Receiver}>(/public/flowTokenReceiver)

          signer.storage.save(<- OragaESports.createGamer(nickname: nickname, flow_vault_receiver: FlowTokenReceiver), to: /storage/OragaESportsGamer)
          let cap = signer.capabilities.storage.issue<&OragaESports.Gamer>(/storage/OragaESportsGamer)
          signer.capabilities.publish(cap, at: /public/OragaESportsGamer)
        }
        execute {
          log("success")
        }
      }
    `,
    args: (arg, t) => [arg(nickname, t.String)],
    proposer: authz,
    payer: authz,
    authorizations: [authz],
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

#### ゲーム代金を支払ってゲームを開始する

```
  const txId = await mutate({
    cadence: `
      import "OragaESports"
      import "FlowToken"
      import "FungibleToken"

      transaction() {
        prepare(signer: auth(BorrowValue) &Account) {
          let payment <- signer.storage.borrow<auth(FungibleToken.Withdraw) &FlowToken.Vault>(from: /storage/flowTokenVault)!.withdraw(amount: 1.1) as! @FlowToken.Vault

          let gamer = signer.storage.borrow<&OragaESports.Gamer>(from: /storage/OragaESportsGamer)
              ?? panic("Could not borrow reference to the Owner's Gamer Resource.")
          gamer.insert_coin(payment: <- payment)
        }
        execute {
          log("success")
        }
      }
    `,
    args: (arg, t) => [],
    proposer: authz,
    payer: authz,
    authorizations: [authz],
    limit: 999,
  });
```

#### ゲームにチップを支払う

```
  const txId = await mutate({
    cadence: `
      import "OragaESports"
      import "FlowToken"
      import "FungibleToken"

      transaction(amount: UFix64) {
        prepare(signer: auth(BorrowValue) &Account) {
          pre {
            amount == 1.0 || amount == 5.0: "tip is not 1.0FLOW or 5.0FLOW."
          }
          let tip <- signer.storage.borrow<auth(FungibleToken.Withdraw) &FlowToken.Vault>(from: /storage/flowTokenVault)!.withdraw(amount: amount) as! @FlowToken.Vault

          let gamer = signer.storage.borrow<&OragaESports.Gamer>(from: /storage/OragaESportsGamer)
              ?? panic("Could not borrow reference to the Owner's Gamer Resource.")
          gamer.tipping(tip: <- tip)
        }
        execute {
          log("success")
        }
      }
    `,
    args: (arg, t) => [arg(amount, t.UFix64)],
    proposer: authz,
    payer: authz,
    authorizations: [authz],
    limit: 999,
  });
```

#### ゲームの結果を処理（勝利の場合は賞金を支払う。負けの場合は賞金を増やす。）

```
    transaction = `
      import OragaESports from 0xb576a3926d239682

      transaction(gamerId: UInt, outcome: Bool) {
        prepare(signer: auth(BorrowValue) &Account) {
          let admin = signer.storage.borrow<&OragaESports.Admin>(from: /storage/OragaESportsAdmin)
            ?? panic("Could not borrow reference to the Administrator Resource.")
          admin.shootingGameOutcome(gamerId: gamerId, outcome: outcome)
        }
        execute {
          log("success")
        }
      }
    `;
```

#### 無料プレイをする

```
    transaction = `
      import OragaESports from 0xb576a3926d239682

      transaction(gamerId: UInt) {
        prepare(signer: auth(BorrowValue) &Account) {
          let admin = signer.storage.borrow<&OragaESports.Admin>(from: /storage/OragaESportsAdmin)
            ?? panic("Could not borrow reference to the Administrator Resource.")
          admin.useTipJarForFreePlay(gamerId: gamerId)
        }
        execute {
          log("success")
        }
      }
    `;
```

#### スマートコントラクト

```
import "FlowToken"
import "FungibleToken"

access(all) contract OragaESports {

  access(self) var totalCount: UInt
  access(self) let GamerFlowTokenVault: {UInt: Capability<&{FungibleToken.Receiver}>}
  access(self) let FlowTokenVault: Capability<&{FungibleToken.Receiver}>
  access(self) let gamersInfo: GamersInfo

  access(all) event GamerCreatted(gamerId: UInt)
  access(all) event WonThePrize(gamerId: UInt, amount: UFix64)

  // [Struct] GamersInfo
  access(all) struct GamersInfo {
    access(contract) var currentPrize: UInt
    access(contract) var tryingPrize: {UInt: UInt}
    access(contract) var lastTimePlayed: {UInt: UFix64?}
    access(contract) var prizeWinners: {UInt: GamerPrizeInfo}
    access(contract) var freePlayCount: {UInt: UInt8}
    access(contract) var tipJarBalance: UFix64

    access(contract) fun setCurrentPrize(added: UInt, gamerId: UInt, paid: Bool): UInt {
      if (paid == false) {
        self.currentPrize = self.currentPrize + added
        self.unsetTryingPrize(gamerId: gamerId)
        return 0
      } else {
        if let paidPrize = self.tryingPrize[gamerId] {
          if (self.currentPrize - paidPrize >= 0) {
            self.currentPrize = self.currentPrize - paidPrize
            self.unsetTryingPrize(gamerId: gamerId)
            if let prizeHistory = self.prizeWinners[gamerId] {
              self.setPrizeWinners(gamerId: gamerId, prize: paidPrize + prizeHistory.prize, nickname: nil)
            } else {
              self.setPrizeWinners(gamerId: gamerId, prize: paidPrize, nickname: nil)
            }
            return paidPrize + added
          } else {
            // In this case, any other person snatched the prize while playing a game. That's why the prize is only the fee this player paid.
            self.currentPrize = self.currentPrize - added
            self.unsetTryingPrize(gamerId: gamerId)
            if let prizeHistory = self.prizeWinners[gamerId] {
              self.setPrizeWinners(gamerId: gamerId, prize: added + prizeHistory.prize, nickname: nil)
            } else {
              self.setPrizeWinners(gamerId: gamerId, prize: added, nickname: nil)
            }
            return added
          }
        }
        panic("Error. Oops, something is not good.")
      }
    }

    access(contract) fun unsetTryingPrize(gamerId: UInt) {
      self.tryingPrize[gamerId] = 0
      self.lastTimePlayed[gamerId] = nil
    }

    access(contract) fun setTryingPrize(gamerId: UInt) {
      self.tryingPrize[gamerId] = self.currentPrize
      self.lastTimePlayed[gamerId] = getCurrentBlock().timestamp
    }

    access(contract) fun setPrizeWinners(gamerId: UInt, prize: UInt, nickname: String?) {
      if let prizeWinnerInfo = self.prizeWinners[gamerId] {
        prizeWinnerInfo.setPrize(prize: prize)
        if (prize > 0) {
          prizeWinnerInfo.setTotalCountUp()
        }
        self.prizeWinners[gamerId] = prizeWinnerInfo
      } else {
        if (nickname != nil) {
          self.prizeWinners[gamerId] = GamerPrizeInfo(prize: prize, gamerId: gamerId, nickname: nickname!, totalCount: prize > 0 ? 1 : 0)
        }
      }
    }

    access(contract) fun setFreePlayCount(gamerId: UInt) {
      self.freePlayCount[gamerId] = self.freePlayCount[gamerId] == nil ? 1 : (self.freePlayCount[gamerId]! + 1)
    }

    access(contract) fun setTipJarBalance(amount: UFix64) {
      self.tipJarBalance = self.tipJarBalance + amount
    }

    access(contract) fun useTipJarBalance() {
      if (self.tipJarBalance > 0.0) {
        self.tipJarBalance = self.tipJarBalance - 1.0
      } else {
        panic("Sorry, tip jar is empty..")
      }
    }

    init() {
      self.currentPrize = 0
      self.tryingPrize = {}
      self.lastTimePlayed = {}
      self.prizeWinners = {}
      self.freePlayCount = {}
      self.tipJarBalance = 0.0
    }
  }

  // [Struct] GamerPrizeInfo
  access(all) struct GamerPrizeInfo {
    access(contract) var prize: UInt
    access(contract) let gamerId: UInt
    access(contract) let gamerName: String
    access(contract) var totalCount: UInt

    access(contract) fun setPrize(prize: UInt) {
      self.prize = self.prize + prize
    }
    access(contract) fun setTotalCountUp() {
      self.totalCount = self.totalCount + 1
    }

    init(prize: UInt, gamerId: UInt, nickname: String, totalCount: UInt) {
      self.prize = prize
      self.gamerId = gamerId
      self.gamerName = nickname
      self.totalCount = totalCount
    }
  }

  access(all) fun getGamersInfo(): GamersInfo {
    return self.gamersInfo
  }

  access(all) resource Gamer {
    access(all) let gamerId: UInt
    access(all) let nickname: String

    access(all) fun insert_coin(payment: @FlowToken.Vault) {
      pre {
        payment.balance == 1.1: "payment is not 1.1FLOW coin."
      }
      // There is a possibility that double withdrawal may occur by the following logic, comment out this logic.(July/6/2025)
      // if let isStillPlayed = OragaESports.gamersInfo.lastTimePlayed[self.gamerId] {
      //   if (isStillPlayed != nil && isStillPlayed! + 80.0 < getCurrentBlock().timestamp) { // 60 + 10 * 2 (game time + transaction)
      //     // NOTE. If a player cut the network after the game won, we don't care about it.
      //     let prizePaid = OragaESports.gamersInfo.setCurrentPrize(added: 1, gamerId: self.gamerId, paid: false) // set lose
      //   }
      // }
      // if let challenged = OragaESports.gamersInfo.tryingPrize[self.gamerId] {
      //   if (challenged > 0) {
      //     panic("You are now on playing the game. Payment is not accepted.")
      //   }
      // }
      OragaESports.gamersInfo.setTryingPrize(gamerId: self.gamerId)
      OragaESports.FlowTokenVault.borrow()!.deposit(from: <- payment)
    }

    access(all) fun tipping(tip: @FlowToken.Vault) {
      pre {
        tip.balance == 1.0 || tip.balance == 5.0: "tip is not 1.0FLOW or 5.0FLOW coin."
      }
      OragaESports.gamersInfo.setTipJarBalance(amount: tip.balance)
      OragaESports.FlowTokenVault.borrow()!.deposit(from: <- tip)
    }

    init(nickname: String) {
      OragaESports.totalCount = OragaESports.totalCount + 1
      self.gamerId = OragaESports.totalCount
      self.nickname = nickname
      OragaESports.gamersInfo.setPrizeWinners(gamerId: self.gamerId, prize: 0, nickname: nickname)
      emit GamerCreatted(gamerId: self.gamerId)
    }
  }

  access(all) fun createGamer(nickname: String, flow_vault_receiver: Capability<&{FungibleToken.Receiver}>): @OragaESports.Gamer {
    let gamer <- create Gamer(nickname: nickname)

    if (OragaESports.GamerFlowTokenVault[gamer.gamerId] == nil) {
      OragaESports.GamerFlowTokenVault[gamer.gamerId] = flow_vault_receiver
    }
    return <- gamer
  }

  /*
  ** [Resource] Admin (Game Server Processing)
  */
  access(all) resource Admin {
    /*
    ** Save the Gamer's shooting game outcome
    */
    access(all) fun shootingGameOutcome(gamerId: UInt, outcome: Bool) {
      let prizePaid = OragaESports.gamersInfo.setCurrentPrize(added: 1, gamerId: gamerId, paid: outcome)
      if (prizePaid > 0) {
        // Pay the prize.
        let reward <- OragaESports.account.storage.borrow<auth(FungibleToken.Withdraw) &{FungibleToken.Provider}>(from: /storage/flowTokenVault)!.withdraw(amount: UFix64.fromString(prizePaid.toString().concat(".0"))!) as! @FlowToken.Vault
        OragaESports.GamerFlowTokenVault[gamerId]!.borrow()!.deposit(from: <- reward)
        emit WonThePrize(gamerId: gamerId, amount: UFix64.fromString(prizePaid.toString().concat(".0"))!)
      }
    }

    /*
    ** Start the free play using tip jar coin.
    */
    access(all) fun useTipJarForFreePlay(gamerId: UInt) {
      if let freePlayed = OragaESports.gamersInfo.freePlayCount[gamerId] {
        if (freePlayed >= 2) {
          panic("It's not acceptable.")
        }
        // To avoid double withdrawal using multi monitor gaming, comment out this logic.(July/6/2025)
        // if let isStillPlayed = OragaESports.gamersInfo.lastTimePlayed[gamerId] {
        //   if (isStillPlayed != nil && isStillPlayed! + 80.0 < getCurrentBlock().timestamp) { // 60 + 10 * 2 (game time + transaction)
        //     // NOTE. If a player cut the network after the game won, we don't care about it.
        //     let prizePaid = OragaESports.gamersInfo.setCurrentPrize(added: 1, gamerId: gamerId, paid: false) // set lose
        //   }
        // }
        // if let challenged = OragaESports.gamersInfo.tryingPrize[gamerId] {
        //   if (challenged > 0) {
        //     panic("You are now on playing the game. Payment is not accepted.")
        //   }
        // }
        OragaESports.gamersInfo.freePlayCount[gamerId] = freePlayed + 1
        OragaESports.gamersInfo.setTryingPrize(gamerId: gamerId)
        OragaESports.gamersInfo.useTipJarBalance()
      } else {
        OragaESports.gamersInfo.freePlayCount[gamerId] = 1
        OragaESports.gamersInfo.setTryingPrize(gamerId: gamerId)
        OragaESports.gamersInfo.useTipJarBalance()
      }
    }
  }

  init() {
    self.account.storage.save( <- create Admin(), to: /storage/OragaESportsAdmin) // grant admin resource
    self.totalCount = 0
    self.GamerFlowTokenVault = {}
    self.FlowTokenVault = self.account.capabilities.get<&{FungibleToken.Receiver}>(/public/flowTokenReceiver)
    self.gamersInfo = GamersInfo()
  }
}
```
