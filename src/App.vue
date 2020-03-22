<template>
  <div
    id="app"
    class="flex flex-col container mx-auto p-10 h-full min-h-screen"
  >
    <header class="flex justify-between items-center mb-4">
      <h1 class="text-gray-800 text-xl font-bold">
        COVID-19 Localized Transmission Rates
      </h1>
      <v-select
        v-model="selected"
        :disabled="loading"
        :options="countries"
        :placeholder="loading ? `Loading..` : `Choose some countries`"
      />
    </header>

    <main class="flex flex-col flex-1 h-full items-center justify-center">
      <transition name="fade" type="out-in">
        <Loading v-if="loading" />
        <GChart
          v-else
          type="LineChart"
          :data="chartData"
          class="w-full flex-1 chart"
        />
      </transition>
    </main>
  </div>
</template>

<script>
import axios from "axios";
import state from "./state";
import Loading from "./components/Loading";
import vSelect from "vue-select";
export default {
  name: "App",
  components: { Loading, vSelect },
  data: () => ({
    loading: false,
    selected: "Canada"
  }),
  mounted() {
    this.fetchData();
  },
  methods: {
    async fetchData() {
      this.loading = true;
      try {
        const { data } = await axios.get(
          "https://pomber.github.io/covid19/timeseries.json"
        );
        state.countries = data;
      } catch (e) {
        console.error(e);
      }
      this.loading = false;
    }
  },
  computed: {
    countries: () => Object.keys(state.countries) || [],
    chartData() {
      return [
        ["date", "confirmed", "deaths", "recovered"],
        ...state.countries[
          this.selected
        ].map(({ date, confirmed, deaths, recovered }) => [
          date,
          confirmed,
          deaths,
          recovered
        ])
      ];
    }
  }
};
</script>

<style>
.v-select {
  min-width: 300px;
}
.chart {
  max-height: 500px;
}
</style>
