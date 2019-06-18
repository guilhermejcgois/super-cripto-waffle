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
      </q-card>
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
  data () {
    return {
      apiToken: '',
      balances: []
    }
  },
  methods: {
    onSubmit () {
      const bar = this.$refs.bar

      bar.start()

      const API_URL = 'https://api.bitcointrade.com.br/v2'
      const instance = this.$axios.create({
        baseURL: API_URL,
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `ApiToken ${this.apiToken}`
        }
      })
      instance.get('/wallets/balance')
        .then(res => { this.balances = res.data.data })
        .catch(console.error)
        .then(() => { bar.stop() })
    },

    onReset () {
      this.apiToken = null
      this.balances = null
    }
  },
  name: 'PageIndex'
}
</script>
