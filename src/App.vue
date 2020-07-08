<template>
  <div id="app">
    <Home></Home>
  </div>
</template>

<script>
import Vue from 'vue'
import VueCardFormat from 'vue-credit-card-validation'
import Home from './components/Home'
Vue.use(VueCardFormat)

export default {
  name: 'App',
  components: {
    Home, VueCardFormat
  }
}
</script>
