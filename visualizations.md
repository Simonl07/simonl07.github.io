layout: page
title: "Data Visualizations"
permalink: /visualizations

# Data Visualizations


<style>
.chart {
  font: 10px serif;
  background-color: #eaeaea;
  padding: 3px;
  margin: 1px;
  color: white;
}

</style>
<div class="chart"></div>
<script src="https://d3js.org/d3.v6.min.js"></script>
<script>
    var margin = 200,
    width = 1300 - margin,
    height = 1000 - margin;
	const scale = d3.scaleOrdinal(d3.schemeCategory10);
	const color = d => scale(d.group);
	var radius = d3.scaleLinear()
		.domain([0, 40])
		.range([10, 30]);
	var f = d3.scaleLinear()
		.domain([0, 40])
		.range([9, 25]);
	const drag = simulation => {
		function dragstarted(event) {
			if (!event.active) simulation.alphaTarget(0.3).restart();
			event.subject.fx = event.subject.x;
			event.subject.fy = event.subject.y;
		}
		function dragged(event) {
			event.subject.fx = event.x;
			event.subject.fy = event.y;
		}
		function dragended(event) {
			if (!event.active) simulation.alphaTarget(0);
			event.subject.fx = null;
			event.subject.fy = null;
		}
		return d3.drag()
		  .on("start", dragstarted)
		  .on("drag", dragged)
		  .on("end", dragended);
	}
	d3.csv("https://gist.githubusercontent.com/Simonl07/bec96bcef1d46ddcc20d395161e72877/raw/924bce9623075fa3fea6e1fac63909e7c40c2148/soc-firm-hi-tech.csv", function(d) {
	  	return d;
	}).then(function(data) {
		d3.select(".chart")
			.append("svg")
			.attr("width", width)
			.attr("height", height)
		var svg = d3.select("svg")
		var adj_list = {};
		var links = [];
		for (row of data) {
			if (!(row.Source in adj_list)) {
				adj_list[row.Source] = []
			}
			if (!(row.Target in adj_list)) {
				adj_list[row.Target] = []
			}
			adj_list[row.Source].push(row.Target);
			adj_list[row.Target].push(row.Source);
			l = {
				source: row.Source,
				target: row.Target,
				value: 5
			}
			links.push(l)
		}
		node_sequence = Object.keys(adj_list).map(d => parseInt(d)).sort((a, b) => a - b)
		node_objs = []
		for (var n of Object.keys(adj_list)) {
			node_objs.push({id: n, group: 1})
		}
		function hover(id) {
			svg.selectAll(`circle[data-id="${id}"]`).style("fill", '#41b6c4');
			svg.selectAll(`rect[data-id="${id}"]`).style("fill", d =>{
				if (d.value){
					return 'grey'
				}
				return '#7fcdbb'
			});
			lines = svg.selectAll('line');
			lines.style('stroke', d => {
				link_target = node_objs[d.target.index].id
				link_source = node_objs[d.source.index].id
				if (link_target == id || link_source == id) {
					return '#7fcdbb';
				}else{
					return color
				}
			});
		}
		function hover_rect(source, target) {
			svg.selectAll(`circle`).style("fill", '#7382a5');
			svg.selectAll(`line`).style("stroke", '#7382a5');
			svg.select(`circle[data-id="${source}"]`).style("fill", '#41b6c4');
			svg.select(`circle[data-id="${target}"]`).style("fill", '#41b6c4');
			lines = svg.selectAll('line');
			lines.style('stroke', d => {
				link_target = node_objs[d.target.index].id
				link_source = node_objs[d.source.index].id
				if (link_target == target && link_source == source || link_target == source && link_source == target) {
					return '#7fcdbb';
				}else{
					return color
				}
			});
			svg.selectAll(`rect[data-id="${source}"]`).style("fill", d =>{
				if (d.value){
					return 'grey'
				}
				return '#7fcdbb'
			});
			svg.selectAll(`rect[data-column="${target}"]`).style("fill", d =>{
				if (d.value){
					return 'grey'
				}
				return '#7fcdbb'
			});
		}
		function exit_rect(source, target) {
			lines = svg.selectAll('line');
			lines.style('stroke', color);
			svg.selectAll(`circle`).style("fill", '#1c3c94');
			svg.selectAll(`rect[data-id="${source}"]`).style("fill", d =>{
				if (d.value){
					return 'grey'
				}
				return 'none'
			});
			svg.selectAll(`rect[data-column="${target}"]`).style("fill", d =>{
				if (d.value){
					return 'grey'
				}
				return 'none'
			});
		}
		function exit(id) {
			svg.selectAll(`circle[data-id="${id}"]`).style("fill", '#1c3c94');
			svg.selectAll(`rect[data-id="${id}"]`).style("fill", d =>{
				if (d.value){
					return 'grey'
				}
				return 'none'
			});
			svg.selectAll('line').style("stroke", "#2c7fb8")
		}
		const simulation = d3.forceSimulation(node_objs)
		  .force("link", d3.forceLink(links).id(d => d.id))
		  .force("charge", d3.forceManyBody().strength(-200))
		  .force("center", d3.forceCenter(250, 300));
		const link = svg.append("g")
		  .attr("stroke", "#2c7fb8")
		  .attr("stroke-opacity", 1)
		.selectAll("line")
		.data(links)
		.join("line")
		  .attr("class", 'line')
		  .attr("stroke-width", d => Math.sqrt(d.value));
		const node = svg.append("g")
		  .attr("stroke", "#eaeaea")
		  .attr("stroke-width", 2)
		  .selectAll("circle")
		  .data(node_objs)
		  .join("circle")
		  .attr("r", d => radius(adj_list[d.id].length))
		  .attr("fill", '#1c3c94')
		  .attr("data-id", d => d.id)
		  .call(drag(simulation))
		  .on("mouseover", elem => hover(elem.srcElement.attributes['data-id'].value))
		  .on("mouseout", elem => exit(elem.srcElement.attributes['data-id'].value))
		const labels = svg.append("g")
  		  .attr("stroke", "#eaeaea")
  		  .selectAll("text")
  		  .data(node_objs)
  		  .join("text")
		  .attr("x", d => d.x)
		  .attr("y", d => d.y + radius(adj_list[d.id].length) * 0.35)
		  .attr("fill", "#fff")
		  .attr("data-id", d => node_objs[d.index].id)
		  .style("text-anchor","middle")
		  .attr("font-family", "serif")
		  .attr("font-weight", 1)
		  .attr("font-size", d => f(adj_list[d.id].length))
		  .text(d => node_objs[d.index].id)
		  .call(drag(simulation))
		  .on("mouseover", elem => hover(elem.srcElement.attributes['data-id'].value))
		  .on("mouseout", elem => exit(elem.srcElement.attributes['data-id'].value))
		simulation.on("tick", () => {
		link
		    .attr("x1", d => d.source.x)
		    .attr("y1", d => d.source.y)
		    .attr("x2", d => d.target.x)
		    .attr("y2", d => d.target.y);
		node
		    .attr("cx", d => d.x)
		    .attr("cy", d => d.y);
		labels
			.attr("x", d => d.x)
			.attr("y", d => d.y + radius(adj_list[d.id].length) * 0.35);
		});
		const columns_title = svg.append("g")
			.selectAll("text")
			.data(node_sequence)
			.enter()
			.append("text")
			.text(d => d)
			.attr("stroke", "#777")
			.attr("stroke-width", 1)
			.attr("x", d => 500 + node_sequence.indexOf(d) * 18)
			.attr("y", 30)
			.attr("font-size", '12px')
			.style("text-anchor","middle")
			.attr("font-weight", 1)
		const row_title = svg.append("g")
			.selectAll("text")
			.data(node_sequence)
			.enter()
			.append("text")
			.text(d => d)
			.attr("stroke", "#777")
			.attr("stroke-width", 1)
			.attr("x", 480)
			.attr("y", d => 53 + node_sequence.indexOf(d) * 18)
			.attr("font-size", '12px')
			.style("text-anchor","end")
			.attr("font-weight", 1)
		for (var i in node_sequence){
			r = node_sequence[i]
			row = []
			for (var j in node_sequence) {
				c = node_sequence[j]
				if (adj_list[r.toString()].includes(c.toString())){
					row.push({index: j, name: c, value: true})
				}else{
					row.push({index: j, name: c, value: false})
				}
			}
			const row_comp = svg.append("g");
			row_comp.selectAll("rect")
				.data(row)
				.enter()
				.append("rect")
				.attr("x", d => 490 + d.index * 18)
				.attr("y", d => 40 + i * 18)
				.attr("width", 18)
				.attr("height", 18)
				.attr("data-id", r)
				.attr("data-value", d => d.value)
				.attr("data-row", r)
				.attr("data-column", d => d.name)
				.style("stroke", 'grey')
				.style("fill", d =>{
					if (d.value){
						return 'grey'
					}
					return 'none'
				})
				.style("stroke-width", 1)
				.on("mouseover", elem => {
					attr = elem.srcElement.attributes
					if (attr['data-value'].value == 'true'){
						hover_rect(attr['data-id'].value, attr['data-column'].value)
					}
				})
				.on("mouseout", elem => {
					attr = elem.srcElement.attributes
					exit_rect(attr['data-id'].value, attr['data-column'].value)
				});
		}
		return svg.node();
	});
</script>


### A2 : Introduction to Visualizing Amounts using P5

Name: Simon Lu 

Assignment: A2 : Introduction to Visualizing Amounts using P5

Three visualizations of airlines.csv data using P5, basic tool tips are added.

[https://bl.ocks.org/Simonl07/f27edb1bb16104ba64adbdfbff314318](https://bl.ocks.org/Simonl07/f27edb1bb16104ba64adbdfbff314318)
[https://bl.ocks.org/Simonl07/43cdd81e2409c5b90992140299d16381](https://bl.ocks.org/Simonl07/43cdd81e2409c5b90992140299d16381)
[https://bl.ocks.org/Simonl07/d715cc1fae69c970f13bd1f8e3fcbd90](https://bl.ocks.org/Simonl07/d715cc1fae69c970f13bd1f8e3fcbd90)

### A3: Introduction to Visualizing Distributions using P5

Name: Simon Lu

Assignment: A3: Introduction to Visualizing Distributions using P5

Histogram: [https://bl.ocks.org/Simonl07/2f7e9c9eb035ffcd5cba1bb41c0a8527](https://bl.ocks.org/Simonl07/2f7e9c9eb035ffcd5cba1bb41c0a8527)

Box plot: [https://bl.ocks.org/Simonl07/5c544e04c2b444318ff98a078d345beb](https://bl.ocks.org/Simonl07/5c544e04c2b444318ff98a078d345beb)

Strip Chart with jittering: [https://bl.ocks.org/Simonl07/f60d26ffae30385a27f6e81b248f8b66](https://bl.ocks.org/Simonl07/f60d26ffae30385a27f6e81b248f8b66)

