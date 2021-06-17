<template>
    <div class="timeliner-container"></div>
</template>

<script>
import { Timeliner } from "timeliner/src/timeliner";
export default {
    name: "Timeline",
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
            this.timeliner = new Timeliner(this.data);
            layers.forEach(layer => {
                this.timeliner.addLayer(layer.name);
            });

            this.timeliner.on("update", (data) => {
                if (data.length > 0) {
                    this.$emit('update', data);
                }
            });

            this.timeliner.on("keyframe.add", (layerId, index, percentage) => {
                this.$emit('key', layerId, index, percentage);
            });

            this.timeliner.on("keyframe.update", (layerId, index) => {
                this.$emit('rekey', layerId, index);
            });

            this.timeliner.on("keyframe.delete", (layerId, index) => {
                this.$emit('unkey', layerId, index);
            });
        },
    },

    methods: {
        key (index) {
            this.timeliner.fire('keyframe', index);
        }
    }
};
</script>

<style scoped>
</style>
