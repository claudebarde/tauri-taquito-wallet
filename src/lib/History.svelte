<script lang="ts">
  import { onMount } from "svelte";
  import type { InMemorySigner } from "@taquito/signer";
  import List, { Item, Text, PrimaryText, SecondaryText } from "@smui/list";

  export let signer: InMemorySigner;

  let ops = [];

  onMount(async () => {
    if (signer) {
      const pkh = await signer.publicKeyHash();
      try {
        const operations_res = await fetch(
          `https://api.ghostnet.tzkt.io/v1/accounts/${pkh}/operations?type=delegation,reveal,origination,transaction&limit=10`
        );
        if (operations_res) {
          const operations = await operations_res.json();
          ops = operations.map(op => ({
            type: op.type,
            timestamp: op.timestamp,
            hash: op.hash
          }));
        }
      } catch (error) {
        console.error(error);
      }
    }
  });
</script>

<style lang="scss">
  section {
    height: 100%;
    width: 100%;
    display: grid;
    padding: 30px;

    .history {
      display: flex;
      flex-direction: column;
      justify-content: start;
      align-items: stretch;
      gap: 20px;
      height: 80%;
      width: 100%;
      overflow: auto;
    }
  }
</style>

<section style={`place-items:${signer ? "start" : "center"}`}>
  {#if signer}
    <div class="history">
      <List threeLine nonInteractive>
        {#each ops as op}
          <Item>
            <Text>
              <PrimaryText>{op.hash}</PrimaryText>
              <SecondaryText>{op.type}</SecondaryText>
              <SecondaryText>{op.timestamp}</SecondaryText>
            </Text>
          </Item>
        {/each}
      </List>
    </div>
  {:else}
    <div style="text-align:center;margin-top:-100px">
      <div>No account connected</div>
      <div>Please go the account tab and connect with your private key</div>
    </div>
  {/if}
</section>
