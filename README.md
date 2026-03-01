# Blsqui | The Sovereign Wallet Standard

## Bridging Industrial-Grade HSM Security with Seamless Web3 Gaming UX.

### 🦑  Overview
Blsqui is the Semantic Layer for the Flow Blockchain. We solve the "Black Box" transaction problem by translating complex Cadence execution into human-readable, visually verified digital assets. Built for the Cadence 1.0 (Crescendo) era, Blsqui is engineered to be the secure gateway for the next generation of global eSports.

### 🛠 Strategic Architecture
Designed for engineers who demand both performance and provenance.

Semantic Authorization: Leverages FLIX (Flow Interaction Templates) to provide players with real-time, visual verification of transaction intent—eliminating blind signing.

Multi-Network Cryptography: Adaptive support for NIST-standard hashing (SHA2-256 / SHA3-256) and dual-curve signatures (P256 and secp256k1) to ensure Mainnet compliance.

HSM-First Philosophy: Architected to interface with enterprise Hardware Security Modules (HSM), bringing Japanese industrial-grade safety to the open web.

### 🚀 Tech Stack
Core: SvelteKit + Vite (Optimized for <100ms Interaction Latency)

Network: Caddy (Automated SSL & Strict CSP frame-ancestors Security)

Auth: JWT-Session Management + Secure SMTP OTP Delivery

Blockchain: Full FCL-JS Wallet Discovery & Handshake compliance.

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
<img width="400" alt="photo2" src="https://github.com/user-attachments/assets/ea5d7ef2-63cf-437b-83a1-26c419ad7d60" />

<br><br>

### 7. メールアドレスを入れてContinueを押します。
（現在デバッグ中です）
```

```

<br><br>

### 8. 入力したメールアドレスに6桁の数字が送られます。それをコピーしてください。
<img width="400" alt="photo4" src="https://github.com/user-attachments/assets/4da737cb-57ab-48b7-ba53-9f898d7eee3a" />

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
