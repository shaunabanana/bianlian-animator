<template>
  <div id="app">
    <timeliner :data="timeliner"/>
  </div>
</template>

<script>
import Timeliner from './components/Timeliner.vue'

export default {
  name: 'App',
  components: {
    Timeliner
  },

  data () {
    return {
      width: 640,
      height: 480,
      name: 'Untitled',
      shape: null,
      frames: {},
      timeliner: {}
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
