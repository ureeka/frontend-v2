<script setup lang="ts">
import useWeb3 from '@/services/web3/useWeb3';
import usePoolCreation from '@/composables/pools/usePoolCreation';
import TokenInput from '@/components/inputs/TokenInput/TokenInput.vue';
import { onMounted, ref } from 'vue';

/**
 * STATE
 */

const autoOptimise = ref(false);

/**
 * METHODS
 */

function optimiseLiquidity() {
  for (const token of tokenWeights.value) {
    token.amount = optimisedLiquidity.value[token.tokenAddress].balanceRequired;
  }
}

function toggleAutoOptimise() {
  autoOptimise.value = !autoOptimise.value;

  checkLiquidityOptimisation();
}

function checkLiquidityOptimisation() {
  if (!autoOptimise.value) return;

  optimiseLiquidity();
}

function amountUpdated(tokenNum) {
  userModifiedTokenNum.value = tokenNum;

  checkLiquidityOptimisation();
}

/**
 * COMPOSABLES
 */
const { userNetworkConfig } = useWeb3();
const {
  tokenWeights,
  proceed,
  optimisedLiquidity,
  userModifiedTokenNum
} = usePoolCreation();

onMounted(() => {
  optimiseLiquidity();
});
</script>

<template>
  <BalCard>
    <BalStack vertical>
      <BalStack vertical spacing="xs">
        <span class="text-sm text-gray-700">{{ userNetworkConfig?.name }}</span>
        <h5 class="font-bold">Set initial liquidity</h5>
        <div class="flex items-center">
          <BalToggle
            name="autoOptimise"
            :checked="autoOptimise"
            @toggle="toggleAutoOptimise"
          />
          <span class="text-sm pl-2">{{
            $t('autoOptimiseLiquidityToggle.label')
          }}</span>
          <BalTooltip width="64">
            <template v-slot:activator>
              <BalIcon name="info" size="xs" class="text-gray-400 ml-1 flex" />
            </template>
            <div v-html="$t('autoOptimiseLiquidityToggle.tooltip')" />
          </BalTooltip>
        </div>
      </BalStack>
      <BalStack isDynamic vertical>
        <TokenInput
          v-for="(token, i) in tokenWeights"
          :key="token.tokenAddress"
          v-model:amount="tokenWeights[i].amount"
          v-model:address="tokenWeights[i].tokenAddress"
          :weight="tokenWeights[i].weight / 100"
          :name="`initial-token-${tokenWeights[i].tokenAddress}`"
          @update:amount="amountUpdated(i)"
        />
      </BalStack>
      <BalBtn @click="proceed" block color="gradient">Preview</BalBtn>
    </BalStack>
  </BalCard>
</template>
