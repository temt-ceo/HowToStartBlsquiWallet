# Blsqui Walletの始め方

<p>:zap: BlsquiはInteraction Templatesでゲームに特化するように設計されたカストディアル・ウォレットです。安全性が強化され、ゲーム資産に対して強い個性を持ったウォレットになっています。</p>

<br><br>

### :wrench: 以下の手順でBlsquiウォレットを呼び出すことができるゲームの制作環境を準備できます。
<br>

### 1. Svelte Webフレームワークをインストール
```
npm create vite@latest web3-game-frontend -- --template svelte-ts
```
:sparkles: インストールが終わると自動的に[localhost:5173](http://localhost:5173/)がブラウザに立ち上がります。コードを編集するとブラウザに自動的に反映されます。<br>
:sparkles: Svelte は素のHTMLを生成するので、文法はReactに近いですが既存のWebフレームワーク(React, Vue)に比べて高速でゲーム作りに適していると評判です。また、その特徴からブロックチェーンライブラリを全く設定なしで使える、という大きな利点があります。

<br><br>

### 2. ブロックチェーンライブラリをインストール
```
npm install @onflow/fcl
```

<br><br>

### 3. ライブラリのconfigとauthorizeメソッドをimportする
```
import { config, authenticate, unauthenticate, currentUser, mutate, tx } from '@onflow/fcl';
```

<br><br>

### 4. ブロックチェーンに繋ぐ設定をする

:gear: Testnet
```
config({
  "discovery.wallet": "https://lab.blsqui.net/authn",
  "accessNode.api": "https://rest-testnet.onflow.org",
  "flow.network": "testnet",
  "app.detail.title": "Your Game Name",
  "app.detail.icon": "https://yourgame.com/icon.png"
})
```
:gear: Mainnet
```
config({
  "discovery.wallet": "https://wallet.blsqui.net/authn",
  "accessNode.api": "https://rest-mainnet.onflow.org",
  "flow.network": "mainnet",
  "app.detail.title": "Your Game Name",
  "app.detail.icon": "https://yourgame.com/icon.png"
})
```

<br><br>

### 5. HTMLのボタンを配置してclickイベントにauthorizeメソッドを紐付ける
<img width="400" alt="photo1" src="https://github.com/user-attachments/assets/4bcdd094-37c3-48c7-b06a-c8988174eb7c" />

```
<style>.blsqui-btn { background: #5d5fef; color: white; padding: 12px 24px; border-radius: 12px; font-weight: 600; border: none; cursor: pointer; transition: transform 0.1s; } .blsqui-btn:hover { transform: scale(1.02); background: #4a4cd9; }</style>
<button class="blsqui-btn" on:click={authenticate}>Sign In with Blsqui</button>
```

<br><br>

### 6. ボタンを押すとBlsquiウォレットが出現します。
<img width="400" alt="photo2" src="https://github.com/user-attachments/assets/ea5d7ef2-63cf-437b-83a1-26c419ad7d60" />

<br><br>

### 7. メールアドレスを入れてContinueを押します。
（現在デバッグ中です）
```

```

<br><br>

### 8. 入力したメールアドレスに6桁の数字が送られます。それをコピーしてください。
<img width="400" alt="photo5"  src="https://github.com/user-attachments/assets/35dc6d57-313c-4002-bab6-23a98903ddf8" />

<br><br>

### 9. Blsquiウォレットに戻って入力してください。
（現在デバッグ中です）
```

```


<br><br>

### 10. Flow blockchain のアドレスが発行されてBlaquiウォレットにアドレスが表示されます。

（現在デバッグ中です）
```

```
:art: Flow blockchain ではアドレスが作られた時からストレージのデポジット費用がわりの0.001FLOWトークンを所有しています。<br>
