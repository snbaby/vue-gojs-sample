<template>
    <div id="sample">
        <div style="width: 100%; display: flex; justify-content: space-between">
            <div style="width: 135px; margin-right: 2px; background-color: whitesmoke; border: solid 1px black">
                <el-collapse v-model="activeName" @change="handleChange" accordion>
                    <el-collapse-item title="Small items" name="1">
                        <div id="myPaletteSmall" style="width: 140px; height: 360px"></div>
                    </el-collapse-item>
                    <el-collapse-item title="Tall items" name="2">
                        <div id="myPaletteTall" style="width: 140px; height: 360px"></div>
                    </el-collapse-item>
                    <el-collapse-item title="Wide items" name="3">
                        <div id="myPaletteWide" style="width: 140px; height: 360px"></div>
                    </el-collapse-item>
                    <el-collapse-item title="Big items" name="4">
                        <div id="myPaletteBig" style="width: 140px; height: 360px"></div>
                    </el-collapse-item>
                </el-collapse>
            </div>
            <div id="myDiagramDiv" style="flex-grow: 1; height: 500px; border: solid 1px black">

            </div>
        </div>
        <p>
            A <em>planogram</em> is a visual representation of a store's products or services, often used as a tool to
            maximize sales.
        </p>
        <p>
            Drag-and-drop "items" from the Palette onto "racks" in the Diagram.
            "Items" are implemented as Nodes; "racks" are implemented as Groups.
            Once items have been placed on a rack, they can be resized, if necessary.
            The <a>DraggingTool.isGridSnapEnabled</a> and <a>ResizingTool.isGridSnapEnabled</a> are both set
            to true to allow for snapping to the background grid.
            Node and Group templates both use dragging functions to allow for highlighting so the user knows which
            rack an item will belong to.
        </p>
        <p>
            A jQuery Accordion is used to create the four collapsible Palettes.
        </p>
        <div>
            Diagram Model saved in JSON format, automatically updated after each transaction:
            <pre id="savedModel" style="height:250px"></pre>
        </div>
    </div>
</template>

<script>
    import go from 'gojs';

    const jQuery = require("jquery")

    let $ = go.GraphObject.make;

    export default {
        data() {
            return {
                myDiagram: null,
                AllowTopLevel: true,
                CellSize: new go.Size(50, 50),
                groupFill: "rgba(128,128,128,0.2)",
                groupStroke: "gray",
                dropFill: "rgba(128,255,255,0.2)",
                dropStroke: "red",
                green: '#B2FF59',
                blue: '#81D4FA',
                yellow: '#FFEB3B',
                activeName: '1',

                myPaletteSmall: null,
                myPaletteTall: null,
                myPaletteWide: null,
                myPaletteBig: null
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
                            grid: $(go.Panel, "Grid",
                                {gridCellSize: self.CellSize},
                                $(go.Shape, "LineH", {stroke: "lightgray"}),
                                $(go.Shape, "LineV", {stroke: "lightgray"})
                            ),
                            // support grid snapping when dragging and when resizing
                            "draggingTool.isGridSnapEnabled": true,
                            "draggingTool.gridSnapCellSpot": go.Spot.Center,
                            "resizingTool.isGridSnapEnabled": true,
                            // For this sample, automatically show the state of the diagram's model on the page
                            "ModelChanged": function (e) {
                                if (e.isTransactionFinished) {
                                    document.getElementById("savedModel").textContent = self.myDiagram.model.toJson();
                                }
                            },
                            "animationManager.isEnabled": false,
                            "undoManager.isEnabled": true
                        });

                // Regular Nodes represent items to be put onto racks.
                // Nodes are currently resizable, but if that is not desired, just set resizable to false.
                self.myDiagram.nodeTemplate =
                    $(go.Node, "Auto",
                        {
                            resizable: true,
                            resizeObjectName: "SHAPE",
                            // because the gridSnapCellSpot is Center, offset the Node's location
                            locationSpot: new go.Spot(0, 0, self.CellSize.width / 2, self.CellSize.height / 2),
                            // provide a visual warning about dropping anything onto an "item"
                            mouseDragEnter: function (e, node) {
                                e.handled = true;
                                node.findObject("SHAPE").fill = "red";
                                self.highlightGroup(node.containingGroup, false);
                            },
                            mouseDragLeave: function (e, node) {
                                node.updateTargetBindings();
                            },
                            mouseDrop: function (e, node) {  // disallow dropping anything onto an "item"
                                node.diagram.currentTool.doCancel();
                            }
                        },
                        // always save/load the point that is the top-left corner of the node, not the location
                        new go.Binding("position", "pos", go.Point.parse).makeTwoWay(go.Point.stringify),
                        // this is the primary thing people see
                        $(go.Shape, "Rectangle",
                            {
                                name: "SHAPE",
                                fill: "white",
                                minSize: self.CellSize,
                                desiredSize: self.CellSize  // initially 1x1 cell
                            },
                            new go.Binding("fill", "color"),
                            new go.Binding("desiredSize", "size", go.Size.parse).makeTwoWay(go.Size.stringify)),
                        // with the textual key in the middle
                        $(go.TextBlock,
                            {
                                alignment: go.Spot.Center,
                                font: 'bold 16px sans-serif'
                            },
                            new go.Binding("text", "key")
                        )
                    );  // end Node

                self.myDiagram.groupTemplate =
                    $(go.Group,
                        {
                            layerName: "Background",
                            resizable: true,
                            resizeObjectName: "SHAPE",
                            // because the gridSnapCellSpot is Center, offset the Group's location
                            locationSpot: new go.Spot(0, 0, self.CellSize.width / 2, self.CellSize.height / 2)
                        },
                        // always save/load the point that is the top-left corner of the node, not the location
                        new go.Binding("position", "pos", go.Point.parse).makeTwoWay(go.Point.stringify),
                        { // what to do when a drag-over or a drag-drop occurs on a Group
                            mouseDragEnter: function (e, grp, prev) {
                                self.highlightGroup(grp, true);
                            },
                            mouseDragLeave: function (e, grp, next) {
                                self.highlightGroup(grp, false);
                            },
                            mouseDrop: function (e, grp) {
                                let ok = grp.addMembers(grp.diagram.selection, true);
                                if (!ok) {
                                    grp.diagram.currentTool.doCancel();
                                }
                            }
                        },
                        $(go.Shape, "Rectangle",  // the rectangular shape around the members
                            {
                                name: "SHAPE",
                                fill: self.groupFill,
                                stroke: self.groupStroke,
                                minSize: new go.Size(self.CellSize.width * 2, self.CellSize.height * 2)
                            },
                            new go.Binding("desiredSize", "size", go.Size.parse).makeTwoWay(go.Size.stringify),
                            new go.Binding("fill", "isHighlighted", function (h) {
                                return h ? dropFill : groupFill;
                            }).ofObject(),
                            new go.Binding("stroke", "isHighlighted", function (h) {
                                return h ? self.dropStroke : self.groupStroke;
                            }).ofObject())
                    );

                // decide what kinds of Parts can be added to a Group
                self.myDiagram.commandHandler.memberValidation = function (grp, node) {
                    if (grp instanceof go.Group && node instanceof go.Group) {
                        return false;
                    }  // cannot add Groups to Groups
                    // but dropping a Group onto the background is always OK
                    return true;
                };

                // what to do when a drag-drop occurs in the Diagram's background
                self.myDiagram.mouseDragOver = function (e) {
                    if (!self.AllowTopLevel) {
                        // but OK to drop a group anywhere
                        if (!e.diagram.selection.all(function (p) {
                            return p instanceof go.Group;
                        })) {
                            e.diagram.currentCursor = "not-allowed";
                        }
                    }
                };

                self.myDiagram.mouseDrop = function (e) {
                    if (self.AllowTopLevel) {
                        // when the selection is dropped in the diagram's background,
                        // make sure the selected Parts no longer belong to any Group
                        if (!e.diagram.commandHandler.addTopLevelParts(e.diagram.selection, true)) {
                            e.diagram.currentTool.doCancel();
                        }
                    } else {
                        // disallow dropping any regular nodes onto the background, but allow dropping "racks"
                        if (!e.diagram.selection.all(function (p) {
                            return p instanceof go.Group;
                        })) {
                            e.diagram.currentTool.doCancel();
                        }
                    }
                };

                // start off with four "racks" that are positioned next to each other
                self.myDiagram.model = new go.GraphLinksModel([
                    {
                        key: "G1",
                        isGroup: true,
                        pos: "0 0",
                        size: "200 200"
                    },
                    {
                        key: "G2",
                        isGroup: true,
                        pos: "200 0",
                        size: "200 200"
                    },
                    {
                        key: "G3",
                        isGroup: true,
                        pos: "0 200",
                        size: "200 200"
                    },
                    {
                        key: "G4",
                        isGroup: true,
                        pos: "200 200",
                        size: "200 200"
                    }
                ]);

                // initialize the first Palette
                self.myPaletteSmall =
                    $(go.Palette, "myPaletteSmall",
                        { // share the templates with the main Diagram
                            nodeTemplate: self.myDiagram.nodeTemplate,
                            groupTemplate: self.myDiagram.groupTemplate
                        }
                    );

                // specify the contents of the Palette
                self.myPaletteSmall.model = new go.GraphLinksModel([
                    {
                        key: "g",
                        color: self.green
                    },
                    {
                        key: "b",
                        color: self.blue
                    },
                    {
                        key: "y",
                        color: self.yellow
                    }
                ]);

                // initialize the second Palette, of tall items
                self.myPaletteTall =
                    $(go.Palette, "myPaletteTall",
                        { // share the templates with the main Diagram
                            nodeTemplate: self.myDiagram.nodeTemplate,
                            groupTemplate: self.myDiagram.groupTemplate
                        }
                    );

                // specify the contents of the Palette
                self.myPaletteTall.model = new go.GraphLinksModel([
                    {
                        key: "g",
                        color: self.green,
                        size: "50 100"
                    },
                    {
                        key: "b",
                        color: self.blue,
                        size: "50 100"
                    },
                    {
                        key: "y",
                        color: self.yellow,
                        size: "50 100"
                    }
                ]);

                // initialize the third Palette, of wide items
                self.myPaletteWide =
                    $(go.Palette, "myPaletteWide",
                        { // share the templates with the main Diagram
                            nodeTemplate: self.myDiagram.nodeTemplate,
                            groupTemplate: self.myDiagram.groupTemplate
                        });

                // specify the contents of the Palette
                self.myPaletteWide.model = new go.GraphLinksModel([
                    {
                        key: "g",
                        color: self.green,
                        size: "100 50"
                    },
                    {
                        key: "b",
                        color: self.blue,
                        size: "100 50"
                    },
                    {
                        key: "y",
                        color: self.yellow,
                        size: "100 50"
                    }
                ]);

                // initialize the fourth Palette, of big items
                self.myPaletteBig =
                    $(go.Palette, "myPaletteBig",
                        { // share the templates with the main Diagram
                            nodeTemplate: self.myDiagram.nodeTemplate,
                            groupTemplate: self.myDiagram.groupTemplate
                        });

                // specify the contents of the Palette
                self.myPaletteBig.model = new go.GraphLinksModel([
                    {
                        key: "g",
                        color: self.green,
                        size: "100 100"
                    },
                    {
                        key: "b",
                        color: self.blue,
                        size: "100 100"
                    },
                    {
                        key: "y",
                        color: self.yellow,
                        size: "100 100"
                    }
                ]);
            },
            handleChange(val) {
                const self = this;
                self.myPaletteSmall.requestUpdate();
                self.myPaletteTall.requestUpdate();
                self.myPaletteWide.requestUpdate();
                self.myPaletteBig.requestUpdate();
            },
            highlightGroup(grp, show) {
                if (!grp) {
                    return;
                }
                if (show) {  // check that the drop may really happen into the Group
                    let tool = grp.diagram.toolManager.draggingTool;
                    let map = tool.draggedParts || tool.copiedParts;  // this is a Map
                    if (grp.canAddMembers(map.toKeySet())) {
                        grp.isHighlighted = true;
                        return;
                    }
                }
                grp.isHighlighted = false;
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
