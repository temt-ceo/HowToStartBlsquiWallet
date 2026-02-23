# Blsqui Walletの始め方
<b>Blsquiはゲームに特化するようにInteraction Templatesで設計されたカストディアル・ウォレットです。安全が強化され、ゲーム資産に対して強い個性を持ったウォレットになっています。</b>
<hr>

### 以下の手順でBlsquiウォレットを呼び出すことができるゲームの制作環境を準備できます。
<br>

### 1. Svelte Webフレームワークをインストール
```
# Initialize a new Vite project with Svelte framework
npm create vite@latest web3-game-frontend -- --template svelte-ts
```

Svelte は素のHTMLを生成するので、文法はReactに近いですが既存のWebフレームワーク(React, Vue)に比べて高速で、ゲーム作りに適している、と評判です。

また、その特徴からブロックチェーンライブラリを全く設定なしで使える、という大きな利点があります。

<br><br>

### 2. ブロックチェーンライブラリをインストール
```
npm install @onflow/fcl
```

<br><br>

### 3. import文でconfigとauthorizeメソッドをimportする
```
import * as fcl from "@onflow/fcl";
```

<br><br>

### 4. ブロックチェーンに繋ぐ設定をする
```
// FOR TESTING (Testnet)
fcl.config({
  "discovery.wallet": "https://lab.blsqui.net/authn", // Blsqui Discovery Point
  "discovery.wallet.method": "IFRAME/RPC",
  "accessNode.api": "https://rest-testnet.onflow.org",
  "flow.network": "testnet",
  "app.detail.title": "Your Game Name",
  "app.detail.icon": "https://yourgame.com/icon.png"
})

// FOR PRODUCTION (Mainnet)
fcl.config({
  "discovery.wallet": "https://wallet.blsqui.net/authn", // Blsqui Discovery Point
  "discovery.wallet.method": "IFRAME/RPC",
  "accessNode.api": "https://rest-testnet.onflow.org",
  "flow.network": "mainnet",
  "app.detail.title": "Your Game Name",
  "app.detail.icon": "https://yourgame.com/icon.png"
})
```

<br><br>

### 5. ボタンを配置してclickイベントにauthorizeメソッドを紐付ける
```
<style>
  .blsqui-btn { background: #5d5fef; color: white; padding: 12px 24px; border-radius: 12px; font-weight: 600; border: none; cursor: pointer; transition: transform 0.1s; }
  .blsqui-btn:hover { transform: scale(1.02); background: #4a4cd9; }
</style>

<button class="blsqui-btn" on:click={fcl.authenticate}>Sign In with Blsqui</button>
```
<img width="917" height="649" alt="photo1" src="https://github.com/user-attachments/assets/4bcdd094-37c3-48c7-b06a-c8988174eb7c" />

<br><br>

### 6. ボタンを押すとBlsquiウォレットが出現します。
<img width="918" height="657" alt="photo2" src="https://github.com/user-attachments/assets/ea5d7ef2-63cf-437b-83a1-26c419ad7d60" />

<br><br>

### 7. メールアドレスを入れてContinueを押します。
（調整中）
```

```

<br><br>

### 8. 入力したメールアドレスに6桁の数字が送られます。それをコピーしてください。
（調整中）
```

```

<br><br>

### 9. Blsquiウォレットに戻って入力してください。
```

```

<br><br>

### 10. Flow blockchain のアドレスが発行されてBlaquiウォレットにアドレスが表示されます。(Flow blockchain ではアドレスが作られた時からストレージのデポジット費用がわりの0.001FLOWトークンを所有しています。)
```

```
