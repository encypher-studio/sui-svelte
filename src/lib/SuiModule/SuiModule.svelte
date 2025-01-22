<script lang="ts" module>
  import { Transaction } from "@mysten/sui/transactions"
  import {
    AllDefaultWallets,
    ConnectionStatus,
    WalletRadar,
    type IDefaultWallet,
    type IWallet,
    type IWalletAdapter
  } from "@suiet/wallet-sdk"
  import type { IdentifierString, WalletAccount } from "@wallet-standard/base"
  import ConnectModal, { type IConnectModal } from "../ConnectModal/ConnectModal.svelte"
  import type { SuiSignAndExecuteTransactionOutput } from "@mysten/wallet-standard"

  let walletAdapter = $state<IWalletAdapter>()
  let status = $state<ConnectionStatus>(ConnectionStatus.DISCONNECTED)
  let _account = $state<WalletAccount>()
  let connectModal = $state<IConnectModal>()
  export let getConnectModal = () => connectModal
  let _onConnect = $state<() => void>(() => {})

  export const account = {
    get value() {
      return _account
    },
    setAccount(account: WalletAccount) {
      _account = account
    },
    removeAccount() {
      _account = undefined
    }
  }

  export const connectWithModal = async () => {
    if (account.value) return
    let selectedWallet = await connectModal?.openAndWaitForResponse()
    if (selectedWallet) {
      await connect(selectedWallet)
    }
  }

  export const connect = async (wallet: IWallet) => {
    walletAdapter = wallet?.adapter
    if (walletAdapter) {
      status = ConnectionStatus.CONNECTING
      try {
        await walletAdapter.connect()
        account.setAccount(walletAdapter.accounts[0])
        status = ConnectionStatus.CONNECTED
        _onConnect()
      } catch {
        status = ConnectionStatus.DISCONNECTED
      }
    }
  }

  export const disconnect = () => {
    walletAdapter?.disconnect()
    account.removeAccount()
  }

  export const signAndExecuteTransactionBlock = async (
    transaction: Transaction
  ): Promise<SuiSignAndExecuteTransactionOutput> => {
    ensureCallable()
    return await walletAdapter!!.signAndExecuteTransaction({
      account: account.value!!,
      chain: account.value!!.chains[0] as IdentifierString,
      transaction: transaction
    })
  }

  const getAvailableWallets = (defaultWallets: IDefaultWallet[]): IWallet[] => {
    const walletAdapters = detectWalletAdapters()
    return defaultWallets
      .map((item) => {
        const foundAdapter = walletAdapters.find(
          (walletAdapter) => item.name === walletAdapter.name
        )

        return {
          ...item,
          adapter: foundAdapter ? foundAdapter : undefined,
          installed: foundAdapter ? true : false
        }
      })
      .filter((item) => item.installed == true)
  }

  const detectWalletAdapters = (): IWalletAdapter[] => {
    const walletRadar = new WalletRadar()
    walletRadar.activate()
    const walletAdapters = walletRadar.getDetectedWalletAdapters()
    walletRadar.deactivate()

    return walletAdapters
  }

  const ensureCallable = () => {
    if (status !== ConnectionStatus.CONNECTED) {
      throw Error("wallet is not connected")
    }
  }

  let availableWallets = getAvailableWallets(AllDefaultWallets)
</script>

<script lang="ts">
  interface IProps {
    onConnect?: () => void
  }

  const { onConnect }: IProps = $props()
  if (onConnect) {
    _onConnect = onConnect
  }
</script>

<ConnectModal isCustom={$$slots.modal ? true : false} bind:this={connectModal} {availableWallets}>
  {#if $$slots.modal}
    <slot name="modal" />
  {/if}
</ConnectModal>
<slot />
