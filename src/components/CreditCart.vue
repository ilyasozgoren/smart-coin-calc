<template>
<div class="container">
    <form>
    <div class="row">
        <div class="col-sm-12 col-md-12 col-lg-12">
        <div class="form-group">
            <label>Kart Sahibi:</label>
            <input id="card-name" :data-error="(cardErrors.cardNameSurname)?true:false" v-model="cardNameSurname" maxlength="100" class="form-control">
            <div v-if="cardErrors.cardNameSurname" class="error">
            <small>{{ cardErrors.cardNameSurname }}</small>
            </div>
        </div>
        </div>
    </div>
    <div class="row">
        <div class="col-sm-12 col-md-12 col-lg-12">
        <div class="form-group">
            <label>Kart Numarası:</label>
            <div class="input-group mb-0">
            <div class="input-group-prepend">
                <span class="input-group-text" id="basic-addon1"><i :class="cardBrandClass"></i></span>
            </div>
            <input ref="cardNumInput" :data-error="(cardErrors.cardNumber)?true:false" v-model="cardNumber" type="tel" class="form-control" placeholder="#### #### #### ####" v-cardformat:formatCardNumber>
            </div>
            <div v-if="cardErrors.cardNumber" class="error">
            <small>{{ cardErrors.cardNumber }}</small>
            </div>
        </div>
        </div>
    </div>
    <div class="row">
        <div class="col-6 col-lg-6">
        <div class="form-group">
            <label>Son Kullanım Tarihi:</label>
            <input ref="cardExpInput" id="card-exp" :data-error="(cardErrors.cardExpiry)?true:false" v-model="cardExpiry" maxlength="10" class="form-control" v-cardformat:formatCardExpiry>
            <div v-if="cardErrors.cardExpiry" class="error">
            <small>{{ cardErrors.cardExpiry }}</small>
            </div>
        </div>
        </div>
        <div class="col-6 col-lg-6">
        <div class="form-group">
            <label>CVC:</label>
            <input ref="cardCvcInput" :data-error="(cardErrors.cardCvc)?true:false" v-model="cardCvc" class="form-control" v-cardformat:formatCardCVC>
            <div v-if="cardErrors.cardCvc" class="error">
            <small>{{ cardErrors.cardCvc }}</small>
            </div>
        </div>
        </div>
        <div class="col-12 col-sm-12">
        <div class="form-group" v-if="pincode">
            <label>Pin Kodu</label>
            <input v-model="pinnumber" type="password" maxlength="3" class="form-control">
        </div>
        </div>
    </div>
    <div class="row">
        <div class="col-12">
        <button type="button" class="btn btn-success" @click="validate" v-if="!pincode === true">ÖDE</button>
        <button type="button" class="btn btn-success" @click="pinControl(pinnumber)" v-if="pincode === true">ÖDEMEYİ BİTİR</button>
        <button type="button" class="btn btn-danger" @click="reset">İPTAL</button>
        </div>
    </div>
    </form>
</div>
<!-- /.container -->
</template>

<script>
export default {
  name: 'CreditCart',
  computed: {
    cardBrandClass () {
      return this.getBrandClass(this.cardBrand)
    }
  },
  data: function () {
    return {
      cardNameSurname: null,
      cardNumber: null,
      cardExpiry: null,
      cardCvc: null,
      cardPostal: null,
      cardErrors: {},
      cardBrand: null,
      pincode: false,
      pinnumber: null
    }
  },
  methods: {
    validate: function () {
      // init
      this.cardErrors = {}

      // validate card number
      if (!this.$cardFormat.validateCardNumber(this.cardNumber)) {
        this.cardErrors.cardNumber = 'Invalid Credit Card Number.'
      };

      // validate card expiry
      if (!this.$cardFormat.validateCardExpiry(this.cardExpiry)) {
        this.cardErrors.cardExpiry = 'Invalid Expiration Date.'
      };

      // validate card CVC
      if (!this.$cardFormat.validateCardCVC(this.cardCvc)) {
        this.cardErrors.cardCvc = 'Invalid CVC.'
      };
      this.pincode = true
      /// this.ode(this.pinnumber)
      // return true
    },
    reset: function () {
      this.cardErrors = {}
      this.cardNumber = null
      this.cardExpiry = null
      this.cardCvc = null
      this.cardPostal = null
      this.$refs.cardNumInput.focus()
    },
    pinControl: function (pinnumber) {
      console.log(pinnumber)
      if (pinnumber === '123') {
        alert('Ödemeniz başarıyla alındı. Teşekkür ederiz')
      } else {
        alert('Pin kodu Yanlış! Ödeme İşlemi Gerçekleştirelemedi.')
      }
    },
    prefill: function (ccNum) {
      this.cardNumber = ccNum
      this.$cardFormat.setCardType({
        currentTarget: this.$refs.cardNumInput,
        value: ccNum
      })
    },
    getBrandClass: function (brand) {
      let icon = ''
      brand = brand || 'unknown'
      let cardBrandToClass = {
        'visa': 'fab fa-cc-visa',
        'mastercard': 'fab fa-cc-mastercard',
        'amex': 'fab fa-cc-amex',
        'discover': 'fab fa-cc-discover',
        'diners': 'fab fa-cc-diners-club',
        'jcb': 'fab fa-cc-jcb',
        'unknown': 'fa fa-credit-card'
      }
      if (cardBrandToClass[brand]) {
        icon = cardBrandToClass[brand]
      };

      return icon
    }
  },
  watch: {
    // handle auto-focus to next field
    // Note: since CVC can be 3 OR 4 digits we don't watch it
    cardNumber: function (val) {
      if (this.$cardFormat.validateCardNumber(val)) {
        this.$refs.cardExpInput.focus()
      }
    },
    cardExpiry: function (val) {
      if (this.$cardFormat.validateCardExpiry(val)) {
        this.$refs.cardCvcInput.focus()
      }
    }
  },
  mounted () {
    this.$refs.cardNumInput.focus()
  }
}
</script>
