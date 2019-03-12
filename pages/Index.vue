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
            this.load();
        },
        methods: {
            load() {
                this.init();
                this.layout();
            },
            layout() {
                this.myDiagram.model = new go.GraphLinksModel(
                    [
                        { key: "Alpha", color: "lightblue" },
                        { key: "Beta", color: "orange" },
                        { key: "Gamma", color: "lightgreen" },
                        { key: "Delta", color: "pink" }
                    ],
                    [
                        { from: "Alpha", to: "Beta" },
                        { from: "Alpha", to: "Gamma" },
                        { from: "Beta", to: "Beta" },
                        { from: "Gamma", to: "Delta" },
                        { from: "Delta", to: "Alpha" }
                    ]);
            },
            init() {
                //初始化
                this.myDiagram =
                    $(go.Diagram, "myDiagramDiv",
                        {
                            "undoManager.isEnabled": true
                        });
                //默认节点模板
                this.myDiagram.nodeTemplate =
                    $(go.Node, "Auto",  // the Shape will go around the TextBlock
                        $(go.Shape, "RoundedRectangle", {strokeWidth: 0, fill: "white"},
                            // Shape.fill is bound to Node.data.color
                            new go.Binding("fill", "color")),
                        $(go.TextBlock,
                            {margin: 8},  // some room around the text
                            // TextBlock.text is bound to Node.data.key
                            new go.Binding("text", "key"))
                    );
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
