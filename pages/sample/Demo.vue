<template>
    <div id="sample">
        <div style="width: 100%; display: flex; justify-content: space-between">
            <div id="palette"
                 style="width: 450px; height: 500px; margin-right: 2px; background-color: whitesmoke; border: solid 1px black"></div>
            <div id="myDiagramDiv" style="flex-grow: 1; height: 500px; border: solid 1px black"></div>
            <div id="pal"
                 style="width: 450px; height: 500px; margin-left: 2px; background-color: whitesmoke; border: solid 1px black"></div>
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
                palette: null,

                red: "orangered",

            }
        },
        mounted() {
            this.init();
        },
        methods: {
            init() {
                const self = this;
                //初始化
                self.myDiagram =
                    $(go.Diagram, "myDiagramDiv",

                        {
                            initialContentAlignment: go.Spot.Center,
                            layout: $(go.GridLayout,
                                {
                                    comparer: function (a, b) {
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

                                    wrappingColumn: 3,
                                    wrappingWidth:200,
                                    cellSize:new go.Size(200, 40),
                                    spacing: new go.Size(0, 20),
                                    alignment: go.GridLayout.Position
                                }),
                            "SelectionMoved": function () {
                                self.myDiagram.layout.invalidateLayout();
                                self.myDiagram.layoutDiagram();
                            }
                        });

                let inputTemplate =
                    $(go.Node, "Auto",
                        $(go.Shape, "Rectangle",
                            {strokeWidth: 0},
                        ), // override the default fill (from shapeStyle()) to be red
                        $(go.Picture,  // the icon showing the logo
                            // You should set the desiredSize (or width and height)
                            // whenever you know what size the Picture should be.
                            {desiredSize: new go.Size(200, 40)},
                            new go.Binding("source", "category", self.convertKeyImage)),
                    );

                let selectTemplate =
                    $(go.Node, "Auto",
                        $(go.Shape, "Rectangle",
                            {strokeWidth: 0},
                        ),
                        $(go.Picture,  // the icon showing the logo
                            // You should set the desiredSize (or width and height)
                            // whenever you know what size the Picture should be.
                            {desiredSize: new go.Size(200, 40)},
                            new go.Binding("source", "category", self.convertKeyImage)),
                    );

                let inputMaxTemplate =
                    $(go.Node, "Auto",
                        $(go.Shape, "Rectangle",
                            {strokeWidth: 0},
                        ), // override the default fill (from shapeStyle()) to be red
                        $(go.Picture,  // the icon showing the logo
                            // You should set the desiredSize (or width and height)
                            // whenever you know what size the Picture should be.
                            {desiredSize: new go.Size(400, 40)},
                            new go.Binding("source", "category", self.convertKeyImage)),
                    );

                let selectMaxTemplate =
                    $(go.Node, "Auto",
                        $(go.Shape, "Rectangle",
                            {strokeWidth: 0},
                        ),
                        $(go.Picture,  // the icon showing the logo
                            // You should set the desiredSize (or width and height)
                            // whenever you know what size the Picture should be.
                            {desiredSize: new go.Size(400, 40)},
                            new go.Binding("source", "category", self.convertKeyImage)),
                    );

                self.myDiagram.nodeTemplateMap.add("input", inputTemplate);
                self.myDiagram.nodeTemplateMap.add("select", selectTemplate);
                self.myDiagram.nodeTemplateMap.add("inputMax", inputMaxTemplate);
                self.myDiagram.nodeTemplateMap.add("selectMax", selectMaxTemplate);

                self.palette = new go.Palette("palette");

                self.palette.nodeTemplateMap = self.myDiagram.nodeTemplateMap;

                self.palette.model.nodeDataArray = [
                    {category: "input"},
                    {category: "inputMax"},
                    {category: "select"},
                    {category: "selectMax"}
                ];

            },
            convertKeyImage(key) {
                return `/static/${key}.png`;
            }
        }
    }
</script>

<style scoped>
    #myDiagramDiv {
        height: 200px;
        border: solid 1px #d3d3d3;
    }
</style>
