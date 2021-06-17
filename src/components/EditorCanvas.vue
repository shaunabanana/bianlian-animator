<template>
    <canvas id="editor-canvas" hidpi="on"
        :style="{background: background}"
    >
    </canvas>
</template>

<script>
import { paper } from "paper";

export default {
    name: "EditorCanvas",
    props: {
        layers: { type: Array, required: true },
        width: { type: Number, required: true},
        height: { type: Number, required: true},
        background: { type: String, default: 'gray'}
    },

    data () {
        return {
            shapes: [],
            tool: null,
            isEditing: false,
            hitResult: null,
            hitOptions: {
                segments: true,
                stroke: true,
                fill: true,
                handles: true,
                tolerance: 5
            },
            interShape: null,
            fromShape: null,
            toShape: null
        }
    },

    computed: {
        editing: {
            get: function () {
                return this.isEditing;
            },
            set: function (value) {
                if (this.isEditing && !value) {
                    const layerId = this.shapes.indexOf(this.hitResult.item);
                    const svg = this.hitResult.item.exportSVG();
                    this.$emit('update', layerId, svg.attributes['d'].value);
                }
                this.isEditing = value;
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
                const path = new paper.Path(layer.shape);
                path.strokeColor = layer.attributes['stroke'];
                path.strokeWidth = layer.attributes['stroke-width'];
                this.shapes[index] = path;
            });
        }
    },

    mounted () {
        var canvas = document.getElementById('editor-canvas');
        paper.setup(canvas);

        // paper.view.onFrame = this.draw.bind(this);

        this.tool = new paper.Tool();
        this.tool.onMouseDown = this.onMouseDown.bind(this);
        this.tool.onMouseUp = this.onMouseUp.bind(this);
        this.tool.onMouseDrag = this.onMouseDrag.bind(this);

        this.interShape = new paper.Path({insert: false});
        this.fromShape = new paper.Path({insert: false});
        this.toShape = new paper.Path({insert: false});
        
    },

    methods: {
        reset () {
            this.shapes.forEach( shape => { shape.remove() })
            this.shapes = [];
            this.isEditing = false;
            this.hitResult = null;
        },

        getLayerShape(layerId) {
            return this.shapes[layerId];
        },

        interpolate(layer, frame1, frame2, percentage) {
            let keys = this.layers[layer].keys;

            if (frame1 < 0) frame1 = 0;
            if (frame1 > keys.length - 1) frame1 = keys.length - 1;
            if (frame2 < 0) frame2 = 0;
            if (frame2 > keys.length - 1) frame2 = keys.length - 1;

            if (keys.length === 0) {
                this.interShape.pathData = this.layers[layer].shape;
                return this.interShape;
            } else if (frame1 === frame2) {
                this.interShape.pathData = keys[frame1];
                return this.interShape;
            } else {
                this.fromShape.pathData = keys[frame1];
                this.toShape.pathData = keys[frame2];
                this.interShape.interpolate(this.fromShape, this.toShape, percentage);
                return this.interShape;
            }
        },

        pathify(shape) {
            return shape.exportSVG().attributes['d'].value;
        },

        draw () {
            this.layers.forEach ( (layer, layerId) => {
                if (layer.time) {
                    const shape = this.interpolate(
                        layerId, 
                        layer.time.start, layer.time.end, 
                        layer.time.percentage
                    );
                    this.shapes[layerId].pathData = shape.pathData;
                }
            })
            paper.view.draw();
        },

        onMouseDown (event) {
            this.hitResult = paper.project.hitTest(event.point, this.hitOptions);
            this.shapes.forEach( shape => { shape.fullySelected = false });
            if (this.hitResult) {
                this.hitResult.item.fullySelected = true;
            }
        },

        onMouseUp () {
            this.editing = false;
            if (this.hitResult) {
                // this.hitResult.item.onChange();
                this.hitResult = null;
            }
        },

        onMouseDrag (event) {
            if (this.hitResult) {
                this.editing = true;
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
