---
author: andromeda
---
In my last blog post, I talked about exploring concept clusters based on different disciplines' uses of a particular word. However, I got into these neural nets in the first place because I was motivated by the question of "why does browse suck?", and I think machine learning creates possibilities for novel interfaces which might not suck. In short, I really want to explore *documents*, and give people new ways of exploring them. Or, to put it another way: if you liked this thesis, what others might you like?

Visualizing an entire thesis corpus is harder than visualizing the local network around a word, because many of our departments have produced thousands of theses over the years (and also because I know approximately nothing about data visualization and d3). So what I've done here is process the aero-astro department down into subgraphs, where everything in the subgraph is related to at least one other subgraph node by at least a threshold amount. Then we can explore subgraphs of more manageable size, and finally start to see what clusters of related *documents* might emerge.

<script src="https://d3js.org/d3.v4.min.js"></script>

<div class="bit">
<i>click a cluster to see its component theses</i>
</div>
<svg width="300" height="300" font-family="sans-serif" font-size="10" text-anchor="middle" id="department"></svg>

<div id="key" class="bit" style="display:none;">
</div>

<svg width="960" height="760" font-family="sans-serif" font-size="10" text-anchor="middle" id="subgraph"></svg>

So, something's weird about the visualization here - the number of subgraph data files I've generated don't seem to match with the overview in big_picture.json, and the number of nodes visible on screen don't seem to match those in the data subgraph files. But at the same time the *meaning* of the visualization seems roughly correct. Not sure what's going on with that but running short on debugging time today. Pull requests welcome at [mundaneum](https://github.com/MITLibraries/mundaneum).

{% capture datafile %}{{ site.url }}{{ site.baseurl }}/data/subgraphs/aeroastro/big_picture.json{% endcapture %}

<script>
var basegraphname = "{{ site.url }}{{ site.baseurl }}/data/subgraphs/aeroastro/graph";

var svg = d3.select("svg#department"),
    width = +svg.attr("width"),
    height = +svg.attr("height");

var diameter = Math.min(height, width),
    format = d3.format(",d"),
    color = d3.scaleOrdinal(d3.schemeCategory20c);

var bubble = d3.pack()
    .size([diameter, diameter])
    .padding(1.5);

d3.json("{{ datafile }}", function(error, data) {
  if (error) throw error;

  var root = d3.hierarchy(classes(data))
      .sum(function(d) { return d.value; })
      .sort(function(a, b) { return b.value - a.value; });

  bubble(root);
  var node = svg.selectAll(".node")
      .data(root.children)
      .enter().append("g")
      .attr("class", "node")
      .attr("transform", function(d) { return "translate(" + d.x + "," + d.y + ")"; });

  node.append("circle")
      .attr("r", function(d) { return d.value; })
      .style("fill", function(d) {
        return color(d.value);
      })
      .on("click", function(d) {
        var filename = basegraphname + d.data.className + '.json';
        showGraph(filename);
        d3.select('#key').style('display', 'inherit').html('<i>click a thesis to see information about it</i>');
        d3.event.stopPropagation();
      });
});

// Returns a flattened hierarchy containing all leaf nodes under the root.
function classes(root) {
  var classes = [];

  function recurse(name, node) {
    if (node.children) node.children.forEach(function(child) { recurse(node.name, child); });
    else classes.push({packageName: name, className: node.id, value: node.value});
  }

  recurse(null, root);
  return {children: classes};
}

d3.select(self.frameElement).style("height", diameter + "px");

function showGraph(filename) {

  d3.json(filename, function(error, graph) {
    if (error) throw error;

    var key = d3.select('#key');

    var svg_subgraph = d3.select('svg#subgraph'),
        width = svg_subgraph.attr("width"),
        height = svg_subgraph.attr("height"),
        maxRadius = +10;

    svg_subgraph.selectAll('*').remove();

    var simulation = d3.forceSimulation()
        .force("link", d3.forceLink().distance(100).id(function(d) { return d.id; }))
        .force("charge", d3.forceManyBody().strength(-100).distanceMax(250))
        .force("center", d3.forceCenter(width / 2, height / 2))
        .force("collide", d3.forceCollide(20));

    var maxLinkWeight = d3.max(graph.links, function(d) {return d.value;} );

    var color = d3.scaleLinear().domain([graph.threshold, maxLinkWeight])
        .interpolate(d3.interpolateHcl)
        .range([d3.rgb('#dedede'), d3.rgb("#21759B")]);

    var link = svg_subgraph.append("g")
      .attr("class", "links")
      .selectAll("line")
      .data(graph.links)
      .enter().append("line")
        .attr("stroke-width", function(d) { return d.value*10; })
        .attr("stroke", function(d) { return color(d.value); });

    var node = svg_subgraph.selectAll(".node")
        .data(graph.nodes)
        .enter().append("g")
        .attr("class", "node")
        .call(d3.drag()
            .on("start", dragstarted)
            .on("drag", dragged)
            .on("end", dragended))
        .on("click", function(d) {
          showMetadata(d);
        });

    node.append("circle")
      .attr("r", 20)
      .attr('z-index', '1')
      .attr("fill", function(d) {
          return '#0088D0';
      })

    node.append("text")
        .attr("fill", "#111")
        .attr("dx", "-10px")
        .attr("dy", "5px")
        .attr('z-index', '10')
        .attr('height', '12px')
        .attr('font-weight', '500')
        .text(function(d) { return d.id });

    simulation
        .nodes(graph.nodes)
        .on("tick", ticked);

    simulation.force("link")
        .links(graph.links);

    function ticked() {
      link
          .attr("x1", function(d) { return d.source.x; })
          .attr("y1", function(d) { return d.source.y; })
          .attr("x2", function(d) { return d.target.x; })
          .attr("y2", function(d) { return d.target.y; });

      node
        .attr("transform", function(d) {
            return "translate(" + Math.max(maxRadius,
                                  Math.min(width - maxRadius, d.x))
                                + ","
                                + Math.max(maxRadius,
                                                    Math.min(height - maxRadius, d.y))
                                + ")"; })
    }

    function dragstarted(d) {
      if (!d3.event.active) simulation.alphaTarget(0.3).restart();
      d.fx = d.x;
      d.fy = d.y;
    }

    function dragged(d) {
      d.fx = d3.event.x;
      d.fy = d3.event.y;
    }

    function dragended(d) {
      if (!d3.event.active) simulation.alphaTarget(0);
      d.fx = null;
      d.fy = null;
    }

    function showMetadata(d) {
      // See https://stackoverflow.com/questions/14667401/click-event-not-firing-after-drag-sometimes-in-d3-js
      if (d3.event.defaultPrevented === false) {
        document.getElementById('key').innerHTML = '<b><a href="' + d.url + '">' + d.title + '</a></b><br/><i>author</i>: ' + d.author + '</b><br/><i>advisor</i>: ' + d.advisor + '</b><br/><i>department</i>: ' + d.dlc;
      }
    }
  });  
}
</script>
