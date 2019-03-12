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
                myDiagram: null
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
                    $(go.Diagram, "myDiagramDiv",  // create a Diagram for the DIV HTML element
                        {
                            isReadOnly: true,  // don't allow move or delete
                            layout: $(go.CircularLayout,
                                {
                                    radius: 100,  // minimum radius
                                    spacing: 0,   // circular nodes will touch each other
                                    nodeDiameterFormula: go.CircularLayout.Circular,  // assume nodes are circular
                                    startAngle: 270  // first node will be at top
                                }),
                            // define a DiagramEvent listener
                            "LayoutCompleted": function (e) {
                                // now that the CircularLayout has finished, we know where its center is
                                var cntr = self.myDiagram.findNodeForKey("Center");
                                cntr.location = self.myDiagram.layout.actualCenter;
                            }
                        });

                // construct a shared radial gradient brush
                let radBrush = $(go.Brush, "Radial", {0: "#550266", 1: "#80418C"});

                // these are the nodes that are in a circle
                this.myDiagram.nodeTemplate =
                    $(go.Node,
                        $(go.Shape, "Circle",
                            {
                                desiredSize: new go.Size(28, 28),
                                fill: radBrush, strokeWidth: 0, stroke: null
                            }), // no outline
                        {
                            locationSpot: go.Spot.Center,
                            click: self.showArrowInfo,  // defined below
                            toolTip:  // define a tooltip for each link that displays its information
                                $("ToolTip",
                                    $(go.TextBlock, {margin: 4},
                                        new go.Binding("text", "", self.infoString).ofObject())
                                )
                        }
                    );

                this.myDiagram.nodeTemplateMap.add("Center",
                    $(go.Node, "Spot",
                        {
                            selectable: false,
                            isLayoutPositioned: false,  // the Diagram.layout will not position this node
                            locationSpot: go.Spot.Center
                        },
                        $(go.Shape, "Circle",
                            {fill: radBrush, strokeWidth: 0, stroke: null, desiredSize: new go.Size(200, 200)}), // no outline
                        $(go.TextBlock, "Arrowheads",
                            {margin: 1, stroke: "white", font: "bold 14px sans-serif"})
                    ));

                this.myDiagram.linkTemplate =
                    $(go.Link,  // the whole link panel
                        {routing: go.Link.Normal},
                        $(go.Shape,  // the link shape
                            // the first element is assumed to be main element: as if isPanelMain were true
                            {stroke: "gray", strokeWidth: 2}),
                        $(go.Shape,  // the "from" arrowhead
                            new go.Binding("fromArrow", "fromArrow"),
                            {scale: 2, fill: "#D4B52C"}),
                        $(go.Shape,  // the "to" arrowhead
                            new go.Binding("toArrow", "toArrow"),
                            {scale: 2, fill: "#D4B52C"}),
                        {
                            click: self.showArrowInfo,
                            toolTip:  // define a tooltip for each link that displays its information
                                $("ToolTip",
                                    $(go.TextBlock, {margin: 4},
                                        new go.Binding("text", "", self.infoString).ofObject())
                                )
                        }
                    );

                var arrowheads = go.Shape.getArrowheadGeometries().toKeySet().toArray();
                if (arrowheads.length % 2 === 1) {
                    arrowheads.push("");
                }  // make sure there's an even number

                var linkdata = [];
                var i = 0;
                for (var j = 0; j < arrowheads.length; j = j + 2) {
                    linkdata.push({from: "Center", to: i++, toArrow: arrowheads[j], fromArrow: arrowheads[j + 1]});
                }

                this.myDiagram.model =
                    $(go.GraphLinksModel,
                        { // this gets copied automatically when there's a link data reference to a new node key
                            // and is then added to the nodeDataArray
                            archetypeNodeData: {},
                            // the node array starts with just the special Center node
                            nodeDataArray: [{category: "Center", key: "Center"}],
                            // the link array was created above
                            linkDataArray: linkdata
                        });
            },
            showArrowInfo(e, obj) {
                var msg = this.infoString(obj);
                if (msg) {
                    var status = document.getElementById("myArrowheadInfo");
                    if (status) {
                        status.textContent = msg;
                    }
                }
            },
            infoString(obj) {
                var part = obj.part;
                if (part instanceof go.Adornment) {
                    part = part.adornedPart;
                }
                var msg = "";
                if (part instanceof go.Link) {
                    var link = part;
                    msg = "toArrow: " + link.data.toArrow + ";\nfromArrow: " + link.data.fromArrow;
                } else if (part instanceof go.Node) {
                    var node = part;
                    var link = node.linksConnected.first();
                    msg = "toArrow: " + link.data.toArrow + ";\nfromArrow: " + link.data.fromArrow;
                }
                return msg;
            }
        }
    }
</script>

<style scoped>
    #myDiagramDiv {
        height: 800px;
        border: solid 1px #d3d3d3;
    }
</style>
