<template>
  <div class="flex-col">
    <div class="flex justify-center">
      <bounce-loader
        :loading="isLoading"
        :color="'#68d391'"
        :size="100"
      ></bounce-loader>
    </div>
    <template v-if="!isLoading">
      <div class="flex flex-col sm:flex-row justify-around items-center">
        <div class="flex flex-col items-center">
          <img
            :src="
              `https://static.coincap.io/assets/icons/${asset.symbol.toLowerCase()}@2x.png`
            "
            :alt="asset.name"
            class="w-20 h-20 mr-5"
          />
          <h1 class="text-5xl">
            {{ asset.name }}
            <small class="sm:mr-2 text-gray-500">{{ asset.symbol }}</small>
          </h1>
        </div>

        <div class="my-10 flex flex-col">
          <ul>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Ranking</b>
              <span>#{{ asset.rank }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio actual</b>
              <span>{{ asset.priceUsd | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio más bajo</b>
              <span>{{ min | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio más alto</b>
              <span>{{ max | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio Promedio</b>
              <span>{{ avg | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Variación 24hs</b>
              <span>{{ asset.changePercent24Hr | percent }}</span>
            </li>
          </ul>
        </div>

        <div class="my-10 sm:mt-0 flex flex-col justify-center text-center">
          <button
            @click="toggleConverter"
            class="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded"
          >
            {{ fromUsd ? `USD a ${asset.symbol}` : `${asset.symbol} to USD` }}
          </button>

          <div class="flex flex-row my-5">
            <label class="w-full" for="convertValue">
              <input
                v-model="covertValue"
                id="convertValue"
                type="number"
                class="text-center bg-white focus:outline-none focus:shadow-outline border border-gray-300 rounded-lg py-2 px-4 block w-full appearance-none leading-normal"
              />
            </label>
          </div>

          <span class="text-xl">{{ convertResult }}</span>
        </div>
      </div>
      <div id="chart-coin">
        <line-chart
          :id="'chart-coin'"
          v-if="!isLoading"
          class="my-10"
          :colors="['orange']"
          :min="min"
          :max="max"
          :data="chartData"
          height="1000px"
        >
        </line-chart>
      </div>

      <h3 class="text-xl my-10">Mejores Ofertas de Cambio</h3>
      <table>
        <tr
          v-for="m in markets"
          :key="`${m.exchangeId}-${m.priceUsd}`"
          class="border-b"
        >
          <td>
            <b>{{ m.exchangeId }}</b>
          </td>
          <td>{{ m.priceUsd | dollar }}</td>
          <td>{{ m.baseSymbol }} / {{ m.quoteSymbol }}</td>
          <td>
            <px-button
              :is-loading="m.isLoading || false"
              @click="getWebsite(m)"
              v-if="!m.url"
            >
              <slot v-if="!m.isLoading">Obtener Link</slot>
            </px-button>
            <a v-else class="hover:underline text-green-600 " target="_blank">{{
              m.url
            }}</a>
          </td>
        </tr>
      </table>
    </template>
  </div>
</template>

<script>
import { getAsset, getAssetHistory, getExchange, getMarkets } from '../api';
import PxButton from '../components/PxButton';
export default {
  components: { PxButton },
  name: 'CoinDetail',

  data() {
    return {
      asset: {},
      history: [],
      isLoading: false,
      markets: [],
      fromUsd: true,
      covertValue: null
    };
  },

  created() {
    this.getCoin();
  },

  watch: {
    $route() {
      this.getCoin();
    }
  },

  methods: {
    getWebsite(exchange) {
      this.$set(exchange, 'isLoading', true);
      return getExchange(exchange.exchangeId)
        .then(res => {
          this.$set(exchange, 'url', res.exchangeUrl);
        })
        .finally(() => {
          this.$set(exchange, 'isLoading', false);
        });
    },

    toggleConverter() {
      console.log('hola');
      this.fromUsd = !this.fromUsd;
    },
    getCoin() {
      const id = this.$route.params.id;
      this.isLoading = true;
      Promise.all([getAsset(id), getAssetHistory(id), getMarkets(id)])
        .then(([asset, dataHistory, markets]) => {
          this.asset = asset;
          this.history = dataHistory;
          this.markets = markets;
        })
        .finally(() => (this.isLoading = false));
    }
  },

  computed: {
    convertResult() {
      if (!this.covertValue) {
        return 0;
      }
      const result = this.fromUsd
        ? this.covertValue / this.asset.priceUsd
        : this.covertValue * this.asset.priceUsd;

      return result.toFixed(2);
    },

    min() {
      return Math.min(
        ...this.history.map(e => parseFloat(e.priceUsd).toFixed())
      );
    },
    max() {
      return Math.max(
        ...this.history.map(e => parseFloat(e.priceUsd).toFixed())
      );
    },
    avg() {
      return (
        this.history.reduce((a, b) => a + parseFloat(b.priceUsd), 0) /
        this.history.length
      );
    },

    chartData() {
      return this.history.map(h => [h.date, parseFloat(h.priceUsd).toFixed(2)]);
    }
  }
};
</script>

<style scoped>
td {
  padding: 10px;
  text-align: center;
}
</style>
