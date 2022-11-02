<script setup>
/**
 * This Nigeria map is created using TopoJSON
 */
import { onMounted } from 'vue'
import * as d3 from 'd3'
import *  as topojson from 'topojson-client'
import ngrStates from '../nigeria-states.json'

const draw = () =>{
    const width = 1240, height = 750

    const map = d3.select('.topo-map')

    // const svg = map.append('svg')
    //     .attr("viewBox", [0, 0, width, height])

    const reset = () => {
        states.transition().style("fill", null);
        svg.transition().duration(750).call(
            zoom.transform,
            d3.zoomIdentity,
            d3.zoomTransform(svg.node()).invert([width / 2, height / 2])
        );
    }

    const svg = map.append('svg')
        .attr('width', width)
        .attr('height', height)
        .on('click', reset)

    const g = svg.append('g');

    var projection = d3.geoMercator()
        .fitExtent([[50, 50], [width, height]], topojson.feature(ngrStates, ngrStates.objects.NGA_adm1))

    var path = d3.geoPath().projection(projection)

    const statesObj = topojson.feature(ngrStates, ngrStates.objects.NGA_adm1).features

    console.log(statesObj);

    const clicked = (event, d) => {
        const [[x0, y0], [x1, y1]] = path.bounds(d);
        event.stopPropagation();
        d3.select(this).transition().style('fill', '#ffc552');
        states.transition().style("fill", null);
        svg.transition().duration(750).call(
            zoom.transform,
            d3.zoomIdentity
                .translate(width / 2, height / 2)
                .scale(Math.min(8, 0.9 / Math.max((x1 - x0) / width, (y1 - y0) / height)))
                .translate(-(x0 + x1) / 2, -(y0 + y1) / 2),
            d3.pointer(event, svg.node())
        );
    }


    const states = g.append('g')
        .style('fill', '#444')
        .attr("cursor", "pointer")
        .selectAll("path")
        .data(statesObj)
        .enter().append('path')
        .on('click', clicked)
        .attr('d', path)

    g.append("path")
        .attr("fill", "none")
        .attr("stroke", "white")
        .attr("stroke-linejoin", "round")
        .attr("d", path(topojson.mesh(ngrStates, ngrStates.objects.NGA_adm1, (a, b) => a !== b)));

    states.append("title")
        .text(d => `${d.properties.NAME_1} ${d.properties.TYPE_1}`);



    const zoomed = (event) => {
        const { transform } = event;
        g.attr("transform", transform);
        g.attr("stroke-width", 1 / transform.k);
    }

    const zoom = d3.zoom()
        .scaleExtent([1, 6])
        .on("zoom", zoomed);

    svg.call(zoom);

    return svg.node();

}

onMounted(() => {
    draw()

    console.log(topojson);
})


</script>

<template>
    <div class="topo-map flex w:full jc:center"></div>
</template>

<style scoped>

</style>