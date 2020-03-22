<template>
  <div id="app" class="flex flex-col mx-auto py-4 px-4 h-screen">
    <header
      class="flex justify-between items-center mb-4 pb-4 border-b border-gray-200"
    >
      <h1 class="text-gray-500 text-xl font-bold leading-none">
        COVID-19 Confirmed Cases
      </h1>
    </header>

    <main class="flex flex-1 overflow-y-hidden">
      <Loading v-if="loading" />
      <template v-else>
        <ul
          v-if="sidebar"
          class="overflow-y-scroll max-h-full pr-3 mr-3 countries relative text-gray-700"
        >
          <li class="mb-2 sticky top-0">
            <input
              type="search"
              v-model="countryFilter"
              placeholder="filter this list..."
              class="border-2 border-gray-300 w-full px-1 py-1"
            />
          </li>
          <li
            v-for="{ country, cases } in casesPerCountry"
            :key="country"
            class="py-1 my-1 border-b border-gray-200"
            :class="{ 'text-red-600': selected.includes(country) }"
          >
            <label class="flex items-center justify-between ">
              <span>
                <input
                  type="checkbox"
                  v-model="selected"
                  :value="country"
                  class="mr-1"
                />
                {{ country }}
              </span>
              <span>{{ parseFloat(cases).toLocaleString("en") }}</span>
            </label>
          </li>
        </ul>
        <div class="flex-1 flex flex-col justify-center items-center">
          <h2 v-if="selected.length === 0" class="text-blue-400">
            Select some countries to chart their confirmed cases.
          </h2>
          <GChart
            v-if="this.selected.length"
            type="LineChart"
            :data="chartData"
            :settings="{ packages: ['line'] }"
            :createChart="(el, google) => new google.charts.Line(el)"
            class="w-full h-full flex-1 chart"
          />
        </div>
      </template>
    </main>
  </div>
</template>

<script>
import axios from "axios";
import { intersection } from "lodash-es";
import state from "./state";
import Loading from "./components/Loading";

export default {
  name: "App",
  components: { Loading },
  data: () => ({
    loading: false,
    selected: [],
    countryFilter: "",
    sidebar: true
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
    casesPerCountry() {
      return Object.keys(state.countries)
        .filter(country =>
          country.toLowerCase().includes(this.countryFilter.toLowerCase())
        )
        .map(country => ({
          country,
          cases:
            state.countries[country][state.countries[country].length - 1]
              .confirmed
        }))
        .sort((a, b) => a.cases - b.cases)
        .reverse();
    },
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
</style>
