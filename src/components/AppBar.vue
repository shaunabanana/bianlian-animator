<template>
    <v-app-bar app dense>
        <v-app-bar-title>
            <v-icon size="1.8rem">mdi-baby-face</v-icon>
            Animator {{ name.length > 0 ? ' - ' + name : '' }}
        </v-app-bar-title>

        <v-spacer></v-spacer>

        <v-menu offset-y>
            <template v-slot:activator="{ on, attrs }">
                <v-btn text v-bind="attrs" v-on="on"> Project </v-btn>
            </template>
            <v-list dense>
                <v-list-item @click="newProject">
                    <v-list-item-icon>
                        <v-icon>mdi-new-box</v-icon>
                    </v-list-item-icon>
                    <v-list-item-content>
                        <v-list-item-title> New </v-list-item-title>
                        <input
                            class="hidden"
                            type="file"
                            id="new-project"
                            @change="readSVG"
                        />
                    </v-list-item-content>
                </v-list-item>

                <v-list-item @click="newProject">
                    <v-list-item-icon>
                        <v-icon>mdi-folder-open</v-icon>
                    </v-list-item-icon>
                    <v-list-item-content>
                        <v-list-item-title> Open </v-list-item-title>
                        <input class="hidden" type="file" id="open-project" />
                    </v-list-item-content>
                </v-list-item>

                <v-list-item @click="newProject">
                    <v-list-item-icon>
                        <v-icon>mdi-content-save</v-icon>
                    </v-list-item-icon>
                    <v-list-item-content>
                        <v-list-item-title> Save </v-list-item-title>
                    </v-list-item-content>
                </v-list-item>
            </v-list>
        </v-menu>

        <v-menu offset-y>
            <template v-slot:activator="{ on, attrs }">
                <v-btn text v-bind="attrs" v-on="on"> Keyframe </v-btn>
            </template>
            <v-list dense>
                <v-list-item @click="newProject">
                    <v-list-item-icon>
                        <v-icon>mdi-new-box</v-icon>
                    </v-list-item-icon>
                    <v-list-item-content>
                        <v-list-item-title> Load </v-list-item-title>
                        <input
                            class="hidden"
                            type="file"
                            id="new-project"
                            @change="readSVG"
                        />
                    </v-list-item-content>
                </v-list-item>

                <v-list-item @click="newProject">
                    <v-list-item-icon>
                        <v-icon>mdi-folder-open</v-icon>
                    </v-list-item-icon>
                    <v-list-item-content>
                        <v-list-item-title> Save </v-list-item-title>
                        <input class="hidden" type="file" id="open-project" />
                    </v-list-item-content>
                </v-list-item>

                <v-list-item @click="newProject">
                    <v-list-item-icon>
                        <v-icon>mdi-content-save</v-icon>
                    </v-list-item-icon>
                    <v-list-item-content>
                        <v-list-item-title> Copy </v-list-item-title>
                    </v-list-item-content>
                </v-list-item>

                <v-list-item @click="newProject">
                    <v-list-item-icon>
                        <v-icon>mdi-content-save</v-icon>
                    </v-list-item-icon>
                    <v-list-item-content>
                        <v-list-item-title> Paste </v-list-item-title>
                    </v-list-item-content>
                </v-list-item>

                <v-list-item @click="newProject">
                    <v-list-item-icon>
                        <v-icon>mdi-content-save</v-icon>
                    </v-list-item-icon>
                    <v-list-item-content>
                        <v-list-item-title> Reset </v-list-item-title>
                    </v-list-item-content>
                </v-list-item>
            </v-list>
        </v-menu>
    </v-app-bar>
</template>

<script>
export default {
    name: "AppBar",

    props: {
        name: { type: String, default: "" }
    },

    methods: {
        newProject() {
            document.querySelector("input#new-project").click();
        },

        openProject() {
            document.querySelector("input#open-project").click();
        },

        readSVG(event) {
            if (event.target.files.length < 1) return;

            let fileReader = new FileReader();
            fileReader.onload = function (event) {
                this.$emit('new-project', event.target.result);
            }.bind(this);
            fileReader.readAsText(event.target.files[0]);

        },
    }
};
</script>

<style scoped>
</style>
