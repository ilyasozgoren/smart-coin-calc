<template id="calculator">
  <div>
    <div class="timestamp ml-2">
      <span v-if="loading">
        <i class="fa fa-spinner fa-spin mr-2"></i>
      </span>
      Last updated: {{timestamp}}
    </div>
    <div class="container">
    <div class="row">
        <div class="col">
            <div class="price-holder">
                <span>Güncel BTC : {{displayRate}}</span>
            </div>
        </div>
    </div>
    <div class="row">
        <div class="col-sm-6 col-md-6 ml-auto">
            <div class="row">
                <div class="col-12 form-group">
                    <label>Amount to Spend:</label><br>
                    <div class="input-group">
                        <input type="text" class="form-control" placeholder="0.00" v-model="currValue" v-on:blur="coinCalc($event)">
                        <select class="custom-select" v-on:change="setCurrency($event)">
                            <option v-bind:value="opt.name"  v-for="(opt, i) in supportedCurrencies" :key="i">{{ opt.name }}</option>
                        </select>
                    </div>
                </div>
            </div>
        </div>
        <div class="col-sm-6 col-md-6 ml-auto">
            <div class="row">
                <div class="col-12 form-group">
                    <label>Coins to Receive:</label><br>
                    <div class="input-group">
                        <input type="text" class="form-control" v-model="coinValue" v-on:blur="currCalc($event)">
                        <select class="custom-select">
                            <option value="BTC">BTC</option>
                        </select>
                    </div>
                </div>
            </div>
        </div>
    </div>

    </div>
  </div>
</template>
<script>
import axios from 'axios'
import moment from 'moment'
// let currDefault = { change: '', code: '' }
export default {
  name: 'SmartCoinCalculator',
  data: () => ({
    apiUrl: 'https://blockchain.info/ticker',
    trade: 'BTC',
    selectedCurrency: null,
    minCurrency: null,
    maxCurrency: null,
    timestamp: null,
    bpi: Object.assign({}),
    inverted: false,
    value: null,
    supportedCurrencies: [
      {name: 'GBP', min: '20', max: '5000'},
      {name: 'USD', min: '30', max: '5000'},
      {name: 'EUR', min: '25', max: '5000'}
    ],
    loading: false,
    interval: null,
    currValue: '',
    coinValue: ''
  }),
  mounted () {
    console.log(this.supportedCurrencies[0].name)
    this.selectedCurrency = this.supportedCurrencies[0].name
    this.getUpdatedPrice()

    this.interval = setInterval(() => {
      this.getUpdatedPrice()
    }, 100000)
  },
  beforeDestroy () {
    clearInterval(this.interval)
  },
  methods: {
    getUpdatedPrice () {
      this.loading = true
      axios.get(this.apiUrl)
        .then(res => {
          if (res && res.data) {
            this.timestamp = moment().local().format('LT')
            // console.log(this.selectedCurrency)
            this.bpi = Object.assign(res.data[this.selectedCurrency])

            // this.bpi.change = (this.rate - y) * 100 /
            // this.getHistorical()
            console.log(this.bpi.last)
          }

          this.loading = false
        })
        .catch(error => {
          this.loading = false
          console.log(error)
        })
    },
    setCurrency (event) {
      // console.log(event.target.value)
      this.selectedCurrency = event.target.value
      this.getUpdatedPrice()
    },
    coinCalc ($event) {
      // console.log(this.selectedCurrency)
      const curr = this.supportedCurrencies.find(d => d.name === this.selectedCurrency)
      // console.log(curr.min)
      // this.minCurr[this.selectedCurrency]
      const val = event.target.value / this.rate

      if (event.target.value < Number(curr.min) || event.target.value > Number(curr.max)) {
        alert('Hata')
        return false
      }

      let opts = { minimumFractionDigits: this.inverted ? 2 : 8 }

      if (this.inverted) {
        opts.style = 'currency'
        opts.currency = this.currency || 'USD'
      }

      const formatter = new Intl.NumberFormat('en-US', opts)
      const result = formatter.format(val)
      this.coinValue = result
    },
    currCalc ($event) {
      const val = event.target.value * this.rate

      let opts = { minimumFractionDigits: this.inverted ? 2 : 2 }

      if (this.inverted) {
        opts.style = 'currency'
        opts.currency = this.currency || 'USD'
      }

      const formatter = new Intl.NumberFormat('en-US', opts)
      const result = formatter.format(val)
      this.currValue = result
    }
  },
  computed: {
    /* currValue: {
      get () {
        return this.value
      },
      set (value) {
        this.value = this.coinCalc(value)
      }
    },
    coinValue: {
      get () {
        return this.value
      },
      set (value) {
        this.value = this.currCalc(value)
      }
    }, */
    rate () {
      return this.bpi ? this.bpi.last : 0
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
      return this.bpi ? this.selectedCurrency : null
    }
    /* change () {
      const formatter = new Intl.NumberFormat()
      return this.bpi ? formatter.format(this.bpi.change) : 'null'
    }, */
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
