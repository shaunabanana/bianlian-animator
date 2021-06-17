<template>
    <v-app>
        <app-bar :name="name"
            @new-project="newProject" 
            @open-project="openProject" 
            @save-project="saveProject" 
        />

        <v-main>
            <timeline
                ref="timeline"
                :layers="layers"
                :focus-capture="!saveProjectDialogue"
                @update="updateTime"
                @key="addKeyframe"
                @rekey="updateKeyframe"
                @unkey="deleteKeyframe"
            />
            <editor-canvas
                ref="editor"
                :width="width"
                :height="height"
                :layers="layers"
                :background="background"
                @update="updateShape"
            />

            <v-dialog v-model="saveProjectDialogue" width="500">
                <v-card>
                    <v-card-title>
                        Save animation
                    </v-card-title>
                    <v-card-text>
                        <v-form>
                            <v-text-field
                                label="File name"
                                v-model="name"
                            ></v-text-field>
                        </v-form>
                    </v-card-text>
                    <v-divider></v-divider>
                    <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn color="primary" text @click="downloadProject">
                            Save
                        </v-btn>
                    </v-card-actions>
                </v-card>
            </v-dialog>
        </v-main>
    </v-app>
</template>

<script>
import AppBar from "./components/AppBar.vue";
import Timeline from "./components/Timeline.vue";
import EditorCanvas from "./components/EditorCanvas.vue";

import { parse } from "svgson";
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
        background: "gray",
        saveProjectDialogue: false
    }),

    mounted() {},

    methods: {
        reset() {
            this.width = 640;
            this.height = 480;
            this.name = "Untitled";
            this.background = "gray";
            this.layers = [];
            this.saveProjectDialogue = false;
            this.$refs.timeline.reset();
            this.$refs.editor.reset();
        },
        
        newProject(svgString) {
            this.reset();
            parse(svgString).then((svg) => {
                if (svg.attributes.width) {
                    this.width = Number.parseFloat(
                        svg.attributes.width.replace("px", "")
                    );
                }
                if (svg.attributes.height) {
                    this.height = Number.parseFloat(
                        svg.attributes.height.replace("px", "")
                    );
                }
                // See if the SVG has a title element. If so, this will be the project name.
                const titleElements = svg.children.filter(
                    (node) => node.name === "title"
                );
                if (titleElements.length > 0) {
                    if (titleElements[0].children.length > 0) {
                        const titleTexts = titleElements[0].children.filter(
                            (node) => node.type === "text"
                        );
                        if (titleTexts.length > 0) {
                            this.name = titleTexts[0].value;
                        }
                    }
                }

                // Find the first group, and use its children as the elements.
                const groupElements = svg.children.filter(
                    (node) => node.name === "g"
                );
                if (groupElements.length > 0) {
                    if (groupElements[0].children.length > 0) {
                        let index = 1;
                        groupElements[0].children.forEach((node) => {
                            if (node.name === "rect" && !node.attributes.id) {
                                if (node.attributes.fill) {
                                    this.background = node.attributes.fill;
                                }
                            } else {
                                let name = `Shape ${index}`;
                                index++;
                                this.layers.push({
                                    name: name,
                                    keys: [],
                                    time: null,
                                    attributes: node.attributes,
                                    shape: toPath(node),
                                });
                            }
                        });
                    }
                }
                this.$refs.timeline.initTimeliner();
            });
        },

        openProject(projectData) {
            this.reset();
            this.layers = projectData.layers;
            this.name = projectData.name;
            this.width = projectData.width;
            this.height = projectData.height;
            this.background = projectData.background;
            this.$refs.timeline.setData(projectData.timeliner);
        },

        saveProject() {
            this.saveProjectDialogue = true;
        },

        downloadProject() {

            const saveData = {
                width: this.width,
                height: this.height,
                name: this.name,
                layers: this.layers,
                background: this.background,
                timeliner: this.$refs.timeline.getData()
            }

            const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(saveData));
            const downloadAnchorNode = document.createElement('a');
            downloadAnchorNode.setAttribute("href", dataStr);
            downloadAnchorNode.setAttribute("download", this.name + ".json");
            document.body.appendChild(downloadAnchorNode); // required for firefox
            downloadAnchorNode.click();
            downloadAnchorNode.remove();
            this.saveProjectDialogue = false;
        },

        updateTime(time) {
            time.forEach((data, index) => {
                this.layers[index].time = data;
            });
            this.$refs.editor.draw();
        },

        updateShape(layerId) {
            // Let timeline add a keyframe here.
            this.$refs.timeline.key(layerId);
        },

        addKeyframe(layerId, index) {
            const shape = this.$refs.editor.getLayerShape(layerId);
            this.layers[layerId].keys.splice(index, 0, shape.pathData);
        },

        updateKeyframe(layerId, index) {
            const shape = this.$refs.editor.getLayerShape(layerId);
            this.layers[layerId].keys[index] = shape.pathData;
        },

        deleteKeyframe(layerId, index) {
            this.layers[layerId].keys.splice(index, 1);
        },
    },
};
</script>

<style>
input.hidden {
    position: fixed;
    top: -100em;
}
</style>