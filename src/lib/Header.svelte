<script lang="ts">
  import { createEventDispatcher } from "svelte";
  import Tab, { Icon, Label } from "@smui/tab";
  import TabBar from "@smui/tab-bar";

  const dispatch = createEventDispatcher();

  let tabs = [
    {
      icon: "account_balance_wallet",
      label: "Account"
    },
    {
      icon: "history",
      label: "History"
    },
    {
      icon: "image",
      label: "NFTs"
    }
  ];

  let activeTab = tabs[0];
</script>

<header>
  <TabBar {tabs} let:tab bind:active={activeTab}>
    <Tab
      {tab}
      on:click={() => dispatch("change-view", tab.label.toLowerCase())}
    >
      <Icon class="material-icons">{tab.icon}</Icon>
      <Label>{tab.label}</Label>
    </Tab>
  </TabBar>
</header>
