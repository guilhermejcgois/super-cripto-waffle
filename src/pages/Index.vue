<template>
  <q-page>
    <q-ajax-bar
      ref="bar"
      position="bottom"
      color="accent"
      size="10px"
      skip-hijack
    />

    <q-form
      @submit="onSubmit"
      @reset="onReset"
      class="token-form row items-center justify-start">
      <p>Bitcoin Trade API Token:</p>
      <q-input
        filled
        v-model="apiToken"
        lazy-rules
        :rules="[ val => val && val.length > 0 || 'Please type something']"
        />
      <q-btn label="Connect" type="submit" color="primary"/>
      <q-btn label="Disconnect" type="reset" color="primary" flat/>
    </q-form>

    <div class="balances__list q-pa-md row items-start q-gutter-md">
      <q-card class="balances__card-item" v-for="balance in balances" :key="balance.currency_code">
        <q-card-section>
          <div class="text-h6">{{balance.currency_code}}</div>
        </q-card-section>
        <q-card-section>
          <p>Disponível: {{balance.available_amount}}</p>
          <p>Em uso: {{balance.locked_amount}}</p>
        </q-card-section>
        <q-card-actions>
          <q-btn label="Ver Ordens" @click="listOrder(balance.currency_code)" flat/>
        </q-card-actions>
      </q-card>
    </div>

    <div class="orders__list" v-if="orders && orders.length">
      <q-table
        :title="currentCurrency"
        :data="orders"
        :columns="columns"
        hide-bottom
      />
    </div>
  </q-page>
</template>

<style>
  .balances__card-item {
    min-width: 170px;
  }
  .token-form {
    margin-top: 10px;
  }
  .token-form>*:first-child {
    margin-left: 20px;
  }
  .token-form>*:not(:first-child) {
    margin-left: 10px;
  }
  .token-form .q-btn {
    margin-bottom: 20px;
  }
  .token-form .q-field {
    width: 550px;
  }
  .token-form .q-field__control {
    height: 38px;
  }
</style>

<script>
export default {
  axiosInstance: null,
  data () {
    return {
      apiToken: '',
      balances: [],
      columns: [
        { name: 'type', required: true, label: 'Tipo', field: row => row.type },
        { name: 'status', required: true, label: 'Estado', field: row => row.status },
        { name: 'unit_price', required: true, label: 'Preço Unit.', field: row => row.unit_price },
        { name: 'requested_amount', required: true, label: 'Qtde Total', field: row => row.requested_amount },
        { name: 'remaining_amount', required: true, label: 'Qtde Restante', field: row => row.remaining_amount },
        { name: 'executed_amount', required: true, label: 'Qtde Ordem', field: row => row.executed_amount },
        { name: 'total_price', required: true, label: 'Preço Total', field: row => row.total_price }
      ],
      currentCurrency: '',
      orders: []
    }
  },
  methods: {
    listOrder (code) {
      this.orders = []

      const DATE_FORMAT = 'YYYY-MM-DD'
      const bar = this.$refs.bar

      bar.start()

      let url = '/market/user_orders/list?'
      url += `pair=BRL${code}&`
      url += `start_date=${this.$moment.utc().subtract(1, 'months').startOf('month').format(DATE_FORMAT)}&`
      url += `end_date=${this.$moment.utc().format(DATE_FORMAT)}&`
      url += 'page_size=100&current_page=1'
      this.axiosInstance.get(url)
        .then(res => { this.orders = res.data.data.orders })
        .catch(console.error)
        .then(() => { bar.stop() })

      this.currentCurrency = code
    },

    onSubmit () {
      const bar = this.$refs.bar

      bar.start()

      const API_URL = 'https://api.bitcointrade.com.br/v2'
      this.axiosInstance = this.$axios.create({
        baseURL: API_URL,
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `ApiToken ${this.apiToken}`
        }
      })
      this.axiosInstance.get('/wallets/balance')
        .then(res => { this.balances = res.data.data })
        .catch(console.error)
        .then(() => { bar.stop() })
    },

    onReset () {
      this.apiToken = null
      this.balances = null
      this.currentCurrency = null
      this.orders = null
    }
  },
  name: 'PageIndex'
}
</script>
