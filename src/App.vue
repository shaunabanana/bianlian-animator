<template>
    <v-app>
        <app-bar :name="name" @new-project="newProject"/>

        <v-main>
            <timeliner
                :layers="layers"
                @update="updateTime"
            />
            <editor-canvas
                :width="width"
                :height="height"
                :layers="layers"
            />
        </v-main>
    </v-app>
</template>

<script>
import AppBar from "./components/AppBar.vue";
import Timeliner from "./components/Timeliner.vue";
import EditorCanvas from "./components/EditorCanvas.vue";

import { parse } from 'svgson';

export default {
    name: "App",

    components: {
        AppBar,
        Timeliner,
        EditorCanvas,
    },

    data: () => ({
        width: 640,
        height: 480,
        name: "Untitled",
        layers: [],
        timeliner: {},
    }),

    mounted() {},

    methods: {
        newProject (svgString) {
            parse(svgString).then(svg => {
                console.log(svg);
                if (svg.attributes.width) {
                    this.width = Number.parseFloat(svg.attributes.width.replace('px', ''));
                }
                if (svg.attributes.height) {
                    this.height = Number.parseFloat(svg.attributes.height.replace('px', ''));
                }
                // See if the SVG has a title element. If so, this will be the project name.
                const titleElements = svg.children.filter( node => node.name === 'title' );
                if (titleElements.length > 0) {
                    if (titleElements[0].children.length > 0) {
                        const titleTexts = titleElements[0].children.filter( node => node.type === 'text')
                        if (titleTexts.length > 0) {
                            this.name = titleTexts[0].value;
                        }
                    }
                }

                // Find the first group, and use its children as the elements.
                const groupElements = svg.children.filter( node => node.name === 'g' );
                if (groupElements.length > 0) {
                    if (groupElements[0].children.length > 0) {
                        groupElements[0].children.forEach((node, index) => {
                            const name = `Shape ${index}`;
                            this.layers.push({
                                name: name,
                                keys: [],
                                time: null,
                                shape: node
                            });
                        });
                    }
                }
            })
            console.log(this.layers);
        },


        updateTime (time) {
            console.log(time);
            time.forEach( (data, index) => {
                this.layers[index].time = data
            })
        }
    },
};
</script>

<style>
input.hidden {
    position: fixed;
    top: -100em;
}
</style>