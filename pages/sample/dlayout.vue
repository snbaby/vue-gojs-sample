<template>
    <v-content>
        <div id="dd" class="diagram"></div>
    </v-content>
</template>
<script>
    import * as go from "gojs";

    const $ = go.GraphObject.make;

    //import {makeDiagram,loadData} from "@/core/diagram";

    function KanLayout() {
        go.GridLayout.call(this);
        this.alignment = go.GridLayout.Position;
        this.isRealtime = false;
        this.wrappingColumn = 3;
        this.wrappingWidth = 800;
        this.cellSize = new go.Size(400, 40);
        this.spacing = new go.Size(0, 20);
        this.comparer = function (a, b) {

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
        };
    }

    go.Diagram.inherit(KanLayout, go.GridLayout);

    KanLayout.prototype.doLayout = function (coll) {
        /*
        var parts = this.collectParts(coll);
        var x = this.arrangementOrigin.x;
        var y = this.arrangementOrigin.y;
        var offset = new go.Size(20,20);

        var it = parts.iterator;
        while (it.next()) {
          var node = it.value;
          if (!(node instanceof go.Node)) continue;  // ignore Links
          node.move(new go.Point(x, y));
          x += offset.width + node.measuredBounds.width;
          console.log(node)
          //y += offset.height;
        }
        */

        go.GridLayout.prototype.doLayout.call(this, coll);
    }
    var diagram = false;

    function relayoutNodes() {
        diagram.layout.invalidateLayout();
        /*
        diagram.nodes.each(function(node){
            node.layout.invalidateLayout=false;
        });
        */
        diagram.layoutDiagram();
    }

    export default {
        mounted() {

            //图
            diagram = $(go.Diagram, "dd", {
                initialContentAlignment: go.Spot.Center,
                layout: new KanLayout(), //自定义布局
                "SelectionMoved": relayoutNodes
            });

            //节点模版
            diagram.nodeTemplate =
                $(go.Node, "Auto",

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
                );

            diagram.model = new go.Model([
                {key: "Alpha", color: "lightblue", width: 1},
                {key: "Beta", color: "thistle", width: 1},
                {key: "Delta", color: "lightcoral", width: 2},
                {key: "Gamma", color: "thistle", width: 1},
                {key: "Field1", color: "lightgreen", width: 1},
                {key: "Field2", color: "thistle", width: 2},
                {key: "Field3", color: "lightgreen", width: 2}
            ]);

        }
    }
</script>


<style scoped>
    .diagram {
        height: 100%;
        background: #fbfcfa;
    }

</style>
