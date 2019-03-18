<template>
    <div>
        <div id="sample">
            <div style="width: 100%; display: flex; justify-content: space-between">
                <div style="width: 900px; margin-right: 2px; background-color: whitesmoke; border: solid 1px black">
                    <el-collapse v-model="activeName" accordion>
                        <el-collapse-item title="demo" name="1">
                            <div id="myPaletteSmall" style="width: 900px; height: 360px"></div>
                        </el-collapse-item>
                        <el-collapse-item title="input" name="2">
                            <div>控制反馈：通过界面样式和交互动效让用户可以清晰的感知自己的操作；</div>
                            <div>页面反馈：操作后，通过页面元素的变化清晰地展现当前状态。</div>
                        </el-collapse-item>
                    </el-collapse>
                </div>
                <div id="myDiagramDiv" style="flex-grow: 1; height: 500px; border: solid 1px black">

                </div>
            </div>
            <div>
                <pre id="savedModel" style="height:250px"></pre>
            </div>
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
                nodeDataArray: [],
                myPaletteSmall: null,
                activeName: '1'
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
                    $(go.Diagram, "myDiagramDiv",  // must be the ID or reference to div
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
                                    wrappingWidth: 900,
                                    wrappingColumn: 2,
                                    cellSize:new go.Size(400, 40),
                                    spacing: new go.Size(0, 20),
                                    alignment: go.GridLayout.Position
                                }),
                            "SelectionMoved": self.relayoutNodes
                            // other properties are set by the layout function, defined below
                        });
                console.log("go.Shape.getFigureGenerators().toArray()",go.Shape.getFigureGenerators().toArray())
                self.myDiagram.nodeTemplate =
                    $(go.Node, "Auto",
                        //边框
                        $(go.Shape, "Rectangle",
                            {strokeWidth: 2, stroke: 'gray'},
                            new go.Binding("fill", "color")
                        ),
                        //文本
                        $(go.TextBlock,
                            {margin: 5},
                            new go.Binding("text", "key")
                        ),
                        $(go.Picture,  // the icon showing the logo
                            // You should set the desiredSize (or width and height)
                            // whenever you know what size the Picture should be.
                            { desiredSize: new go.Size(400, 50) },
                            new go.Binding("source", "key", self.convertKeyImage)),
                        { // second arg will be this GraphObject, which in this case is the Node itself:
                            click: function(e, node) {
                                alert(node.data.key);
                                console.log("node",node);
                                node.data.key = "ww";
                            }
                        }
                    );

                // initialize the first Palette
                self.myPaletteSmall =
                    $(go.Palette, "myPaletteSmall",
                        { // share the templates with the main Diagram
                            nodeTemplate: self.myDiagram.nodeTemplate
                        }
                    );

                // specify the contents of the Palette
                self.myPaletteSmall.model = new go.GraphLinksModel([
                    {
                        key: "输入框-S",
                        color: go.Brush.randomColor(),
                        width: 1
                    },
                    {
                        key: "输入框-B",
                        color: go.Brush.randomColor(),
                        width: 2
                    },
                ]);

                self.myDiagram.model = new go.Model([
                    {key: "Alpha", color: "lightblue", width: 1},
                    {key: "Beta", color: "thistle", width: 1},
                    {key: "Delta", color: "lightcoral", width: 2},
                    {key: "Gamma", color: "thistle", width: 1},
                    {key: "Field1", color: "lightgreen", width: 1},
                    {key: "Field2", color: "thistle", width: 2},
                    {key: "Field3", color: "lightgreen", width: 2}
                ]);
                /*self.myDiagram.model = new go.Model(self.nodeDataArray);*/

            },
            relayoutNodes() {
                const self = this;
                self.myDiagram.layout.invalidateLayout();
                self.myDiagram.layoutDiagram();
            },
            convertKeyImage(key) {
                if (!key) {
                    key = "NE";
                }
                return "/static/img/icons/apple-touch-icon-180x180.png";
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
