<template>
  <div id="app" class="container">

    <h1>Курсы криптовалют</h1>

    <chart v-show="!error"
           :chart-data="chartData"
           :options="chartOptions"
           css-classes="coin-values-chart mt-4" />
    <b-alert v-if="error" show variant="danger">{{ error.message }}</b-alert>

    <b-form @submit.prevent="onSubmit" class="mt-4">
      <b-form-input id="coin"
                    v-model="coinName"
                    required
                    placeholder="Enter currency ticker" />
      <b-button type="submit" variant="primary" class="mt-4">Показать</b-button>
    </b-form>

    <div class="loading" v-if="isLoading">Loading&#8230;</div>

  </div>
</template>

<script>
import axios from 'axios';
import moment from 'moment';

import chart from './components/Chart.vue';

const API = {
  HISTODAY: 'https://min-api.cryptocompare.com/data/histoday',
};

export default {
  name: 'app',
  components: {
    chart,
  },
  data() {
    return {
      fsym: 'BTC',
      chartData: {
        labels: [],
        datasets: [],
      },
      chartOptions: {
        maintainAspectRatio: false,
      },
      error: '',
      isLoading: true,
    };
  },
  computed: {
    coinName: {
      get() {
        return this.fsym;
      },
      set(newValue) {
        this.fsym = newValue.toUpperCase();
      },
    },
  },
  methods: {
    async getCryptoCurrency(fsym = this.fsym) {
      this.isLoading = true;
      try {
        const results = await axios.get(API.HISTODAY, {
          params: {
            fsym,
            tsym: 'USD',
            limit: 10,
            aggregate: 1,
          },
        });
        if (results.data.Response === 'Success') {
          this.error = '';
          this.prepareChartData(results.data.Data);
        } else {
          throw new Error(results.data.Message);
        }
        this.isLoading = false;
      } catch (error) {
        this.error = error;
        console.log('error', error);
        this.isLoading = false;
      }
    },
    prepareChartData(data) {
      this.chartData.labels = data.map(value => moment(value.time).format('DD/MM/YY, h:mm'));
      this.chartData.datasets = [
        {
          label: this.coinName,
          backgroundColor: '#f87979',
          lineTension: 0,
          data: data.map(value => this.getAverage(value)),
        },
      ];
    },
    getAverage(value) {
      return (value.high + value.low) / 2;
    },
    onSubmit() {
      this.getCryptoCurrency(this.coinName);
    },
  },
  created() {
    this.getCryptoCurrency();
  },
};
</script>

<style lang="scss">
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
  .coin-values-chart {
    position: relative;
    height: 500px;
  }
  .isLoading {
    &:before {
      content: '';
      position: absolute;
      top: 0;
      left: 0;
      display: block;
      height: 100vh;
      background: rgba(255, 255, 255, .8);
    }
  }
  // Взял готовое решение, обычно пишу сам
  /* Absolute Center Spinner */
  .loading {
    position: fixed;
    z-index: 999;
    height: 2em;
    width: 2em;
    overflow: visible;
    margin: auto;
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
  }

  /* Transparent Overlay */
  .loading:before {
    content: '';
    display: block;
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0,0,0,0.3);
  }

  /* :not(:required) hides these rules from IE9 and below */
  .loading:not(:required) {
    font: 0/0 a;
    color: transparent;
    text-shadow: none;
    background-color: transparent;
    border: 0;
  }

  .loading:not(:required):after {
    content: '';
    display: block;
    font-size: 10px;
    width: 1em;
    height: 1em;
    margin-top: -0.5em;
    animation: spinner 1500ms infinite linear;
    border-radius: 0.5em;
    box-shadow: rgba(0, 0, 0, 0.75) 1.5em 0 0 0,
                rgba(0, 0, 0, 0.75) 1.1em 1.1em 0 0,
                rgba(0, 0, 0, 0.75) 0 1.5em 0 0,
                rgba(0, 0, 0, 0.75) -1.1em 1.1em 0 0,
                rgba(0, 0, 0, 0.75) -1.5em 0 0 0,
                rgba(0, 0, 0, 0.75) -1.1em -1.1em 0 0,
                rgba(0, 0, 0, 0.75) 0 -1.5em 0 0,
                rgba(0, 0, 0, 0.75) 1.1em -1.1em 0 0;
  }

  /* Animation */

  @-webkit-keyframes spinner {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }
  @-moz-keyframes spinner {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }
  @-o-keyframes spinner {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }
  @keyframes spinner {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }
</style>
