<template>
    <div class="timeliner-container"></div>
</template>

<script>
import { Timeliner } from "timeliner/src/timeliner";
export default {
    name: "Timeliner",
    props: {
        layers: { type: Array, required: true },
    },
    
    data () {
        return {
            timeliner: null,
            data: {}
        }
    },

    mounted() {
        
    },

    watch: {
        layers(layers) {
            console.log(layers);
            if (this.timeliner) this.timeliner.dispose();

            this.data = {};
            layers.forEach(layer => {
                this.data[layer.name] = 0;
            });
            console.log(this.data);

            // Initialize timeliner
            let timeliner = new Timeliner(this.data);
            layers.forEach(layer => {
                timeliner.addLayer(layer.name);
            });

            timeliner.on("update", (data) => {
                if (data.length > 0) {
                    this.$emit('update', data);
                }
            });
        },
    },
};
</script>

<style scoped>
</style>
