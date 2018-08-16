<template>
    <button>Update</button>
</template>
<script src="https://d3js.org/d3.v4.min.js"></script>

<script>
import bullet from '../assets/bullet.js'
import data from '../assets/bullets.json'


export default {
  name: 'histogram',
  data() {
    return {
        
    }
  },
  methods: {
      
  },
mounted(){
            
    var svg = d3.select("svg"),
        width = +svg.attr("width"),
        height = +svg.attr("height");

        var format = d3.format(",d");

    var treemap = d3.treemap()
        .tile(d3.treemapResquarify)
        .size([width, height])
        .round(true)
        .paddingInner(1);

    d3.json("allPipelineData.json", function(error, data) {
    if (error) throw error;

    var arr = []
    for(var i = 0; i < data.length; i++){
        if(i === 0){
            arr.push({
                assignee: data[i].assignee,
                cards: parseInt(data[i].Card_Is_Open)
            })
        }
        else{
            var obj = arr.find(function(o){
                return o.assignee === data[i].assignee
            })
            if(obj != null){
                obj.cards = obj.cards + parseInt(data[i].Card_Is_Open)
            }
            else{
                arr.push({
                    assignee: data[i].assignee,
                    cards: parseInt(data[i].Card_Is_Open)
                })
            }
        }
    }

    var allData = {
        name: "",
        children: arr
    }

    var root = d3.hierarchy(allData)
        .eachBefore(function(d) { d.data.id = d.data.assignee; })
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
            if (d.data.assignee == null) {
                return "";
            }
            else {
                var name = d.data.assignee + " " + d.data.cards; 
                return name.split(/(?=[A-Z][^A-Z])/g); 
            }})
        .enter().append("tspan")
        .attr("x", 4)
        .attr("y", function(d, i) { return 13 + i * 10; })
        .text(function(d) { return d; });

    cell.append("title")
        .text(function(d) { return d.data.id + "\n" + format(d.value); });

        }); 

    function sumBySize(d) {
    return d.cards;
    }

  }
}
</script>

<style>

body {
  font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
  margin: auto;
  padding-top: 40px;
  position: relative;
  width: 960px;
}

button {
  position: absolute;
  right: 10px;
  top: 10px;
}

.bullet { font: 10px sans-serif; }
.bullet .marker { stroke: #000; stroke-width: 2px; }
.bullet .tick line { stroke: #666; stroke-width: .5px; }
.bullet .range.s0 { fill: #eee; }
.bullet .range.s1 { fill: #ddd; }
.bullet .range.s2 { fill: #ccc; }
.bullet .measure.s0 { fill: lightsteelblue; }
.bullet .measure.s1 { fill: steelblue; }
.bullet .title { font-size: 14px; font-weight: bold; }
.bullet .subtitle { fill: #999; }

</style>