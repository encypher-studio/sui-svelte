<script lang="ts">
  import { ConnectButton } from "$lib/ConnectButton"
  import { SuiModule, account } from "$lib/SuiModule"

  // (Optional) Callback to call when a Sui account is connected
  const onConnect = () => {
    if (account.value) {
      console.log("Connected address: ", account.value.address)
    }
  }
</script>

<SuiModule />
<ConnectButton />
{#if account.value}
  <div>Connected address: {account.value.address}</div>
{/if}
