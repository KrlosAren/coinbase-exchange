<template>
  <div>
    <bounce-loader
      :size="100"
      :loading="isLoading"
      :color="'#68d391'"
    ></bounce-loader>
    <px-assets-table v-if="!isLoading" :assets="assets"></px-assets-table>
  </div>
</template>

<script>
import PxAssetsTable from '@/components/PxAssetsTable.vue';
import { getAssets } from '@/api';

export default {
  name: 'Home',
  components: {
    PxAssetsTable
  },

  data() {
    return {
      isLoading: false,
      assets: []
    };
  },

  created() {
    this.isLoading = true;
    getAssets()
      .then(assets => (this.assets = assets))
      .finally(() => (this.isLoading = false));
  }
};
</script>
