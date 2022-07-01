<script lang="ts">
  import { createEventDispatcher, afterUpdate } from "svelte";
  import type { TezosToolkit } from "@taquito/taquito";
  import type { InMemorySigner } from "@taquito/signer";
  import Paper, { Subtitle, Content } from "@smui/paper";
  import Textfield from "@smui/textfield";
  import Icon from "@smui/textfield/icon";
  import HelperText from "@smui/textfield/helper-text";
  import Button, { Label } from "@smui/button";
  import type { SnackbarComponentDev } from "@smui/snackbar";
  import Snackbar, { Label as SnackbarLabel } from "@smui/snackbar";

  export let signer: InMemorySigner, Tezos: TezosToolkit;

  let privateKey: string;
  let publicKeyHash: string;
  const dispatch = createEventDispatcher();
  let recipientAddress: string;
  let tezToSend: number;
  let snackbar: SnackbarComponentDev;
  let snackbarText = "";
  let sendingTez = false;

  const sendTez = async () => {
    sendingTez = true;
    snackbarText = `Sending ${tezToSend} tez, please wait`;
    snackbar.open();
    try {
      const op = await Tezos.contract.transfer({
        to: recipientAddress,
        amount: tezToSend
      });
      op.confirmation();
      snackbarText = `${tezToSend} tez successfully sent!`;
      recipientAddress = undefined;
      tezToSend = undefined;
      setTimeout(() => {
        snackbar.close();
      }, 3000);
    } catch (error) {
      console.error(error);
      snackbar.close();
    } finally {
      sendingTez = false;
    }
  };

  afterUpdate(async () => {
    if (signer) {
      publicKeyHash = await signer.publicKeyHash();
    }
  });
</script>

<style lang="scss">
  section {
    height: 100%;
    width: 100%;
    display: grid;
    place-items: center;
    margin-top: -50px; // this is to counter the space taken by the tabs

    .interactions {
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: stretch;
      gap: 20px;
    }
  }
</style>

<section>
  {#if signer}
    <div class="interactions">
      <Paper style="text-align:center">
        <Subtitle>Connected as</Subtitle>
        <Subtitle>{publicKeyHash}</Subtitle>
        <Button
          variant="raised"
          on:click={() => {
            publicKeyHash = undefined;
            privateKey = undefined;
            dispatch("reset");
          }}
        >
          <Label>Disconnect</Label>
        </Button>
      </Paper>
      <Paper>
        <Subtitle>Send tez to a Tezos account</Subtitle>
        <Content>
          <Textfield
            type="text"
            bind:value={recipientAddress}
            label="Recipient's address"
            disabled={sendingTez}
            input$emptyValueUndefined
            style="width: 100%;"
            helperLine$style="width: 100%;"
          >
            <Icon class="material-icons" slot="leadingIcon">person</Icon>
          </Textfield>
          <Textfield
            type="number"
            bind:value={tezToSend}
            label="Amount to send"
            disabled={sendingTez}
            input$emptyValueUndefined
            style="width: 100%;"
            helperLine$style="width: 100%;"
          >
            <svelte:fragment slot="leadingIcon">
              {#if sendingTez}
                <Icon class="material-icons" slot="leadingIcon">
                  schedule_send
                </Icon>
              {:else}
                <Icon class="material-icons" slot="leadingIcon">send</Icon>
              {/if}
            </svelte:fragment>
          </Textfield>
          <br /><br />
          <Button
            variant="raised"
            disabled={(!tezToSend && !recipientAddress) || sendingTez}
            style="float:right"
            on:click={sendTez}
          >
            <Label>Confirm</Label>
          </Button>
        </Content>
      </Paper>
    </div>
  {:else}
    <Paper>
      <Subtitle>Please input your private key in the field below</Subtitle>
      <Content>
        <Textfield
          type="password"
          bind:value={privateKey}
          label="Your private key"
          input$emptyValueUndefined
          style="width: 100%;"
          helperLine$style="width: 100%;"
        >
          <Icon class="material-icons" slot="leadingIcon">key</Icon>
          <HelperText slot="helper">Starts with "edsk"</HelperText>
        </Textfield>
        <br />
        <Button
          variant="raised"
          disabled={!privateKey}
          style="float:right"
          on:click={() => dispatch("new-key", privateKey)}
        >
          <Label>Confirm</Label>
        </Button>
      </Content>
    </Paper>
  {/if}
</section>
<Snackbar bind:this={snackbar} labelText={snackbarText} timeoutMs={-1}>
  <SnackbarLabel />
</Snackbar>
