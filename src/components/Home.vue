<template id="calculator">
  <div>
    <div class="container">
      <div class="row">
          <div class="col-8">
              <div class="price-area">
                  <span>Güncel BTC : {{displayBTCPrice}}</span>
              </div>
                <span v-if="loading">
                    <i class="fa fa-spinner fa-spin mr-2"></i>
                </span>
                Son Güncelleme: {{timestamp}}
            </div>
          <!--<div class="col-4 text-right">
              <div class="price-area" v-if="timerCount>0">
                  <span v-if="currValue>0">Kalan Süre : {{timerCount}}</span>
              </div>
          </div>-->
      </div>
      <div class="row">
          <div class="col-12">
              <hr>
          </div>
      </div>
      <div class="row">
          <div class="col-sm-6 col-md-6 ml-auto">
              <div class="row">
                  <div class="col-12 form-group">
                      <label>Yatırılacak Tutar:</label><br>
                      <div class="input-group">
                          <input type="text" class="form-control" placeholder="0.00" v-model="currValue">
                          <select class="custom-select" @input="setCurrency($event)">
                              <option v-bind:value="opt.name"  v-for="(opt, i) in supportedCurrencies" :key="i">{{ opt.name }}</option>
                          </select>
                      </div>
                  </div>
              </div>
          </div>
          <div class="col-sm-6 col-md-6 ml-auto">
              <div class="row">
                  <div class="col-12 form-group">
                      <label>Alınacak Coin Miktarı:</label><br>
                      <div class="input-group">
                          <input type="text" class="form-control" v-model="coinValue">
                          <select class="custom-select">
                              <option value="BTC">BTC</option>
                          </select>
                      </div>
                  </div>
              </div>
          </div>
          <div class="col-sm-12 col-md-12">
            <div class="row">
                  <div class="col-12 form-group">
                      <button class="btn btn-primary" @click="addToCart(currValue,coinValue);">Satın Al</button>
                      <button type="button" class="btn btn-danger" @click="reset()">Sıfırla</button>
                  </div>
            </div>
          </div>
      </div>

    </div>
    <div v-if="showModal">
    <transition name="modal">
      <div class="modal-mask">
        <div class="modal-wrapper">
          <div class="modal-dialog" role="document">
            <div class="modal-content">
              <div class="modal-header">
                <h5 class="modal-title">Ödeme Bilgileri</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                  <span aria-hidden="true" @click="showModal = false">&times;</span>
                </button>
              </div>
              <div class="modal-body">
                <CoinBasketPage :items="cartItems" v-if="payment === true"></CoinBasketPage>
                <CreditCart v-if="payment === true"></CreditCart>
              </div>
            </div>
          </div>
        </div>
      </div>
    </transition>
  </div>
  </div>
</template>
<script>
import axios from 'axios'
import moment from 'moment'
import 'moment/locale/tr'
import CoinBasketPage from './CoinBasketPage'
import CreditCart from './CreditCart'

let itemDefault = { Coin: '', Price: '', Symbol: '' }
export default {
  name: 'Home',
  components: {
    CoinBasketPage, CreditCart
  },
  data: () => ({
    apiUrl: 'https://blockchain.info/ticker',
    trade: 'BTC',
    selectedCurrency: null,
    timestamp: null,
    blockapi: {},
    inverted: false,
    value: null,
    payment: false,
    supportedCurrencies: [
      {name: 'USD', min: '30', max: '5000'},
      {name: 'GBP', min: '20', max: '5000'},
      {name: 'EUR', min: '25', max: '5000'}
    ],
    interval: null,
    paymentTime: null,
    loading: false,
    cartItems: Object.assign({}, itemDefault),
    items: [],
    showModal: false,
    counterStart: false,
    timerCount: 10,
    coinValue: 0,
    isRunning: false
  }),
  mounted () {
    this.selectedCurrency = this.supportedCurrencies[0].name
    this.getUpdatedBTCValue()

    this.interval = setInterval(() => {
      if (this.showModal === false && this.currValue > 0) {

        this.showModal = false
        this.cartItems = { Coin: '', Price: '', Symbol: '' }
        alert('DİKKAT! 10 saniye içinde işlem yapmadığınız için tutar yeniden hesaplanacak!')
        this.getUpdatedBTCValue()
      }
    }, 10000)
  },
  beforeDestroy () {
    clearInterval(this.interval)
  },
  methods: {
    getUpdatedBTCValue () {
      this.loading = true
      axios.get(this.apiUrl)
        .then(res => {
          if (res && res.data) {
            this.timestamp = moment().local('tr').format('LLL')
            this.blockapi = Object.assign(res.data[this.selectedCurrency])
            console.log(this.blockapi.last)
          }
          this.loading = false
        })
        .catch(error => {
          console.log(error)
        })
    },
    coinControl (currValue) {
      const curr = this.supportedCurrencies.find(d => d.name === this.selectedCurrency)
      if (currValue < Number(curr.min)) {
        alert('En az ' + Number(curr.min) + ' ' + this.selectedCurrency + ' değerinde alım yapabilirsiniz!')
        this.currValue = 0
        this.timerCount = 0
        return false
      }
      if (currValue > Number(curr.max)) {
        alert('En fazla ' + Number(curr.max) + ' ' + this.selectedCurrency + ' değerinde alım yapabilirsiniz!')
        this.currValue = 0
        this.timerCount = 0
        return false
      }
    },
    setCurrency (event) {
      this.selectedCurrency = event.target.value
      this.getUpdatedBTCValue()
    },
    addToCart (curr, coin) {
      this.coinControl(curr)
      console.log(curr)
      console.log(coin)
      if (curr > 0) {
        this.cartItems.Coin = coin
        this.cartItems.Price = curr
        this.cartItems.Symbol = this.selectedCurrency
        this.payment = true
        this.showModal = true
      }
    },
    reset () {
      this.currValue = 0
      this.coinValue = 0
      this.timerCount = 0
      this.cartItems = { Coin: '', Price: '', Symbol: '' }
    }
  },
  computed: {
    currValue: {
      get () {
        return this.value
      },
      set (value) {
        this.value = value
      }
    },
    btc () {
      return this.blockapi ? this.blockapi.last : null
    },
    displayBTCPrice () {
      const formatter = new Intl.NumberFormat('en-US', {
        style: 'currency',
        currency: this.currency || 'USD',
        minimumFractionDigits: 2
      })

      return formatter.format(this.btc)
    },
    currency () {
      return this.blockapi ? this.selectedCurrency : null
    },
    conversion () {
      const val = this.value / this.btc

      let opts = { minimumFractionDigits: this.inverted ? 2 : 8 }

      if (this.inverted) {
        opts.style = 'currency'
        opts.currency = this.currency || 'USD'
      }

      const formatter = new Intl.NumberFormat('en-US', opts)
      const result = formatter.format(val)
      return result
    },
    conversionBTC () {
      const val = this.value * this.btc

      let opts = { minimumFractionDigits: 2 }

      if (this.inverted) {
        opts.style = 'currency'
        opts.currency = this.currency || 'USD'
      }

      const formatter = new Intl.NumberFormat('en-US', opts)
      const result = formatter.format(val)
      return result
    }
  },
  watch: {
    conversion: function (val) {
      this.coinValue = val
      console.log(val)
      if (Number(val) > 0) { this.timerCount = 10 }
      if (Number(val) === 0) {
        this.showModal = false
      }
    }
  }
}
</script>

<style>
body {
  background-color: #e1e1e1;
}
.container {
  margin-top: 4em;
  background-color: #fff;
  border-radius: 0.25rem;
  border: solid 1px darkgray;
}
.price-area {
  padding: 5px 10px;
  margin: 10px 0;
  border-radius: 0.2em;
  color: #4be33c;
  text-align: center;
  font-size: 1.5em;
  font-weight: bold;
  display: flex;
  align-items: center;
  justify-content: left;
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
.label {
  font-size: 0.5em;
  padding: 4px;
  border-radius: 0.2em;
  font-weight: 900;
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
