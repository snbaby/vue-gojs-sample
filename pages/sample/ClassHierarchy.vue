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
                //初始化
                this.myDiagram =
                    $(go.Diagram, "myDiagramDiv",  // id of DIV
                        { // Automatically lay out the diagram as a tree;
                            // separate trees are arranged vertically above each other.
                            layout: $(go.TreeLayout, {nodeSpacing: 3})
                        });
                //默认节点模板
                this.myDiagram.nodeTemplate =
                    $(go.Node,
                        $("HyperlinkText",
                            // compute the URL to open for the documentation
                            function (node) {
                                return "../api/symbols/" + node.data.key + ".html";
                            },
                            // define the visuals for the hyperlink, basically the whole node:
                            $(go.Panel, "Auto",
                                $(go.Shape, {fill: "#1F4963", stroke: null}),
                                $(go.TextBlock,
                                    {
                                        font: "bold 13px Helvetica, bold Arial, sans-serif",
                                        stroke: "white", margin: 3
                                    },
                                    new go.Binding("text", "key"))
                            )
                        )
                    );
                this.diagram.linkTemplate =
                    $(go.Link,
                        {
                            curve: go.Link.Bezier,
                            toEndSegmentLength: 30,
                            fromEndSegmentLength: 30
                        },
                        $(go.Shape, {strokeWidth: 1.5}) // the link shape, with the default black stroke
                    );

                // Collect all of the data for the model of the class hierarchy
                let nodeDataArray = [];

                // Iterate over all of the classes in "go"
                for (k in go) {
                    var cls = go[k];
                    if (!cls) {
                        continue;
                    }
                    let proto = cls.prototype;
                    if (!proto) {
                        continue;
                    }
                    if (k === 'EnumValue' || k === 'TextBlockMetrics') {
                        continue;
                    } // undocumented classes
                    proto.constructor.className = k;  // remember name
                    // find base class constructor
                    var base = Object.getPrototypeOf(proto).constructor;
                    if (base === Object) {  // "root" node?
                        nodeDataArray.push({key: k});
                    } else {
                        // add a node for this class and a tree-parent reference to the base class name
                        nodeDataArray.push({key: k, parent: base.className});
                    }
                }

                // Create the model for the hierarchy diagram
                this.diagram.model = new go.TreeModel(nodeDataArray);

                // Now collect all node data that are singletons
                let singlesArray = [];  // for classes that don't inherit from another class
                this.diagram.nodes.each(function (node) {
                    if (node.linksConnected.count === 0) {
                        singlesArray.push(node.data);
                    }
                });

                // Remove the unconnected class nodes from the main Diagram
                this.diagram.model.removeNodeDataCollection(singlesArray);

                // Display the unconnected classes in a separate Diagram
                let singletons =
                    $(go.Diagram, "mySingletons",
                        {
                            nodeTemplate: self.diagram.nodeTemplate, // share the node template with the main Diagram
                            layout:
                                $(go.GridLayout,
                                    {
                                        wrappingColumn: 1,  // put the unconnected nodes in a column
                                        spacing: new go.Size(3, 3)
                                    }),
                            model: new go.Model(singlesArray)  // use a separate model
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
