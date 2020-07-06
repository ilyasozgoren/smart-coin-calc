<template id="calculator">
  <div>
    <div class="timestamp ml-2">
      <span v-if="loading">
        <i class="fa fa-spinner fa-spin mr-2"></i>
      </span>
      Last updated: {{timestamp}}
    </div>
    <div class="container">
      <div class="row" >
        <div class="col">
          <div class="price-holder">
            <span>{{displayRate}}</span>
            <span class="label ml-2" :class="bpi.change >= 0 ? 'up' : 'down'">
              {{change}}%
            </span>
          </div>
        </div>
        <div class="col ml-auto my-auto">
          <ul class="nav justify-content-end">
            <li class="nav-item"
                v-for="(opt, i) in supportedCurrencies"
                :key="i">
              <a class="nav-link"
                 v-bind:class="{'active' : currency === opt }"
                 v-on:click="setCurrency(opt)">
                {{opt}}
              </a>
            </li>
          </ul>
        </div>
      </div>
      <div class="row align-items-center converter">
        <div class="col-sm-10 col-md-6 ml-auto">
          <div class="input-group">
            <div class="input-group">
              <input type="text"
                     class="form-control reverse"
                     placeholder="0.0"
                     v-model="inputValue">
              <span class="underline"></span>
            </div>
            <span class="input-group-addon">
              <span>{{ inverted ?  trade : currency }}</span>
            <span class="sm" :class="{'blurry' : blurred }">{{conversion}}</span>
            </span>
          </div>
        </div>
        <button type="button"
                class="c-toggle btn btn-default col-sm-2"
                v-on:click="inverted = !inverted">
          <i class="fas fa-sync fa-2x my-2"></i>
        </button>
      </div>
    </div>
  </div>
</template>
<script>
import axios from 'axios'
import moment from 'moment'
let bpiDefault = { change: '', code: '' }
export default {
  name: 'CoinCalculator',
  data: () => ({
    apiUrl: 'https://api.coindesk.com/v1/bpi/',
    trade: 'BTC',
    selectedCurrency: null,
    timestamp: null,
    bpi: Object.assign({}, bpiDefault),
    inverted: false,
    value: null,
    supportedCurrencies: ['GBP', 'USD', 'EUR'],
    loading: false,
    interval: null,
    blurred: false
  }),
  mounted () {
    this.selectedCurrency = this.supportedCurrencies[0]
    this.getUpdatedPrice()

    this.interval = setInterval(() => {
      this.getUpdatedPrice()
    }, 10000)
  },
  beforeDestroy () {
    clearInterval(this.interval)
  },
  methods: {
    getUpdatedPrice () {
      this.loading = true
      axios.get(this.apiUrl + 'currentprice/' + this.selectedCurrency + '.json')
        .then(res => {
          if (res && res.data) {
            this.timestamp = moment().local().format('LT')
            this.bpi = Object.assign(bpiDefault, res.data.bpi[this.selectedCurrency])
            this.getHistorical()
          }

          this.loading = false
        })
        .catch(error => {
          this.loading = false
          console.log(error)
        })
    },
    getHistorical () {
      axios.get(this.apiUrl + 'historical/close.json?currency=' + this.selectedCurrency + '&for=yesterday')
        .then(res => {
          if (res && res.data) {
            const y = Object.values(res.data.bpi)[0]
            this.bpi.change = (this.rate - y) * 100 / y
          }
        })
        .catch(error => {
          alert(error)
        })
    },
    setCurrency (code) {
      this.selectedCurrency = code
      this.getUpdatedPrice()
    }
  },
  computed: {
    inputValue: {
      get () {
        return this.value
      },
      set (value) {
        this.blurred = isNaN(value)
        if (this.blurred) {
          return
        }
        this.value = Number(value)
      }
    },
    rate () {
      return this.bpi ? this.bpi.rate_float : null
    },
    displayRate () {
      const formatter = new Intl.NumberFormat('en-US', {
        style: 'currency',
        currency: this.currency || 'USD',
        minimumFractionDigits: 2
      })

      return formatter.format(this.rate)
    },
    currency () {
      return this.bpi ? this.bpi.code : null
    },
    change () {
      const formatter = new Intl.NumberFormat()
      return this.bpi ? formatter.format(this.bpi.change) : 'null'
    },
    conversion () {
      const val = this.inverted ? this.value * this.rate : this.value / this.rate

      let opts = { minimumFractionDigits: this.inverted ? 2 : 4 }

      if (this.inverted) {
        opts.style = 'currency'
        opts.currency = this.currency || 'USD'
      }

      const formatter = new Intl.NumberFormat('en-US', opts)
      const result = formatter.format(val)
      return this.inverted ? result : result + ' ' + this.trade
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
body {
  background-color: #e6e8e9;
}
.btn:focus,
.btn:active {
  box-shadow: none !important;
  animation: btn-scale 0.5s ease-in-out;
}
.container {
  margin-top: 4em;
  background-color: #fff;
  border-radius: 0.25rem;
  border: solid 1px darkgray;
}
.converter {
  border-top: solid 1px lightgray;
}
.converter input[type="text"].reverse {
  text-align: right;
  font-size: 2em;
  font-weight: 400;
  height: 1.8em;
}
.converter input[type="text"].reverse:focus {
  box-shadow: none;
}
.converter input[type="text"].reverse:focus + .underline {
  left: 0;
}
.converter .c-toggle {
  padding: 1.5em;
  color: #0e85ec;
  cursor: pointer;
  border: none;
  border-top-left-radius: 0;
  border-bottom-left-radius: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.converter .c-toggle:hover {
  transition: all 0.5s ease;
  color: #192c52;
}
.converter .c-toggle:hover svg {
  animation: toggle-spin 0.5s ease-in-out;
}
.underline {
  background-color: dodgerblue;
  display: inline-block;
  height: 3px;
  position: absolute;
  right: 0;
  left: 100%;
  bottom: 1.5em;
  -webkit-transition: all 0.1s ease-in-out;
  transition: all 0.1s ease-in-out;
  z-index: 5;
}
.price-holder {
  padding: 5px 10px;
  background-color: #4f6080;
  margin: 10px 0;
  width: fit-content;
  border-radius: 0.2em;
  color: white;
  text-align: center;
  font-size: 1.2em;
  display: flex;
  align-items: center;
  justify-content: center;
}
.input-group-addon {
  background-color: transparent;
  border: none;
  font-weight: 500;
  font-size: 2em;
  display: flex;
  flex-direction: column;
}
.input-group-addon .sm {
  font-size: 0.5em;
  color: lightgray;
}
.nav-link {
  padding: 0.5rem 0.5rem;
}
.nav-item {
  cursor: pointer;
  font-weight: 500;
}
.nav-item:hover {
  color: darkgray;
}
.nav-item:hover a.active {
  color: #0e85ec;
}
.nav-item a.active {
  color: #0e85ec;
}
.timestamp {
  color: #4f6080;
}
.blurry {
  text-shadow: 0 0 1em rgba(1, 1, 1, 0.5);
  color: transparent;
}
.label {
  font-size: 0.5em;
  padding: 4px;
  border-radius: 0.2em;
  font-weight: 900;
}
.label.up {
  background-color: #438a43;
}
.label.down {
  background-color: #cc3232;
}
@keyframes btn-scale {
  50% {
    transform: scale(1.05);
  }
  100% {
    transform: scale(1);
  }
}
@keyframes toggle-spin {
  0% {
    color: #0e85ec;
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

</style>
