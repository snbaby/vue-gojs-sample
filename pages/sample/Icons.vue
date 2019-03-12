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
                colors: {
                    blue: "#00B5CB",
                    orange: "#F47321",
                    green: "#C8DA2B",
                    gray: "#888",
                    white: "#F5F5F5"
                }
            }
        },
        mounted() {
            this.init();
        },
        methods: {
            init() {
                //初始化
                this.myDiagram =
                    $(go.Diagram, "myDiagramDiv",
                        {
                            "undoManager.isEnabled": true,
                            layout: $(go.TreeLayout)
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
