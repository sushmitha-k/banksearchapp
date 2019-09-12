<template lang="html">
  <div class="listdata">
    <div class="ld-header">
      <div class="ld-leftarrow" @click="goBack()">&larr;</div>
      <h1 class="ld-heading">Bank Data</h1>
    </div>
    <div class="ld-gridview">
      <div class="ld-container">
        <span class="ld-cell-h">Bank Name</span>
        <span class="ld-cell-h">IFSC</span>
        <span class="ld-cell-h">City</span>
        <span class="ld-cell-h">State</span>
        <span class="ld-cell-h">Address</span>
      </div>
      <div class="ld-row">
        <span class="ld-cell">{{bankData.bank_name}}</span>
        <span class="ld-cell">{{bankData.ifsc}}</span>
        <span class="ld-cell">{{bankData.city}}</span>
        <span class="ld-cell">{{bankData.state}}</span>
        <span class="ld-cell">{{bankData.address}}</span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ListData',
  props: {
    bankData: {}
  },
  methods: {
    goBack: function () {
      this.$emit('goBack')
    }
  }
}
</script>

<style lang="css" scoped>
</style>
