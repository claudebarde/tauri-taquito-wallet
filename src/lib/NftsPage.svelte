<script lang="ts">
  import { onMount } from "svelte";
  import type { TezosToolkit } from "@taquito/taquito";
  import type { InMemorySigner } from "@taquito/signer";
  import { tzip12 } from "@taquito/tzip12";
  import Card, {
    Content,
    PrimaryAction,
    Media,
    MediaContent,
    Actions,
    ActionButtons,
    ActionIcons
  } from "@smui/card";
  import Button, { Label } from "@smui/button";
  import IconButton, { Icon } from "@smui/icon-button";

  export let signer: InMemorySigner, Tezos: TezosToolkit;

  let nfts = [];
  let loading = true;

  onMount(async () => {
    if (signer) {
      const pkh = await signer.publicKeyHash();
      // fetches NFT from a specific contract
      const exampleContractAddress = "KT1MFJzDbHF5deD5YU6RvsZLFNJkyJwcJXHb";
      const exampleContract = await Tezos.contract.at(
        exampleContractAddress,
        tzip12
      );
      const metadata = await exampleContract.tzip12().getTokenMetadata(0);
      console.log(metadata);
      nfts = [{ contract: exampleContractAddress, tokenId: 0, metadata }];

      loading = false;

      /*try {
        const tokens_res = await fetch(
          `https://api.ghostnet.tzkt.io/v1/tokens/balances?account=${pkh}&token.standard=fa2&limit=10`
        );
        if (tokens_res) {
          const tokens = await tokens_res.json();
          let nfts_ = tokens
            .filter(tk => tk.balance == 1)
            .map(tk => ({
              contract: tk.token.contract.address,
              tokenId: tk.token.tokenId
            }));
          nfts_ = await Promise.all(
            nfts_.map(async nft => {
              try {
                const contract = await Tezos.contract.at(nft.contract, tzip12);
                const metadata = await contract
                  .tzip12()
                  .getTokenMetadata(nft.tokenId);
                return { ...nft, metadata };
              } catch (error) {
                return { ...nft, metadata: undefined };
              }
            })
          );
          nfts = [...nfts_.filter(nft => !!nft.metadata)];
        } else {
          throw "No response from TzKT Tokens API";
        }
      } catch (error) {
        console.error(error);
      } finally {
        loading = false;
      }*/
    } else {
      loading = false;
    }
  });
</script>

<style lang="scss">
  section {
    height: 100%;
    width: 100%;
    text-align: center;
    display: grid;
    padding: 30px;

    .nfts {
      display: flex;
      flex-direction: column;
      justify-content: start;
      align-items: center;
      gap: 20px;
      height: 80%;
      width: 80%;
      margin: 0 auto;
      overflow: auto;

      .nft-image {
        width: 100%;
        height: 100%;
      }
    }
  }
</style>

<section style={`place-items:${signer ? "start" : "center"}`}>
  {#if signer}
    {#if loading}
      <div style="width:100%;text-align:center">Loading, please wait...</div>
    {:else}
      <div class="nfts">
        {#each nfts as nft}
          <Card>
            <Media class="card-media-16x9" aspectRatio="16x9">
              <MediaContent>
                <img
                  class="nft-image"
                  src={`https://gateway.pinata.cloud/ipfs/${nft.metadata.displayUri.slice(
                    7
                  )}`}
                  alt="nft-display"
                />
              </MediaContent>
            </Media>
            <Content class="mdc-typography--body2">
              <PrimaryAction>
                <h2>{nft.metadata.attributes[0].value}</h2>
                <h3>{nft.metadata.publishers[0]}</h3>
              </PrimaryAction>
              <Actions>
                <ActionButtons>
                  <Button>
                    <Label>View</Label>
                  </Button>
                  <Button>
                    <Label>Sell</Label>
                  </Button>
                </ActionButtons>
                <ActionIcons>
                  <IconButton
                    aria-label="Add to favorites"
                    title="Add to favorites"
                  >
                    <Icon class="material-icons" on>favorite</Icon>
                    <Icon class="material-icons">favorite_border</Icon>
                  </IconButton>
                  <IconButton class="material-icons" title="Share">
                    share
                  </IconButton>
                  <IconButton class="material-icons" title="More options">
                    more_vert
                  </IconButton>
                </ActionIcons>
              </Actions>
            </Content>
          </Card>
        {:else}
          <div>No NFT to display</div>
        {/each}
      </div>
    {/if}
  {:else}
    <div style="text-align:center;margin-top:-100px">
      <div>No account connected</div>
      <div>Please go the account tab and connect with your private key</div>
    </div>
  {/if}
</section>
