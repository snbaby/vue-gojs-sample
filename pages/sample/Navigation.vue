<template>
    <div id="sample">
        <div id="displays" style="width:100%; white-space:nowrap;">
            <div id="myDiagramDiv"
                 style="border: solid 1px black; height: 560px; display: inline-block; vertical-align: top; width: 70%"></div>
            <div id="controls"
                 style="border: solid 1px black; height: 560px; width: 200px; display: inline-block; vertical-align: top;">
                <div id="buttons"
                     style="border-radius: 10px; border: solid 1px gray; background-color: #eaeaea; margin: 5px;">
                    <b style="margin: 5px">Related Parts Highlighted</b> <br/>
                    <label><input type="radio" name="highlight" @click="updateHighlights($event.currentTarget)" id="None"
                                  checked="checked">Unhighlight All</label><br/>
                    <label><input type="radio" name="highlight" @click="updateHighlights($event.currentTarget)" id="linksIn">Links Into</label><br/>
                    <label><input type="radio" name="highlight" @click="updateHighlights($event.currentTarget)" id="linksOut">Links Out
                        Of</label><br/>
                    <label><input type="radio" name="highlight" @click="updateHighlights($event.currentTarget)" id="linksAll">Links
                        Connected</label><br/>
                    <label><input type="radio" name="highlight" @click="updateHighlights($event.currentTarget)" id="nodesIn">Nodes Into</label><br/>
                    <label><input type="radio" name="highlight" @click="updateHighlights($event.currentTarget)" id="nodesOut">Nodes Out
                        Of</label><br/>
                    <label><input type="radio" name="highlight" @click="updateHighlights($event.currentTarget)" id="nodesConnect">Nodes
                        Connected</label><br/>
                    <label><input type="radio" name="highlight" @click="updateHighlights($event.currentTarget)" id="nodesReach">Nodes
                        Reachable</label><br/>
                    <label><input type="radio" name="highlight" @click="updateHighlights($event.currentTarget)" id="group">Containing
                        Group (Parent)</label><br/>
                    <label><input type="radio" name="highlight" @click="updateHighlights($event.currentTarget)" id="groupsAll">Containing
                        Groups (All)</label><br/>
                    <label><input type="radio" name="highlight" @click="updateHighlights($event.currentTarget)" id="nodesMember">Member
                        Nodes (Children)</label><br/>
                    <label><input type="radio" name="highlight" @click="updateHighlights($event.currentTarget)" id="nodesMembersAll">Member
                        Nodes (All)</label><br/>
                    <label><input type="radio" name="highlight" @click="updateHighlights($event.currentTarget)" id="linksMember">Member
                        Links (Children)</label><br/>
                    <label><input type="radio" name="highlight" @click="updateHighlights($event.currentTarget)" id="linksMembersAll">Member
                        Links (All)</label><br/>
                </div>
                <div id="colorKey"
                     style="border-radius: 10px; border: solid 1px gray; background-color: #eaeaea; margin: 5px;">
                    <b style="margin: 5px">Relationship Colors</b>
                    <table>
                        <tr>
                            <td>
                                <div
                                    style="float: left; margin: 5px; height: 20px; width: 20px; background-color: black;"></div>
                            </td>
                            <td>Not related</td>
                        </tr>
                        <tr>
                            <td>
                                <div
                                    style="float: left; margin: 5px; height: 20px; width: 20px; background-color: blue;"></div>
                            </td>
                            <td>Directly related</td>
                        </tr>
                        <tr>
                            <td>
                                <div
                                    style="float: left; margin: 5px; height: 20px; width: 20px; background-color: green;"></div>
                            </td>
                            <td>2 relationships apart</td>
                        </tr>
                        <tr>
                            <td>
                                <div
                                    style="float: left; margin: 5px; height: 20px; width: 20px; background-color: orange;"></div>
                            </td>
                            <td>3 relationships apart</td>
                        </tr>
                        <tr>
                            <td>
                                <div
                                    style="float: left; margin: 5px; height: 20px; width: 20px; background-color: red;"></div>
                            </td>
                            <td>4 relationships apart</td>
                        </tr>
                        <tr>
                            <td>
                                <div
                                    style="float: left; margin: 5px; height: 20px; width: 20px; background-color: purple;"></div>
                            </td>
                            <td>Very indirectly related</td>
                        </tr>
                    </table>
                </div>
            </div>
        </div>
        <div>
            <p>This sample displays relationships between different parts of a diagram.</p>
            <p>
                Select a node or link and one of the radio buttons to highlight parts with a certain relationship to the
                one selected.
                The highlighting color depends on the "distance" between the parts.
            </p>
        </div>
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
                    $(go.Diagram, "myDiagramDiv",  // Diagram refers to its DIV HTML element by id
                        {
                            maxSelectionCount: 1 // no more than 1 element can be selected at a time
                        });

                // define the node template
                this.myDiagram.nodeTemplate =
                    $(go.Node, "Auto",
                        new go.Binding("location", "loc"),
                        {
                            locationSpot: go.Spot.Center,
                            toEndSegmentLength: 30,
                            fromEndSegmentLength: 30
                        },
                        $(go.Shape, "Rectangle",
                            {
                                name: "OBJSHAPE",
                                fill: "white",
                                desiredSize: new go.Size(30, 30)
                            }),
                        $(go.TextBlock,
                            {margin: 4},
                            new go.Binding("text", "key")),
                        {
                            toolTip:  //  define a tooltip for each node that displays its information
                                $("ToolTip",
                                    $(go.TextBlock, {margin: 4},
                                        new go.Binding("text", "", self.getInfo))
                                )
                        }
                    );

                this.myDiagram.linkTemplate =
                    $(go.Link,
                        {
                            selectionAdornmentTemplate:
                                $(go.Adornment,
                                    $(go.Shape,
                                        {isPanelMain: true, stroke: "dodgerblue", strokeWidth: 3}),
                                    $(go.Shape,
                                        {toArrow: "Standard", fill: "dodgerblue", stroke: null, scale: 1})
                                ),
                            routing: go.Link.Normal,
                            curve: go.Link.Bezier,
                            toShortLength: 2
                        },
                        $(go.Shape,  //  the link shape
                            {name: "OBJSHAPE"}),
                        $(go.Shape,  //  the arrowhead
                            {name: "ARWSHAPE", toArrow: "Standard"}),
                        {
                            toolTip:  //  define a tooltip for each link that displays its information
                                $("ToolTip",
                                    $(go.TextBlock, {margin: 4},
                                        new go.Binding("text", "", self.getInfo))
                                )
                        }
                    );

                // define the group template
                this.myDiagram.groupTemplate =
                    $(go.Group, "Spot",
                        {
                            selectionAdornmentTemplate: // adornment when a group is selected
                                $(go.Adornment, "Auto",
                                    $(go.Shape, "Rectangle",
                                        {fill: null, stroke: "dodgerblue", strokeWidth: 3}),
                                    $(go.Placeholder)
                                ),
                            toSpot: go.Spot.AllSides, // links coming into groups at any side
                            toEndSegmentLength: 30, fromEndSegmentLength: 30
                        },
                        $(go.Panel, "Auto",
                            $(go.Shape, "Rectangle",
                                {
                                    name: "OBJSHAPE",
                                    parameter1: 14,
                                    fill: "rgba(255,0,0,0.10)"
                                },
                                new go.Binding("desiredSize", "ds")),
                            $(go.Placeholder,
                                {padding: 16})
                        ),
                        $(go.TextBlock,
                            {
                                name: "GROUPTEXT",
                                alignment: go.Spot.TopLeft,
                                alignmentFocus: new go.Spot(0, 0, -4, -4),
                                font: "Bold 10pt Sans-Serif"
                            },
                            new go.Binding("text", "key")),
                        {
                            toolTip:  //  define a tooltip for each group that displays its information
                                $("ToolTip",
                                    $(go.TextBlock, {margin: 4},
                                        new go.Binding("text", "", self.getInfo))
                                )
                        }
                    );

                // add nodes, including groups, and links to the model
                this.myDiagram.model = new go.GraphLinksModel(
                    [ // node data
                        {key: "A", loc: new go.Point(320, 100)},
                        {key: "B", loc: new go.Point(420, 200)},
                        {key: "C", group: "Psi", loc: new go.Point(250, 225)},
                        {key: "D", group: "Omega", loc: new go.Point(270, 325)},
                        {key: "E", group: "Phi", loc: new go.Point(120, 225)},
                        {key: "F", group: "Omega", loc: new go.Point(200, 350)},
                        {key: "G", loc: new go.Point(180, 450)},
                        {key: "Chi", isGroup: true},
                        {key: "Psi", isGroup: true, group: "Chi"},
                        {key: "Phi", isGroup: true, group: "Psi"},
                        {key: "Omega", isGroup: true, group: "Psi"}
                    ],
                    [  // link data
                        {from: "A", to: "B"},
                        {from: "A", to: "C"},
                        {from: "A", to: "C"},
                        {from: "B", to: "B"},
                        {from: "B", to: "C"},
                        {from: "B", to: "Omega"},
                        {from: "C", to: "A"},
                        {from: "C", to: "Psi"},
                        {from: "C", to: "D"},
                        {from: "D", to: "F"},
                        {from: "E", to: "F"},
                        {from: "F", to: "G"}
                    ]);

                // whenever selection changes, run updateHighlights
                this.myDiagram.addDiagramListener("ChangedSelection",
                    function () {
                        self.updateHighlights(self.getRadioButton());
                    });

            },
            getInfo(model, obj) {
                var x = obj.panel.adornedPart; // the object that the mouse is over
                var text = ""; // what will be displayed
                if (x instanceof go.Node) {
                    if (x instanceof go.Group) text += "Group: "; else text += "Node: ";
                    text += x.data.key;
                    var toLst = nodesTo(x, 0); // display names of nodes going into this node
                    if (toLst.count > 0) {
                        toLst.sort(function (a, b) {
                            return a < b ? -1 : 1
                        });
                        text += "\nNodes into: ";
                        toLst.each(function (key) {
                            if (key !== text.substring(text.length - 3, text.length - 2)) {
                                text += key + ", ";
                            }
                        });
                        text = text.substring(0, text.length - 2);
                    }
                    var frLst = nodesFrom(x, 0); // display names of nodes coming out of this node
                    if (frLst.count > 0) {
                        frLst.sort(function (a, b) {
                            return a < b ? -1 : 1
                        });
                        text += "\nNodes out of: ";
                        frLst.each(function (key) {
                            if (key !== text.substring(text.length - 3, text.length - 2)) {
                                text += key + ", ";
                            }
                        });
                        text = text.substring(0, text.length - 2);
                    }
                    var grpC = containing(x, 0); // if the node is in a group, display its name
                    if (grpC !== null) text += "\nContaining SubGraph: " + grpC.data.key;
                    if (x instanceof go.Group) {
                        // if it"s a group, also display nodes and links contained in it
                        text += "\nMember nodes: ";
                        var children = childNodes(x, 0);
                        children.sort(function (a, b) {
                            return a < b ? -1 : 1
                        });
                        children.each(function (key) {
                            if (key !== text.substring(text.length - 3, text.length - 2)) {
                                text += key + ", ";
                            }
                        });
                        text = text.substring(0, text.length - 2);

                        var linkChildren = childLinks(x, 0);
                        if (linkChildren.count > 0) {
                            text += "\nMember links: ";
                            var linkStrings = new go.List(/*"string"*/);
                            linkChildren.each(function (link) {
                                linkStrings.add(link.data.from + " --> " + link.data.to);
                            });
                            linkStrings.sort(function (a, b) {
                                return a < b ? -1 : 1
                            });
                            linkStrings.each(function (str) {
                                text += str + ", ";
                            });
                            text = text.substring(0, text.length - 2);
                        }
                    }
                } else if (x instanceof go.Link) {
                    // if it"s a link, display its to and from nodes
                    text += "Link: " + x.data.from + " --> " + x.data.to +
                        "\nNode To: " + x.data.to + "\nNode From: " + x.data.from;
                    var grp = containing(x, 0); // and containing group, if it has one
                    if (grp !== null) text += "\nContaining SubGraph: " + grp.data.key;
                }
                return text;
            },
            linksTo(x, i) {
                if (x instanceof go.Node) {
                    x.findLinksInto().each(function (link) {
                        link.highlight = i;
                    });
                }
            },
            linksFrom(x, i) {
                if (x instanceof go.Node) {
                    x.findLinksOutOf().each(function (link) {
                        link.highlight = i;
                    });
                }
            },
            linksAll(x, i) {
                if (x instanceof go.Node) {
                    x.linksConnected.each(function (link) {
                        link.highlight = i;
                    });
                }
            },
            nodesTo(x, i) {
                var nodesToList = new go.List(/*"string"*/);
                if (x instanceof go.Link) {
                    x.fromNode.highlight = i;
                    nodesToList.add(x.data.from);
                } else {
                    x.findNodesInto().each(function (node) {
                        node.highlight = i;
                        nodesToList.add(node.data.key);
                    });
                }
                return nodesToList;
            },
            nodesFrom(x, i) {
                var nodesFromList = new go.List(/*"string"*/);
                if (x instanceof go.Link) {
                    x.toNode.highlight = i;
                    nodesFromList.add(x.data.to);
                } else {
                    x.findNodesOutOf().each(function (node) {
                        node.highlight = i;
                        nodesFromList.add(node.data.key);
                    });
                }
                return nodesFromList;
            },
            nodesReach(x, i) {
                const self = this;
                if (x instanceof go.Link) {
                    x.toNode.highlight = i;
                    self.nodesReach(x.toNode, i + 1);
                } else {
                    x.findNodesOutOf().each(function (node) {
                        if (node.highlight === 0 || node.highlight > i) {
                            node.highlight = i;
                            self.nodesReach(node, i + 1);
                        }
                    });
                }
            },
            nodesConnect(x, i) {
                if (x instanceof go.Link) {
                    x.toNode.highlight = i;
                    x.fromNode.highlight = i;
                } else {
                    x.findNodesConnected().each(function (node) {
                        node.highlight = i;
                    });
                }
            },
            containing(x, i) {
                var container = x.containingGroup;
                if (container !== null) container.highlight = i;
                return container;
            },
            containingAll(x, i) {
                const self = this;
                self.containing(x, i);
                var container = x.containingGroup;
                if (container !== null) {
                    self.containingAll(container, i + 1);
                }
            },
            childNodes(x, i) {
                const self = this;
                var childLst = new go.List(/*"string"*/);
                if (x instanceof go.Group) {
                    self.myDiagram.nodes.each(function (node) {
                        if (node.containingGroup === x) {
                            node.highlight = i;
                            childLst.add(node.data.key);
                        }
                    });
                }
                return childLst;
            },
            allMemberNodes(x, i) {
                const self = this;
                if (x instanceof go.Group) {
                    self.myDiagram.nodes.each(function (node) {
                        if (node.containingGroup === x) {
                            node.highlight = i;
                            self.allMemberNodes(node, i + 1);
                        }
                    });
                }
            },
            childLinks(x, i) {
                const self = this;
                var childLst = new go.List(/*go.Link*/);
                self.myDiagram.links.each(function (link) {
                    if (link.containingGroup === x) {
                        link.highlight = i;
                        childLst.add(link);
                    }
                });
                return childLst;
            },
            allMemberLinks(x, i) {
                const self = this;
                self.childLinks(x, i);
                self.myDiagram.nodes.each(function (node) {
                    if (node instanceof go.Group && node.containingGroup === x) {
                        self.allMemberLinks(node, i + 1);
                    }
                });
            },
            highlight(shp, obj2, hl) {
                var color;
                var width = 3;
                if (hl === 0) {
                    color = "black";
                    width = 1;
                } else if (hl === 1) {
                    color = "blue";
                } else if (hl === 2) {
                    color = "green";
                } else if (hl === 3) {
                    color = "orange";
                } else if (hl === 4) {
                    color = "red";
                } else {
                    color = "purple";
                }
                shp.stroke = color;
                shp.strokeWidth = width;
                if (obj2 !== null) {
                    obj2.stroke = color;
                    obj2.fill = color;
                }
            },
            getRadioButton() {
                var radio = document.getElementsByName("highlight");
                for (var i = 0; i < radio.length; i++)
                    if (radio[i].checked) return radio[i];
            },
            updateHighlights(e) {
                const self = this;
                // Set highlight to 0 for everything before updating
                self.myDiagram.nodes.each(function (node) {
                    node.highlight = 0;
                });
                self.myDiagram.links.each(function (link) {
                    link.highlight = 0;
                });

                // Get the selected GraphObject and run the appropriate method
                var sel = self.myDiagram.selection.first();
                if (sel !== null) {
                    switch (e.id) {
                        case "linksIn":
                            self.linksTo(sel, 1);
                            break;
                        case "linksOut":
                            self.linksFrom(sel, 1);
                            break;
                        case "linksAll":
                            self.linksAll(sel, 1);
                            break;
                        case "nodesIn":
                            self.nodesTo(sel, 1);
                            break;
                        case "nodesOut":
                            self.nodesFrom(sel, 1);
                            break;
                        case "nodesConnect":
                            self.nodesConnect(sel, 1);
                            break;
                        case "nodesReach":
                            self.nodesReach(sel, 1);
                            break;
                        case "group":
                            self.containing(sel, 1);
                            break;
                        case "groupsAll":
                            self.containingAll(sel, 1);
                            break;
                        case "nodesMember":
                            self.childNodes(sel, 1);
                            break;
                        case "nodesMembersAll":
                            self.allMemberNodes(sel, 1);
                            break;
                        case "linksMember":
                            self.childLinks(sel, 1);
                            break;
                        case "linksMembersAll":
                            self.allMemberLinks(sel, 1);
                            break;
                    }
                }

                // Give everything the appropriate highlighting ( color and width of stroke )
                // nodes, including groups
                self.myDiagram.nodes.each(function (node) {
                    var shp = node.findObject("OBJSHAPE");
                    var grp = node.findObject("GROUPTEXT");
                    var hl = node.highlight;
                    self.highlight(shp, grp, hl);
                });
                // links
                self.myDiagram.links.each(function (link) {
                    var hl = link.highlight;
                    var shp = link.findObject("OBJSHAPE");
                    var arw = link.findObject("ARWSHAPE");
                    self.highlight(shp, arw, hl);
                });
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
