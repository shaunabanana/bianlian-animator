<template>
  <div class="timeliner-container">
    
  </div>
</template>

<script>
import {Timeliner} from 'timeliner/src/timeliner'
export default {
  name: 'Timeliner',
  props: {
    data: {type: Object, required: true}
  },

  mounted () {
    // Initialize timeliner
    let timeliner = new Timeliner(this.config);
    // this.Object.forEach(key => {
    //   timeliner.addLayer(key);
    // });
    timeliner.on('update', data => {
      if (data.length > 0) {
        console.log(data);
      }
    })
  },

  watch: {
    layers (value) {
      console.log('Layers changed', value);
    }
  }
}
</script>

<style scoped>

</style>
