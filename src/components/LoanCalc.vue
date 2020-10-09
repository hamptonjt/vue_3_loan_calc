<template>
  <div class="container is-fluid">
    <nav class="navbar" role="navigation" aria-label="main navigation">
      <div class="navbar-brand">
        <div class="title has-text-centered">Vue 3 Loan Calculator</div>
      </div>
    </nav>
    <div class="tile is-ancestor">
      <div class="tile is-parent is-vertical is-4">
        <div class="tile is-child box">
          <div class="field">
            <label class="label" for="amount">Loan Amount</label>
            <input class="input" type="text" placeholder="Loan Amount" v-model="state.amount" id="amount" name="amount" />
          </div>
          <div class="field">
            <label class="label" for="amount">Duration (Months)</label>
            <input class="input" type="text" placeholder="Duration" v-model="state.months" id="months" name="months" />
          </div>
          <div class="field">
            <label class="label" for="amount">Interest (%)</label>
            <input class="input" type="text" placeholder="Interest Rate" v-model="state.interest" id="interest" name="interest" />
          </div>
          <div class="control">
            <button class="button is-link" @click="calcTable">Calculate</button>
          </div>
        </div>
      </div>
      <div class="tile is-parent is-vertical is-4">
        <div class="tile is-child box">
          <div class="title has-text-centered">Loan Totals</div>
          <label class="label">Monthly Payment</label>{{ state.filteredMonthlyPayment }}
          <label class="label">Total Cost</label>{{ state.filteredTotalCost }}
          <label class="label">Total Interest</label>{{ state.filteredTotalInterest }}
        </div>
      </div>
    </div>
    <div class="table-container" width="100%" v-if="state.payData.length > 0">
      <table class="table is-bordered is-striped is-fullwidth">
        <thead>
          <tr>
            <th>Payment Number</th>
            <th>Starting Balance</th>
            <th>Payment Amount</th>
            <th>Principal Applied</th>
            <th>Interest Paid</th>
            <th>New Balance</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="p in state.payData" :key="p.paymentNum">
            <td>{{p.paymentNum}}</td>
            <td>{{p.startingBalance}}</td>
            <td>{{p.paymentAmount}}</td>
            <td>{{p.principalApplied}}</td>
            <td>{{p.interestAmount}}</td>
            <td>{{p.balanceAfterPayment}}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import { reactive, computed } from 'vue'

export default {
  name: 'LoanCalc',
  setup() {
    const digitsRE = /(\d{3})(?=\d)/g
    const state = reactive({
      amount: 0.0,
      months: 0,
      interest: 0.0,
      annualRate: computed(() => (state.interest * 0.01) / 12),
      discountFactor: computed(() => Number(((((1 + state.annualRate) ** state.months) - 1) / (state.annualRate * (1 + state.annualRate) ** state.months)).toFixed(4))),
      monthlyPayment: computed(() => Number(state.amount / state.discountFactor)),
      filteredMonthlyPayment: computed(() => currencyFilter(state.monthlyPayment)),
      totalCost: computed(() => Number(state.monthlyPayment * state.months)),
      filteredTotalCost: computed(() => currencyFilter(state.totalCost)),
      totalInterest: computed(() => Number(state.totalCost - state.amount)),
      filteredTotalInterest: computed(() => currencyFilter(state.totalInterest)),
      payData: []
    })
    const calcTable = () => {
      let runningBalance = state.amount
      state.payData = []
      for (var i=1; i<=state.months; i++) {
        let interestAmount = (((state.interest * 0.01) / 12) * runningBalance)
        let newBalance = runningBalance - (state.monthlyPayment - interestAmount)
        let principalPaid = runningBalance - newBalance
        let paymentInfo = {
          paymentNum: i,
          startingBalance: currencyFilter(runningBalance),
          paymentAmount: currencyFilter(state.monthlyPayment),
          principalApplied: currencyFilter(principalPaid),
          interestAmount: currencyFilter(interestAmount),
          balanceAfterPayment: currencyFilter(newBalance)
        }
        state.payData.push(paymentInfo)
        runningBalance = newBalance
      }
    }
    function currencyFilter (value, currency, decimals) {
      value = parseFloat(value)
      if (!isFinite(value) || (!value && value !== 0)) return ''
      currency = currency != null ? currency : '$'
      decimals = decimals != null ? decimals : 2
      var stringified = Math.abs(value).toFixed(decimals)
      var _int = decimals
        ? stringified.slice(0, -1 - decimals)
        : stringified
      var i = _int.length % 3
      var head = i > 0
        ? (_int.slice(0, i) + (_int.length > 3 ? ',' : ''))
        : ''
      var _float = decimals
        ? stringified.slice(-1 - decimals)
        : ''
      var sign = value < 0 ? '-' : ''
      return sign + currency + head +
        _int.slice(i).replace(digitsRE, '$1,') +
        _float
    }
    return { state, calcTable }
  }

}
</script>
