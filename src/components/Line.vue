<template>
    <div :id="id">
        <div id = "main" style = "display : flex ; flex-direction : row;">
        <div style = "display : flex ; flex-direction : column;" >
            <svg id="amyOpened"></svg>
            <svg id="julieOpened"></svg>
            <svg id="beckyOpened"></svg>
        </div>
        <div style = "display : flex ; flex-direction : column;" >
            <svg id="amyCompleted"></svg>
            <svg id="julieCompleted"></svg>
            <svg id="beckyCompleted"></svg>
        </div>
        </div>

        <div style = "display : flex ; flex-direction : row ; margin-left : 16em ; margin-top : 1em;">
                <label for="Both">Both</label>
                <input type="radio" name="linesShown" id="Both" value="Both" checked><br>
                <label for="2017">2017</label>
                <input type="radio" name="linesShown" id="2017" value="2017"><br>
                <label for="2018">2018</label>
                <input type="radio" name="linesShown" id="2018" value="2018"><br><br>
        </div>
    </div>
</template>

<script>

import uuid from 'uuid/v4'

export default {
    name: 'lineGraph',
    data() {
        return {
            id: null
        }
    },
    props: ["data"],
    created() {
        this.id = "line_" + uuid()
    },
    mounted(){
            var data = this.data

            var months = {
                1 : "Jan",
                2 : "Feb",
                3 : "Mar",
                4 : "Apr",
                5 : "May",
                6 : "Jun",
                7 : "Jul",
                8 : "Aug",
                9 : "Sep",
                10 : "Oct",
                11 : "Nov",
                12 : "Dec"
            }

            var width = 320;
            var height = 167;

        var svgAmyOpened = d3.select(`#${this.id} #amyOpened`).attr('width', width).attr('height', height);
        var svgJulieOpened = d3.select(`#${this.id} #julieOpened`).attr('width', width).attr('height', height);
        var svgBeckyOpened = d3.select(`#${this.id} #beckyOpened`).attr('width', width).attr('height', height);
        var svgAmyCompleted = d3.select(`#${this.id} #amyCompleted`).attr('width', width).attr('height', height);
        var svgJulieCompleted = d3.select(`#${this.id} #julieCompleted`).attr('width', width).attr('height', height);
        var svgBeckyCompleted = d3.select(`#${this.id} #beckyCompleted`).attr('width', width).attr('height', height);

        var margin = {top: 4, right: 20, bottom: 30, left: 50},
            width = width - margin.left - margin.right,
            height = height - margin.top - margin.bottom;

        var x = d3.scaleLinear()
            .rangeRound([0, width]);

        var y = d3.scaleLinear()
            .rangeRound([height, 0]);

        var line = d3.line()
            .x(function(d, i) { return x (i+ 1); })
            .y(function(d) { return y(d); });

        d3.selectAll("input")
            .data(['both', '2017', '2018'])
            .on("change", changed);

        function changed(chosen) {

            var line1 = d3.selectAll("#line1")
            var line2 = d3.selectAll("#line2")

            if(chosen === '2017' || chosen === 'both'){
                line1.transition()
                .duration(1000)
                .attr("opacity", 1)
            }
            if(chosen === '2018' || chosen === 'both'){
                line2.transition()
                .duration(1000)
                .attr("opacity", 1)
            }
            if(chosen != '2017' && chosen != 'both'){
                line1.transition()
                .duration(1000)
                .attr("opacity", 0)
            }
            if(chosen != '2018' && chosen != 'both'){
                line2.transition()
                .duration(1000)
                .attr("opacity", 0)
            }

        }

        var arrOpened = data.opened
        var arrCompleted = data.completed

        var amy2017Opened = arrOpened.find(function(d){
            return d.year === '2017'
        }).amy
        var amy2018Opened = arrOpened.find(function(d){
            return d.year === '2018'
        }).amy
        var julie2017Opened = arrOpened.find(function(d){
            return d.year === '2017'
        }).julie
        var julie2018Opened = arrOpened.find(function(d){
            return d.year === '2018'
        }).julie
        var becky2017Opened = arrOpened.find(function(d){
            return d.year === '2017'
        }).becky
        var becky2018Opened = arrOpened.find(function(d){
            return d.year === '2018'
        }).becky
        // Completed
        var amy2017Completed = arrCompleted.find(function(d){
            return d.year === '2017'
        }).amy
        var amy2018Completed = arrCompleted.find(function(d){
            return d.year === '2018'
        }).amy
        var julie2017Completed = arrCompleted.find(function(d){
            return d.year === '2017'
        }).julie
        var julie2018Completed = arrCompleted.find(function(d){
            return d.year === '2018'
        }).julie
        var becky2017Completed = arrCompleted.find(function(d){
            return d.year === '2017'
        }).becky
        var becky2018Completed = arrCompleted.find(function(d){
            return d.year === '2018'
        }).becky

            makeSvg(svgAmyOpened, amy2017Opened, amy2018Opened , "Amy")
            makeSvg(svgJulieOpened, julie2017Opened, julie2018Opened, "Julie")
            makeSvg(svgBeckyOpened, becky2017Opened, becky2018Opened, "Becky")
            makeSvg(svgAmyCompleted, amy2017Completed, amy2018Completed , "Amy")
            makeSvg(svgJulieCompleted, julie2017Completed, julie2018Completed, "Julie")
            makeSvg(svgBeckyCompleted, becky2017Completed, becky2018Completed, "Becky")

        function makeSvg(svg, arr2017, arr2018, name){

            console.log(svg)
        
            var g = svg.append("g").attr("transform", "translate(" + margin.left + "," + margin.top + ")");

            x.domain(d3.extent(arr2017, function(d, i) { return i + 1; }));
            y.domain(d3.extent(arr2017.concat(arr2018), function(d) { return d; }));

            if (name === "Becky") {
                g.append("g")
                    .attr("transform", "translate(0," + height + ")")
                    .call(d3.axisBottom(x).tickFormat(function(d, i){return months[i + 1]}))
                .append("text")
                    .attr("fill", "#000")
                    .attr("y", 20)
                    .attr("x" , 140)
                    .attr("dy", "0.71em")
                    .attr("text-anchor", "end")
                    .text("Months");
            }

            g.append("g")
                .call(d3.axisLeft(y))
            .append("text")
                .attr("fill", "#000")
                .attr("transform", "rotate(-90)")
                .attr("y", 6)
                .attr("dy", "0.71em")
                .attr("text-anchor", "end")
                .text(name);

            g.append("path")
                .datum(arr2017)
                .attr("fill", "none")
                .attr("id", "line1")
                .attr("opacity", 1)
                .attr("stroke", "steelblue")
                .attr("stroke-linejoin", "round")
                .attr("stroke-linecap", "round")
                .attr("stroke-width", 1.5)
                .attr("d", line(arr2017.filter(function(d) {
                    return d != 0;
                })));

            g.append("path")
                .datum(arr2018)
                .attr("fill", "none")
                .attr("id", "line2")
                .attr("opacity", 1)
                .attr("stroke", "red")
                .attr("stroke-linejoin", "round")
                .attr("stroke-linecap", "round")
                .attr("stroke-width", 1.5)
                .attr("d", line(arr2018.filter(function(d) {
                    return d != 0;
                })));
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