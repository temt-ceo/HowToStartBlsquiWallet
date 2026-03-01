# Blsqui | The Sovereign Wallet Standard

## Bridging Industrial-Grade HSM Security with Seamless Web3 Gaming UX.

### 🦑  Overview
Blsqui is the Semantic Layer for the Flow blockchain. We solve the "Black Box" transaction problem by translating complex Cadence execution into human-readable, visually verified digital assets. Built for the Cadence 1.0 (Crescendo) era, Blsqui is engineered to be the secure gateway for the next generation of global eSports.

### 🛠 Strategic Architecture
Semantic Authorization: Leverages FLIX (Flow Interaction Templates) to provide players with real-time, visual verification of transaction intent—eliminating blind signing.

Multi-Network Cryptography: Adaptive support for NIST-standard hashing (SHA2-256 / SHA3-256) and dual-curve signatures (P256 and secp256k1) to ensure Mainnet compliance.

HSM-First Philosophy: Architected to interface with enterprise Hardware Security Modules (HSM), bringing Japanese industrial-grade safety to the open web.

### 🚀 Tech Stack
Full FCL-JS Wallet Discovery & Handshake compliance.

### 📈 Roadmap
Phase 1 (Current): Testnet Beta & Semantic UI Launch.

Phase 2: Mainnet Migration & Strategic HSM Partnership Integration.

Phase 3: Institutional Security Audit & FLIX-based Scalability.

### 🤝 Strategic Inquiry
We are currently opening discussions for infrastructure partners and institutional funding.

HSM Integration Standards

Strategic Infrastructure Partnership

Cadence 1.0 Security Migration

Technical Lead: inquiry@tem-technologies.com

#### Developed by TEM Technologies Co. & Blsqui Co., Ltd.

<br><br><br>

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
<img width="400" alt="photo2"  src="https://github.com/user-attachments/assets/f696409e-ac63-4eea-8a2e-d4214675155f" />

<br><br>

### 7. メールアドレスを入れてContinueを押します。

<img width="400" alt="photo3" src="https://github.com/user-attachments/assets/d134a467-32ae-461b-8b86-de359551a7d1" />

<br><br>

### 8. 入力したメールアドレスに6桁の数字が送られます。それをコピーしてください。
<img width="400" alt="photo4" src="https://github.com/user-attachments/assets/832ca6f2-ea4a-4d19-a0ef-a48e8afb4b13" />

<br><br>

### 9. Blsquiウォレットに戻って入力してください。
<img width="400" alt="photo5" src="https://github.com/user-attachments/assets/95bc7ce5-e0f1-4c40-8e99-f30cea217833" />

<br><br>

#### 10. Flow blockchain のアドレスが発行されて、アカウントに紐付くブロックチェーンのストレージ内容をゲーム開発者が自由に考えて変更することが可能なブロックチェーンゲームを作成することができます。
<img width="400" alt="photo6" src="https://github.com/user-attachments/assets/bb7c8ed6-7bd1-450a-9291-396c2ad7dc0e" />

:art: Flow blockchain ではアドレスが作られた時からストレージのデポジット費用がわりの0.001FLOWトークンを所有しています。<br>
