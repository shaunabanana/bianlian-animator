<template>
    <canvas id="editor-canvas" hidpi="on">
    </canvas>
</template>

<script>
import { paper } from "paper";
import toPath from "element-to-path";

export default {
    name: "EditorCanvas",
    props: {
        layers: { type: Array, required: true },
        width: { type: Number, required: true},
        height: { type: Number, required: true}
    },

    data () {
        return {
            shapes: [],
            tool: null,
            editing: false,
            hitResult: null,
            hitOptions: {
                segments: true,
                stroke: true,
                fill: true,
                handles: true,
                tolerance: 5
            }
        }
    },

    watch: {
        width () {
            paper.view.viewSize = new paper.Size(this.width, this.height);
        },

        height () {
            paper.view.viewSize = new paper.Size(this.width, this.height);
        },

        layers () {
            this.shapes = [];
            this.layers.forEach ( (layer, index) => {
                const pathData = toPath(layer.shape);
                const path = new paper.Path(pathData);
                path.strokeColor = layer.shape.attributes['stroke'];
                path.strokeWidth = layer.shape.attributes['stroke-width'];
                this.shapes[index] = path;
            });
        }
    },

    methods: {
        draw () {
            this.layers.forEach ( (layer) => {
                if (layer.time) {
                    //
                }
            })
            paper.view.draw();
        }
    },

    mounted () {
        // paper.install(window);
        var canvas = document.getElementById('editor-canvas');
        console.log(canvas);
        paper.setup(canvas);

        paper.view.onFrame = this.draw.bind(this);

        this.tool = new paper.Tool();
        this.tool.onMouseDown = (event) => {
            console.log(event);
            this.hitResult = paper.project.hitTest(event.point, this.hitOptions);
            this.shapes.forEach( shape => { shape.fullySelected = false });
            if (this.hitResult) {
                this.hitResult.item.fullySelected = true;
            }
        }

        this.tool.onMouseUp = (event) => {
            console.log(event);
            this.editing = false;
            if (this.hitResult) {
                // this.hitResult.item.onChange();
                this.hitResult = null;
            }
        }

        this.tool.onMouseDrag = (event) => {
            this.editing = true;
            if (this.hitResult) {
                console.log(this.hitResult.type);
                if (this.hitResult.type === 'fill' || this.hitResult.type === 'stroke') {
                    let item = this.hitResult.item;
                    item.position.x += event.delta.x;
                    item.position.y += event.delta.y;
                } if (this.hitResult.type === 'segment') {
                    let segment = this.hitResult.segment;
                    segment.point.x += event.delta.x;
                    segment.point.y += event.delta.y;
                } else if (this.hitResult.type === 'handle-in') {
                    let handle = this.hitResult.segment.handleIn;
                    handle.x += event.delta.x;
                    handle.y += event.delta.y;
                    if (!paper.Key.isDown('shift')) {
                        let otherHandle = this.hitResult.segment.handleOut;
                        otherHandle.angle = handle.angle + 180
                    }
                } else if (this.hitResult.type === 'handle-out') {
                    let handle = this.hitResult.segment.handleOut;
                    handle.x += event.delta.x;
                    handle.y += event.delta.y;
                    if (!paper.Key.isDown('shift')) {
                        let otherHandle = this.hitResult.segment.handleIn;
                        otherHandle.angle = handle.angle + 180
                    }
                }
            } else {
                this.shapes.forEach( shape => { shape.fullySelected = false });
            }
        }
        
    }
};
</script>

<style scoped>

canvas {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -60%);
}

</style>
