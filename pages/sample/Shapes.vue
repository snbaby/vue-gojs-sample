<template>
    <div>
        <div id="myDiagramDiv"></div>
    </div>

</template>

<script>
    import go from 'gojs';

    let $ = go.GraphObject.make;

    export default {
        data() {
            return {
                myDiagram: null,
                builtIn: ["Rectangle", "Square", "RoundedRectangle", "Border", "Ellipse", "Circle", "TriangleRight", "TriangleDown", "TriangleLeft", "TriangleUp", "Triangle", "Diamond", "LineH", "LineV", "None", "BarH", "BarV", "MinusLine", "PlusLine", "XLine"]
            }
        },
        mounted() {
            this.init();
        },
        methods: {
            init() {
                const self = this;
                //初始化
                this.myDiagram =
                    $(go.Diagram, "myDiagramDiv",  // create the Diagram for the HTML DIV element
                        {
                            layout: $(go.GridLayout, {sorting: go.GridLayout.Forward}), // use a GridLayout
                            padding: new go.Margin(5, 5, 25, 5) // to see the names of shapes on the bottom row
                        });

                this.myDiagram.nodeTemplate =
                    $(go.Node, "Vertical",
                        {
                            mouseEnter: self.mouseEnter,
                            mouseLeave: self.mouseLeave,
                            locationSpot: go.Spot.Center,  // the location is the center of the Shape
                            locationObjectName: "SHAPE",
                            selectionAdorned: false,  // no selection handle when selected
                            resizable: true, resizeObjectName: "SHAPE",  // user can resize the Shape
                            rotatable: true, rotateObjectName: "SHAPE",  // rotate the Shape without rotating the label
                            // don't re-layout when node changes size
                            layoutConditions: go.Part.LayoutStandard & ~go.Part.LayoutNodeSized
                        },
                        new go.Binding("layerName", "isHighlighted", function (h) {
                            return h ? "Foreground" : "";
                        }).ofObject(),
                        $(go.Shape,
                            {
                                name: "SHAPE",  // named so that the above properties can refer to this GraphObject
                                width: 70, height: 70,
                                strokeWidth: 3
                            },
                            // Color the built in shapes green, and the figures.js shapes Pink
                            new go.Binding("fill", "key", function (k) {
                                return self.isBuiltIn(k) ? "palegreen" : "lightpink";
                            }),
                            new go.Binding("stroke", "key", function (k) {
                                return self.isBuiltIn(k) ? "darkgreen" : "#C2185B";
                            }),
                            // bind the Shape.figure to the figure name, which automatically gives the Shape a Geometry
                            new go.Binding("figure", "key")),
                        $(go.TextBlock,  // the label
                            {
                                margin: 4,
                                font: "bold 18px sans-serif",
                                background: 'white'
                            },
                            new go.Binding("visible", "isHighlighted").ofObject(),
                            new go.Binding("text", "key"))
                    );

                this.myDiagram.model.nodeDataArray = go.Shape.getFigureGenerators().toArray();
            },
            mouseEnter(e, obj) {
                obj.isHighlighted = true;
            },
            mouseLeave(e, obj) {
                obj.isHighlighted = false;
            },
            isBuiltIn(shapeName) {
                return this.builtIn.find(function (element) {
                    return element.toLowerCase() === shapeName.toLowerCase()
                }) !== undefined;
            }
        }
    }
</script>

<style scoped>
    #myDiagramDiv {
        height: 500px;
        border: solid 1px #d3d3d3;
    }
</style>
