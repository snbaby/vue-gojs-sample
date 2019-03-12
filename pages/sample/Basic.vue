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
                    $(go.Diagram, "myDiagramDiv",
                        {
                            // allow double-click in background to create a new node
                            "clickCreatingTool.archetypeNodeData": {text: "Node", color: "white"},

                            // allow Ctrl-G to call groupSelection()
                            "commandHandler.archetypeGroupData": {text: "Group", isGroup: true, color: "blue"},

                            // enable undo & redo
                            "undoManager.isEnabled": true
                        });

                let partContextMenu =
                    $("ContextMenu",
                        self.makeButton("Properties",
                            function (e, obj) {  // OBJ is this Button
                                var contextmenu = obj.part;  // the Button is in the context menu Adornment
                                var part = contextmenu.adornedPart;  // the adornedPart is the Part that the context menu adorns
                                // now can do something with PART, or with its data, or with the Adornment (the context menu)
                                if (part instanceof go.Link) {
                                    alert(self.linkInfo(part.data));
                                } else if (part instanceof go.Group) {
                                    alert(self.groupInfo(contextmenu));
                                } else {
                                    alert(self.nodeInfo(part.data));
                                }
                            }),
                        self.makeButton("Cut",
                            function (e, obj) {
                                e.diagram.commandHandler.cutSelection();
                            },
                            function (o) {
                                return o.diagram.commandHandler.canCutSelection();
                            }),
                        self.makeButton("Copy",
                            function (e, obj) {
                                e.diagram.commandHandler.copySelection();
                            },
                            function (o) {
                                return o.diagram.commandHandler.canCopySelection();
                            }),
                        self.makeButton("Paste",
                            function (e, obj) {
                                e.diagram.commandHandler.pasteSelection(e.diagram.lastInput.documentPoint);
                            },
                            function (o) {
                                return o.diagram.commandHandler.canPasteSelection();
                            }),
                        self.makeButton("Delete",
                            function (e, obj) {
                                e.diagram.commandHandler.deleteSelection();
                            },
                            function (o) {
                                return o.diagram.commandHandler.canDeleteSelection();
                            }),
                        self.makeButton("Undo",
                            function (e, obj) {
                                e.diagram.commandHandler.undo();
                            },
                            function (o) {
                                return o.diagram.commandHandler.canUndo();
                            }),
                        self.makeButton("Redo",
                            function (e, obj) {
                                e.diagram.commandHandler.redo();
                            },
                            function (o) {
                                return o.diagram.commandHandler.canRedo();
                            }),
                        self.makeButton("Group",
                            function (e, obj) {
                                e.diagram.commandHandler.groupSelection();
                            },
                            function (o) {
                                return o.diagram.commandHandler.canGroupSelection();
                            }),
                        self.makeButton("Ungroup",
                            function (e, obj) {
                                e.diagram.commandHandler.ungroupSelection();
                            },
                            function (o) {
                                return o.diagram.commandHandler.canUngroupSelection();
                            })
                    );


                //默认节点模板
                this.myDiagram.nodeTemplate =
                    $(go.Node, "Auto",
                        {locationSpot: go.Spot.Center},
                        $(go.Shape, "RoundedRectangle",
                            {
                                fill: "white", // the default fill, if there is no data bound value
                                portId: "", cursor: "pointer",  // the Shape is the port, not the whole Node
                                // allow all kinds of links from and to this port
                                fromLinkable: true, fromLinkableSelfNode: true, fromLinkableDuplicates: true,
                                toLinkable: true, toLinkableSelfNode: true, toLinkableDuplicates: true
                            },
                            new go.Binding("fill", "color")),
                        $(go.TextBlock,
                            {
                                font: "bold 14px sans-serif",
                                stroke: '#333',
                                margin: 6,  // make some extra space for the shape around the text
                                isMultiline: false,  // don't allow newlines in text
                                editable: true  // allow in-place editing by user
                            },
                            new go.Binding("text", "text").makeTwoWay()),  // the label shows the node data's text
                        { // this tooltip Adornment is shared by all nodes
                            toolTip:
                                $("ToolTip",
                                    $(go.TextBlock, {margin: 4},  // the tooltip shows the result of calling nodeInfo(data)
                                        new go.Binding("text", "",self.nodeInfo))
                                ),
                            // this context menu Adornment is shared by all nodes
                            contextMenu: partContextMenu
                        }
                    );

                this.myDiagram.linkTemplate =
                    $(go.Link,
                        { toShortLength: 3, relinkableFrom: true, relinkableTo: true },  // allow the user to relink existing links
                        $(go.Shape,
                            { strokeWidth: 2 },
                            new go.Binding("stroke", "color")),
                        $(go.Shape,
                            { toArrow: "Standard", stroke: null },
                            new go.Binding("fill", "color")),
                        { // this tooltip Adornment is shared by all links
                            toolTip:
                                $("ToolTip",
                                    $(go.TextBlock, { margin: 4 },  // the tooltip shows the result of calling linkInfo(data)
                                        new go.Binding("text", "", self.linkInfo))
                                ),
                            // the same context menu Adornment is shared by all links
                            contextMenu: partContextMenu
                        }
                    );

                this.myDiagram.groupTemplate =
                    $(go.Group, "Vertical",
                        {
                            selectionObjectName: "PANEL",  // selection handle goes around shape, not label
                            ungroupable: true  // enable Ctrl-Shift-G to ungroup a selected Group
                        },
                        $(go.TextBlock,
                            {
                                font: "bold 19px sans-serif",
                                isMultiline: false,  // don't allow newlines in text
                                editable: true  // allow in-place editing by user
                            },
                            new go.Binding("text", "text").makeTwoWay(),
                            new go.Binding("stroke", "color")),
                        $(go.Panel, "Auto",
                            { name: "PANEL" },
                            $(go.Shape, "Rectangle",  // the rectangular shape around the members
                                {
                                    fill: "rgba(128,128,128,0.2)", stroke: "gray", strokeWidth: 3,
                                    portId: "", cursor: "pointer",  // the Shape is the port, not the whole Node
                                    // allow all kinds of links from and to this port
                                    fromLinkable: true, fromLinkableSelfNode: true, fromLinkableDuplicates: true,
                                    toLinkable: true, toLinkableSelfNode: true, toLinkableDuplicates: true
                                }),
                            $(go.Placeholder, { margin: 10, background: "transparent" })  // represents where the members are
                        ),
                        { // this tooltip Adornment is shared by all groups
                            toolTip:
                                $("ToolTip",
                                    $(go.TextBlock, { margin: 4 },
                                        // bind to tooltip, not to Group.data, to allow access to Group properties
                                        new go.Binding("text", "", self.groupInfo).ofObject())
                                ),
                            // the same context menu Adornment is shared by all groups
                            contextMenu: partContextMenu
                        }
                    );

                this.myDiagram.toolTip =
                    $("ToolTip",
                        $(go.TextBlock, { margin: 4 },
                            new go.Binding("text", "", self.diagramInfo))
                    );

                this.myDiagram.contextMenu =
                    $("ContextMenu",
                        self.makeButton("Paste",
                            function(e, obj) { e.diagram.commandHandler.pasteSelection(e.diagram.lastInput.documentPoint); },
                            function(o) { return o.diagram.commandHandler.canPasteSelection(); }),
                        self.makeButton("Undo",
                            function(e, obj) { e.diagram.commandHandler.undo(); },
                            function(o) { return o.diagram.commandHandler.canUndo(); }),
                        self.makeButton("Redo",
                            function(e, obj) { e.diagram.commandHandler.redo(); },
                            function(o) { return o.diagram.commandHandler.canRedo(); })
                    );

                let nodeDataArray = [
                    { key: 1, text: "Alpha", color: "lightblue" },
                    { key: 2, text: "Beta", color: "orange" },
                    { key: 3, text: "Gamma", color: "lightgreen", group: 5 },
                    { key: 4, text: "Delta", color: "pink", group: 5 },
                    { key: 5, text: "Epsilon", color: "green", isGroup: true }
                ];
                let linkDataArray = [
                    { from: 1, to: 2, color: "blue" },
                    { from: 2, to: 2 },
                    { from: 3, to: 4, color: "green" },
                    { from: 3, to: 1, color: "purple" }
                ];
                this.myDiagram.model = new go.GraphLinksModel(nodeDataArray, linkDataArray);

            },
            makeButton(text, action, visiblePredicate) {
                return $("ContextMenuButton",
                    $(go.TextBlock, text),
                    {click: action},
                    // don't bother with binding GraphObject.visible if there's no predicate
                    visiblePredicate ? new go.Binding("visible", "", function (o, e) {
                        return o.diagram ? visiblePredicate(o, e) : false;
                    }).ofObject() : {});
            },
            nodeInfo(d) {
                var str = "Node " + d.key + ": " + d.text + "\n";
                if (d.group)
                    str += "member of " + d.group;
                else
                    str += "top-level node";
                return str;
            },
            linkInfo(d) {
                return "Link:\nfrom " + d.from + " to " + d.to;
            },
            groupInfo(adornment) {  // takes the tooltip or context menu, not a group node data object
                var g = adornment.adornedPart;  // get the Group that the tooltip adorns
                var mems = g.memberParts.count;
                var links = 0;
                g.memberParts.each(function (part) {
                    if (part instanceof go.Link) links++;
                });
                return "Group " + g.data.key + ": " + g.data.text + "\n" + mems + " members including " + links + " links";
            },
            diagramInfo(model) {  // Tooltip info for the diagram's model
                return "Model:\n" + model.nodeDataArray.length + " nodes, " + model.linkDataArray.length + " links";
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
