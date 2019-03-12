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
                names:{}
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
                            initialAutoScale: go.Diagram.UniformToFill,
                            // define the layout for the diagram
                            layout: $(go.TreeLayout, {nodeSpacing: 5, layerSpacing: 30})
                        });
                // Define a simple node template consisting of text followed by an expand/collapse button
                this.myDiagram.nodeTemplate =
                    $(go.Node, "Horizontal",
                        {selectionChanged: self.nodeSelectionChanged},  // this event handler is defined below
                        $(go.Panel, "Auto",
                            $(go.Shape, {fill: "#1F4963", stroke: null}),
                            $(go.TextBlock,
                                {
                                    font: "bold 13px Helvetica, bold Arial, sans-serif",
                                    stroke: "white", margin: 3
                                },
                                new go.Binding("text", "key"))
                        ),
                        $("TreeExpanderButton")
                    );
                // Define a trivial link template with no arrowhead.
                this.myDiagram.linkTemplate =
                    $(go.Link,
                        {selectable: false},
                        $(go.Shape));  // the link shape

                // create the model for the DOM tree
                this.myDiagram.model =
                    $(go.TreeModel, {
                        isReadOnly: true,  // don't allow the user to delete or copy nodes
                        // build up the tree in an Array of node data
                        nodeDataArray: self.traverseDom(document.activeElement)
                    });
            },
            nodeSelectionChanged(node) {
                if (node.isSelected) {
                    this.names[node.data.name].style.backgroundColor = "lightblue";
                } else {
                    this.names[node.data.name].style.backgroundColor = "";
                }
            },
            traverseDom(node, parentName, dataArray) {
                if (parentName === undefined) {
                    parentName = null;
                }
                if (dataArray === undefined) {
                    dataArray = [];
                }
                // skip everything but HTML Elements
                if (!(node instanceof Element)) {
                    return;
                }
                // Ignore the navigation menus
                if (node.id === "navindex" || node.id === "navtop") {
                    return;
                }
                // add this node to the nodeDataArray
                var name = this.getName(node);
                var data = {key: name, name: name};
                dataArray.push(data);
                // add a link to its parent
                if (parentName !== null) {
                    data.parent = parentName;
                }
                // find all children
                var l = node.childNodes.length;
                for (var i = 0; i < l; i++) {
                    this.traverseDom(node.childNodes[i], name, dataArray);
                }
                return dataArray;
            },
            getName(node) {
                var n = node.nodeName;
                if (node.id) {
                    n = n + " (" + node.id + ")";
                }
                var namenum = n;  // make sure the name is unique
                var i = 1;
                while (this.names[namenum] !== undefined) {
                    namenum = n + i;
                    i++;
                }
                this.names[namenum] = node;
                return namenum;
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
