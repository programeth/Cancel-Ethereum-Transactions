<template>
  <div>
    <q-checkbox v-model="buyBeer" size="sm" @input="setTxPayload">
      <span style="font-size: 1rem;">Buy me a beer</span>
      <span class="q-ml-sm" style="font-size: 1.1rem;">🍺</span>
    </q-checkbox>
    <div class="text-caption">
      <span>
        Sends&nbsp;
        <q-input
          v-model.number="beerPrice"
          dense
          input-class="text-center text-caption"
          step="0.01"
          style="max-width: 50px; display: inline-block;"
          type="number"
          @input="setTxPayload"
        />&nbsp;ETH
        <span v-if="ethUsdPrice !== 0">
          (about ${{ Math.round(Number(ethUsdPrice * beerPrice)) }})
        </span>
        to the developer as part of the cancellation
      </span>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, onMounted, ref } from '@vue/composition-api';
import { ethers } from 'ethers';
import useTxStore from 'src/store/tx';
import useWalletStore from 'src/store/wallet';
import useEthUsdPrice from 'src/utils/ethUsdPrice';

function useDonationData() {
  const { userAddress } = useWalletStore();
  const { setTxTo, setTxValue } = useTxStore();

  const buyBeer = ref(true);
  const beerPrice = ref(0.01); // in ETH

  function setTxPayload() {
    const donationAddress = '0x3a9bE12aB20Ef966f35325763C21EAa764D639C3';
    const recipient = buyBeer.value ? donationAddress : userAddress.value;
    const amount = buyBeer.value
      ? ethers.utils.parseEther(String(beerPrice.value))
      : ethers.constants.Zero;
    setTxValue(amount);
    setTxTo(String(recipient));
  }

  onMounted(() => setTxPayload());

  return { buyBeer, beerPrice, setTxPayload };
}

export default defineComponent({
  name: 'TransactionPayloadDonation',

  setup() {
    return { ...useEthUsdPrice(), ...useDonationData() };
  },
});
</script>
