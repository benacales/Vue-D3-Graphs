<template>
    <div class = "histoGraph" :id="id" @mousemove="onmousemove" ref="histoGraph">
        <svg ref = "histoGraphSVG" width="1000" height="600" style="padding-left: 40px ; padding-top: 10px"></svg>
    </div>
</template>

<script>
// import bullet from '../assets/bullet.js'
import uuid from 'uuid/v4'

export default {
    name: 'histoGraph',
    props: ['data'],
    data() {
        return {
            id: null,
            onmousemove: function() {}
        }
    },

    created() {
        this.id = "hist_" + uuid()
    },

    mounted() {
        var data = this.data,
        svg = d3.select(`#${this.id} > svg`)
        var margin = {top: 10, right: 30, bottom: 40, left: 60}
        var padding = {top: 10, right: 0, bottom: 0, left: 40},
        width = svg.attr('width') - margin.left - margin.right - 30,
        height = svg.attr('height') - margin.top - margin.bottom - 30;

        if(width > height){
            width = (3/2) * height;
        }
        else {
            height = (2/3) * width;
        }

        var max = 0;
        var maxY= 0;
        var arrTicks = []
        var dataJSON = []
    
        var color =  d3.scaleLinear()
            .domain([0, 1, 2, 3, 4, 5, 6])
            .range(['#005BB7', '#2a53aa', '#55619d', '#7f6490', '#a96683', '#d46976','#FE6C69' ]);
        
        svg
            .append("g")
            .attr("transform",
                "translate(" + margin.left + "," + margin.top + ")");                

        var div = d3.select(`#${this.id}`).append('div')
            .attr("class", "tooltip")				
            .style("opacity", 0);

            var data = this.data.map( d => {
                d.cycleTime = parseInt(d.cycleTime);
                d.leadTime = parseInt(d.leadTime);
                return d;
            })
            
            dataJSON = data

            for (var i = 0 ; i < data.length ; i++) {
                if (data[i].cycleTime > max) {
                    max = data[i].cycleTime
                }
            }
        
            max = Math.ceil(max/100)*100
            for (var i = 0 ; i < max/100 ; i++){
                arrTicks.push(0)
            }

            for (var i = 0 ; i < data.length ; i++) {
                var column = Math.floor(data[i].cycleTime/100)

                arrTicks[column]++
            }

            for (var i = 0 ; i < arrTicks.length ; i ++) {
                if (arrTicks[i] > maxY) {
                    maxY = arrTicks[i];
                }
            }

            var x = d3.scaleLinear()
                .domain([0 , max])
                .rangeRound([0, width]);
            var y = d3.scaleLinear()
                .range([height, 0]);

            var histogram = d3.histogram()
                .value(function(d) { return d.cycleTime; })
                .domain(x.domain())
                .thresholds(x.ticks(max/100));

            var bins = histogram(data);

            y.domain([0, maxY]);


            svg.selectAll("rect")
                .data(bins)
                .enter().append("rect")
                .attr("id", function(d, i) {
                    return "rect" + i
                })
                .attr("class", "bar")
                .attr("x", 1)
                .attr("transform", function(d) {
                    return "translate(" + x(d.x0) + "," + y(d.length) + ")"; })
                .attr("width", function(d) { 
                    if ( x(d.x1) - x(d.x0) === 0){
                        return 0;
                    }
                    return x(d.x1) - x(d.x0) -1 ; })
                .attr("height", function(d) { return height - y(d.length); })
                .attr("fill", function(d,i){ return color(i)})

            function positionOf(el) {
                // for (var lx=0, ly=0;
                //     el != null;
                //     lx += el.offsetLeft, ly += el.offsetTop, el = el.offsetParent);
                var rect = el.getBoundingClientRect()
                var body = document.querySelector('body').getBoundingClientRect()

                return {y: rect.top - body.y, x: rect.left};
            }

            this.onmousemove = () => {
                var x = event.pageX;
                var y = event.pageY;

                console.log({x, y})

                var divPos = positionOf(this.$refs.histoGraph)

                // var divBorders = {minX: divPos.x, maxX: divPos.x + width, minY: divPos.y, maxY: divPos.y + height}

                // if((margin.left < x  && x < (width + margin.left)) && ( 0 < y  && y < height)){
                if ((divPos.x + padding.left) < x && x < (divPos.x + width + padding.left) && (divPos.y + padding.top) < y && y < (divPos.y + height + padding.top) ) {

                    var rectWidth = width/arrTicks.length;
                
                    var xPosition = x - divPos.x - padding.left
                    
                    var index = Math.floor(xPosition / rectWidth)
                    
                    div.transition()
                        .duration(180)
                        .style("opacity", .9)
                    div.html(arrTicks[index])
                        .style("left", (x - divPos.x + 15 - padding.left) + "px")
                        .style("top", (y - 25) + "px")

                }
                else {
                    div.transition()		
                    .duration(500)		
                    .style("opacity", 0);	
                }
            
            }

            svg.append("g")
                .attr("transform", "translate(0," + height + ")")
                .call(d3.axisBottom(x).ticks(max/100));
                
            svg.append("g")
                .call(d3.axisLeft(y));
            svg.append("text")             
                .attr("transform" , "translate(" + (width/2) + " ," + (height + margin.top + 25) + ")")
                .style("text-anchor", "middle")
                .text("Number of Cycle Days");
            svg.append("text")
                .attr("transform", "rotate(-90)")
                .attr("y", 0 - margin.left)
                .attr("x",0 - (height / 2))
                .attr("dy", "1em")
                .style("text-anchor", "middle")
                .text("Number of Tasks Completed");  

            window.addEventListener('resize' , function(){
                svg.selectAll("*").remove();
                width = svg.attr('width') - margin.left - margin.right - 30;
                height = svg.attr('height') - margin.top - margin.bottom - 30;

                if(width > height){
                    width = (3/2) * height;
                }
                else {
                    height = (2/3) * width;
                }
                // console.log("what is width and height now", width, height, dataJSON)
                svg.attr("width", width + margin.left + margin.right )
                .attr("height", height + margin.top + margin.bottom)

                    
                var x = d3.scaleLinear()
                    .domain([0 , max])
                    .rangeRound([0, width]);
                var y = d3.scaleLinear()
                    .range([height, 0]);

                var histogram = d3.histogram()
                .value(function(d) { return d.cycleTime; })
                .domain(x.domain())
                .thresholds(x.ticks(max/100));

                var bins = histogram(dataJSON);

                y.domain([0, maxY]);


                svg.selectAll("rect")
                    .data(bins)
                    .enter().append("rect")
                    .attr("id", function(d, i) {
                        return "rect" + i
                    })
                    .attr("class", "bar")
                    .attr("x", 1)
                    .attr("transform", function(d) {
                        return "translate(" + x(d.x0) + "," + y(d.length) + ")"; })
                    .attr("width", function(d) { 
                        if ( x(d.x1) - x(d.x0) === 0){
                            return 0;
                        }
                        return x(d.x1) - x(d.x0) -1 ; })
                    .attr("height", function(d) { return height - y(d.length); })
                    .attr("fill", function(d,i){ return color(i)})

                svg.append("g")
                    .attr("transform", "translate(0," + height + ")")
                    .call(d3.axisBottom(x).ticks(max/100));
                    
                svg.append("g")
                    .call(d3.axisLeft(y));
                svg.append("text")             
                    .attr("transform" , "translate(" + (width/2) + " ," + (height + margin.top + 25) + ")")
                    .style("text-anchor", "middle")
                    .text("Number of Cycle Days");
                svg.append("text")
                    .attr("transform", "rotate(-90)")
                    .attr("y", 0 - margin.left)
                    .attr("x",0 - (height / 2))
                    .attr("dy", "1em")
                    .style("text-anchor", "middle")
                    .text("Number of Tasks Completed");  
            }) 
    }
}
</script>

<style>

div.tooltip {	
    position: absolute;			
    text-align: center;			
    width: 60px;					
    height: 28px;					
    padding: 2px;				
    font: 12px sans-serif;		
    background: lightsteelblue;	
    border: 0px;		
    border-radius: 8px;			
    pointer-events: none;			
}   

</style>