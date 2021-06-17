<template>
    <v-app>
        <app-bar :name="name" @new-project="newProject"/>

        <v-main>
            <timeline ref="timeline"
                :layers="layers"
                @update="updateTime"
                @key="addKeyframe"
                @rekey="updateKeyframe"
                @unkey="deleteKeyframe"
            />
            <editor-canvas ref="editor"
                :width="width"
                :height="height"
                :layers="layers"
                :background="background"
                @update="updateShape"
            />
        </v-main>
    </v-app>
</template>

<script>
import AppBar from "./components/AppBar.vue";
import Timeline from "./components/Timeline.vue";
import EditorCanvas from "./components/EditorCanvas.vue";

import { parse } from 'svgson';
import toPath from "element-to-path";

export default {
    name: "App",

    components: {
        AppBar,
        Timeline,
        EditorCanvas,
    },

    data: () => ({
        width: 640,
        height: 480,
        name: "Untitled",
        layers: [],
        timeliner: {},
        background: "gray",
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
                        let index = 1;
                        groupElements[0].children.forEach((node) => {

                            console.log(node);
                            if (node.name === 'rect' && !node.attributes.id) {
                                if (node.attributes.fill) {
                                    this.background = node.attributes.fill;
                                }
                            } else {
                                let name = `Shape ${index}`;
                                index ++;
                                this.layers.push({
                                    name: name,
                                    keys: [],
                                    time: null,
                                    attributes: node.attributes,
                                    shape: toPath(node)
                                });
                            }
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
            this.$refs.editor.draw();
        },

        updateShape (layerId, data) {
            console.log("Updating layer shape", layerId, data);
            // Let timeline add a keyframe here.
            this.$refs.timeline.key(layerId);
        },

        addKeyframe (layerId, index) {
            const shape = this.$refs.editor.getLayerShape(layerId);
            this.layers[layerId].keys.splice(index, 0, shape.pathData);
            console.log(this.layers[layerId]);
        },

        updateKeyframe (layerId, index) {
            console.log('Updating keyframe', layerId, index);
            const shape = this.$refs.editor.getLayerShape(layerId);
            this.layers[layerId].keys[index] = shape.pathData;
            console.log(this.layers[layerId]);
        },

        deleteKeyframe (layerId, index) {
            this.layers[layerId].keys.splice(index, 1);
            console.log(this.layers[layerId]);
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