<template>
    <div class="timeliner-container"></div>
</template>

<script>
import { Timeliner } from "timeliner/src/timeliner";
export default {
    name: "Timeline",
    props: {
        layers: { type: Array, required: true },
        focusCapture: { type: Boolean, default: false }
    },
    
    data () {
        return {
            timeliner: null,
            data: {}
        }
    },

    watch: {
        focusCapture () {
            if (!this.timeliner) return;
            this.timeliner.setFocusCapture(this.focusCapture);
        }
    },

    methods: {
        reset () {
            if (this.timeliner) this.timeliner.dispose();
            this.timeliner = null;
            this.data = null;
        },

        key (index) {
            this.timeliner.fire('keyframe', index);
        },

        getData () {
            return this.timeliner.getData();
        },

        setData (data) {
            this.data = {};
            data.layers.forEach(layer => {
                this.data[layer.name] = 0;
            });
            this.initTimeliner();
            this.timeliner.load(data);
            console.log(this.timeliner)
        },

        initTimeliner () {
            if (this.timeliner) {
                this.timeliner.dispose();
                this.timeliner = null;
            }

            // Initialize timeliner
            this.timeliner = new Timeliner(this.data);
            document.querySelector('timeliner').id = "timeliner";

            this.layers.forEach(layer => {
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
        }
    }
};
</script>

<style>
#timeliner select {
    color: text;
    background-color: -webkit-control-background;
    box-sizing: border-box;
    -webkit-appearance: menulist;
}
</style>
