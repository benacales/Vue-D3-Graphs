<template>
    <div :id="id">
        <svg width="960" height="570"></svg>
    </div>
</template>

<script>
import uuid from 'uuid/v4'

export default {
    name: 'treemap',
    props: ["data"],
    data() {
        return {
            id: null
        }
    },

    created() {
        this.id = "tree_" + uuid()
    },

    mounted(){
        var data = this.data

        var svg = d3.select(`#${this.id} > svg`),

            width = +svg.attr("width"),
            height = +svg.attr("height");

            var format = d3.format(",d");

        var treemap = d3.treemap()
            .tile(d3.treemapResquarify)
            .size([width, height])
            .round(true)
            .paddingInner(1);

        var arr = []
                for(var i = 0; i < data.length; i++){
                    if(i === 0){
                        arr.push({
                            chosenPerson: data[i].chosenPerson,
                            num: parseInt(data[i].num)
                        })
                    }
                    else{
                        var obj = arr.find(function(o){
                            return o.chosenPerson === data[i].chosenPerson
                        })
                        if(obj != null){
                            obj.num = obj.num + parseInt(data[i].num)
                        }
                        else{
                            arr.push({
                                chosenPerson: data[i].chosenPerson,
                                num: parseInt(data[i].num)
                            })
                        }
                    }
                }

                var allData = {
                    name: "",
                    children: arr
                }

        var root = d3.hierarchy(allData)
            .eachBefore(function(d) { d.data.id = d.data.chosenPerson; })
            .sum(sumBySize)
            .sort(function(a, b){
            return b.height - a.height || b.value - a.value; });

        treemap(root);

        var cell = svg.selectAll("g")
            .data(root.leaves())
            .enter().append("g")
            .attr("transform", function(d) { return "translate(" + d.x0 + "," + d.y0 + ")"; });

        cell.append("rect")
            .attr("id", function(d) { return d.data.id; })
            .attr("width", function(d) { return d.x1 - d.x0; })
            .attr("height", function(d) { return d.y1 - d.y0; })
            .attr("fill", "lightblue");

        cell.append("clipPath")
            .attr("id", function(d) { return "clip-" + d.data.id; })
            .append("use")
            .attr("xlink:href", function(d) { return "#" + d.data.id; });

        cell.append("text")
            .attr("clip-path", function(d) { return "url(#clip-" + d.data.id + ")"; })
            .selectAll("tspan")
            .data(function(d) { 
                if (d.data.chosenPerson == null) {
                    return "";
                }
                else {
                    var name = d.data.chosenPerson + " " + d.data.num; 
                    return name.split(/(?=[A-Z][^A-Z])/g); 
                }})
            .enter().append("tspan")
            .attr("x", 4)
            .attr("y", function(d, i) { return 13 + i * 10; })
            .text(function(d) { return d; });

        cell.append("title")
            .text(function(d) { return d.data.id + "\n" + format(d.value); });

        function sumBySize(d) {
        return d.num;
        }

  }
}
</script>

<style>

form {
  font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
}

svg {
  font: 10px sans-serif;
}

</style>