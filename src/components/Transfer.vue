<template>
  <div class="transfer">

      <form novalidate @submit.stop.prevent="submit">
          <md-input-container  :title="$t('transfer.to_address_label_tooltip')">
              <label>{{ $t('transfer.to_address_label') }}</label>
              <md-input v-model="targetAddress"></md-input>
          </md-input-container>
          <md-input-container>
              <label style="text-align:left">{{ $t('transfer.amount_label') }} (max: {{ maxToTransfer }} ADM)</label>
              <md-input type="number" min=0 :max="maxToTransfer" v-model="targetAmount"></md-input>
          </md-input-container>
          <md-input-container>
              <label>{{ $t('transfer.commission_label') }}</label>
              <md-input type="number" readonly v-model="commission"></md-input>
          </md-input-container>
          <md-input-container>
              <label style="text-align:left">{{ $t('transfer.final_amount_label') }}</label>
              <md-input type="number" readonly v-model="finalAmount"></md-input>
          </md-input-container>
          <md-layout md-align="center" md-gutter="16">
              <md-button class="md-raised md-primary" :title="$t('transfer.send_button_tooltip')" v-on:click="transfer">{{ $t('transfer.send_button') }}</md-button>

          </md-layout>
      </form>
      <md-snackbar md-position="bottom center" md-accent ref="transferSnackbar" md-duration="2000">
          <span>{{ formErrorMessage }}</span>
      </md-snackbar>
      <md-dialog-confirm
              :md-title="$t('transfer.confirm_title')"
              :md-content-html="$t('transfer.confirm_message', {amount: targetAmount, target: targetAddress })"
              :md-ok-text="$t('transfer.confirm_approve')"
              :md-cancel-text="$t('transfer.confirm_cancel')"
              @close="onClose"
              ref="confirm_transfer_dialog">
      </md-dialog-confirm>

  </div>
</template>

<script>
export default {
  name: 'home',
  methods: {
    errorMessage (message) {
      this.formErrorMessage = this.$t('transfer.' + message)
      this.$refs.transferSnackbar.open()
    },
    onClose (type) {
      if (type === 'ok') {
        this.transferFunds(this.targetAmount, this.targetAddress)
      }
    },
    transfer: function () {
      if (!this.targetAddress) {
        this.errorMessage('error_no_address')
        return
      }
      if (!(/U([0-9]{6,})$/.test(this.targetAddress))) {
        this.errorMessage('error_incorrect_address')
        return
      }
      if (!this.targetAmount) {
        this.errorMessage('error_no_amount')
        return
      }
      if (this.targetAmount < 0) {
        this.errorMessage('error_incorrect_amount')
        return
      }
      if ((parseFloat(this.targetAmount) + this.commission) > parseFloat(this.$store.state.balance)) {
        this.errorMessage('error_not_enough')
        return
      }
      this.$refs['confirm_transfer_dialog'].open()
    }
  },
  computed: {
    maxToTransfer: function () {
      this.amountToTransfer = (Math.floor((parseFloat(this.$store.state.balance) - this.commission) * 100) / 100).toFixed(2)
      if (this.amountToTransfer < 0) {
        this.amountToTransfer = 0
      }
      return this.amountToTransfer
    }
  },
  watch: {
    targetAmount (to, from) {
      var fixedPoint = 2
      if (to.toString().indexOf('.') > -1) {
        fixedPoint = to.toString().length - to.toString().indexOf('.') - 1
        if (fixedPoint < 2) {
          fixedPoint = 2
        }
      }
      this.finalAmount = (parseFloat(to) + 0.5).toFixed(fixedPoint)
    },
    'language' (to, from) {
      this.$i18n.locale = to
    }
  },
  data () {
    return {
      finalAmount: 0,
      formErrorMessage: '',
      commission: 0.5,
      amountToTransfer: 0,
      targetAddress: '',
      targetAmount: ''
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
    .md-dialog-container.md-active .md-dialog {
        background: #fefefe;
    }
    .md-input-container.md-has-value input.md-input[readonly] {
        color: rgba(0, 0, 0, 0.54);
    }
    @media (max-width: 800px) {
        .transfer {
            padding-left: 1rem;
            padding-right: 1rem;
        }
    }
</style>
