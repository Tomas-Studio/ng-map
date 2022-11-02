<script setup>
/**
 * This Nigeria map is created using geoJSON
 */
import { onMounted } from 'vue'
import * as d3 from 'd3'
import statesGeoJSON from '../statesGeoJSON.json'
import populationByState from '../wave4PopulationByState.json'


const drawMap = () =>{
    const width = 800, height = 600;

    const area = d3.select('.map')

    const svg = area.append('svg')
        .attr('width', width)
        .attr('height', height)

    const popValues = Object.values(populationByState)

    const minPop = Math.min(...popValues)
    const maxPop = Math.max(...popValues)

    const colorScale = d3.scaleLinear().domain([minPop, maxPop]).range([
        d3.interpolateGreens(0.1), d3.interpolateGreens(0.9)
        ])

    const getStateColor = d => {
        const state = d.properties.admin1Name === "Federal Capital Territory" ? "Abuja" : d.properties.admin1Name
        return colorScale(populationByState[state])
    }


    var projection = d3.geoMercator()
        .fitExtent([[20, 20], [width, height]], statesGeoJSON)

    var path = d3.geoPath()
        .projection(projection)

    svg.append('g').selectAll('path')
        .data(statesGeoJSON.features)
        .enter().append('path')
        .attr('d', path)
        .attr('cursor', 'pointer')
        .style('fill', getStateColor)
        .style('stroke', 'white')
        .on('mouseover', function(d){
            d3.select(this).style('fill', '#ffc552')
        })
        .on('mouseout', function (d) {
            d3.select(this).style('fill', getStateColor)
        })
        .append('title')
        .text(d => d.properties.admin1Name)

    //console.log('area:', area)
}



onMounted(()=>{
    drawMap()
})

</script>

<template>
    
    <div class="map flex w:full jc:center"></div>

</template>

<style lang="scss" scoped>

</style>