<script>
  import Proxy from './lib/Proxy';
  import Networks from './conf/networks';
  import Token from './components/Token.svelte';
  import { createEventDispatcher } from 'svelte';

  export let contract;
  export let tokenId = null;
  export let owner = null;
  export let network = Networks.mainnet;
  export let resizable = true;
  export let width = '388px';
  export let height = '560px';
  export const version = process.env.npm_package_version;

  let chaindId = Networks.mainnet;
  if ('string' === typeof network && Networks[network]) {
    chaindId = Networks[network];
  } else {
    chaindId = parseInt(network);
  }

  let loaded = false;
  let error;
  let proxy;
  let uris;

  $: {
    if (contract && tokenId !== null) {
      if (!uris) {
        getURIs();
      } else if (uris.tokenURI) {
        loaded = true;
      }
    }
  }

  async function getURIs() {
    try {
      proxy = new Proxy(contract, tokenId, chaindId);
      await proxy.connect();
      uris = await proxy.uris();
    } catch (e) {
      error = e.message;
    }
  }
</script>

<div class="beyondembeddable">
  {#if error}
    <p class="beyondembeddable__error">{error}</p>
  {:else if loaded}
    <Token
      {owner}
      {uris}
      {resizable}
      {width}
      {height}
      on:loaded
      on:error={(e) => (error = e.detail.message)} />
  {/if}
</div>
