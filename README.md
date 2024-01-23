# Sui Svelte
[![npm version](https://badge.fury.io/js/sui-svelte.svg)](https://badge.fury.io/js/sui-svelte)

Provides components for interacting with Sui wallets.

# Showcase

[SuiForge](https://suiforge.com/) uses sui-svelte to interact with SUI wallets.

# Setup

Barebones example:

routes/+page.svelte
```ts
<script lang="ts">
    import {SuiModule, ConnectButton} from "@encypher/sui-svelte"
</script>

<SuiModule />
<ConnectButton />
 ```

 # Getting connected account

 ```ts
 <script lang="ts">
	import { account } from "@encypher/sui-svelte/SuiModule"

	console.log("Connected account object:", account)

    constole.log("Connected address:", account.address)
</script>
  ```

 # Sending transactions

 Refer to [Sui guide on building transactions](https://docs.sui.io/guides/developer/sui-101/building-ptb).

 ```ts
 <script lang="ts">
	import { signAndExecuteTransactionBlock } from "@encypher/sui-svelte/SuiModule"
	import { TransactionBlock } from "@mysten/sui.js/transactions"

	const sendTx = () => {
        const txb = new TransactionBlock()
        // Create a new coin with balance 100, based on the coins used as gas payment
        const [coin] = txb.splitCoins(txb.gas, [txb.pure(100)])
        // Transfer the split coin to a specific address
        txb.transferObjects([coin], txb.pure("0xSomeSuiAddress"))

        // Execute using sui-svelte
        signAndExecuteTransactionBlock(txb)
    }
</script>
  ```

# Customization

## ConnectButton

Is a \<button> componnet and can be customized by passing a class or styles prop.

```html
<ConnectButton class="my-button-class" style="background: red;" />
```

## ConnectModal

The modal for connecting can be customized by passing a "modal" slot to the SuiModule.

You should always open the modal using the ConnectButton or the connect exported method from SuiModule, since this starts the process that ends when resolve gets called.

```html
<script lang="ts">
    import {SuiModule} from "@encypher/sui-svelte/SuiModule"
    import {connectModal, resolve} from "@encypher/sui-svelte/ConnectModal"
    import type { IWallet } from "@suiet/wallet-sdk"

    // Get the wallet from the browser
    func getWallet = (): IWallet  => {
        ...
        return suietWallet
    }

    // Call the modal resolve function with the wallet
    const selectWallet = () => {
        const wallet = getWallet()
        resolve.val(wallet)
    }

    // If closed, call the close method of connectModal
    const onClose = () => {
        connectModal.close()
    }
</script>

<SuiModule>
    <div slot="modal">
        <button onclick={onClose}> Close </button>
        <button onclick={selectWallet}> Select wallet </button>
    </div>
</SuiModule>
```

# Issues and contributions

Feel free to submit PRs, or issues for any doubts or feature requests.
