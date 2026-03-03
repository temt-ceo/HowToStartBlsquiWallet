<script lang="ts">
  import svelteLogo from './assets/svelte.svg'
  import viteLogo from '/vite.svg'
  import { config, authenticate, unauthenticate, currentUser, mutate, authz } from '@onflow/fcl';
  let blsquiWalletUser: any;

  // FOR TESTING (Testnet)
  config({
    "discovery.wallet": "https://lab.blsqui.net/authn", // Blsqui Discovery Point
    "accessNode.api": "https://rest-testnet.onflow.org",
    "flow.network": "testnet",
    "app.detail.title": "Your Game Name",
    "app.detail.icon": "https://yourgame.com/icon.png"
  });
  currentUser.subscribe(async (user) => {
    blsquiWalletUser = user
  });

  const transactionTest = async function () {
    await mutate({
      cadence: ` transaction { prepare(signer: &Account) { log("Squid Authz Test Successful") } } `,
      args: (arg, t) => [],
      proposer: authz,
      payer: authz,
      authorizations: [authz],
      limit: 999
    });
  };
</script>

<main>
  <div>
    <a href="https://vite.dev" target="_blank" rel="noreferrer">
      <img src={viteLogo} class="logo" alt="Vite Logo" />
    </a>
    <a href="https://svelte.dev" target="_blank" rel="noreferrer">
      <img src={svelteLogo} class="logo svelte" alt="Svelte Logo" />
    </a>
  </div>
  <h1>Vite + Svelte</h1>

  <div class="card">

    {#if blsquiWalletUser?.addr}
      <button class="blsqui-btn" on:click={unauthenticate}>Sign Out from Blsqui</button>
      <button class="blsqui-btn" on:click={transactionTest}>トランザクションテスト</button>
    {:else}
      <button class="blsqui-btn" on:click={authenticate}>Sign In with Blsqui</button>
    {/if}
    {blsquiWalletUser?.addr}
  </div>
</main>

<style>
  .blsqui-btn { background: #5d5fef; color: white; padding: 12px 24px; border-radius: 12px; font-weight: 600; border: none; cursor: pointer; transition: transform 0.1s; }
  .blsqui-btn:hover { transform: scale(1.02); background: #4a4cd9; }

  .logo {
    height: 6em;
    padding: 1.5em;
    will-change: filter;
    transition: filter 300ms;
  }
  .logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
  }
  .logo.svelte:hover {
    filter: drop-shadow(0 0 2em #ff3e00aa);
  }
  .read-the-docs {
    color: #888;
  }
</style>
