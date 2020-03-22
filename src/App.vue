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
        multiple
        :disabled="loading"
        :options="countries"
        :placeholder="loading ? `Loading..` : `Choose some countries`"
      />
    </header>

    <main class="flex flex-col flex-1 h-full items-center justify-center">
      <transition name="fade" type="out-in">
        <Loading v-if="loading" />
        <GChart
          v-else-if="!loading && this.selected.length"
          type="LineChart"
          :data="chartData"
          :settings="{ packages: ['line'] }"
          :createChart="(el, google) => new google.charts.Line(el)"
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
import { flatten, intersection, intersectionBy } from "lodash-es";
export default {
  name: "App",
  components: { Loading, vSelect },
  data: () => ({
    loading: false,
    selected: ["Canada", "Japan"]
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
      if (
        this.selected &&
        this.selected.length === 1 &&
        state.countries[this.selected[0]]
      ) {
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

      if (this.selected && this.selected.length >= 2) {
        const dateRange = intersection(
          ...this.selected.map(country =>
            state.countries[country].map(({ date }) => date)
          )
        );

        const plot = dateRange.map(date => {
          return [
            date,
            ...this.selected.map(country => {
              return (
                state.countries[country].find(data => data.date === date)
                  .confirmed || 0
              );
            })
          ];
        });

        return [["date", ...this.selected], ...plot];
      }

      return [];
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
