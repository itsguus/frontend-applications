<template>
  <div id="app">
    <Chart />
  </div>
</template>

<script>
import Chart from  './components/Chart.vue';

export default {
  name: 'App',
  components: {
    Chart
  },
  mounted() {
    console.log("App loaded");
  }
}
</script>
<style>
  @import './assets/styles/style.css';
  @import './assets/styles/fonts.css';
  #app { display:flex; justify-content: center; width: calc(100% - 2rem); }
</style>