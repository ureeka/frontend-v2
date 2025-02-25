<script setup lang="ts">
import { useTradeState } from '@/composables/trade/useTradeState';
import useBreakpoints from '@/composables/useBreakpoints';
import QUERY_KEYS from '@/constants/queryKeys';
import { balancerSubgraphService } from '@/services/balancer/subgraph/balancer-subgraph.service';
import useWeb3 from '@/services/web3/useWeb3';
import { getAddress } from '@ethersproject/address';
import { getUnixTime } from 'date-fns';
import { computed } from 'vue';
import { useQuery } from 'vue-query';

type TrendingPair = {
  symbol: string;
  address: string;
};

const { setTokenOutAddress, setTokenInAddress } = useTradeState();
const { chainId: userNetworkId } = useWeb3();
const { upToLargeBreakpoint } = useBreakpoints();

const getTrendingTradePairs = async () => {
  return await balancerSubgraphService.tradePairSnapshots.get({
    orderBy: 'totalSwapVolume',
    orderDirection: 'desc',
    where: {
      timestamp_gte: getUnixTime(new Date().setUTCHours(0, 0, 0, 0))
    },
    first: 5
  });
};

const { data: tradePairSnapshots } = useQuery(
  QUERY_KEYS.Tokens.TrendingPairs(userNetworkId),
  () => getTrendingTradePairs()
);
const trendingPairs = computed(() => {
  return (tradePairSnapshots.value || []).map(pairSnapshot => [
    {
      symbol: pairSnapshot.pair.token0.symbol,
      address: getAddress(pairSnapshot.pair.token0.address)
    },
    {
      symbol: pairSnapshot.pair.token1.symbol,
      address: getAddress(pairSnapshot.pair.token1.address)
    }
  ]);
});

const setTradePair = (pair: TrendingPair[]) => {
  setTokenInAddress(pair[0].address);
  setTokenOutAddress(pair[1].address);
};
</script>

<template>
  <BalCard
    :square="upToLargeBreakpoint"
    noPad
    growContent
    :noBorder="upToLargeBreakpoint"
    shadow="false"
  >
    <div class="trending-pairs">
      <div
        v-if="!upToLargeBreakpoint"
        class="flex justify-between p-3 lg:shadow-lg"
      >
        <h6>{{ $t('trendingPairs') }}</h6>
      </div>
      <div class="px-1 lg:p-3 flex flex-wrap">
        <button
          class="py-1 px-2 bg-transparent hover:bg-blue-500 hover:text-white text-sm rounded-lg lg:shadow my-2 mr-2 font-medium lg:font-normal"
          v-for="(pair, i) in trendingPairs"
          :key="`trendingPair-${i}`"
          @click="setTradePair(pair)"
        >
          {{ pair[0].symbol }}/{{ pair[1].symbol }}
        </button>
      </div>
    </div>
  </BalCard>
</template>

<style scoped>
.trending-pairs {
  @apply flex flex-col bg-transparent;
  min-height: 200px;
}
</style>
