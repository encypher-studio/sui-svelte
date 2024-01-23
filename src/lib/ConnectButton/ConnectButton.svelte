<script lang="ts">
  import { connectWithModal, disconnect, account } from "$lib/SuiModule"

  interface IProps {
    class?: string
    style?: string
  }

  const { class: className = "", style = "" } = $props<IProps>()

  const onClick = function () {
    if (!account.val) {
      connectWithModal()
    } else {
      disconnect()
    }
  }
</script>

<button type="button" class={className} {style} onclick={onClick}>
  {account.val ? "Disconnect" : "Connect"}
</button>
