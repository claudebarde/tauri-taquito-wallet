<script lang="ts">
  import { onMount } from "svelte";
  import { TezosToolkit } from "@taquito/taquito";
  import { InMemorySigner } from "@taquito/signer";
  import { Tzip12Module } from "@taquito/tzip12";
  import Header from "./lib/Header.svelte";
  import Interactions from "./lib/Interactions.svelte";
  import History from "./lib/History.svelte";
  import NftsPage from "./lib/NftsPage.svelte";
  import Dialog, { Title, Content, Actions } from "@smui/dialog";
  import Button, { Label } from "@smui/button";

  let rpcUrl = "https://ghostnet.ecadinfra.com/";
  let currentPage: "interactions" | "history" | "nfts-page" = "interactions";
  let Tezos: TezosToolkit;
  let signer: InMemorySigner;
  // edskRpm2mUhvoUjHjXgMoDRxMKhtKfww1ixmWiHCWhHuMEEbGzdnz8Ks4vgarKDtxok7HmrEo1JzkXkdkvyw7Rtw6BNtSd7MJ7
  let newSignerError = false;

  const changeView = event => {
    switch (event.detail) {
      case "account":
        currentPage = "interactions";
        break;
      case "history":
        currentPage = "history";
        break;
      case "nfts":
        currentPage = "nfts-page";
        break;
    }
  };

  const connect = event => {
    const privateKey = event.detail;
    try {
      signer = new InMemorySigner(privateKey);
      Tezos.setSignerProvider(signer);
    } catch (error) {
      console.error(error);
      newSignerError = true;
    }
  };

  const reset = () => {
    signer = undefined;
  };

  onMount(async () => {
    Tezos = new TezosToolkit(rpcUrl);
    Tezos.addExtension(new Tzip12Module());
  });
</script>

<style lang="scss">
  main {
    height: 100%;
    width: 100%;
    position: relative;
  }
</style>

<main>
  <Header on:change-view={changeView} />
  {#if currentPage === "interactions"}
    <Interactions {signer} {Tezos} on:new-key={connect} on:reset={reset} />
  {:else if currentPage === "history"}
    <History {signer} />
  {:else}
    <NftsPage {signer} {Tezos} />
  {/if}
  <Dialog
    bind:open={newSignerError}
    aria-labelledby="wrong-private-key"
    aria-describedby="wrong-private-key-content"
    style="background-color:blue;color:white"
  >
    <Title id="wrong-private-key">Unable to set up the signer</Title>
    <Content id="wrong-private-key-content">
      The wallet couldn't be' set up with the key you provided. The private key
      may be wrong or another issue occurred.
    </Content>
    <Actions>
      <Button on:click={() => (newSignerError = false)}>
        <Label>OK</Label>
      </Button>
    </Actions>
  </Dialog>
</main>
