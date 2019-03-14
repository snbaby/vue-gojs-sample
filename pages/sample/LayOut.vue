<template>
    <div id="sample">
        <div style="width: 100%; display: flex; justify-content: space-between">
            <div style="width: 135px; margin-right: 2px; background-color: whitesmoke; border: solid 1px black">
                <div id="myPaletteSmall" style="width: 140px; height: 360px"></div>
            </div>
            <div id="myDiagramDiv" style="flex-grow: 1; height: 500px; border: solid 1px black">

            </div>
        </div>
        <div>
            <pre id="savedModel" style="height:250px"></pre>
        </div>
    </div>
</template>

<script>
    import go from 'gojs';

    let $ = go.GraphObject.make;

    export default {
        data() {
            return {
                myDiagram: null,
                CellSize: null,
                groupFill: "rgba(128,128,128,0.2)",
                groupStroke: "gray",
                dropFill: "rgba(128,255,255,0.2)",
                dropStroke: "red",
                green: '#B2FF59',
                blue: '#81D4FA',
                yellow: '#FFEB3B',
                activeName: '1',

                myPaletteSmall: null,
            }
        },
        mounted() {
            this.init();
        },
        methods: {
            init() {
                const self = this;
                self.CellSize = new go.Size(document.getElementById('myDiagramDiv').clientWidth / 24, document.getElementById('myDiagramDiv').clientWidth / 24),
                    //初始化
                    self.myDiagram =
                        $(go.Diagram, "myDiagramDiv",
                            {
                                initialContentAlignment: go.Spot.Center,
                                layout: $(go.GridLayout,
                                    {
                                        comparer: function(a, b) {

                                            var ay = a.location.y;
                                            var by = b.location.y;
                                            if (isNaN(ay) || isNaN(by)) {
                                                return 0;
                                            }
                                            if (ay < by) {
                                                return -1;
                                            }
                                            if (ay > by) {
                                                return 1;
                                            }


                                            var ax = a.location.x;
                                            var bx = b.location.x;
                                            if (isNaN(ax) || isNaN(bx)) {
                                                return 0;
                                            }
                                            if (ax < bx) {
                                                return -1;
                                            }
                                            if (ax > bx) {
                                                return 1;
                                            }
                                            return 0;
                                        },
                                        wrappingWidth: 800,
                                        wrappingColumn:3,
                                        alignment:go.GridLayout.Position
                                    }),
                                "SelectionMoved": self.relayoutNodes
                                // other properties are set by the layout function, defined below
                            });

                // Regular Nodes represent items to be put onto racks.
                // Nodes are currently resizable, but if that is not desired, just set resizable to false.
                self.myDiagram.nodeTemplate =
                    $(go.Node, "Auto",


                        // this is the primary thing people see
                        //边框
                        $(go.Shape, "Rectangle",
                            {fill: "#eee", height: 40, strokeWidth: 0},
                            new go.Binding("fill", "color"),
                            new go.Binding("width", "width", function (v) {
                                console.log(v);
                                return v * 400
                            })
                        ),

                        //文本
                        $(go.TextBlock,
                            {margin: 5},
                            new go.Binding("text", "key")
                        )
                    );  // end Node


                // initialize the first Palette
                self.myPaletteSmall =
                    $(go.Palette, "myPaletteSmall",
                        { // share the templates with the main Diagram
                            nodeTemplate: self.myDiagram.nodeTemplate
                        }
                    );

                // specify the contents of the Palette
                self.myPaletteSmall.model = new go.Node([
                    {key: "Beta", color: "thistle", width: 1},
                    {key: "Delta", color: "lightcoral", width: 2},
                ]);
            }
        }
    }
</script>

<style scoped>
    #myDiagramDiv {
        height: 600px;
        border: solid 1px #d3d3d3;
    }
</style>
