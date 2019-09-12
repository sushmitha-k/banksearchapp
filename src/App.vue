<template>
  <div id="app">
    <bankList></bankList>
  </div>
</template>

<script>
import bankList from './components/bankList.vue'

export default {
  name: 'app',
  components: {
    bankList
  }
}
</script>

<style>
  @import './scss/core.scss';
</style>
