<template>
  <div id="app">
    <Introduction />
    <Explanation />
    <Chart />
    <Disclaimer />
  </div>
</template>

<script>
import Introduction from  './components/Introduction.vue';
import Explanation from  './components/Explanation.vue';
import Chart from  './components/Chart.vue';
import Disclaimer from  './components/Disclaimer.vue';

export default {
  name: 'App',
  components: {
    Introduction,
    Explanation,
    Chart,
    Disclaimer
  },
  mounted() {
    console.log("App loaded");
  }
}
</script>
<style>
  @import './assets/styles/style.css';
  @import './assets/styles/fonts.css';
  #app { display:flex; flex-direction: column; align-items: center; width: 100%; background: rgba(70,130,180, 0.3); transition: background 500ms ease;}
  #app.perc  { background: rgba(128,0,0, 0.3);}
</style>