<template lang="pug">
  section#app
    .container.mt-5
      h1 EOS javascript API demo
      b-card(no-body)
        b-tabs(card)
          b-tab(title="Generate keys")
            h3 Press Generate to create brand new keypair
            b-form
              b-form-group(label="Private key in WIF format: keep it safe to access your wallet")
                b-form-input(v-model="keyPair.private")
              b-form-group(label="Public key: it's your address in EOS")
                b-form-input(v-model="keyPair.public")
              b-button(primary @click="generateKeyPair") Generate
          b-tab(title="Create account")
            h3 Use generated private key from previous tab as password. See console for result
            b-form
              b-form-group(label="New account name (2-12 unique symbols)")
                b-form-input(v-model="newAccount.login")
              b-form-group(label="Private key in WIF format: keep it safe to access your wallet")
                b-form-input(v-model="newAccount.password")
              b-form-group(label="Result")
                b-form-textarea(v-model="newAccount.result" rows="10")
              b-button(primary @click="createAccountWithPrivateKey(newAccount.login, newAccount.password)") Create
          b-tab(title="Send tokens")
            h3 Type from, to, funds owner private key, amount to issue and and press "Send"
            b-form
              b-form-group(label="From account")
                b-form-input(v-model="sendToken.from")
              b-form-group(label="To account")
                b-form-input(v-model="sendToken.to")
              b-form-group(label="Amount (N CUR)")
                b-form-input(v-model="sendToken.amount")
              b-form-group(label="Password")
                b-form-input(v-model="sendToken.password")
              b-form-group(label="Result")
                b-form-textarea(v-model="sendToken.result" rows="10")
              b-button(primary @click="transferTokens(sendToken.from, sendToken.to, sendToken.amount, sendToken.password)") Claim
          b-tab(title="Issue tokens")
            h3 Type account name, amount to issue and and press "Claim"
            b-form
              b-form-group(label="Existing account name")
                b-form-input(v-model="issueToken.name")
              b-form-group(label="Amount (N CUR)")
                b-form-input(v-model="issueToken.amount")
              b-form-group(label="Result")
                b-form-textarea(v-model="issueToken.result" rows="10")
              b-button(primary @click="issueTokens(issueToken.name, issueToken.amount)") Claim
          b-tab(title="Account info")
            h3 Type account name and press "Fetch"
            b-form
              b-form-group(label="Existing account name")
                b-form-input(v-model="accountInfo.name")
              b-form-group(label="Account information")
                b-form-textarea(v-model="accountInfo.info" rows="10")
              b-button(primary @click="getAccountInfo(accountInfo.name)") Fetch info
          b-tab(title="Account balance")
            h3 Type account name and press "Fetch"
            b-form
              b-form-group(label="Existing account name")
                b-form-input(v-model="accountInfo.name")
              b-form-group(label="Account balance")
                b-form-textarea(v-model="accountInfo.info" rows="10")
              b-button(primary @click="getAccountBalance(accountInfo.name)") Fetch balance
          b-tab(title="Account history")
            h3 Type account name and press "Fetch"
            b-form
              b-form-group(label="Existing account name")
                b-form-input(v-model="accountHistory.name")
              b-form-group(label="Account history")
                b-form-textarea(v-model="accountHistory.info" rows="10")
              b-button(primary @click="getAccountHistory(accountHistory.name)") Fetch history
</template>

<script>
import * as Eos from 'eosjs'
let {ecc} = Eos.modules

export default {
  name: 'app',
  data () {
    this.CONTRACT_NAME = 'currency'
    return {
      title: 'eosjs test app',
      privateKey: '',
      masterPrivate: '5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3',
      keyPair: {
        private: '',
        public: ''
      },
      newAccount: {
        login: '',
        password: '',
        result: ''
      },
      accountInfo: {
        name: '',
        info: ''
      },
      accountHistory: {
        name: '',
        info: ''
      },
      issueToken: {
        name: '',
        amount: '',
        result: ''
      },
      sendToken: {
        password: '',
        from: '',
        to: '',
        amount: '',
        result: ''
      },
      eos: null
    }
  },
  async mounted () {
    this.eos = window.EOS = Eos.Localnet({keyProvider: this.masterPrivate})
    window.ecc = ecc
  },
  methods: {
    async createAccountWithPrivateKey (name, privateKey) {
      const publicKey = ecc.privateToPublic(privateKey)
      // creater need permissions to create new accounts
      const result = await this.eos.newaccount({
        creator: 'eosio',
        name: this.newAccount.login,
        owner: publicKey,
        active: publicKey,
        recovery: 'eosio'
      })
      this.newAccount.result = JSON.stringify(result, null, 2)
    },

    async generateKeyPair () {
      const privateKey = await ecc.unsafeRandomKey() // wif key
      const publicKey = ecc.privateToPublic(privateKey) // EOS address, public key
      this.keyPair.private = privateKey
      this.keyPair.public = publicKey
    },

    async getAccountInfo (name) {
      const result = await this.eos.getAccount(name)
      this.accountInfo.info = JSON.stringify(result, null, 2)
    },

    async getAccountBalance (name) {
      const result = await this.eos.getCurrencyBalance(this.CONTRACT_NAME, name)
      this.accountInfo.info = JSON.stringify(result, null, 2)
    },

    async issueTokens (name, amount) {
      let currency = await this.eos.contract(this.CONTRACT_NAME)
      const result = await currency.issue(name, amount, '', {authorization: 'currency'})
      this.issueToken.result = JSON.stringify(result, null, 2)
    },

    async getAccountHistory (name) {
      const result = await this.eos.getTransactions(name)
      this.accountHistory.info = JSON.stringify(result, null, 2)
    },

    async transferTokens(from, to, amount, password) {
      this.eos = window.EOS = Eos.Localnet({keyProvider: password})
      let currency = await this.eos.contract(this.CONTRACT_NAME)
      const result = await currency.transfer(from, to, amount, 'memo')
      this.sendToken.result = JSON.stringify(result, null, 2)
      this.eos = window.EOS = Eos.Localnet({keyProvider: this.masterPrivate})
    }
  }
}
</script>
