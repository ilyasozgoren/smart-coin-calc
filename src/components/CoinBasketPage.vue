<template>
  <div class="contaier">
    <div class="row">
        <div class="col-12"></div>
    </div>
    <div class="row">
        <div class="col-12">
          <table class="table table-striped">
              <thead class="thead-dark">
              <tr>
                <th scope="col">#</th>
                <th scope="col">Coin Miktar</th>
                <th scope="col">Ödenecek Tutar</th>
              </tr>
              </thead>
              <tbody>
              <tr>
                <th scope="row">1</th>
                <td style="width:120px">{{items.Coin}}</td>
                <td class="text-right">{{items.Price}} {{items.Symbol}}</td>
              </tr>
              </tbody>
          </table>
        </div>
    </div>
  </div>
<!-- /.container -->
</template>

<script>
export default {
  name: 'CoinBasketPage',
  props: ['items'],
  data () {
    return {
    }
  },
  methods: {

  }
}
</script>
