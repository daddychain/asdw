<template>
  <div class="home">
    <button @click="swap">Hello world</button>
  </div>
</template>
<script>
import {mapState} from 'vuex'
import {connectNetwork} from '@/utils/getWeb3'
import {getBalance} from "@/utils/common"
import countTo from 'vue-count-to'
import config from '../service/index'
import Web3 from 'web3'

export default {
  name: 'Home',
  components: {
    countTo
  },
  filters: {
    timeValue (val) {
      if (val < 10) return '0' + val;
      return val;
    }
  },
  data () {
    return {}
  },
  destroyed () {
    if (this.interval) {
      clearInterval(this.interval)
    }
  },
  computed: {
    web3Register () {
      return this.$store.state.web3Register
    },
    ...mapState({
      balance: state => state.balance
    }),
  },
  watch: {
    web3Register (newval, old) {
      if (newval.accounts) {
        this.$utils.getBalance(newval.accounts)
      }
    }
  },
  mounted () {
    this.getAllowance()
    this.fnCountdown()
    const _this = this
    this.interval = setInterval(() => {
      _this.fnCountdown()
    }, 1000)
  },
  methods: {
    swap () {
      this.$gtag.event('swap', { method: 'gi' })
    },
    approve () {
      const {contract, symbol_abi} = this.$config
      const _contract = new this.$metaMaSKWeb3.eth.Contract(symbol_abi, contract.symbol_contract)
      return new Promise((resolve, reject) => {
        const sum = this.$metaMaSKWeb3.utils.toWei(String(10000000000), 'ether')
        this.overlay = true
        _contract.methods.approve(contract.swap_contract, sum).send({
          from: this.web3Register.accounts,
        }).then(res => {
          this.overlay = false
          if (res.status) {
            this.isApprove = true
            this.$msg({message: 'Authorization Succeeded', type: 'success', customClass: 'msg'})
          }
        }).catch((err) => {
          this.overlay = false
          this.$msg({message: 'Cancel Authorization', type: 'error', customClass: 'msg'})
        })
      })
    },
    getAllowance (item) {
      if (this.web3Register.accounts) {
        const {contract, symbol_abi} = this.$config
        const _contract = new this.$metaMaSKWeb3.eth.Contract(symbol_abi, contract.symbol_contract)
        _contract.methods.allowance(this.web3Register.accounts, contract.swap_contract).call((err, result) => {
          if (err) return
          if (Number(this.$metaMaSKWeb3.utils.fromWei(result, 'ether')) > 0) {
            this.isApprove = true
          }
        })
      }
    },
    fnCountdown () {
      const _this = this
      let currentTime = this.$moment().valueOf();
      if (currentTime < this.startTime) {
        let t = this.startTime - currentTime;
        _this.showtime = t
      } else {
        this.isStart = true
      }
    }
  }
}
</script>
<style scoped lang="scss"></style>
